---
layout: post
author: jean-romain krupa
title: "From a CSV to your Database"
date: 2021-04-01 11:39:51 +0100
tags: rails SQL project
---

Sometimes datas are on a a CSV and you have to store those datas in your DB. My first intuition, (which is rarely the most performant) to populate the DB would be to: open, read and an create objects with a Model.find_or_create_by. Something like this :

```ruby
require 'csv'
appellations_filepath = File.join(__dir__,'../../db/data/appellations_bourgogne.csv')

CSV.foreach(appellations_filepath) do |row|
  Appellation.find_or_create_by(name: row[0],
                                wine_region: row[1],
                                country: row[2])
end
```

It works fine, but let's have a look to the perf. For 128 lines this method last approximatively : 0.80s on my machine. you can test by adding those lines in between your function :

```ruby
start = Time.now
# the function you want to test
finish = Time.now
p execution_time = finish - start
# 0.80613
```

So now, here is the trick, instead of creating an element at every row of the CSV, we can use plain old SQL with the COPY function.

```ruby
appellation_filepath = File.join(__dir__,'../../db/data/appellations_bourgogne.csv')

	# the magic is there 👇
  sql = <<-SQL
  COPY public.appellations (name, wine_sub_region, wine_region, country, origin_label)
  FROM '#{appellation_filepath}'
  DELIMITER ','
  CSV HEADER QUOTE '"'
  SQL

  ActiveRecord::Base.connection.execute(sql)

	finish = Time.now
  p execution_time = finish - start
	# 0.001766s !!!
```

And guess what ! It's bloody more efficient **0.001766s VS 0.80.** At this stage you may think well it's useless. But for 1 million lines it's **10s VS > 1h**

So thank you [JOJO](https://www.malt.fr/profile/josephblanchard) for the tip !
