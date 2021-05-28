# Learning Management System - Moodle

Moodle is the most popular and widely used Learning Management System (LMS). Moodle is short for Modular Object-Oriented Dynamic Learning Environment.
Moodle is designed to provide teachers, administrators and students with an open, robust, secure and to create and deliver personalized learning environments.


## Software Architecture

Moodle can be accessed via a web browser, desktop application and mobile application.

Moodle can be installed on any Linux server (eg. Centos, RedHat, Ubuntu).
Moodle requires a web server (eg. Nginx, Apache), database (eg. MariaDB, Postgresql), and PHP to run.


## Deployment

### Prepare server

Install and configure:

* web server 
* install database
* install PHP
* install SSL certificate

We prefer Nginx, Postgresql and php-fpm for our installation.
This combination provides greater efficiency and reliability,
especially in large installations.

### Install Moodle

The installation procedure can be found in [the official documentation](https://docs.moodle.org/311/en/Installing_Moodle).


## Integration with SSO

To integrate with external SSO, Moodle requires installation of
[the OpenID Connect plugin](https://moodle.org/plugins/auth_oidc).
The plugin must be configured with:

* Client ID 
* Client secret
* Authorization Endpoint
* Token Endpoint
 
It is a good practice to check the "Force redirect" box.
If it is checked, after clicking Login, the user will be automatically transferred to the SSO login page and not to the Moodle login page.

The default OpenID Connect scope values are "openid, profile, mail".


## Scaling up

For most Moodle sites, the default configuration should be sufficient, and it is not necessary to change the configuration.

For larger Moodle sites (multi-board servers, over 500 users) you can use additional cache options.<br>

For more information look at [the official documentation](https://docs.moodle.org/311/en/Caching).

## Notes

* It is important to allocate enough disk space before installation.
* Mount a separate disk for your backups with a capacity greater than or equal to the moodle data.
* Disable cron via web browser.
* Before update moodle test
* Before upgrading moodle, please make a backup of your moodle files and database.
* It is best to test the update on a separate instance.
