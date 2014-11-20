---
layout: post
title: "php-cooking"
date: 2014-11-07 11:03:51 +0700
comments: true
author: codeforfoods
fb_img: https://lh6.googleusercontent.com/-yoM7_7UiHBY/VFxCYAUZpJI/AAAAAAAAABs/xbd3d4fabn8/s184/README_md_-__Users_congdang_my-training_codeforfoods_php-cooking_-_Atom.png
categories: [chef, cooking, php]
---

This article would help you to setup Chef for PHP stuff step by step

#### Step 1 - Install required tools
* Ruby

```
	https://www.ruby-lang.org/en/installation/
```

* Knife Solo

```
	gem install knife-solo
```

* librarian-chef

```
	gem install librarian-chef
```

#### Step 2 - Setup

* Create new directory for your project

		mkdir /your-path/php-cooking

* Move to php-cooking folder

		cd /your-path/php-cooking


* Initialize librarian-chef

		librarian-chef init

* Initialize knife solo

		knife solo init .

After initialazing, you should see the folder structure like below (notice that I added the `www` folder for containing the php's web app )

{% img https://lh5.googleusercontent.com/-e9FKrxVfB50/VFxCHoV-CaI/AAAAAAAAABg/aBM_nLe5EMY/s418/README_md_-__Users_congdang_my-training_codeforfoods_php-cooking_-_Atom.png %}

Ok, now is time for adding the necessary cookbooks to `Cheffile`

``` ruby Cheffile
	#!/usr/bin/env ruby
	#^syntax detection

	site 'https://supermarket.getchef.com/api/v1'


	cookbook 'php', '~> 1.4.6'

	cookbook "php55",
	  :git => "https://github.com/aporat/php55-cookbook"

	cookbook 'php-mcrypt', '~> 1.0.0'

	cookbook 'apache2', '>= 1.0.0'

	cookbook 'mysql', '~> 5.5.2'

	cookbook 'composer', '~> 1.0.5'
```

This is including `php`, `apache`, `mysql` and `composer`

Next we will need to install all cookbooks which be specified in `Cheffile`

		librarian-chef install

After installing, you should see the cookbooks folder as below

{% img https://lh4.googleusercontent.com/-zg-x4_s1JLw/VFxCwKlTkfI/AAAAAAAAACE/sVvPi3G5hGA/s492/README_md_-__Users_congdang_my-training_codeforfoods_php-cooking_-_Atom.png %}

That's good so far, now we should setup `roles`, `enviroments` and `owner cookbooks` for install the web server and deploy your application to web server.

* Setup `dev environent`: create a file called `dev.json` in enviroment folder

{% img https://lh4.googleusercontent.com/-zg-x4_s1JLw/VFxCwKlTkfI/AAAAAAAAACE/sVvPi3G5hGA/s492/README_md_-__Users_congdang_my-training_codeforfoods_php-cooking_-_Atom.png %}

``` json dev.json
	{
	  "name": "dev",
	  "default_attributes": {
	    "apache2": {
    	  "listen_ports": [
        	"80",
	        "443"
    	  ]
	    },
    	"php": {
	      "ext_conf_dir": "/etc/php5/apache2/conf.d"
    	},
	    "myapp": {
    	  "user": "vagrant",
	      "root_path": "/vagrant",
    	  "server_name": "phpdev.codeforfoods.net"
	    }
	  },
	  "json_class": "Chef::Environment",
	  "description": "",
	  "chef_type": "environment"
	}

```

* Setup `myapp role` and `myapp deploy role`: create 2 file `myapp.json` and `myapp_deploy.json` in roles folder

{% img https://lh6.googleusercontent.com/-yoM7_7UiHBY/VFxCYAUZpJI/AAAAAAAAABs/xbd3d4fabn8/s184/README_md_-__Users_congdang_my-training_codeforfoods_php-cooking_-_Atom.png %}


``` json myapp.json

	{
		"name": "myapp",
		"description": "The role for app servers",
		"json_class": "Chef::Role",
		"default_attributes": {},
		"override_attributes": {},
		"chef_type": "role",
		"run_list": [
			"recipe[apt]",
			"recipe[build-essential]",
			"recipe[composer]",
			"recipe[myapp]",
			"recipe[myapp::deploy]"
		],
		"env_run_lists": {
			"dev": [
				"recipe[apt]",
				"recipe[build-essential]",
				"recipe[composer]",
				"recipe[myapp]",
				"recipe[myapp::deploy]"
			]
		}
	}


```

``` json myapp_deploy.json

	{
		"name": "myapp_deploy",
		"description": "The role for app servers",
		"json_class": "Chef::Role",
		"default_attributes": {},
		"override_attributes": {},
		"chef_type": "role",
		"run_list": [
			"recipe[myapp::deploy]"
		],
		"env_run_lists": {
			"dev": [
				"recipe[myapp::deploy]"
			]
		}
	}


```

* Create a owner cookbook called `myapp`, this should be store in `site-cookbooks` folder

			knife cookbook create myapp

{% img https://lh3.googleusercontent.com/-7wagSNwnOWI/VFxESOwCZCI/AAAAAAAAACs/Xe7vU8MlycA/s415/README_md_-__Users_congdang_my-training_codeforfoods_php-cooking_-_Atom.png %}

Notes:

1) 2 recipes, one for myapp default and other for myapp deploy

``` ruby defaut.rb
	include_recipe "apache2"
	include_recipe "mysql::client"
	include_recipe "mysql::server"

	include_recipe "php55"


	include_recipe "php::module_mysql"
	include_recipe "php::module_curl"
	include_recipe "php::module_memcache"
	include_recipe "apache2::mod_php5"
	include_recipe "apache2::mod_expires"

	include_recipe "php-mcrypt"

	# disble default site.
	apache_site "default" do
		enable false
	end

```

``` ruby deploy.rb
	# grant permission for webroot
	directory node["myapp"]["root_path"] do
		owner "root"
		group "root"
		mode "0755"
		action :create
		recursive true
	end

	# reading the data bag
	#secrets = Chef::EncryptedDataBagItem.load("secrets", "myapp")

	if node.chef_environment == "dev"
			# enable kizang-api site.
			web_app 'myapp' do
			template 'site.conf.erb'
			docroot node['myapp']['root_path']
			server_name node['myapp']['server_name']
	end
	else

	end

```

2) the Apache configuration template file

``` ruby site.conf.erb
	<VirtualHost *:80>
		ServerName <%= @params[:server_name] %>
		DocumentRoot <%= @params[:docroot] %>

		<Directory <%= @params[:docroot] %>>
			Options FollowSymLinks
			AllowOverride All

			<% if node['apache']['version'] == '2.4' -%>
					Require all granted
				<% elsif node['apache']['version'] == '2.2' -%>
				Order allow,deny
				Allow from all
				<% end -%>

			RewriteEngine On
			RewriteBase /
			RewriteRule ^index\.php$ - [L]
			RewriteCond %{REQUEST_FILENAME} !-f
			RewriteCond %{REQUEST_FILENAME} !-d
			RewriteRule . /index.php [L]


		</Directory>

		<Directory />
			Options FollowSymLinks
			AllowOverride None
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/myapp.error.log
		# Possible values include: debug, info, notice, warn, error, crit,
		# alert, emerg.
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/myapp.access.log combined
	</VirtualHost>

```

#### Step 3 - Setup vagrant for create development environment

Ok! Almost Chef setting up is done, now you should create a `Vagrantfile` and use chef-solo as provision for install development server

``` ruby Vagrantfile

	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
	VAGRANTFILE_API_VERSION = "2"
	CHEF_PATH = "."
	SYNC_PATH = "./www"
	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

		config.vm.box = "ubuntu14.04"
		config.vm.box_url = "https://oss-binaries.phusionpassenger.com/vagrant/boxes/latest/ubuntu-14.04-amd64-vbox.box"
		config.vm.network "private_network", ip: "192.168.34.100"
		config.vm.hostname = "devphp.congdang.com"
		config.ssh.forward_agent = true
		config.ssh.forward_x11   = true

		config.vm.provider "virtualbox" do |vb|
			vb.customize(["modifyvm", :id, "--natdnshostresolver1", "off"  ])
			vb.customize(["modifyvm", :id, "--natdnsproxy1",        "off"  ])
			vb.customize(["modifyvm", :id, "--memory",              "1024" ])
		end



		config.omnibus.chef_version = '11.16.0'
		config.vm.provision :chef_solo do |chef|


		chef.cookbooks_path = "#{CHEF_PATH}/cookbooks", "#{CHEF_PATH}/site-cookbooks"
		chef.environments_path = "#{CHEF_PATH}/environments"
		chef.environment = "dev"
		chef.roles_path = "#{CHEF_PATH}/roles"
		chef.add_role('myapp')
		end

		config.vm.synced_folder("#{SYNC_PATH}", "/vagrant",
														:owner => "vagrant",
														:group => "vagrant",
														:mount_options => ['dmode=777','fmode=777'])
	end


```

Notice that, I used `www` vagrant's sync folder. That means you should store your web application in `www` folder. In this article I just created a simple php file in that folder

``` php index.php

	<?php
		echo "Hello PHP world!!!!";
	?>

```

Finally you need to run vagrant and check the site on the browser

			vagrant up

Once vagrant upping is done, you can check your application on browser

			http://192.168.34.100

#### Conclusion

You can pull all code from my github

``` bash repository on gihub
	https://github.com/code-for-food/php-cooking.git # fork from github

```

In this Chef, I've setup `mysql` as well but I just only used `php` and `apache`. You can add more code in `defaul.rb` recipe for using `mysql` for your project. If you have any question, you can drop an email to me [codeforfoods](mailto:code.for.food.14@gmail.com).

Thanks for your reading!!!
