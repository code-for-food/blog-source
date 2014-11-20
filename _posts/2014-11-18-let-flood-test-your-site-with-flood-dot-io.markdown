---
layout: post
title: "Let Flood test your site with Flood.io"
date: 2014-11-18 23:45:08 +0700
comments: true
author: codeforfoods
source_name: flood.io
source_site: https://flood.io/help/floods/getting_started_using_jmeter
fb_img: https://s3.amazonaws.com/flood-io-support/Flood_IO_2014-10-06_17-49-17.jpg
categories: [test, flood]
---

Flood IO is a cloud based load testing platform for [JMeter](https://flood.io/help/floods/jmeter.apache.org) and [Gatling](https://flood.io/help/floods/gatling-tool.org) that lets you scale out your tests across the globe using our on demand or your own hosted AWS infrastructure. In this article we introduce you to JMeter and how to run your first Flood test.

In this article, I'd like to use `JMeter` as testing scripts

###Getting JMeter

JMeter can be downloaded [here](http://jmeter.apache.org/download_jmeter.cgi) and typically you can start JMeter using the `jmeter.cmd` or `jmeter.sh` scripts located in the downloaded archive `bin` directory.

{% img https://s3.amazonaws.com/flood-io-support/jmeter-example.jmx_UserstimkoopmansGitfloodflood-sitepublicjmeter-example.jmx_-_Apache_JMeter_2.11_r1554548_2014-10-06_17-37-40.jpg %}

###Creating a Test Plan
Once you have created a [basic test plan](https://flood.io/jmeter-example.jmx) and saved it locally as a `.jmx` file, you can upload this to Flood IO to create your first Flood test.

###Creating a Flood Test

After you login to Flood IO you can click the "+" icon or from your [Floods dashboard](https://flood.io/dashboard/floods) you can click the "Create Flood" button to create a new Flood.

{% img https://s3.amazonaws.com/flood-io-support/Flood_IO_2014-10-06_17-36-50.jpg %}

In the new Flood form, add the `.jmx` test plan you created using JMeter to the Files input and select a Grid to run your Flood test on. You can optionally set the "Name" or "Tags" to help identify the test in your dashboard.

{% img https://s3.amazonaws.com/flood-io-support/Flood_IO_2014-10-06_17-49-17.jpg %}

###Start your Flood Test
Read about more [advanced settings] or simply click the "Start Flood" button to launch your test.

{% img https://s3.amazonaws.com/flood-io-support/Flood_IO_2014-10-06_17-53-22.jpg %}

You will be redirected to a Flood report which will show you results in real time from the test you just executed on the chosen Grid.

###The results in real time

* Summary about AVG Response Time, Concurence Users and Transaction Rate

{% img https://lh3.googleusercontent.com/-cbQyegUkAt8/VGt77yF3o_I/AAAAAAAAADE/zIceQUvDRnY/w957-h199-no/Flood_IO.png %}

* Summary report by chart

{% img https://lh3.googleusercontent.com/-LdsF3ANppGY/VGt8J5PJa3I/AAAAAAAAADM/dN75wBnBQ7M/w958-h404-no/Flood_IO.png %} 

* Detail transaction

{% img https://lh3.googleusercontent.com/-NLjHvr3TwME/VGt8m92lEoI/AAAAAAAAADc/9o8vDRsEzBE/w957-h384-no/Flood_IO.png %}

* You can trace your http connection

{% img https://lh3.googleusercontent.com/ItYy1EO4CTEt8fFFxFNKxkwdgR9uey33boD5RZ_MyA=w400-h207-p-no %}


