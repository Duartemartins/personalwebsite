---
layout: post
title: How to export Substack posts to Jekyll
description: A Ruby script to export Substack posts to Jekyll
date: 2022-09-21 17:45:46 +0100
published: true
categories:
tags:
lang:
---
A golden rule of the internet is that you should own your content, and not have it subject to a third party's arbitrary decisions. Substack is one such third party, however it is a great tool to syndicate content towards. So, how do we get the best of both worlds and get our Substack content into Jekyll? 

Substack doesn't have an API but you can export all of your Substack content fairly easily, see [their help article](https://support.substack.com/hc/en-us/articles/360037466012-How-do-I-export-my-posts-).

However, Substack exports everything in HTML, whereas Jekyll uses markdown. Luckily there is a Ruby gem built to convert HTML to Markdown called reverse_markdown, and it does a pretty good job of it. Using some simple Ruby scripting, we can add our usual font matter, and because Substack provides a CSV file with data on our posts, we can retrieve the date and time that the Substack post was published as well.

The following is a script I created to that effect. To use it with Ruby you'll need to run `gem install reverse markdown` in your terminal/shell and then use `ruby your_script_name.rb` in the directory you've saved the script in. The way it's currently written assumes that you've saved the script in the folder that the Substack export created. 
```ruby
require 'reverse_markdown'
require 'csv'
require 'date'

# csv = CSV.parse(File.read(Dir.pwd + '/posts.csv'), headers: true)

def front_matter(date,title)
return %(---
layout: post
title: #{title}
description:
date: #{DateTime.parse(date).strftime('%Y-%m-%d %H:%M:%S')} +0100
published: true
categories:
tags:
lang:
---
)
end

Dir.foreach('/your/directory/posts/') do |filename|
  next if filename == '.' || filename == '..' || File.extname(filename) != '.html'
  # skip if file does not end in .html
  CSV.foreach(Dir.pwd + '/posts.csv', headers: true) do |row|
    @date = row[1] if row[0].to_s == File.basename(filename.chomp, File.extname(filename)) && row[1]
  end
  #gets post date if in posts.csv file
  puts filename
  file = File.open(Dir.pwd + "/posts/#{filename.chomp}").read
  result = ReverseMarkdown.convert file
  title = File.basename(filename.chomp, File.extname(filename)).split('.').last
  date = !@date.nil? ? @date : '2022-09-21 16:16:38'
  # get post date if it has been published, otherwise use a set date and time
  File.open(Dir.pwd + "/posts/#{DateTime.parse(date).strftime('%Y-%m-%d').to_s + title}.markdown", 'w+') do |f|
    f.write front_matter(date,title) + result
  end
  # Create new markdown file
end
```
I hope that's useful. Obligatory Substack newsletter plug: <https://interessant3.substack.com/>

In addition, if you're looking for jobs in data with an effective social impact check out <https://www.gooddatajobs.com>

If you're looking for data analysis work for your organisation, feel free to DM/email me. See details below.