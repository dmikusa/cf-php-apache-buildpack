CloudFoundry PHP &amp; Apache HTTPD Buildpack
=============================================

This project is a build pack for deploying PHP applications to CloudFoundry.  It deploys and automatically configures Apache HTTPD and PHP (HTTPD 2.2 uses mod_fcgid and HTTPD 2.4 uses mod_proxy_fcgi) to host your applications.


Instructions
------------

The build pack has some conventions for the format of your application.  It expects a folder structure like this.

  - htdocs -> your application files & static resources
  - config -> PHP and HTTPD specific configuration settings (optional)
  - lib    -> PHP libraries and files used by your application (directory is on the include_path), these files are not publically visible.
  
There is no further structure for the *htdocs* and *lib* directories.  You can set them up however you wish.  


### Configuration

The build pack comes with sane defaults for small to medium sized applications.  Ideally you won't need to adjust these setting much if at all.  However you can do so by creating a ```config``` directory in your application folder.  This folder can contain configuration settings that will override the defaults provided by the build pack.  

If you'd like to customize things further, you have the following options.


#### options.json

In your application, create the file *config/options.json* file, you have the following options:

  - ADMIN_EMAIL    -> the administrator email defined in HTTPD (default admin@localhost)
  - DOWNLOAD_URL   -> the URL from where to download the PHP & HTTPD binaries
  - HTTPD_VERSION  -> the version of HTTPD to install (default latest 2.2 release)
  - PHP_VERSION    -> the version of PHP to install (default latest 5.4 release)
  - NR_INSTALL_KEY -> your New Relic key, if specified New Relic support will be enabled
  - NEWRELIC_DOWNLOAD_URL -> location of New Relic binaries.  Defaults to download.newrelic.com.
  - NEWRELIC_VERSION -> the version to download, defaults to the latest version

As the file extension indicates, the file should be valid JSON.


#### php.ini

This allows you to override the *php.ini* file that is used by your application.  The default php.ini file is a minimal example and does not include any of the additional extensions.  If you include a custom *php.ini* file it will completely override the default one used by the build pack.  Because of this, it is recommended that you start with the file from the build pack for your version and customize it:  [5.3](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php/5.3/php.ini), [5.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php/5.4/php.ini), [5.5](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php/5.5/php.ini). 


#### php-fpm.conf

This allows you to override the *php-fpm.conf* file that is used by the HTTPD 2.4 configuration.  The default php-fpm.conf file and it should be good for most situations.  If your application is large, you'll likely want to configure this file to adjust the number of applications in the worker pool.  If you include a custom *php-fpm.conf* file it will completely override the default one used by the build pack.  Because of this, it is recommended that you start with the file from the build pack for your version and customize it:  [5.3](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php/5.3/php-fpm.conf), [5.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php/5.4/php-fpm.conf), [5.5](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php/5.5/php-fpm.conf).


#### httpd directory
  
The configuration that is used by the build pack for HTTPD is broken down into multiple files.  Each file allows you to configure a specific part of HTTPD.  Your application can include one or more of the configuration files in the *config/httpd* directory and those files will override the defaults used by the build pack.

The following configuration files are used by the build pack.

  - httpd-default.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-default.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-default.conf) -> some default options used by HTTPD
  - httpd-logging.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-logging.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-logging.conf) -> the logging configuration used by HTTPD
  - httpd-modules.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-modules.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-modules.conf) -> the modules loaded by HTTPD
  - httpd-mpm.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-mpm.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-mpm.conf) -> the configuration options for the worker MPM module
  - httpd-directories.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-directories.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-directories.conf) -> directory options
  - httpd-mime.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-mime.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-mime.conf) -> the mime configuration & directory index settings
  - httpd-php.conf - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/extra/httpd-php.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/extra/httpd-php.conf) -> the PHP & mod_fcgid configuration

While not possible to override, here is a link to the main httpd.conf file - [2.2](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.2/httpd.conf) | [2.4](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd/2.4/httpd.conf).

Usage
-----

To use the build pack, specify the ```--buildpack``` option to the *cf* command.

Ex:

```
   cd my-app-dir   # top level dir for your project
   cf push --buildpack=https://github.com/dmikusa-pivotal/cf-php-apache-buildpack.git
```

New Relic
---------

This build pack can install the New Relic plugin for PHP to monitor your application with the New Relic SaaS application monitoring tool. 

To indicate that it should be installed, simply specify your New Relic license key (found on the "Account Settings" screen when you login to New Relic) in your application's options.json file.  Please see the configuration section above for more details on options.json.

Then simply push your application as normal.  The build pack will install and configure New Relic for you.  If you would like to handle the configuration manually, you can simply create an application specific php.ini file and manually add the New Relic configuration options.  The presence of the word "newrelic" will disable the build pack's automatic configuration.

Currently the build pack's support is limited to users that have registered directly with New Relic.  Support for users who bind a New Relic server to their application through CloudFoundry may be added in a future release.

Troubleshooting
---------------

Here are some helpful hints when things go wrong.

1. Check the memory usage.  You can use the *cf stats <app>* command to check the usage of a running application.  Additionally you can use the *cf curl get <url-for-events>/<guid>* command to view the app events which will show you if the application was killed for consuming too much memory.
2. Look at the startup logs with the *cf logs <app>* command.  This will show any problems with the initial start up of the server.
3. Look at the Apache HTTPD logs with the command *cf files <app> app/httpd/logs/error_log*.  

Builds
------

The build pack makes use of binary versions of Apache HTTPD and PHP.  These are unmodified versions of the software that are downloaded into the environment in a binary form so that they do not have to be compiled prior to being used.  In the event that you'd like to compile your own versions of the software, check out this repo which has some helper scripts and instructions for building your own binaries.

## License
The CloudFoundry PHP Build Pack is released under version 2.0 of the [Apache License](http://www.apache.org/licenses/LICENSE-2.0).

