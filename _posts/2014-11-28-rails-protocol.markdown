---
layout: post
title: "Rails Protocol"
date: 2014-11-28 10:05:34 +0700
source_name: rails-protocol
source_site: https://github.com/thoughtbot/guides/tree/master/protocol/rails
comments: true
categories: [rails]
author: chicken-dp
---

A guide for writing great web apps.

#### Set Up Laptop

Set up your laptop with : 

* https://github.com/thoughtbot/laptop 

* https://github.com/thoughtbot/dotfiles

#### Create App

Get Suspenders.

```
	gem install suspenders
```

Create the app.

```
	suspenders app --heroku true --github organization/app
```

#### Set Up App

Get the code 

```
	git clone git@github.com:organization/app.git
```

Set up the app's dependencies

```
	cd project
	./bin/setup
```

Use Heroku config to get ENV variables. (https://github.com/ddollar/heroku-config)

```
	heroku config:pull --remote staging
```

Delete extra lines in .env, leaving only those needed for app to function properly. For example: BRAINTREE_MERCHANT_ID and S3_SECRET.

Use Foreman to run the app locally (https://github.com/ddollar/foreman)

```
	foreman start
```

It uses your .env file and Procfile to run processes like Heroku's Cedar stack. (https://devcenter.heroku.com/articles/cedar)

#### Git protocol

https://github.com/thoughtbot/guides/tree/master/protocol/git

#### Product Review

https://github.com/thoughtbot/guides/tree/master/protocol/product-review

#### Code Review

https://github.com/thoughtbot/guides/tree/master/code-review

* Review data integrity closely, such as migrations that make irreversible changes to the data, and whether there is a related todo to make a database backup during the staging and production deploys.

* Review SQL queries for potential SQL injection.

* Review whether dependency upgrades include a reason in the commit message, such as a link to the dependency's 
ChangeLog or NEWS file.

* Review whether new database indexes are necessary if new columns or SQL queries were added.

* Review whether new scheduler (cron) tasks have been added and whether there is a related todo in the project management system to add it during the staging and production deploys.

#### Deploy 

View a list of new commits. View changed files.

```
	git fetch staging
	git log staging/master..master
	git diff --stat staging/master
```

If necessary, add new environment variables.

```
	heroku config:add NEW_VARIABLE=value --remote staging
```

Deploy to Heroku

```
	git push staging
```

If necessary, run migrations and restart the dynos.

```
	heroku run rake db:migrate --remote staging
	heroku restart --remote staging
```

Introspect to make sure everything's ok.

```
	watch heroku ps --remote staging
```

Test the feature in browser.

Deploy to production.

```
	git fetch production
	git log production/master..master
	git diff --stat production/master
	heroku config:add NEW_VARIABLE=value --remote production
	git push production
	heroku run rake db:migrate --remote production
	heroku restart --remote production
	watch heroku ps --remote production
```

Watch logs and metrics dashboards.

Close pull request and comment Merged.

#### Set Up Production Environment

* Make sure that your Procfile is set up to run Unicorn. (https://devcenter.heroku.com/articles/procfile)

* Make sure the PG Backups add-on is enabled.

* Create a read-only Heroku Follower for your production database. If a Heroku database outage occurs, Heroku can use the follower to get your app back up and running faster. (https://devcenter.heroku.com/articles/heroku-postgres-follower-databases#high-availability-with-followers)
