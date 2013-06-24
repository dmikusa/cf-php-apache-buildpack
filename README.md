CloudFoundry PHP &amp; Apache HTTPD Buildpack
=============================================

This project is a build pack for deploying PHP applications to CloudFoundry.  It deploys and automatically configures Apache HTTPD and mod_fcgid to host your PHP applications.


Instructions
------------

The build pack has some conventions for the format of your application.  It expects a folder structure like this.

  - htdocs -> your application files & static resources
  - config -> PHP and HTTPD specific configuration settings
  - lib    -> PHP libraries used by your application (directory is on the include_path)
  
There is no further structure for the *htdocs* and *lib* directories.  You can set them up however you wish.  


### Configuration

The build pack comes with sane defaults for small to medium sized applications.  Ideally you won't need to adjust these setting much if at all.  If you'd like to customize things further, you have the following options.


#### options.json

In the *config/options.json* file, you have the following options:

  - ADMIN_EMAIL   -> the administrator email defined in HTTPD (default admin@localhost)
  - DOWNLOAD_URL  -> the URL from where to download the PHP & HTTPD binaries
  - HTTPD_VERSION -> the version of HTTPD to install (default latest 2.2 release)
  - PHP_VERSION   -> the version of PHP to install (default latest 5.4 release)

As the file extension indicates, the file should be valid JSON.


#### php.ini

This allows you to override the *php.ini* file that is used by your application.  If you include a custom *php.ini* file it will completely override the default one used by the build pack.  Because of this, it is recommended that you start with the [file from the build pack](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php.ini) and customisze it.


#### httpd directory
  
The configuration that is used by the build pack for HTTPD is broken down into multiple files.  Each file allows you to configure a specific part of HTTPD.  Your application can include one or more of the configuration files in the *config/httpd* directory and those files will override the defaults used by the build pack.

The following configuration files are used by the build pack.

  - [httpd-default.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-default.conf) -> some default options used by HTTPD
  - [httpd-logging.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-logging.conf) -> the logging configuration used by HTTPD
  - [httpd-modules.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-modules.conf) -> the modules loaded by HTTPD
  - [httpd-mpm.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-mpm.conf) -> the configuration options for the worker MPM module
  - [httpd-directories.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-directories.conf) -> directory options
  - [httpd-mime.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-mime.conf) -> the mime configuration & directory index settings
  - [httpd-php.conf](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/extra/httpd-php.conf) -> the PHP & mod_fcgid configuration

While not possible to override, here is a link to the [main httpd.conf file](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/httpd.conf).

Usage
-----

To use the build pack, specify the ```--buildpack``` option to the *cf* command.

Ex:

```
   cd my-app-dir   # top level dir for your project
   cf push --buildpack=https://github.com/dmikusa-pivotal/cf-php-apache-buildpack.git
```

Troubleshooting
---------------

Here are some helpful hints when things go wrong.

1. Check the memory usage.  You can use the *cf stats <app>* command to check the usage of a running application.  Additionally you can use the *cf curl get <url-for-events>/<guid>* command to view the app events which will show you if the application was killed for consuming too much memory.
2. Look at the startup logs with the *cf logs <app>* command.  This will show any problems with the initial start up of the server.
3. Look at the Apache HTTPD logs with the command *cf files <app> app/httpd/logs/error_log*.  

