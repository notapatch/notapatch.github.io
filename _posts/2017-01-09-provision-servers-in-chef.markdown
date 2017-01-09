---
layout: post
title:  "Provision Servers with Chef"
date:   2017-01-09 11:11:11 +0000
categories: devops chef
---

### Provisioning Server
Provisioning a server is about running actions on a machine to make it ready for operation. These actions can be any number of configuration changes and installations of software.  
Infrastructure as code was about making these configurations a programming problem rather than a scripting job for the sysadmin. The leading solutions for infrastructure as code are [Ansible](https://www.ansible.com/) [Chef](https://www.chef.io/chef/) and [Puppet](https://puppet.com/). Any choice which groups people into camps leads to divisive exchanges of views - [feel free to follow these](https://www.google.co.uk/search?q=puppet+vs+chef+vs+ansible) - but I chose to use Chef.  

Chef is normally run on a [Chef Server](https://docs.chef.io/server_components.html) but I am running the infrastructure of a small business managing only a few servers. I chose to configure everything through the Workstation tool [knife-solo](https://matschaffer.github.io/knife-solo/) - [an excellent introduction](https://jenssegers.com/55/server-provisioning-with-chef-and-knife-solo). 

### CookBook Patterns
DevOps is a new area of development, [first popularised in 2009](https://en.wikipedia.org/wiki/DevOps),This is also the first year that [Chef was released](https://en.wikipedia.org/wiki/Chef_(software)).We wanted to avoid a common solution of forking everything -  
[The Environment Cookbook Pattern by Jamie Winsor - actually covers all the cookbook patterns](http://blog.vialstudios.com/the-environment-cookbook-pattern/)
[Suggested Environment Cookbook folder Structure - ifeltsweet](https://github.com/berkshelf/berkshelf/issues/535) - the whole thread is worth reading. 

### Environment Base Cookbook
Understanding cookbook patterns was helpful to get things started. However, I did not understand how they managed code reuse between Environment Cookbooks. The idea was that you had an environment cookbook for production, staging etc. So I added a ‘Base’ cookbook which all the environment cookbooks had as their ancestor.  Environment Base Cookbook.
Environment Base Cookbook - is what I call the common code for the Environment Cookbook - So I have Production Environment Cookbook and a Staging Environment Cookbook and their common code is in Environment Base Cookbook

````
  +---------------------------------------------+
  | Book: MyWebServer                           |
  | Type: Environment                           |
  |                                             |
  | /attributes/defaults                        |
  | default['ruby']['version'] = '2.3'          |
  |                                             |
  | /recipies/production                        |
  | include_recipe 'website_cookbook::default'  |
  |                                             |
  | /recipies/staging                           |
  | default['ruby']['version'] = '2.1'          |
  | include_recipe 'website_cookbook::default'  |
  |                                             |
  +---------------------------------------------+

```

### Cookbooks

| Cookbook Type             | Repository                                                                 |
| ------------------------- |:---------------------------------------------------------------------------|
| Environment               | [website-example](https://github.com/BCS-io-provision/website-example)     |
| Environment Base          | [website-cookbook](https://github.com/BCS-io-provision/website-cookbook)   |
| Base                      | [bcs_common_sytem](https://github.com/BCS-io-provision/bcs_common_system)  |
| Wrapper                   | [bcs_monit](https://github.com/BCS-io-provision/bcs_monit)                 |
{:.table}
