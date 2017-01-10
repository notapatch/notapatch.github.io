---
layout: post
title:  "Server Provisioning with Chef"
date:   2017-01-09 11:11:11 +0000
categories: devops chef
---

### How to get a server into operation
Readying a server for use is the act of provisioning it. Each server takes time to get going. We need to speed this up.

A first try at speeding up provisioning is configuring servers the same. While quick, diverse demands on the server will lead to compromise. If you can no longer compromise on shared settings, then a second try is to use configuration management tools such as [Ansible](https://www.ansible.com/) [Chef](https://www.chef.io/chef/) and [Puppet](https://puppet.com/). We will use Chef.

The recommended way to run Chef is on [Chef Server](https://docs.chef.io/server_components.html) but we are maintaining a small business network. We configured servers through the Workstation tool [knife-solo](https://matschaffer.github.io/knife-solo/).

#### Reference
* [Server Provisioning with Chef and Knife-Solo](https://jenssegers.com/55/server-provisioning-with-chef-and-knife-solo). 
* [Configuration management and the golden image](http://russell.ballestrini.net/configuration-management-and-the-golden-image/)

### CookBook Patterns
Chef is a young framework having been [released in 2009](https://en.wikipedia.org/wiki/Chef_(software)). At the time we began developing it was a common to forking everything and build on top - we were trying to avoid that and found the following patterns interesting:
* [The Environment Cookbook Pattern by Jamie Winsor - covers all the cookbook patterns](http://blog.vialstudios.com/the-environment-cookbook-pattern/)
* [Suggested Environment Cookbook folder Structure - ifeltsweet](https://github.com/berkshelf/berkshelf/issues/535) - the whole thread is worth reading. 

### Provision Architecture



````
     +---------------------------------------------+     
     | Book: Website-example                       |
     | Type: Environment                           |
     |                                             |
     | /attributes/defaults                        |
     | default['ruby']['version'] = '2.3'          |
     |                                             |
     | /recipies/production                        |
     | include_recipe 'website_cookbook::default' -+------+
     |                                             |      |
     | /recipies/staging                           |      |
     | default['ruby']['version'] = '2.1'          |      |
     | include_recipe 'website_cookbook::default'  |      |
     |                                             |      |
     +---------------------------------------------+      |
                                                          |
     +---------------------------------------------+      |
     | Book: Website-cookbook                      |      |
     | Type: Environment Base                      |      |
     |                                             |      |
     | /recipies/default  -------------------------+------+
     | ...                                         |
     | include_recipe 'bcs_common_system::default' +------+
     | ...                                         |      |
     | ...                                         |      |
     | include_recipe 'bcs_monit::default' --------+---+  |
     |                                             |   |  |
     +---------------------------------------------+   |  |
                                                       |  |
     +---------------------------------------------+   |  |
     | Book: bcs_monit                             |   |  |
     | Type: wrapper                               |   |  |
     |                                             |   |  |
     | /attributes/defaults                        |   |  |
     | default['monit']['polling_frequency'] = 30  |   |  |
     |                                             |   |  |
     | /recipies/defaults  ------------------------+---+  |
     | include_recipe 'monit::default'             |      |
     | monit_monitrc 'load' do                     |      |
     |   template_cookbook 'monit'                 |      |
     | end                                         |      |
     |                                             |      |
     | node['monit']['probe'].each do |probe|      |      |
     |   monit_monitrc probe do                    |      |
     |     template_cookbook 'bcs_monit'           |      |
     |   end                                       |      |
     | end                                         |      |
     |                                             |      |
     +---------------------------------------------+      |
                                                          |
     +---------------------------------------------+      |
     | Book: bcs_common_system                     |      |
     | Type: Base                                  |      |
     |                                             |      |
     | /recipies/default  -------------------------+------+
     | include_recipe 'bcs_common_system::basic'   |
     | include_recipe 'bcs_common_system::server'  |
     |                                             |
     | /recipies/basic                             |
     | include_recipe 'apt::default'               |
     | include_recipe 'apt-repository::default'    |
     | ...                                         |
     | include_recipe 'bcs_git::default' ----------+---+
     |                                             |   |
     +---------------------------------------------+   |
                                                       |
     +---------------------------------------------+   |
     | Book: bcs_git                               |   |
     | Type: wrapper                               |   |
     |                                             |   |
     | /recipies/basic ----------------------------+---+
     | include_recipe 'git::default'               |
     |                                             |
     | git_config 'color' do                       |
     |   key 'color.ui'                            |
     |   value 'auto'                              |
     | end                                         |
     |                                             |
     +---------------------------------------------+

```

### Cookbooks

| Cookbook Type             | Repository                                                                 |
| ------------------------- |:---------------------------------------------------------------------------|
| Environment               | [website-example](https://github.com/BCS-io-provision/website-example)     |
| Environment Base          | [website-cookbook](https://github.com/BCS-io-provision/website-cookbook)   |
| Base                      | [bcs_common_system](https://github.com/BCS-io-provision/bcs_common_system) |
| Wrapper                   | [bcs_monit](https://github.com/BCS-io-provision/bcs_monit)                 |
| Wrapper                   | [bcs_git](https://github.com/BCS-io-provision/bcs_git)                     |
{:.table}
