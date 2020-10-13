---
layout: post
title:  "Finding duplicate records in a CSV file column"
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

This runs over hundreds of rows of CSV data in seconds.