---
layout: post
title: "Crystal: Getting Started"
date: 2014-11-27 16:43:51 +0700
comments: true
categories: 
---

###Introduction:
Crystal is a programming language that resembles Ruby but compiles to native code and tries to be much more efficient, at the cost of disallowing certain dynamic aspects of Ruby.

###Installation
####1. On Debian and Ubuntu
```
curl http://dist.crystal-lang.org/apt/setup.sh | sudo bash
apt-key adv --keyserver keys.gnupg.net --recv-keys 09617FD37CC06B54
echo "deb http://dist.crystal-lang.org/apt crystal main" > /etc/apt/sources.list.d/crystal.list
sudo apt-get install crystal
```
####2. On Mac OSX using Homebrew
```
brew tap manastech/crystal
brew install crystal
```

###Hello World
This is the simplest way to write the Hello World program in Crystal:
```
puts "Hello World"
```
First create a file hello.cr containing your code. Then type in the console:
```
chmod +x hello.cr
crystal hello.cr
```

That's all! Hope you guys love it :D
I will have another post about syntax and semantics :)
Thanks