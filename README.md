# Super Duper Local Web Development Environment

A self provisioning [Vagrant](https://www.vagrantup.com/ "Learn More About Vagrant") box based on Ubuntu 16.04 (ubuntu/xenial64).  No custom boxes, just a ready to go local web development environment you can customize to your heart's content.

## Ready to Go

At your first ```vagrant up```, the following is automatically installed and configured.

* Apache (http and https)
* PHP 7.2
* Composer
* Node JS
* NPM
* MySQL (MariaDB 10.3)
* Mongo DB
* Redis
* Memcached
* Gulp
* Yarn
* Bower
* Sass
* MailHog
* WP-CLI
* Git
* Vim
* Curl

## Develop Your Project

The provided public folder is automatically synced to your machine at /var/www/public and acts as your web root.  This allows you to work on your project with your favorite code editor locally on your computer.

## Access Your Project

You can access your project from either http://192.168.33.10 or https://192.168.33.10.  

Alternatively, you can set your hosts file to point any domain you want to this ip address. 

## Connect to MySQL (MariaDB 10.3)

No need for an SSH tunnel.  MySQL (MariaDB 10.3) is configured automatically to allow external connections.

Host: 192.168.33.10 (User: root  Password: root)

## MailHog Interface

If you're developing emails, you can visit the [MailHog](https://github.com/mailhog/MailHog "Learn More About MailHog") interface at http://192.168.33.10:8025.

## Want Something Different?

You're not tied to a custom box with Super Duper Local Web Development Environment.  Simply change the provisioning in Vagrantfile.




