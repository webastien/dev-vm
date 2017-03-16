# Development virtual machine

This is my implementation of the nice couple [Vagrant](http://vagrantup.com) / [Ansible](https://www.ansible.com) to build a virtual machine for my developments (mainly
using [PHP](http://php.net) with [Drupal](http://drupal.org)).

## About
The generated server is not supposed to be used in production, it's not the perfect one ; It probably has security holes, lack of important packages, ... It's only fit my
actual needs and will probably be improved in the future.

I didn't test it on other environment than mine, so if you try it, you could encounter errors (for example, synchronised folders could not properly), but... don't worry, it will not break your system, that's the power of Virtual machines.

## My environment
MacBook pro / iMac with last MacOS version installed. I use [iTerm2](https://www.iterm2.com) for terminal, with default Mac Bash, and the less as possible dev tools installed on my Macs (no mysql for example). I prefer deferred them into the VM.

## Requirements
* [Vagrant](http://vagrantup.com)
* [Ansible](https://www.ansible.com)
* [VirtualBox](https://www.virtualbox.org)

Actualy, 2 Vagrant plugins are also required:
* [Vagrant Host Manager](https://github.com/devopsgroup-io/vagrant-hostmanager)
* [Vagrant Auto-network](https://github.com/oscar-stack/vagrant-auto_network)

**The first time you play the Vagrantfile, they will be automaticaly installed.**

**Note:** Each time you up / down the VM, Vagrant Host Manager will ask your password to update the `/etc/hosts` file. If you don't want to type it again and again, follow this [procedure](https://github.com/devopsgroup-io/vagrant-hostmanager#passwordless-sudo). I recommand because with it, this lovely plugin becomes transparent.

Suggested vagrant plugins:
* [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier)
* [vagrant-scp](https://github.com/invernizzi/vagrant-scp)

**You vill be prompted to install it, you can accept (y) refuse once (n) or type "never".**

**Note:** Choosing "never" add the plugin in `configuration/yours/vagrant-plugins-ignored.yml`. It's a simple YAML file you can edit or remove. All required plugins will not be affected by this one, only suggested. So, if you had choose "never" and change your mind, you just have to suppress the plugin name from this file. The next time you, it will be suggested again.

## What's in the box?
* [Debian](http://debian.org) 8 (Jessie) with VB guest additions (*), [Dotdeb](https://www.dotdeb.org) and non-free packages repositories are activated
* [Mysql](https://www.mysql.com) configured to use utf8
* [PHP-FPM](https://php-fpm.org) (php7) as fastcgi
* [Apache2](https://httpd.apache.org)

\* : I had problems with sync folder when using boxes without those guest additions, so I choose **garbetjie/debian8** from the [public Vagrant box catalog](https://atlas.hashicorp.com/boxes/search). No time to test for now, but the Vagrant plugin [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) could solve the problem.

Specific for web / Drupal developments:
* [Composer](https://getcomposer.org)
* [Drush](http://www.drush.org)
* [Drupal console](https://drupalconsole.com)
* [Adminer](https://www.adminer.org)

But keep in mind the playbook is configurable, so you can modify what the VM will have inside.

## Why did I made this instead of using Drupal-VM?
My first tries to use [Drupal-VM](https://github.com/geerlingguy/drupal-vm) was unsuccessful because there were opened issues breaking its usage on my system (because of MacOS? don't remember... That was last year), so I've decided to write mine. *But now, it seems very stable* and it's very configurable. Honestly, you should prefer this than mine, there is a large community around and it's actively maintened. If you have more time, try mine.

Other reasons: I wanted to understand how Vagrant/Ansible playbook work. Now, I can use mine, use yours, use [Drupal-VM](https://github.com/geerlingguy/drupal-vm) or whatever recipe given to me. I'm able to debug and modify Ansible playbook and Vagrant files, practice more [Jinja2](http://jinja.pocoo.org/docs/2.9/) syntax and [Ruby language](https://www.ruby-lang.org) (the Vagrantfile use it). It's also a good way to learn how to install and configure some server softwares when I will need them before integrate them inside. I don't regret the experience and become an addict.

## What my VM offers?
* As mentioned further: Composer, Drush and Drupal console are installed by default.
* You can easily add Drupal 7/8 sites (D8 trusted hosts are configured automaticaly), adminer, phpinfo with a few lines.
* You can modify and extend easily the "starters".
* [My own config](https://github.com/webastien/vim) for my favorite editor: [VIm](http://www.vim.org/)

## How to install?
Simply clone/copy this repository somewhere and, in this directory, run "**vagrant up**". Now, grab a cup of coffee because it takes a few minutes, depending on your internet connection and your hardware. That's it. You can optionaly override default settings, see below.

## How to setup / extend
Read the [Wiki pages](https://github.com/webastien/dev-vm/wiki) first, then if you're brave, the source code, or at least the config YAML files: I put lots of comments.

## Quickfix temporary in place
* **Drupal 8 site name**

There is a (minor) bug with Drupal 8 when installed by drush command and specify the site name: [It's not taken in consideration](https://github.com/drush-ops/drush/issues/2462). I've fixed this in [drupal8.yml](https://github.com/webastien/dev-vm/blob/master/configuration/default/starters/drupal8.yml) with a code not bad but which should not exists if the bug wasn't here.  I'll suppress this later when fixed in Drush / Drupal8.

* **Drupal console autocompletion**

When running **drush init** from ansible, the generated **console.rc** file uses a wrong name for the binary ("sh" instead of "drupal" by default). This is fixed in [drupalConsole-install.yml](https://github.com/webastien/dev-vm/blob/master/provisioning/roles/webastien.dev-vm/extras/drupalConsole/main.yml).

