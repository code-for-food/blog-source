---
layout: post
title: "[aws] - Adding a Ec2 node to an Elastic Load Balancer"
date: 2014-11-03 21:42:35 +0700
comments: true
categories: [bash, aws, elb]
---

### Installing

* Install `Python`

				brew install python

	
* Install `AWS CLI` - AWS CLI (<http://aws.amazon.com/cli/>) is command line tool for working with Amazon Web Service. In this project, we used it for checking the `Elastic Load Balancer` (ELB) is existed or not, creating an ELB, adding the `Health check` for ELB, retrieving the `EC2 instance` information registering `EC2 instances` to ELB.

		pip install awscli
		
* Create config for aws at ~/.aws/config and add below content

``` bash Config file http://aws.amazon.com/cli/ link
	[profile default]
	output = json
	region = us-east-1
	aws_access_key_id = <access key id>
	aws_secret_access_key = <secret access key>
```

* Install `jq` <http://xmodulo.com/how-to-parse-json-string-via-command-line-on-linux.html>.

	`jq` is tool for parser, filter, map, and transform JSON-structured data effortlessly by command-line. We used it for parser the json data which retured form AWS CLI. [Read more on my post](/blog/2014/11/03/command-line-json-processor/)
	
		brew install jq	
		
### Coding

``` bash add-node-2-elb.sh
	#!/bin/bash
	#

	# ask to add the node to load balancer

	node=$1;
	role=$2;
	env=$3;
	lbName="MyLoadBalancer"
if [ "$node" != "" ] && [ "$role" != "" ]; then

	if [ "$role" == "web" ]; then
		if [ "$env" == "prod" ]; then
			lbName="MyLoadBalancerProd"
		else
			lbName="MyLoadBalancerStaging"
		fi	
	else	
		printf "Please enter a valid role ('web') \n";
	fi	

	# http://docs.aws.amazon.com/cli/latest/reference/elb/describe-load-balancers.html
	MyLoadBalancer=$(
		aws elb describe-load-balancers \
			--load-balancer-name $lbName \
		);

	# the load balancer is node existed, create the ELB first
	if [ "$MyLoadBalancer" == '' ]; then
		
		# http://docs.aws.amazon.com/cli/latest/reference/elb/create-load-balancer.html	
		aws elb create-load-balancer \
			--load-balancer-name $lbName \
			--listeners Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80 \
			--availability-zones us-east-1a \
			--security-groups sg-bac6eddf \

		#http://docs.aws.amazon.com/cli/latest/reference/elb/configure-health-check.html
		aws elb configure-health-check \
			--load-balancer-name $lbName \
			--health-check Target=HTTP:80/index.php,Interval=30,UnhealthyThreshold=2,HealthyThreshold=2,Timeout=3

	else
		printf "$MyLoadBalancer\n";
	fi

	# http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html
	# http://xmodulo.com/how-to-parse-json-string-via-command-line-on-linux.html
	# MAC - brew install jq

	nodeId=$(aws ec2 describe-instances --filters "Name=ip-address,Values=$node" | jq ".Reservations[0].Instances[0].InstanceId")
	nodeId="${nodeId%\"}"; 
	# http://docs.aws.amazon.com/cli/latest/reference/elb/register-instances-with-load-balancer.html
	aws elb register-instances-with-load-balancer \
		--load-balancer-name $lbName \
		--instances "${nodeId#\"}"
else
	printf "Please enter the node and role adding to ELB.\n";
fi
```		
	
### Usage

	./add-node-2-elb.sh 123.456.789.910 web staging	

