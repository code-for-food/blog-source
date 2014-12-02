---
layout: post
title: "Scaling Ruby on Rails by Caching Your Database Queries"
date: 2014-11-28 08:34:54 +0700
source_name: scaling-ruby-on-rails-by-caching-your-database-queries
source_site: http://www.codebeerstartups.com/2014/11/scaling-ruby-on-rails-by-caching-your-database-queries/
comments: true
categories: [database, rails]
author: chicken-dp 
---

This article would help you to scaling your app with caching your database queries.

Its pretty good to use active-record in ruby on rails, relationships, find_by methods, where queries everything makes your life much more simpler as a developer but sooner or later your application becomes slow. At that time you need to work on optimizations. There are lot of work around for optimizations and one of them in caching the queries. The company I work for is StudyPad and we have a product ie splashmath and that has over 9 millions users using it across multiple platforms. The amount of traffic that we get is pretty huge and caching one of the thing that helped us to solve out scaling problems. Here is quick idea to cache your queries in memcache.

#### Basic requirements

There are few requirements for this. First is the dalli gem and memcache installed on your machine.

#### Basic structure of the application.

We have lot of static/seed data which change very less and the queries on those things are pretty high. So first thing to handle that static data.

```
	class Grade < ActiveRecord::Base
	  has_many :skills
	end

	class Skills < ActiveRecord::Base
	  belongs_to :grade
	end
```

So in order to get skills for a grade, its pretty simple

```
	grade = Grade.first
	skills = grade.skills
```

Now for that matter every time we are fetching skills for a grade its gonna hit the database and get the results. So its time to get this query and its results cached.

Assuming you have configured dalli gem easily and you have memcached up and running. Here is a simple method to cache the results

```
	def cached_skills
	  Rails.cache.fetch([self.class.name, id, :skills], expires_in: 240.hours) {
	    skills.to_a
	  }
end
```

So instead of fetching skills like

```
	grade.skills
```

we are gonna get it like

```
	grade.cached_skills
```

So whats its gonna do. Its gonna fetch the results first time from the database and store it in cache. Next time its gonna return the results from the cache. Note few things here

* Understand the pattern of the key. [self.class.name, id, :skills] is your key here.

* Cache will expire in 240.hours. You can customize it as per your needs. Better keep a constant for this somewhere in your application.

* In cached_skills methods we are keep records not the active-record relations that why we have to convert into array by using to_a else active-record-relation will be cached and database query will be executed.

```
	skills.to_a
```

#### Expiring the cache.

We are caching the query results but we are not expiring the results. What if some skill has changed. Grade object is not getting any notification for that, so cache is stale, we need to expire it. So we can write a after_commit hook for skill to expire its grade object’s cache

```
	# in skill model
	after_commit :flush_cache

	def flush_cache
	  Rails.cache.delete([self.grade.class.name, self.grade_id, :skills])
	end
```

This is enough to make sure you cache is never stale. There is another way to handle the expiring cache. Lets see that.

#### Another way

We redefine the models like this

```
	class Grade < ActiveRecord::Base
	  has_many :skills
	end

	class Skills < ActiveRecord::Base
	  belongs_to :grade, touch: true
	end
```

Note we have added touch: true in skills, and now we redefine our cached_skills method again:

```
	def cached_skills
	  Rails.cache.fetch([self.class.name, updated_at.to_i, :skills], expires_in: 240.hours) {
	    skills.to_a
	  }
	end
```

Now just caching this we don’t need to expire the cache manually, when ever skills get updated it will touch its parent object object grade, that will update its updated_at value and that specific cache will be never used, as key attribute updated_at has been changed.

#### The problem with second approach

But there is a problem. Assume you have 10 different has_many relationships for grade and you are caching it all, now everytime a skill has be changed all the other cache keys for grade relationships will be useless too. For example: Grade has_many topics

```
	class Grade < ActiveRecord::Base
	  has_many :skills
	end

	class Skill < ActiveRecord::Base
	  belongs_to :grade, touch: true
	end

	class Topic < ActiveRecord::Base
	  belongs_to :grade, touch: true
	end
```
	
```
	# In Grade model
	def cached_skills
	  Rails.cache.fetch([self.class.name, updated_at.to_i, :skills], expires_in: 240.hours) {
	    skills.to_a
	  }
	end

	def cached_topics
	  Rails.cache.fetch([self.class.name, updated_at.to_i, :topics], expires_in: 240.hours) {
	    topics.to_a
	  }
	end
```

Now in this case changing any skill will make topics cache useless, but that’s not the case when you are trying to expire it manually. So both approach has pros and cons, first will ask you write more code and second expire cache more frequently. You have to make that choice as per your needs.

#### What else?

Using the same base principle you can cache lot of queries like

```
Grade.cached_find_by_id(1)
Skill.first.cached_grade
```

This approach helped us to reduce the load on RDS and make things pretty fast. I hope this will help you too. Let me know you feedback or some tips that made you system more faster