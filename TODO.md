Things to do...
---------------

1. Possibly add support for other servers
    - lighttpd
    - nginx

2. Status monitoring for:
   - php-fpm
   - HTTPD
   - mod_fcgid

3. Automatically enable PHP modules, if specific services are bound to the application.  Ex:  If a mysql database is attached, enable MySQL.

Bugs
----

1. Custom environment variables do not get passed through to PHP scripts
    - Look at automatically generating FcgidInitialEnv options based on the environment when the build pack runs
    - Look at using a custom wrapper script that would source the correct environment first
    - Possibly some other way to automate this with HTTPD
