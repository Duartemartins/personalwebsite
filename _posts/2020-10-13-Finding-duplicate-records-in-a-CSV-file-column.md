---
layout: post
title:  "Finding duplicate records in a CSV file column"
description: "A Ruby script to get duplicate values in a CSV."
date:   2020-10-13 19:20:00 +0100
categories: programming
tags: ruby data
---
I sometimes need to retreive duplicate records from large CSV files, so I wrote this neat little two line Ruby script. It gets all the duplicate records in a column and prints them in the console.

You'll need Ruby >2.7.1 for this since it uses two features introduced in 2.7, namely the tally method, and numbered parameters.

* The tally method counts the occurrences of a unique value in an array, and returns a hash.

* Numbered parameters let you access enumerator variables in order.

The logic of this script is the following: I first get a CSV and read it as a table with headers. I then get the column by header name, in this case 'Email', and tally all the emails. This would return a hash like so: ```{"john@smith.com"=>10, "satya@gmail.com"=>25, "seamus@icloud.com"=>12}```. I then sort that hash (remember hashes are key => value pairs) so that the value, in this case represented by ```_2```, is more than 1. This is so that only duplicate records are returned. I then sort by the largest to smallest amount of occurrences, and map only the first value so that we only get the key of the hash, in this case the emails, and not the amount of times it occurs.

{% highlight ruby %}
require 'csv'

csv = CSV.read('export.csv', headers: true)

puts csv['Email'].tally.filter { _2 > 1 }.sort_by { -_2 }.map(&:first)

{% endhighlight %}

If we wanted to join several rows and find duplicate records with several variables in common, say first name, job title, country, and email, we could do the following:

{% highlight ruby %}
require 'csv'

col_data = []

CSV.foreach('export.csv', headers: true) do |row|
  col_data << "#{row[1]}, #{row[2]}, #{row[3]}, #{row[4]}"
end

col_data.tally.filter { _2 > 1 }.sort_by { -_2 }.each do |k, v|
  puts "#{k}, #{v}"
end

{% endhighlight %}

This puts the CSV rows into an array, then returns a hash like in the first example, the difference being that it then returns the variables with the occurrence count next to it, e.g.: ```Dwight, Assistant to the Regional Manager, USA, dwight@dmifflin.com, 34```. This is quite efficient and runs over hundreds of thousands of rows of CSV data in seconds.