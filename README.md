## What is this repository for? ##

This repository contains an [Ansible](https://docs.ansible.com/ansible/2.10) playbook (basically a script) which lets you prepare a server for WordPress and/or NodeJS web applications in an automated fashion depending on the parameters you set as inputs.

This playbook can be used to automatically perform the following tasks:

* lock the root user
* create an alternative admin user with full SUDO rights
* create a deploy user with limited SUDO rights (only commands specified in input)
* disallow SSH access to root user
* only allow SSH with public-private key authentication
* install basic required packages (build-essential, git, etc)
* configure UFW with sensible defaults
* change default SSH port 22 to a custom port
* set SSH authorized keys of admin/deploy users by specifying employee names (public keys are automatically downloaded from the [sg-public-keys repository](https://bitbucket.org/singleton-group/sg-public-keys/))
* set up fail2ban to automatically ban IPs who are brute-forcing the host
* generate HTTPS certificates with Certbot and schedule automated renewal 
* create a user model with permissions set in such a way that each services are isolated from each other
* install PHP and packages needed by WordPress
* set up PHP-FPM
* install MySQL
* install and configure WordPress
* install Nginx
* set up Nginx HTTPS and HTTP (redirects to former) virtual hosts for WordPress with PHP-FPM proxying
* install NodeJS 
* configure PM2 to run on startup
* set up Nginx HTTPS and HTTP (redirects to former) virtual hosts with proxying to Node server
* create symlinks to web application directories from deploy user home
## How do I get set up? ##
### Assumptions ###

This playbook assumes the following is true:

* you have installed Ansible locally 
* you have already set up domains
* you have a public key for SSH
* your public key is among the authorized keys for either the root or admin user on the server you're targeting
* the server your targeting is running on either Debian/Ubuntu distribution of Linux
* you want the server to run on Nginx
* one of the web applications to run on the server is either WordPress or a NodeJS-based application.

* Summary of set up
* Configuration
* Dependencies
* Database configuration
* How to run tests
* Deployment instructions

### Who do I talk to? ###

The base for this project was written by Dan Bondarenko (dan@singleton.ee). Please contact me if you have any questions.