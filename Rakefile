require 'Date'

desc 'create a new draft post'
task :post do
  title = ARGV[1].dup unless ARGV[1].nil?
  slug = "#{Date.today}-#{title.downcase.gsub(/[^\w]+/, '-')}"

  file = File.join(
    File.dirname(__FILE__),
    '_posts',
    slug + '.markdown'
  )

  File.open(file, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    description:
    date: #{Time.new}
    published: false
    categories:
    tags:
    lang: 
    ---

    EOS
  end

  system ("#{ENV['EDITOR']} #{file}")
end

desc 'List all draft posts'
task :drafts do
  puts `find ./_posts -type f -exec grep -H 'published: false' {} \\;`
end
