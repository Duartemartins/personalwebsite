---
layout: post
title: "A Rakefile to automate your ruby builds"
description: "Creating a rakefile to automate your build tasks"
date: 2021-02-13 19:20:00 +0100
categories: programming
tags: ruby
---

There are a few tasks I like to perform every time I start a new project in Ruby:

1. Create the project directory
2. Create a Gemfile with a few gems I require
3. Create a folder structure for my minitest tests
4. Run Bundler

With that in mind and as a fan of automation, I set about researching how to perform these tasks automatically. Ruby has a great build automation library (gem) called Rake. The name comes from C's Make library - Rake being short for *Ruby Make*. It doesn't have the most straightforward syntax however, so I hope this example can help you get started using it if you're looking at making your Ruby development faster. 

The gems I usually install are:

- 'minitest' - a testing library
- 'rubocop' - for formatting
- 'solargraph' - this nearly transforms vscode from a text editor into an IDE
- 'ruby-debug-ide' - for debugging in vscode
- 'debase' - required for the above
- 'ripper-tags' - a gem for ctags in Ruby. These let you check method and class definitions easily.
- 'rake'

My idea was to start every project by creating a simple Rake command in my code directory and pass it the name of the new directory I wanted to create. This way I could automate the folder structure creation too. To do this, one needs to place the Rakefile in the root folder of where one's projects go.

Rake can take command line arguments which makes it great to build a CLI for Ruby projects. Now evert time I want to start a new ruby project, I change into my code directory and use `Rake default dir=foldername`. Default is simply the name of the task, and the argument it takes is the new directory name.

This creates the following structure:

{% highlight html %}
.
Rakefile <!--The Rakefile with the code below-->
├── foldername
│   └── Gemfile
│   └── lib
│   └── bin
...
{% endhighlight %}

Here's the Rakefile code:

{% highlight ruby linenos%}
desc 'Default task is builder:start'
task default: 'builder:start'

dir = Dir.pwd + "/ruby/#{ENV['dir'].to_s}"

gems = %w[
  'minitest'
  'rubocop'
  'solargraph'
  'ruby-debug-ide'
  'debase'
  'ripper-tags'
  'rake'
]

namespace :builder do
  desc 'Create Gemfile and start build process'
  task :start do |_t, args|
    mkdir dir
    puts "Directory is #{dir}"
    Dir.chdir(dir) do
      touch './Gemfile'
      File.open('Gemfile', 'w') do |f|
        f <<
          '# Gemfile
source "https://rubygems.org"' + "\n\n"
        gems.each { |g| f << "gem #{g}" + "\n" }
        f.close
      end
      mkdir 'lib'
      mkdir 'bin'
      sh 'bundle install'
    end
  end
end
{% endhighlight %}

I hope this helps!