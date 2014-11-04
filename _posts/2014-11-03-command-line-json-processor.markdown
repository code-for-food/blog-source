---
layout: post
title: "Command-line JSON processor"
date: 2014-11-03 11:23:05 +0700
comments: true
categories: [bash, json]
---

`jq` is a command-line JSON processor.

If you want to learn to use jq, read the documentation at <http://stedolan.github.io/jq>. This documentation is generated from the docs/ folder of this repository. You can also try it online at <http://jqplay.org>.

### Basic filters

``` bash Example1 http://stedolan.github.io/jq/manual/exemple1 exemple
	jq '.'
	Input	"Hello, world!"
	Output	"Hello, world!"
```
``` bash Example2 http://stedolan.github.io/jq/manual/exemple2 exemple
	jq '.foo'
	Input	{"foo": 42, "bar": "less interesting data"}
	Output	42
```


### More details please check it out from original website

``` bash Original website http://stedolan.github.io/jq/manual original website
	http://stedolan.github.io/jq/manual/	
```

Next post, I'd like to use this for parser the `aws` result. (coming soon.)