---
layout: post
title: "Rails Email Interception"
date: 2017-07-25 11:11:11 +0000
categories: rails solidus spree email interceptor
---

How do you change an email you did not create? Rails Action mailer emails have a ready-made solution: [ActionMailer emails interceptor][0]. 

My problem was that my E-commerce engine, Solidus a Spree fork, [sends the customer an email when a customer orders notifying no one else.][1] By intercepting and changing the email addresses's 'bcc' field you can tell the shipping office without affecting the e-commerce engine. 
 
### Steps

Email interception requires several steps to complete:

* Email Interceptor to change the order email
* Register the interceptor class with Rails
* Supply email addresses to copy into the email's bcc field
* Testing Interceptor
    1. Unit test
    2. Feature test
   
&nbsp; 


### Email Interceptor to change the order email

Example 1, is the actual code that changes the order mail. The guard clause, line 5, prevents [the interceptor from changing any other ActionMailer delivery][3]. [Your class should make any needed modifications directly to the passed in Mail::Message instance.][4] With the bcc email addresses being taken from `Rails.application.secrets` though I've seen programmers use a separate yml file. 

Example 1\: Changing the order mail - `lib/bcc_email_interceptor.rb`

{% highlight ruby linenos %}
class BccEmailInterceptor
  REVISABLE_MAILERS = [Spree::OrderMailer].freeze

  def self.delivering_email(message)
    return unless REVISABLE_MAILERS.include?(message.delivery_handler)

    message.bcc = Rails.application.secrets['order']['notify']
  end
end
{% endhighlight %}
   

### Register the interceptor class with Rails

Rails needs the interceptor class registering before it can work, see example 2 below.

Example 2\: Register interceptor class with Rails `config/initializers/bcc_email_interceptor.rb`
{% highlight ruby linenos %}
require "bcc_email_interceptor"

ActionMailer::Base.register_interceptor(BccEmailInterceptor)
{% endhighlight %}
It is more common to see code that only registers interceptors on production (`if Rails.env.production?`) but I didn't want to do this for simplicity unless it became a problem - I filter out example.com email addresses with a rule at the email service level and [Rails does not send emails in test environment by default][9]. 
&nbsp; 

&nbsp; 

### Supply email addresses to copy into the email's bcc field

Bcc Email addresses are taken from a configuration file. In example 3, I am only showing the full production structure but test and development are the same format.
 
Example 3\: Email addresses to notify on a new order - `config/secrets.yml`

{% highlight yml %}
development:
  ...
test:
  ...
production:
  order:
    notify:
      - tell-warehouse@example.com
{% endhighlight %}
   

### Testing Interceptor

Testing the interceptor requires\:
1. Unit test examining that interceptor alters an email as expected.
2. Feature test examining if the interceptor is hooking into Rails

For more information the [Rails guide has a section on testing mailers][5]
   
   

#### 1. Unit test examining that interceptor alters an email as expected

A test isolating the interceptor from the rest of the system as shown in example 4 below - showing it alters the bcc.

Example 4\: unit test to see the interceptor code worked `spec/bcc_email_interceptor_spec.rb`
{% highlight ruby %}
require 'rails_helper'

describe BccEmailInterceptor do
  describe 'OrderMailer email' do
    it 'leaves an unchanged bcc when overlooked by interceptor' do
      mail = Spree::OrderMailer.confirm_email(FactoryGirl.create(:order))

      expect(mail.bcc).to be_nil
    end

    it 'changes bcc when handled by the interceptor' do
      mail = Spree::OrderMailer.confirm_email(FactoryGirl.create(:order))

      BccEmailInterceptor.delivering_email(mail)

      expect(mail.bcc).to eq ['tell-admin@example.com']
    end
  end
end
{% endhighlight %}
   


#### 2. Feature test examining if the interceptor is hooking into the Rails

Test the interceptor is active within Rails with an integration test as shown in example 5 below. Rails functional mailer tests call the deliver method and then test the queued mail - we can assume that Rails has tested ActiveMailer.

Example 5\: feature test to see email interception was working within Rails `spec/features/bcc_email_interceptor_spec.rb`
{% highlight ruby %}
require "rails_helper"

RSpec.feature "BccEmailInterceptor", type: :feature do
  before { ActionMailer::Base.deliveries.clear }

  scenario "OrderMailer deliveries are changed" do
    order = FactoryGirl.create(:order)

    Spree::OrderMailer.confirm_email(order).deliver
    order_mail = ActionMailer::Base.deliveries.last

    expect(order_mail.bcc).to eq ['tell-admin@example.com']
  end
end
{% endhighlight %}

### Summary   

Intercepting emails is a feature of Rails's ActiveMailer that adds flexibility to your system without incurring the cost of maintaining a patch to someone else's library.


### References

- Email interception idea came from [Solidus support channel thread][10]
- [Action Mailer Interceptors by Josh][6] 
- [Mail interceptors for different Rails environments][7]

[0]: http://guides.rubyonrails.org/action_mailer_basics.html#intercepting-emails 
[1]: https://github.com/solidusio/solidus/blob/v2.2/core/app/models/spree/order.rb#L409-L427
[3]: https://stackoverflow.com/questions/19408376/rails-mail-interceptor-only-for-a-specific-mailer
[4]: https://github.com/rails/rails/blob/v5.1.2/actionmailer/lib/action_mailer/base.rb#L261-L264
[5]: http://guides.rubyonrails.org/testing.html#testing-your-mailers
[6]: https://wearestac.com/blog/action-mailer-interceptors
[7]: http://www.eq8.eu/blogs/9-mail-interceptor-for-different-rails-environments
[9]: https://github.com/rails/rails/blob/v5.1.2/railties/lib/rails/generators/rails/app/templates/config/environments/test.rb.tt#L33-L36
[10]: https://solidusio.slack.com

&nbsp; 
