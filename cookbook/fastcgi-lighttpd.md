---
layout: default
title: Webpy + LightTTPD with FastCGi
---

# Webpy + LightTTPD with FastCGi

The following applies to lighttpd version 1.4.18

Earlier version of lighttpd may organize the .conf files differently but the same principles applied to them as well.

##ligghttpd Configuration under Debian GNU/Linux

<pre>
Files and Directories in /etc/lighttpd:
---------------------------------------

lighttpd.conf:
         main configuration file

conf-available/
        This directory contains a series of .conf files. These files contain
        configuration directives necessary to load and run webserver modules.
        If you want to create your own files they names should be
        build as nn-name.conf where "nn" is two digit number (number
        is used to find order for loading files)

conf-enabled/
        To actually enable a module for lighttpd, it is necessary to create a
        symlink in this directory to the .conf file in conf-available/.

Enabling and disabling modules could be done by provided
/usr/sbin/lighty-enable-mod and /usr/sbin/lighty-disable-mod scripts.

For web py you should enable mod_fastcgi and mod_rewrite, thus run:  
/usr/sbin/lighty-enable-mod  
and supply <b>fastcgi</b>
</pre>

Below are instructions for the following files:
 * /etc/lighttpd/lighttpd.conf  
 * /etc/lighttpd/conf-available/10-fastcgi.conf  
 * code.py  

<code>/etc/lighttpd/lighttpd.conf</code>

<pre>
server.modules              = (
            "mod_access",
            "mod_alias",
            "mod_accesslog",
            "mod_compress",
)
server.document-root       = "/path/to/webpy/app/root-dir"
</pre>

In my case I used postgresql and therefore runs lighttpd as postgres in order to grant permissions to the database, therefore I added the line:

<pre>
server.username = "postgres"
</pre>

<code>conf-enabled/10-fastcgi.conf</code>

<pre>
server.modules   += ( "mod_fastcgi" )
server.modules   += ( "mod_rewrite" )

 fastcgi.server = ( "/code.py" =>
 (( "socket" => "/tmp/fastcgi.socket",
    "bin-path" => "/path/to/code.py",
    "max-procs" => 1,
   "bin-environment" => (
     "REAL_SCRIPT_NAME" => ""
   ),
   "check-local" => "disable"
 ))
 )

 url.rewrite-once = (
   "^/favicon.ico$" => "/static/favicon.ico",
   "^/static/(.*)$" => "/static/$1",
   "^/(.*)$" => "/code.py/$1",
 )
</pre>

<code>/code.py</code>
at the top of the file add:

<pre>
#!/usr/bin/env python
</pre>