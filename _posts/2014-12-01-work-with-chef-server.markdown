---
layout: post
title: "work with chef server"
date: 2014-12-01 16:44:10 +0700
comments: true
author: codeforfoods
fb_img: https://lh5.googleusercontent.com/-PeezzBJhRrE/VHwwAKgzq7I/AAAAAAAAABk/OBw78YVf5Mk/w707-h604-no/Sign_Up_-_Chef_Manage.png
categories: [php,chef]
---

In [previous post](blog/2014/11/07/php-cooking/) I have wrote how to use Chef Solo to setup and deploy a Server with PHP stuff. Today I'd like to use Chef Server for storing Chef's configurations (Cookbooks, Roles, Enviroments, Data Bags) and Bootstrap a server use Chef Server.


Setup Chef Server
===
The easiest way to get started with a Chef server is to use Opscode’s [Hosted Chef](http://www.opscode.com/hosted-chef/)–it’s free for up to 5 nodes. Once you exceed 5 nodes you can decide whether to pay a monthly fee or install your own Chef server.

* Register your free trial of hosted Chef

{% img https://lh5.googleusercontent.com/-PeezzBJhRrE/VHwwAKgzq7I/AAAAAAAAABk/OBw78YVf5Mk/w707-h604-no/Sign_Up_-_Chef_Manage.png %}

* Get pem key from profile

{% img https://lh6.googleusercontent.com/-_tKhvVNO0u8/VHww8xtE59I/AAAAAAAAABw/O_cbtONexe0/w791-h640-no/Chef_-_Account_Management.png %}

* Generate Knife config

{% img https://lh4.googleusercontent.com/-xyw-Lzc3V5Y/VHwzDvnqQwI/AAAAAAAAACY/RJsXXJP_6VY/w710-h640-no/Chef_Manage_and_php-cooking_%E2%80%94_bash_%E2%80%94_162%C3%9730.png %}

* Fork the `php-cooking` from [previous post](blog/2014/11/07/php-cooking/)


```
https://github.com/code-for-food/php-cooking.git # fork from github
```

* Move to `php-cooking` and copy client key and validation key to `.chef` folder.

* Replace the knife.rb in .chef folder

```ruby

current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "codeforfoods"
client_key               "#{current_dir}/codeforfoods.pem" # replace by your key
validation_client_name   "chicken-validator"
validation_key           "#{current_dir}/chicken-validator.pem" # replace by your key
chef_server_url          "https://api.opscode.com/organizations/chicken"
cache_type               'BasicFile'
cache_options( :path => "#{ENV['HOME']}/.chef/checksums" )


cookbook_path    ["cookbooks", "site-cookbooks"]
node_path        "nodes"
role_path        "roles"
environment_path "environments"
data_bag_path    "data_bags"
encrypted_data_bag_secret "#{current_dir}/encrypted_data_bag_secret"


```


* Upload cookbooks to Chef server

		knife cookbook upload --all
	
{% img https://lh6.googleusercontent.com/-0WxqaSjoQfE/VHw9TgYqHEI/AAAAAAAAACo/qyVX1k3ZFdY/w266-h293-no/php-cooking_%E2%80%94_bash_%E2%80%94_162%C3%9730.png %}

View on Chef server

{% img https://lh3.googleusercontent.com/-_jRaxKzYeac/VHw92i4RIYI/AAAAAAAAACw/ybd_zPQg7gA/w775-h591-no/Chef_Manage_and_Picasa_3.png %}

* Upload Roles to Chef Server


		 knife role from file roles/myapp.json 
		 knife role from file roles/myapp_deploy.json

View on Server

{% img https://lh6.googleusercontent.com/-ARi99k01P8E/VHw_QKAtUPI/AAAAAAAAAC8/weO6WZ8ub-o/w558-h186-no/Chef_Manage.png %}	 

* Create new enviroment called `staging` by create staging.json file in `enviroments` folder

```json staging.json
{
  "name": "staging",
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
      "user": "ubuntu",
      "root_path": "/var/www/codeforfoods",
      "server_name": "phpdev.codeforfoods.net"
    }
  },
  "json_class": "Chef::Environment",
  "description": "",
  "chef_type": "environment"
}
```

* Upload Enviroments to Chef Server

		knife environment from file environments/dev.json 
		knife environment from file environments/staging.json 

View on Server

{% img https://lh4.googleusercontent.com/-kUMwE9lMI1k/VHxAFiJZKcI/AAAAAAAAADE/tVMTf4MMHhQ/w333-h240-no/Chef_Manage.png %}	

* Create an encrypted data bag

	* Create encrypted data bag key, the key would help to decrypted the data bag
		
			openssl rand -base64 512 | tr -d 'rn' > ~/.chef/encrypted_data_bag_secret
			chmod 600 ~/.chef/encrypted_data_bag_secret
	* Add below line to `.chef/knife.rb`
	
			encrypted_data_bag_secret "#{current_dir}/encrypted_data_bag_secret"
	* Create the encrypted data bage
			 
			 knife data bag create --secret-file ~/.chef/encrypted_data_bag_secret secrets myapp
	
	{% img https://lh6.googleusercontent.com/--OFW4K4ZuFk/VHxfvrX4DCI/AAAAAAAAADU/SGCTIKaQW9s/w290-h249-no/php-cooking_%E2%80%94_vim_%E2%80%94_162%C3%9730.png %}
	
	View on Server
	
	{% img https://lh3.googleusercontent.com/-1W37bGO_9FU/VHxifiyiOCI/AAAAAAAAADg/VsHn4LbYVf4/w958-h421-no/Chef_Manage.png %}
	
Setup Vagrant use chef-client as provision
===

```ruby Vagrantfile
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

  # Your organization name for hosted Chef 
  orgname = "chicken"

  # Set the Chef node ID based on environment variable NODE, if set. Otherwise default to vagrant-$USER
  node = ENV['NODE']
  node ||= "vagrant-codeforfoods"

  config.vm.provision :chef_client do |chef|
    chef.chef_server_url = "https://api.opscode.com/organizations/#{orgname}"
    chef.validation_key_path = "#{CHEF_PATH}/.chef/#{orgname}-validator.pem"
    chef.validation_client_name = "#{orgname}-validator"
    chef.encrypted_data_bag_secret_key_path = "#{CHEF_PATH}/.chef/encrypted_data_bag_secret"
    chef.node_name = "#{node}"
    chef.provisioning_path = "/etc/chef"
    chef.log_level = :debug
    #chef.log_level = :info

    chef.environment = "dev" 
    chef.add_role("myapp")
    
    #chef.json.merge!({ :mysql_password => "foo" }) # You can do this to override any default attributes for this node.
  end   


  config.vm.synced_folder("#{SYNC_PATH}", "/vagrant",
                          :owner => "vagrant",
                          :group => "vagrant",
                          :mount_options => ['dmode=777','fmode=777'])
end


```	

Starting `Vagrant`

		vagrant up
		
Chef the webapp on browser

```

http://192.168.34.100

```		

Bootstrap a Server
===
[Cloudshare](http://cloudshare.com) provides a easy way to create new node for testing [More detail here](http://learn.getchef.com/ubuntu/bootstrap-your-node/)

* Get a Linux machine to bootstrap

{% img https://lh5.googleusercontent.com/-rJtVKEyp_-k/VHx18JYfv0I/AAAAAAAAADw/fECENQ9LPqo/w661-h524-no/Machines.png %}

* Bootstrap your node

		knife bootstrap uvo1vinqr3ty38kwnpn.vm.cld.sr -x sysadmin -P Oi9YB7HTfP --sudo -N node1 -r 'role[myapp]' -E staging

** If you meet the error while creating the node, please see the comment in this https://groups.google.com/forum/#!topic/learnchef-fundamentals-webinar/LYe_i_QIWdU

Error

```bash
54.79.58.34   knife ssl check -c /etc/chef/client.rb
54.79.58.34 ```
54.79.58.34 
54.79.58.34 * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
54.79.58.34 
54.79.58.34 Starting Chef Client, version 11.16.4
54.79.58.34 
54.79.58.34 ================================================================================
54.79.58.34 Chef encountered an error attempting to load the node data for "testnode"
54.79.58.34 ================================================================================
54.79.58.34 
54.79.58.34 Authentication Error:
54.79.58.34 ---------------------
54.79.58.34 Failed to authenticate to the chef server (http 401).
54.79.58.34 
54.79.58.34 Server Response:
54.79.58.34 ----------------
54.79.58.34 Failed to authenticate as 'testnode'. Ensure that your node_name and client key are correct.
54.79.58.34 
54.79.58.34 Relevant Config Settings:
54.79.58.34 -------------------------
54.79.58.34 chef_server_url   "https://api.opscode.com/organizations/cd-chef"
54.79.58.34 node_name         "testnode"
54.79.58.34 client_key        "/etc/chef/client.pem"

```

SSH to server and do this command
		
		sudo rm -rf /etc/chef/client.pem