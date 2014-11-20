---
layout: post
title: "Nodejs cooking"
date: 2014-11-03 09:47:33 +0700
comments: true
author: codeforfoods
categories: [chef, cooking, nodejs]
---

* This project is Chef-Solo, this help to setup development base on Nodejs and Mysql

* This also includes a demo about Simple API which using Express and Mysql

#### What was including

* Using [Chef-solo](https://docs.getchef.com/chef_solo.html) for cooking
* Using [Knife-solo](http://matschaffer.github.io/knife-solo) for initialazing, adding, editing `cookbooks`, `roles`, `environment`
* Using [librarian-chef](https://github.com/applicationsonline/librarian-chef) for managing `cookbooks`

#### Installing

* Installing `librarian-chef` and `knife-solo`
* initialazing librarian, this would create new forder with named `cookbooks` and a `Cheffile`

		librarian-chef init
		
* Adding `nvm` and `msql` cookbook to `Cheffile`

		cookbook 'mysql', '~> 5.5.2'
		cookbook 'nvm', '~> 0.1.0'	
	
* Initialazing `knife-solo`, this would create the folder structure

		knife-solo init
		
* Installing cookbooks from `Cheffile`

		libratian-chef install
		

* Creating an encryption key

		openssl rand -base64 512 | tr -d 'rn' > ~/.chef/encrypted_data_bag_secret
		chmod 600 ~/.chef/encrypted_data_bag_secret

* Adding the path of encryption key to `.chef/knife.rb`

		knife[:secret_file] = ".chef/encrypted_data_bag_secret"


* Creating a data bag, this stores the security inforamtion (ex database's password)

		 knife solo data bag create secrets myapp
		
Enter something

		{
		  	"id": "myapp",
			 "aws_config": {
		    	"AWSAccessKeyId": "xxx",
		    	"AWSSecretKey": "xxx"
			  },
		  "db_config": {
		    "dev": [
      			{
			        "name": "default",
			        "mysql_root_password": "abc@123",
			        "hostname": "localhost",
			        "username": "root",
			        "password": "abc@123",
			        "database": "myqpp"
      			}
		    ]
		  }
		}
		
* The encrypted data bag will be stored into `data-bag/secrets/myapp.js`
		
		{
		  "id": "myapp",
		  "aws_config": {
		    "encrypted_data": 				"RjARHBKLm0DTbHDeIB5XjTjdWovyvv5T3V9T79UyEOq4/5Km6LaZ4y7leP7/\nNLCNQaJsUZQASaAII3bzkmwamecn03q5XUp4yDzeYcCqyRhP18BnyCxYDcOm\nc0HD1xEVuOanTt5R7ppJMj3kzN02YESN4CA/+70f0Qkp6yexhVo=\n",
		    "iv": "Yr9pDrhjZjhLRxHEBna0nw==\n",
		    "version": 1,
		    "cipher": "aes-256-cbc"
		  },
		  "db_config": {
		    "encrypted_data": "F6iPNUMY+dzao52SpQqHEWlFbGWUBMWSBAy4FinaCQ0Tbh06nDPgSdNI7qoJ\n3DjfrLBZOV1/8xD9t+SkJI3BEvW+yxCnnDPEz+iiZ8/jYJaypy/Bhrn42EC4\ni2VPb66HnVYD4Uq0pObVWzhLATXfqNZ/79I/bTdHTuhGcynDNzRNjyYEDfIF\nG9SuooNjUAduwmLotW//XxMM4uFNZ04hww==\n",
		    "iv": "DbN/KOPibOqNtAkLbdBr0w==\n",
		    "version": 1,
		    "cipher": "aes-256-cbc"
		  }
		}

* Creating a owner cookbook (I created `myapp` cookbook), this should create at `site-cookbook` folder

		knife cookbook create  myapp
		


* Configurating the default root password for mysql::server add `site-cookbooks/myapp/attributes/default.rb` (create one if this file is not existed)

		secrets = Chef::EncryptedDataBagItem.load("secrets", "myapp")

		default['mysql']['server_root_password']   = secrets["db_config"][node.chef_environment][0]["mysql_root_password"]
		default['mysql']['server_debian_password'] = secrets["db_config"][node.chef_environment][0]["mysql_root_password"]
		default['mysql']['server_repl_password']   = secrets["db_config"][node.chef_environment][0]["mysql_root_password"]
		
* Installing `mysql` and `nvm` and `node 0.10.33` in `site-cookbooks/myapp/attributes/recipes/default.rb`

		include_recipe "mysql::client"
		include_recipe "mysql::server"
		include_recipe 'nvm'

		# install node.js v0.10.5
		nvm_install 'v0.10.33'  do
	    	from_source true
	    	alias_as_default true
		    action :create
		end

* Creating `deploy recipe`, this recipe would  help to deploy latest code from `github` to server and `start/restart server`

* Creating the `enviroments` (`dev` for Vagrant and `prod` for real server)

		kinfe evironment create dev
		
Enter something

		{
  			  "name": "dev",
			  "description": "",
			  "cookbook_versions": {

			  },
			  "json_class": "Chef::Environment",
			  "chef_type": "environment",
			  "default_attributes": {
			    "myapp": {
			      "root_path": "/vagrant"
			    }
		  	},
		  	"override_attributes": {

  			}
		}
		
* Creating the `roles`

		knife role create myapp
		
Enter something

		{
		  "name": "myapp",
		  "description": "",
		  "json_class": "Chef::Role",
		  "default_attributes": {

		  },
		  "override_attributes": {

		  },
		  "chef_type": "role",
		  "run_list": [

		  ],
		  "env_run_lists": {
		    "dev": [
		      "recipe[myapp]",
		      "recipe[myapp::deploy]"
		    ]
		  }
		}	
		
* Creating `Vagrantfile` which would use `chef-solo` as provision

``` ruby Vagrantfile https://github.com/code-for-food/node-cooking/blob/master/Vagrantfile Link
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
		  config.vm.hostname = "devnodejs.chicken.com"
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
			   chef.data_bags_path = "#{CHEF_PATH}/data_bags"
			   chef.encrypted_data_bag_secret_key_path = "#{CHEF_PATH}/.chef/encrypted_data_bag_secret"
			   chef.add_role('myapp')
		  end

		  config.vm.synced_folder("#{SYNC_PATH}", "/vagrant",
                          :owner => "vagrant",
                          :group => "vagrant",
                          :mount_options => ['dmode=777','fmode=777']) 
		end

```
		

	
### Usage

#### Develop Environment using Vagrant

Notes: 

* You must install [Vagrant](https://www.vagrantup.com/) first
* Install `vagrant-omnibus` plugin

		vagrant plugin install vagrant-omnibus

Vagrant up

		vagrant up


You see something like that

		Running handlers:
		[2014-11-02T07:16:20+00:00] INFO: Running report handlers
		Running handlers complete
		[2014-11-02T07:16:20+00:00] INFO: Report handlers complete
		Chef Client finished, 30/36 resources updated in 756.082469267 seconds

#####Congratulation!!! You are Chef now :)

``` ruby git repos https://github.com/code-for-food/node-cooking/blob/master/Vagrantfile repos
$ git clone https://github.com/code-for-food/node-cooking.git # fork from github
```

### License
Written by [code-for-food](mailto:code.for.food.14@gmail.com)

Copyright (c) 2014 Code-For-Food.

Welcome any comments, please contact me via

* Email: code.for.food.14@gmail.com



