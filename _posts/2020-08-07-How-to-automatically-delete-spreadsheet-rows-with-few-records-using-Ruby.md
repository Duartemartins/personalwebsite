---
layout: post
title:  "How to automatically delete spreadsheet rows with few records using Ruby"
date:   2020-08-07 18:20:52 +0100
categories: tech, ruby
---
![A wind turbine assembly line, making extensive use of automation. Photo by Science in HD on Unsplash](https://miro.medium.com/max/1400/0*Z0d5VOYF5jMx_iEe)

Ruby is a great scripting language, it’s easy to read and write, and it can automate pretty much anything. It’s a prime candidate for data analysis and general business process automation.

When working with CRM exports to carry out audience analyses, one simple time saver would be if we could instantly delete irrelevant columns that haven’t got enough data in them to be useful. To that end, I wrote this little ruby script that deletes columns with fewer than 10 records, it uses nothing other than ruby’s standard CSV library.
{% highlight ruby %}
require ‘csv’
table = CSV.table(‘./spreadsheet.csv’, headers:true).by_col! 
{% endhighlight %}
This opens the CSV as a table and sets it to be handled by columns
{% highlight ruby %}
table.delete_if do |col|
 col[1].compact.size <10
end
{% endhighlight %}
Here we delete columns if they have fewer than 10 records, using .compact to discount empty cells
{% highlight ruby %}
File.open(‘cleanspreadsheet.csv’, ‘w’) do |f|
 f.write(table.to_csv)
end
{% endhighlight %}
Finally, we save the file in a new CSV.

I hope that by sharing this I can save others in the same situation some time. 👍