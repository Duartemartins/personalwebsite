require 'Date'

desc 'create a new draft post'
task :post do
  # Prompt for user input
  print "Enter the title of the post: "
  title = STDIN.gets.chomp
  print "Enter a short description: "
  description = STDIN.gets.chomp
  print "Enter categories (comma-separated): "
  categories = STDIN.gets.chomp.split(",").map(&:strip)
  print "Enter tags (comma-separated): "
  tags = STDIN.gets.chomp.split(",").map(&:strip)
  print "Enter language (default: english): "
  lang = STDIN.gets.chomp
  lang = lang.empty? ? "english" : lang
  print "Publish now? (yes/no, default: no): "
  published = STDIN.gets.chomp.downcase
  published = published == "yes"

  # Generate file details
  date = Date.today
  time = Time.now.strftime("%Y-%m-%d %H:%M:%S %z")
  slug = title.downcase.gsub(/[^a-z0-9\s]/i, '').gsub(/\s+/, '-')
  file_name = "#{date}-#{slug}.markdown"
  post_dir = "_posts"
  FileUtils.mkdir_p(post_dir)
  file_path = File.join(post_dir, file_name)

  # Generate front matter
  front_matter = <<~FRONTMATTER
    ---
    layout: post
    title: "#{title}"
    description: "#{description}"
    date: #{time}
    published: #{published}
    categories: #{categories.join(", ")}
    tags: #{tags.join(", ")}
    lang: #{lang}
    ---
  FRONTMATTER

  # Write the file
  File.open(file_path, "w") do |file|
    file.puts front_matter
    file.puts
    file.puts "## Context"
    file.puts
    file.puts "Provide context for your post here."
    file.puts
    file.puts "## Previous Arguments"
    file.puts
    file.puts "1. Summarise key arguments for the initial position."
    file.puts
    file.puts "---"
    file.puts
    file.puts "## What made me change my mind"
    file.puts
    file.puts "1. Use numbered or bulleted lists for points supporting the shift in perspective."
    file.puts
    file.puts "---"
    file.puts
    file.puts "*Conclusion:* Conclude with your updated position and any remaining nuances."
  end

  puts "New post created: #{file_path}"
end

desc 'List all draft posts'
task :drafts do
  puts `find ./_posts -type f -exec grep -H 'published: false' {} \\;`
end
