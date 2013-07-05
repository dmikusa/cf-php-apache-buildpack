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

This allows you to override the *php.ini* file that is used by your application.  The default php.ini file is a minimal example and does not include any of the additional extensions.  If you include a custom *php.ini* file it will completely override the default one used by the build pack.  Because of this, it is recommended that you start with the [file from the build pack](https://github.com/dmikusa-pivotal/cf-php-apache-buildpack/blob/master/default/php.ini) and customisze it.


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

Builds
------

The build pack makes use of binary versions of Apache HTTPD and PHP.  These are unmodified versions of the software that are downloaded into the environment in a binary form so that they do not have to be compiled prior to being used.  In the event that you'd like to compile your own versions of the software, the following instructions show how to do that.

###Environment

The environment used to compile the files was a fully up-to-date Ubuntu 10.04.4 LTS system.  The easiest way to get all of the build tools is to install the build-essential and autoconf packages.  Beyond that, you'll need libraries and dev packages for things like MySQL, PostgresSQL, aspell, mcrypt, libc-client, gettext, openssl, gdb, bz2 and anything else that you want to compile support for into PHP.

###Apache Build

Use the typical installation instructions for building Apache HTTPD.  When you run the "./configure" step, use the following options.

```
./configure --prefix=/tmp/staged/app/httpd --enable-mods-shared=all --enable-so --with-mpm=worker
```

####mod_fcgid

In addition to building Apache HTTPD, you need to build mod_fcgid.  This is done with the following command.

```
APXS=/tmp/staged/app/httpd/bin/apxs ./configure.apxs && make && make install
```

After this completes, the mod_fcgid.so file should be located at */tmp/staged/app/httpd/modules/mod_fcgid.so*.

####Packaging

The following steps are used to package Apache HTTPD for the build pack.

1. Remove these extra directories.

    ```rm -rf build cgi-bin/ error/ icons/ include/ man/ manual/ htdocs/```

2. Remove these configuration files.  The build pack will create them.

    ```rm -rf conf/extra/* conf/httpd.conf conf/httpd.conf.bak conf/magic conf/original```

3. Edit the following files and replace */tmp/staged/app* with *${HOME}*.

    ```
    vi apachectl apr-1-config apu-1-config envvars envvars-std
    %s/\/tmp\/staged\/app/${HOME}/g
    ```

4. Edit *apachectl*, locate the *HTTPD* variable at the top of the script.  Change the surrounding quotes from single quotes to double quotes.

5. Rename the folder */tmp/staged/app/httpd* to */tmp/staged/app/httpd-${version}-bin*.

6. Tar and gzip the file.  ```tar czf httpd-${version}-bin.tar.gz httpd-${version}-bin```

###PHP Build

The PHP build is a bit more complicated as it depends on quite a few things.  The default build provided with this build pack tries to include all of the modules that you would need to connect to the services available on CloudFoundry.  This includes MySQL, PostgreSQL, Redis, Mongo and RabbitMQ.  Here are the instructions used to 

Use the typical installation instructions for building PHP.  When you run the "./configure" step, use the following options. Note, these are for bulding PHP 5.4.x.  The options will likely differ with different releases.

```
./configure
    --prefix=/tmp/staged/app/php
    --with-config-file-path=/home/vcap/app/php/etc
    --enable-cli
    --enable-ftp
    --enable-sockets
    --enable-soap
    --enable-fileinfo
    --enable-bcmath
    --enable-calendar
    --with-kerberos
    --enable-zip
    --enable-pear
    --with-bz2=shared
    --with-curl=shared
    --enable-dba=shared
    --with-inifile
    --with-flatfile
    --with-cdb
    --with-gdbm
    --with-mcrypt=shared
    --with-mhash=shared
    --with-mysql=mysqlnd
    --with-mysqli=mysqlnd
    --with-pdo-mysql=mysqlnd
    --with-gd=shared
    --with-pdo-pgsql=shared
    --with-pgsql=shared
    --with-pspell=shared
    --with-gettext=shared
    --with-gmp=shared
    --with-imap=shared
    --with-imap-ssl=shared
    --with-ldap=shared
    --with-ldap-sasl
    --enable-mbstring
    --enable-mbregex
    --with-exif=shared
    --with-openssl=shared
```

####Additional Require Libraries

Some of the extensions that are built into PHP require additional shared libraries.  These should be copied to the */tmp/staged/app/php/lib* directory, which is added to the LD_LIBRARY_PATH by the build pack.

 - libaspell.so.15
 - libc-client.so.2007e
 - libmcrypt.so.4
 - libpspell.so.15
 - librabbitmq.so.1

####Build External Extensions

Some of the bundled extensions are not part of the standard PHP distribution.  These need to be downloaded and installed separately.  These include the following.

 - apc
 - amqp
 - mongo
 - redis
 - xdebug

Follow the standard build and install instructions for these extensions. 

```
cd <mod-folder>
/tmp/staged/app/php/bin/phpize
./configure --with-php-config=/tmp/staged/app/bin/php-config <other required opts>
make && make install
```

After *make install* completes the extension will exist in */tmp/staged/app/php/lib/php/extensions/no-debug-non-zts-20100525*.

####Packaging

The following steps are used to package PHP for the build pack.

1. Remove these extra directories.

    ```rm -rf include/ php/```

2.  Rename the folder */tmp/staged/app/php* to */tmp/staged/app/php-${version}-bin*.

3. Tar and gzip the file.  ```tar czf php-${version}-bin.tar.gz php-${version}-bin```

###Distribution

Distribution of your builds is simple.  Place them on a web server so that they are public.  From there edit *options.json* and set the *DOWNLOAD_URL* option to point to the directory on your web server.  As long as you have followed the packaging instructions above, the build pack should be able to consume your binaries.

