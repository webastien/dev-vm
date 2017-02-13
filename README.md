# Development virtual machine

This is my implementation of the nice couple [Vagrant](http://vagrantup.com) / [Ansible](https://www.ansible.com) to build a virtual machine for my developments (mainly
using [PHP](http://php.net) with [Drupal](http://drupal.org)).

## About
The generated server is not supposed to be used in production, it's not the perfect one ; It probably has security holes, lack of important packages, ... It's only fit my
actual needs and will probably be improved in the future.

I didn't test it on other environment than mine, so if you try it, you could encounter errors (for example, synchronised folders could not works with "nfs" on other
system), but... don't worry, it will not break your system, that's the power of Virtual machines.

## My environment
MacBook pro / iMac with last MacOS version installed. I use [iTerm2](https://www.iterm2.com) for terminal, with default Mac Bash, and the less as possible dev tools
installed on my Macs (no mysql for example). I prefer deferred them into the VM.

## Requirements
* [Vagrant](http://vagrantup.com)
* [Ansible](https://www.ansible.com)
* [VirtualBox](https://www.virtualbox.org)

Actualy, 2 Vagrant plugins are also required:
* [Vagrant Host Manager](https://github.com/devopsgroup-io/vagrant-hostmanager)
* [Vagrant Auto-network](https://github.com/oscar-stack/vagrant-auto_network)

The first time you play the Vagrantfile, they will be automaticaly installed. Note a third one is not required but strongly recommanded:
[vagrant-cachier](https://github.com/fgrehm/vagrant-cachier).

## What's in the box?
* [Debian](http://debian.org) 8 (Jessie) with VB guest additions (*), [Dotdeb](https://www.dotdeb.org) and non-free packages repositories are activated
* [Mysql](https://www.mysql.com) configured to use utf8
* [PHP-FPM](https://php-fpm.org) (php7) as fastcgi
* [Apache2](https://httpd.apache.org)

\* : I had problems with sync folder when using boxes without those guest additions, so I choose **garbetjie/debian8** from the [public Vagrant box
catalog](https://atlas.hashicorp.com/boxes/search).

Specific for web / Drupal developments:
* [Composer](https://getcomposer.org)
* [Drush](http://www.drush.org)
* [Drupal console](https://drupalconsole.com)
* [Adminer](https://www.adminer.org)

But keep in mind the playbook is configurable, so you can modify what the VM will have inside. The
[config.yml](https://github.com/webastien/dev-vm/blob/master/provisioning/vars/config.yml) file is fully documented.

## Why did I made this instead of using Drupal-VM?
First of all, I wanted to understand how Vagrant/Ansible playbook are working. Now, I can use mine, use yours, use [Drupal-VM](https://github.com/geerlingguy/drupal-vm)
or whatever recipe given to me. I also can debug and modify it if required.

Also, my first tries to use Drupal-VM was unsuccessful because there were opened issue breaking its usage on my system (because of MacOS? don't remember... That was last
year), so I've decided to write mine. But now, it seems very stable and usable so I can use both.

## What my VM offers?
* As mentioned further: Composer, Drush and Drupal console are installed by default.
* In the [config.yml](https://github.com/webastien/dev-vm/blob/master/provisioning/vars/config.yml) file you can easily add Drupal 7/8 sites, adminer, phpinfo and it's
  easy to extend the list of "starter sites" you can install with a few lines (Installed drupal 8 sites has their trusted configured)
* [My own config](https://github.com/webastien/vim) for my favorite editor: [VIm](http://www.vim.org/)

## How to install?
Simply clone/copy this repository somewhere and, in this directory, run "**vagrant up**". Now, go find a coffee cause it takes a few minutes. That's it. You can optionaly
override default config by copy **provisioning/vars/default-config.yml** to **provisioning/vars/config.yml** and make your changes in it.

## Quickfix temporary in place
* **Drupal 8 site name**

There is a (minor) bug with Drupal 8 when installed by drush command and specify the site name: [It's not taken in
consideration](https://github.com/drush-ops/drush/issues/2462). I've fixed this in
[drupal8.yml](https://github.com/webastien/dev-vm/blob/master/provisioning/starters/drupal8.yml) with a code not bad but which should not exists if the bug wasn't here.
I'll suppress this later when fixed in Drush / Drupal8.

* **Drupal console autocompletion**

When running **drush init** from ansible, the generated **console.rc** file uses a wrong name for the binary ("sh" instead of "drupal" by default). This is fixed in [drupalConsole-install.yml](https://github.com/webastien/dev-vm/blob/master/provisioning/tasks/php/drupalConsole-install.yml).

