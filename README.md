# What is this repository for? #

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
  
___

# Assumptions #

This playbook assumes the following is true:

* you have installed Ansible locally 
* you have already set up domains
* you have a public key for SSH
* your public key is among the authorized keys for either the root or admin user on the server you're targeting
* the server your targeting is running on either Debian/Ubuntu distribution of Linux
* you want the server to run on Nginx
* one of the web applications to run on the server is either WordPress or a NodeJS-based application.
* you have entered all the required input parameters in the input files

# Running the playbook #

Before running the playbook, you must install Ansible dependencies:

* `ansible-galaxy install -r requirements.yaml`

Depending whether you want to set the server up for WordPress or NodeJS or both, run any the following commands:

* both: `OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook --inventory-file hosts default_setup.yaml`
* WordPress: `OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook --inventory-file hosts wordpress_setup.yaml`
* NodeJS-based: `OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook --inventory-file hosts node_setup.yaml`

Note, the `OBJC_DISABLE_INITIALIZE_FORK_SAFETY` environment variable must be set due to an issue Mac OS has with Python process forking; [see this issue on GitHub](https://github.com/ansible/ansible/issues/32499#issuecomment-341578864).

# Input parameters #

In order to run the playbook, you need to set the following input parameters which are distributed among four files:

* `hosts`
* `input_shared.yaml`
* `input_wordpress.yaml`
* `input_web.yaml`

### The `hosts` (a.k.a. inventory) file ###

In the `hosts` file you must enter the domain names for the servers which you want to set up for NodeJS (a.k.a web) and/or WordPress.

For example:

```ini
[wordpress]
wp.sg-web.tartu.tech

[web]
sg-web.tartu.tech
```

### Shared parameters (`input_shared.yaml`, `input_wordpress.yaml` or `input_web.yaml`) ###

The following parameters are used by both the WordPress and the NodeJS setups. If any of the parameters differs depending on which target you're setting up for, then you can set the value in either `input_wordpress.yaml` or `input_web.yaml` instead of `input_shared.yaml`.

* `domain_name` 
    * String: The domain name which the Nginx server should accept. 
    * Example: `domain_name: example.com`
* `prefix_www_to_domain_name`
    * Boolean: Whether the Nginx server should also accept the `domain_name` with a `www.` prefix (e.g. if `domain_name` is `hello.world.com`, whether `www.hello.world.com` should also be accepted).
    * Example: `prefix_www_to_domain_name: true`
* `current_ssh_port`
    * String: The port which the server is using for SSH when the playbook is run. Usually this is port 22 by default.
    * Example: `current_ssh_port: "22"`
* `new_ssh_port`
    * String: The port which you want the server to use for SSH connections. This can be the same as `current_ssh_port` if you don't want to change the port.
    * Example: `new_ssh_port: "12345"`
* `start_certbot`
    * Boolean: Whether to run certbot and generate a HTTPS certificate.
    * Example: `start_certbot: true`
* `admin_username`
    * String: The username for the admin user (user with full sudo rights). This shouldn't be longer than 32 characters. You can increase security by making this username hard to guess.
    * Example: `admin_username: admin-2c45c1d3732d`
* `authorized_employees_for_admin`
    * List of strings: The names from the [sg-public-keys repository](https://bitbucket.org/singleton-group/sg-public-keys/) (without the `.pub` suffix) who are allowed to enter the admin user with SSH.
    * Example: `authorized_employees_for_admin: [dan, kaarel, markus]`
* `deploy_username`
    * String: The username for the deploy user (user with sudo rights for limited commands). This shouldn't be longer than 32 characters. You can increase security by making this username hard to guess.
    * Example: `deploy_username: deploy-06372f89aa57`
* `authorized_employees_for_deploy`
    * List of strings: The names from the [sg-public-keys repository](https://bitbucket.org/singleton-group/sg-public-keys/) (without the `.pub` suffix) who are allowed to enter the deploy user with SSH.
    * Example: `authorized_employees_for_deploy: [roman, jan, inga]`
* `commands_allowed_for_deploy`
    * List of strings: The commands which the deploy user is allowed to run with sudo.
    * Example: `commands_allowed_for_deploy: ["systemctl restart nginx", "pm2 deploy production update"]`
* `certbot_notifications_email`
    * String: Email address to which Certbot sends notifications (e.g. when the certificate is expiring). This email address is also used to agree to Certbot TOS.
    * Example: `certbot_notifications_email: dan@singleton.ee`
* `fail2ban_notifications_email`
    * String: Email address to which fail2ban sends notifications
    * Example: `fail2ban_notifications_email: dan@singleton.ee`

___

### WordPress-specific parameters (`input_wordpress.yaml` only) ###

* `php_version`
    * String: PHP version to install. In order to avoid issues with PHP packages, consider using [actively supported](https://www.php.net/supported-versions.php) version.
    * Example: `php_version: "7.4"`
* `_mysql_root_password`
    * String: The MySQL root password. Keep this secure and make it strong.
    * Example: `_mysql_root_password: "XOJ88Jb5CI*Jo773VQIXEk?mJ6Acwf7s"` 
* `_mysql_wordpress_password`
    * String: The MySQL root password. Keep this secure and make it strong.
    * Example: `_mysql_wordpress_password: "HWN@D@?CiG6dQ5YV&xSQ5O6Nml_0&Y+S"` 

### NodeJS-specific parameters (`input_web.yaml` only) ###

* `node_process_port`
    * String: The port at which the NodeJS server process is listening. Nginx will redirect to this process.
    * Example: `node_process_port: "3000"`

# Who do I talk to? #

The base for this project was written by Dan Bondarenko (dan@singleton.ee). Please contact me if you have any questions or ideas for improvements.