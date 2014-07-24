---
layout: post
title: "Automated your development environment"
date: 2013-05-26 12:49
comments: true
categories: [chef, vagrant, ruby]
---

When MacOS have a major update every time, for example, from 10.7 to 10.8, there’s always a lot of changes with my development environments: default gcc changes, Xcode version change, and lot of others library need to re-compile, some library need to re-install using tools like “brew” or “port”

It’s really headache for me to get back to work before fix those problems. And the time to fix those dependences talk like a few hours, not a few minutes.

Recently, I am thinking to get rid of those mess with Mac OS dependence, and wanna automatic my development env in Mac OS or other OS like Windows. I have a Windows XP desktop and linux base VM, using putty from Windows to do development job at work, and also got a MacBook Pro for daily hacking or development at home. I want the same environment both in Windows & MacOS, same editor setting(vim), same ruby version ( using rvm ) and hope that I can easily reset/clone all my environment in a few minutes so that I can get back to work ASAP.

Inspire by [rails-dev-box](http://github.com/rails/rails-dev-box) and [this post](http://sidekicksrc.com/post/if-you-love-automation-why-is-your-development-environment-manual/), I have come up a solution.

Vagrant + Virtual Box + Chef-solo

## Vagrant + Virtual Box
[Vagrant](http://www.vagrantup.com) using a Vagrantfile to manage virtual box, make it automatically download a specific linux base-system vary from Ubuntu, Centos, Debian, and others linux distrubution. All with a single command  “vagrant up”, and in a few minutes, you got a fresh new linux base VM.

Here's the example of setting up a Ubuntu base system with a few command in the vagrant website.

``` bash
$ vagrant box add base http://files.vagrantup.com/lucid32.box
$ vagrant init
$ vagrant up
```

## PROVISIONING
provisioning allow you to install software automatically, alter configuration file and many manual things you need to setup a VM. Vagrant support Puppet or Chef Solo as provisioning system. 

I choose chef, since it’s ruby-friendly, configuration is pure ruby, that's a bonus!

## Chef
{% blockquote %}
Chef is an automation platform that transforms infrastructure into code
{% endblockquote %}
[Chef](http://www.opscode.com/chef/) can help install software in VM with ease using pre-defined code. The main concept on Chef is “Infrastructure as code”, all the software and it’s dependence can be coded and when it run, software can automatically installed in base-system vagrant choose, and Chef called this "cookbook". 

I choose “vim”, “git”, “mysql”, “rvm” cookbook, and cookbook dependences managed by Berkshelf ( berkshelf to cookbook, as bundler to gems ). Chef is not free, but [Chef-solo](http://docs.opscode.com/chef_solo.html) is free, and Vagrant can use Chef solo to do provisionning jobs!

## Where my code store
Vagrant use a share-folder concept to share file within mother-system and VMs, all my code can be checkout into this share-folder in my mother-system, this share-folder is located in the folder where you place Vagrantfile. And it’s mount under /vagrant automatically in VMs. this can have a lot of benifit, you can use you favour editor in mother system, and test code in VM, or you can use tmux/screen to login into VM, using vim/emacs to edit code and test under VM. This keep the flexibility of choosing favoured editor.

## Conclution
With a single “vagrant up”, no matter what system you are using right now (vagrant support Windows, MacOS or linux), and wait a few minutes to download base-system, to automatically install vim, git, mysql-server, and other configuration with chef cookbook, the fresh built environment for development is done.

And If something go wrong, such as system crash, I can just destroy it, and re-build it with ease into a fresh new status in a few minutes.

More on this topic next week.
