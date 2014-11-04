---
layout: post
title: "CORS on Apache"
date: 2014-11-04 08:32:41 +0700
comments: true
author: codeforfoods
author_site: http://codeforfoods.net/about-me
categories: [apache, crors-domain]
---

To add the CORS authorization to the header using Apache, simply add the following line inside either the `<Directory>`, `<Location>`, `<Files>` or `<VirtualHost>` sections of your server config (usually located in a *.conf file, such as `httpd.conf` or `apache.conf`), or within a `.htaccess` file:

		Header set Access-Control-Allow-Origin "*"
		
To ensure that your changes are correct, it is strongly reccomended that you use

		apachectl -t
		
to check your configuration changes for errors. After this passes, you may need to reload Apache to make sure your changes are applied by running the command

		sudo service apache2 reload
		
More detail

```
	http://enable-cors.org/server_apache.html
```