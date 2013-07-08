Things to do...
---------------

1. Possibly add support for other servers
    - lighttpd
    - nginx
2. Use mod_proxy & mod_proxy_fcgid with HTTPD 2.4.x

Bugs
----

1. Custom environment variables do not get passed through to PHP scripts
    - Look at automatically generating FcgidInitialEnv options based on the environment when the build pack runs
    - Look at using a custom wrapper script that would source the correct environment first
    - Possibly some other way to automate this with HTTPD
