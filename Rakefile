require 'date'
require 'csv'
require 'fileutils'
require 'reverse_markdown'
require 'net/http'
require 'uri'
require 'digest'
require 'dotenv/load'
require 'yaml'
require 'json'

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

SUBSTACK_SYNC_FILE = '.substack_synced.yml'
SUBSTACK_PULLED_FILE = '.substack_pulled.yml'

# Helper: Load sync tracking data
def load_sync_data
  File.exist?(SUBSTACK_SYNC_FILE) ? YAML.load_file(SUBSTACK_SYNC_FILE) || {} : {}
end

# Helper: Save sync tracking data
def save_sync_data(data)
  File.write(SUBSTACK_SYNC_FILE, data.to_yaml)
end

# Helper: Load pulled posts tracking data
def load_pulled_data
  File.exist?(SUBSTACK_PULLED_FILE) ? YAML.load_file(SUBSTACK_PULLED_FILE) || {} : {}
end

# Helper: Save pulled posts tracking data
def save_pulled_data(data)
  File.write(SUBSTACK_PULLED_FILE, data.to_yaml)
end

# Helper: Calculate content hash for change detection
def post_content_hash(file_path)
  content = File.read(file_path)
  Digest::MD5.hexdigest(content)
end

# Helper: Check if post should be synced to Substack
def should_sync_to_substack?(file_path, frontmatter)
  # Skip newsletter posts (they came FROM Substack)
  categories = frontmatter['categories'].to_s.downcase
  return false if categories.include?('newsletter')
  
  # Skip unpublished drafts
  return false if frontmatter['published'] == false
  
  # Skip posts with substack_sync: false
  return false if frontmatter['substack_sync'] == false

  # Skip posts published before 2026
  post_date = frontmatter['date']
  if post_date
    date_obj = post_date.is_a?(Time) || post_date.is_a?(Date) ? post_date : Date.parse(post_date.to_s)
    return false if date_obj.year < 2026
  end
  
  # Skip Portuguese posts unless explicitly enabled
  lang = frontmatter['lang'].to_s.downcase
  return false if lang == 'pt' && frontmatter['substack_sync'] != true
  
  true
end

# Helper: Initialize Substack client
def get_substack_client
  require 'substack'
  
  begin
    Substack::Client.new
  rescue => e
    email = ENV['SUBSTACK_EMAIL']
    password = ENV['SUBSTACK_PASSWORD']
    
    unless email && password
      puts "❌ No saved session and missing credentials in .env"
      puts "Add SUBSTACK_EMAIL and SUBSTACK_PASSWORD to your .env file"
      exit 1
    end
    
    puts "🔐 Authenticating with Substack..."
    Substack::Client.new(email: email, password: password)
  end
end

namespace :substack do
  desc 'Import posts from Substack export'
  task :import, [:dry_run] do |t, args|
    dry_run = args[:dry_run] == 'true'
    
    csv_path = 'substack_posts/posts.csv'
    html_dir = 'substack_posts'
    posts_dir = '_posts'
    images_dir = 'assets/images/substack'
    
    FileUtils.mkdir_p(posts_dir) unless dry_run
    FileUtils.mkdir_p(images_dir) unless dry_run
    
    # Track stats
    imported = 0
    skipped = 0
    errors = []
    
    # Read CSV
    csv_data = CSV.read(csv_path, headers: true)
    
    csv_data.each do |row|
      post_id = row['post_id']&.split('.')&.first
      title = row['title']
      subtitle = row['subtitle']
      post_date = row['post_date']
      is_published = row['is_published'] == 'true'
      audience = row['audience']
      
      # Skip unpublished or paid-only posts
      unless is_published
        puts "⏭️  Skipping unpublished: #{title}"
        skipped += 1
        next
      end
      
      if audience == 'only_paid'
        puts "⏭️  Skipping paid-only: #{title}"
        skipped += 1
        next
      end
      
      # Skip posts without a date
      if post_date.nil? || post_date.empty?
        puts "⏭️  Skipping (no date): #{title}"
        skipped += 1
        next
      end
      
      # Find matching HTML file
      html_files = Dir.glob("#{html_dir}/#{post_id}.*.html")
      if html_files.empty?
        puts "⚠️  No HTML found for: #{title} (ID: #{post_id})"
        errors << "Missing HTML: #{post_id} - #{title}"
        next
      end
      
      html_file = html_files.first
      
      # Parse date
      begin
        date = DateTime.parse(post_date)
        date_str = date.strftime('%Y-%m-%d')
        time_str = date.strftime('%Y-%m-%d %H:%M:%S %z')
      rescue => e
        puts "⚠️  Date parse error for #{title}: #{e.message}"
        errors << "Date error: #{post_id} - #{title}"
        next
      end
      
      # Generate slug from title
      slug = title.downcase
        .gsub(/[^\w\s-]/, '') # Remove special chars except spaces and hyphens
        .gsub(/\s+/, '-')      # Replace spaces with hyphens
        .gsub(/-+/, '-')       # Collapse multiple hyphens
        .gsub(/^-|-$/, '')     # Trim leading/trailing hyphens
        .slice(0, 50)          # Limit length
      
      file_name = "#{date_str}-#{slug}.markdown"
      file_path = File.join(posts_dir, file_name)
      
      # Check for duplicates
      if File.exist?(file_path)
        puts "⏭️  Already exists: #{file_name}"
        skipped += 1
        next
      end
      
      # Read and process HTML
      html_content = File.read(html_file)
      
      # Strip Substack boilerplate before conversion
      html_content = strip_substack_boilerplate(html_content)
      
      # Convert HTML to Markdown
      markdown_content = ReverseMarkdown.convert(html_content, unknown_tags: :bypass, github_flavored: true)
      
      # Clean up markdown
      markdown_content = clean_markdown(markdown_content)
      
      # Download images and rewrite URLs
      unless dry_run
        markdown_content = download_and_rewrite_images(markdown_content, images_dir, post_id)
      end
      
      # Build frontmatter
      # Escape quotes in title and subtitle, and strip emojis
      safe_title = strip_emojis(title).gsub('"', '\"')
      safe_subtitle = strip_emojis(subtitle || '').gsub('"', '\"')
      
      front_matter = <<~FRONTMATTER
        ---
        layout: post
        title: "#{safe_title}"
        description: "#{safe_subtitle}"
        date: #{time_str}
        published: true
        categories: newsletter
        lang: en
        ---
      FRONTMATTER
      
      full_content = front_matter + "\n" + markdown_content
      
      if dry_run
        puts "🔍 Would create: #{file_path}"
        puts "   Title: #{title}"
        puts "   Date: #{date_str}"
      else
        File.write(file_path, full_content)
        puts "✅ Created: #{file_path}"
      end
      
      imported += 1
    end
    
    puts "\n" + "=" * 50
    puts "Import complete!"
    puts "  Imported: #{imported}"
    puts "  Skipped: #{skipped}"
    puts "  Errors: #{errors.length}"
    
    if errors.any?
      puts "\nErrors:"
      errors.each { |e| puts "  - #{e}" }
    end
    
    if dry_run
      puts "\n⚠️  DRY RUN - no files were created. Run without dry_run to import."
    end
  end

  desc 'Pull latest posts from Substack API'
  task :pull, [:dry_run] do |t, args|
    require 'substack'
    
    dry_run = args[:dry_run] == 'true'
    subdomain = 'interessant3'
    posts_dir = '_posts'
    images_dir = 'assets/images/substack'
    
    FileUtils.mkdir_p(posts_dir) unless dry_run
    FileUtils.mkdir_p(images_dir) unless dry_run
    
    puts "📡 Fetching latest posts from #{subdomain}.substack.com..."
    
    # Use Substack gem's endpoint
    api_url = Substack::Endpoints::POSTS_FEED.call(subdomain) + "?limit=50"
    uri = URI(api_url)
    response = Net::HTTP.get(uri)
    posts = JSON.parse(response)
    
    imported = 0
    skipped = 0
    
    # Load tracking data
    pulled_data = load_pulled_data
    
    # Get existing titles, filenames, and newsletter numbers to avoid duplicates
    existing_titles = {}
    existing_files = {}
    existing_newsletter_numbers = {}
    Dir.glob("#{posts_dir}/*.{md,markdown}").each do |f|
      begin
        content = File.read(f)
        fm = parse_frontmatter(content)
        title = fm['title']&.downcase&.strip
        existing_titles[title] = f if title
        
        basename = File.basename(f)
        existing_files[basename] = true
        
        # Extract newsletter number from filename (e.g., interessant3-133)
        if basename =~ /interessant3-(\d+)/
          existing_newsletter_numbers[$1] = f
        end
      rescue
        # Skip files that can't be parsed
      end
    end
    
    posts.each do |post_data|
      title = post_data['title']
      subtitle = post_data['subtitle']
      post_date = post_data['post_date']
      is_published = post_data['is_published']
      audience = post_data['audience']
      post_id = post_data['id'].to_s
      
      # Generate slug from title (more reliable than Substack's slugs which can be duplicated)
      slug = title.downcase
        .gsub(/[^\w\s-]/, '')  # Remove special chars except spaces and hyphens
        .gsub(/\s+/, '-')       # Replace spaces with hyphens
        .gsub(/-+/, '-')        # Collapse multiple hyphens
        .gsub(/^-|-$/, '')      # Trim leading/trailing hyphens
        .slice(0, 50)           # Limit length
      
      # Skip if already pulled (tracked by Substack post ID)
      if pulled_data[post_id]
        skipped += 1
        next
      end
      
      # Skip unpublished or paid-only
      unless is_published
        skipped += 1
        next
      end
      
      if audience == 'only_paid'
        skipped += 1
        next
      end
      
      # Parse date
      begin
        date = DateTime.parse(post_date)
        date_str = date.strftime('%Y-%m-%d')
        time_str = date.strftime('%Y-%m-%d %H:%M:%S %z')
      rescue => e
        puts "⚠️  Date parse error for #{title}: #{e.message}"
        next
      end
      
      file_name = "#{date_str}-#{slug}.markdown"
      file_path = File.join(posts_dir, file_name)
      
      # Extract newsletter number from title (e.g., "Interessant3 #133 | ..." -> "133")
      newsletter_number = title =~ /interessant3\s*#?(\d+)/i ? $1 : nil
      
      # Check for duplicates by filename, title, or newsletter number
      if existing_files[file_name]
        puts "⏭️  Already exists (filename): #{file_name}"
        pulled_data[post_id] = { 'title' => title, 'file' => file_name, 'pulled_at' => Time.now.iso8601 }
        save_pulled_data(pulled_data) unless dry_run
        skipped += 1
        next
      end
      
      if existing_titles[title.downcase.strip]
        puts "⏭️  Already exists (title match): #{title}"
        pulled_data[post_id] = { 'title' => title, 'file' => existing_titles[title.downcase.strip], 'pulled_at' => Time.now.iso8601 }
        save_pulled_data(pulled_data) unless dry_run
        skipped += 1
        next
      end
      
      if newsletter_number && existing_newsletter_numbers[newsletter_number]
        puts "⏭️  Already exists (newsletter ##{newsletter_number}): #{File.basename(existing_newsletter_numbers[newsletter_number])}"
        pulled_data[post_id] = { 'title' => title, 'file' => File.basename(existing_newsletter_numbers[newsletter_number]), 'pulled_at' => Time.now.iso8601 }
        save_pulled_data(pulled_data) unless dry_run
        skipped += 1
        next
      end
      
      puts "📥 Pulling: #{title}"
      
      html_content = post_data['body_html']
      if html_content.nil? || html_content.empty?
        puts "  ⚠️  No content found for: #{title}"
        next
      end
      
      # Process content
      html_content = strip_substack_boilerplate(html_content)
      markdown_content = ReverseMarkdown.convert(html_content, unknown_tags: :bypass, github_flavored: true)
      markdown_content = clean_markdown(markdown_content)
      
      unless dry_run
        markdown_content = download_and_rewrite_images(markdown_content, images_dir, post_id)
      end
      
      # Build frontmatter
      safe_title = strip_emojis(title).gsub('"', '\"')
      safe_subtitle = strip_emojis(subtitle || '').gsub('"', '\"')
      
      front_matter = <<~FRONTMATTER
        ---
        layout: post
        title: "#{safe_title}"
        description: "#{safe_subtitle}"
        date: #{time_str}
        published: true
        categories: newsletter
        lang: en
        substack_id: #{post_id}
        ---
      FRONTMATTER
      
      full_content = front_matter + "\n" + markdown_content
      
      if dry_run
        puts "  🔍 Would create: #{file_path}"
      else
        File.write(file_path, full_content)
        puts "  ✅ Created"
        
        # Track as pulled
        pulled_data[post_id] = { 'title' => title, 'file' => file_name, 'pulled_at' => Time.now.iso8601 }
        save_pulled_data(pulled_data)
      end
      
      imported += 1
    end
    
    puts "\nDone!"
    puts "  Imported: #{imported}"
    puts "  Skipped: #{skipped}"
    
    if dry_run
      puts "\n⚠️  DRY RUN - no files were created. Run without dry_run to pull."
    end
  end
  
  desc 'List Jekyll posts that can be published to Substack (excludes newsletter category)'
  task :list do
    posts_dir = '_posts'
    sync_data = load_sync_data
    
    posts = Dir.glob("#{posts_dir}/*.{md,markdown}").select do |file|
      content = File.read(file)
      frontmatter = parse_frontmatter(content)
      should_sync_to_substack?(file, frontmatter)
    end.sort.reverse
    
    if posts.empty?
      puts "No publishable posts found."
    else
      puts "📋 Posts available for Substack publishing:"
      puts "-" * 70
      posts.first(20).each do |file|
        content = File.read(file)
        frontmatter = parse_frontmatter(content)
        title = frontmatter['title'] || File.basename(file)
        published = frontmatter['published']
        
        # Check sync status
        basename = File.basename(file)
        current_hash = post_content_hash(file)
        synced_hash = sync_data[basename]
        
        if synced_hash.nil?
          status = " [NEW]"
        elsif synced_hash != current_hash
          status = " [MODIFIED]"
        else
          status = " [SYNCED ✓]"
        end
        
        status += " [DRAFT]" if published == false
        
        puts "#{basename}#{status}"
        puts "  Title: #{title}"
        puts ""
      end
      puts "-" * 70
      puts "Showing 20 most recent of #{posts.length} total posts"
      puts "\nCommands:"
      puts "  rake \"substack:publish[_posts/filename.markdown]\"  - Publish single post"
      puts "  rake substack:sync                                  - Sync all new/modified posts"
      puts "  rake substack:status                                - Show sync status"
    end
  end

  desc 'Show sync status - which posts are synced, new, or modified'
  task :status do
    posts_dir = '_posts'
    sync_data = load_sync_data
    
    posts = Dir.glob("#{posts_dir}/*.{md,markdown}").select do |file|
      content = File.read(file)
      frontmatter = parse_frontmatter(content)
      should_sync_to_substack?(file, frontmatter)
    end.sort.reverse
    
    synced = []
    new_posts = []
    modified = []
    
    posts.each do |file|
      basename = File.basename(file)
      current_hash = post_content_hash(file)
      synced_hash = sync_data[basename]
      
      if synced_hash.nil?
        new_posts << file
      elsif synced_hash != current_hash
        modified << file
      else
        synced << file
      end
    end
    
    puts "📊 Substack Sync Status"
    puts "=" * 50
    puts "  ✅ Synced:    #{synced.length}"
    puts "  🆕 New:       #{new_posts.length}"
    puts "  ✏️  Modified:  #{modified.length}"
    puts "  📝 Total:     #{posts.length}"
    puts ""
    
    if new_posts.any?
      puts "🆕 New posts to sync:"
      new_posts.first(10).each do |file|
        content = File.read(file)
        frontmatter = parse_frontmatter(content)
        puts "  - #{frontmatter['title'] || File.basename(file)}"
      end
      puts "  ... and #{new_posts.length - 10} more" if new_posts.length > 10
      puts ""
    end
    
    if modified.any?
      puts "✏️  Modified posts (will be re-synced):"
      modified.first(10).each do |file|
        content = File.read(file)
        frontmatter = parse_frontmatter(content)
        puts "  - #{frontmatter['title'] || File.basename(file)}"
      end
      puts "  ... and #{modified.length - 10} more" if modified.length > 10
      puts ""
    end
    
    if new_posts.any? || modified.any?
      puts "Run 'rake substack:sync' to sync #{new_posts.length + modified.length} posts"
    else
      puts "✨ Everything is in sync!"
    end
  end

  desc 'Sync all new and modified posts to Substack as drafts'
  task :sync, [:dry_run] do |t, args|
    dry_run = args[:dry_run] == 'true'
    posts_dir = '_posts'
    sync_data = load_sync_data
    
    posts = Dir.glob("#{posts_dir}/*.{md,markdown}").select do |file|
      content = File.read(file)
      frontmatter = parse_frontmatter(content)
      should_sync_to_substack?(file, frontmatter)
    end.sort
    
    to_sync = posts.select do |file|
      basename = File.basename(file)
      current_hash = post_content_hash(file)
      synced_hash = sync_data[basename]
      synced_hash.nil? || synced_hash != current_hash
    end
    
    if to_sync.empty?
      puts "✨ All posts are already synced to Substack!"
      exit 0
    end
    
    puts "🔄 Syncing #{to_sync.length} posts to Substack..."
    puts ""
    
    client = nil
    unless dry_run
      client = get_substack_client
    end
    
    user_id = client&.get_user_id
    
    synced_count = 0
    errors = []
    
    to_sync.each_with_index do |file, index|
      content = File.read(file)
      frontmatter = parse_frontmatter(content)
      body = parse_body(content)
      
      title = frontmatter['title'] || 'Untitled'
      subtitle = frontmatter['description'] || ''
      basename = File.basename(file)
      
      puts "[#{index + 1}/#{to_sync.length}] #{title}"
      
      if dry_run
        puts "  📋 Would sync: #{basename}"
      else
        begin
          post = Substack::Post.new(title: title, subtitle: subtitle, user_id: user_id)
          markdown_to_substack(body, post)
          draft = post.get_draft
          client.post_draft(draft)
          
          # Update sync tracking
          sync_data[basename] = post_content_hash(file)
          save_sync_data(sync_data)
          
          puts "  ✅ Synced"
          synced_count += 1
          
          # Rate limiting - be nice to Substack
          sleep(2) if index < to_sync.length - 1
        rescue => e
          puts "  ❌ Error: #{e.message}"
          errors << { file: basename, error: e.message }
        end
      end
    end
    
    puts ""
    puts "=" * 50
    puts "Sync complete!"
    puts "  ✅ Synced:  #{synced_count}"
    puts "  ❌ Errors:  #{errors.length}"
    
    if errors.any?
      puts "\nErrors:"
      errors.each { |e| puts "  - #{e[:file]}: #{e[:error]}" }
    end
    
    if dry_run
      puts "\n⚠️  DRY RUN - no posts were synced. Run without dry_run to sync."
    else
      puts "\n📝 Review drafts at: https://interessant3.substack.com/publish/posts"
    end
  end

  desc 'Mark a post as synced without actually publishing (for already published posts)'
  task :mark_synced, [:post_path] do |t, args|
    post_path = args[:post_path]
    
    unless post_path && File.exist?(post_path)
      puts "❌ Error: Please provide a valid post path"
      puts "Usage: rake \"substack:mark_synced[_posts/2025-01-01-my-post.markdown]\""
      exit 1
    end
    
    sync_data = load_sync_data
    basename = File.basename(post_path)
    sync_data[basename] = post_content_hash(post_path)
    save_sync_data(sync_data)
    
    puts "✅ Marked as synced: #{basename}"
  end

  desc 'Mark all existing posts as synced (useful for initial setup)'
  task :mark_all_synced do
    posts_dir = '_posts'
    sync_data = load_sync_data
    
    posts = Dir.glob("#{posts_dir}/*.{md,markdown}").select do |file|
      content = File.read(file)
      frontmatter = parse_frontmatter(content)
      should_sync_to_substack?(file, frontmatter)
    end
    
    posts.each do |file|
      basename = File.basename(file)
      sync_data[basename] = post_content_hash(file)
    end
    
    save_sync_data(sync_data)
    puts "✅ Marked #{posts.length} posts as synced"
  end

  desc 'Reset sync tracking (will re-sync all posts on next sync)'
  task :reset_sync do
    if File.exist?(SUBSTACK_SYNC_FILE)
      File.delete(SUBSTACK_SYNC_FILE)
      puts "✅ Sync tracking reset. Run 'rake substack:sync' to sync all posts."
    else
      puts "ℹ️  No sync tracking file found."
    end
  end

  desc 'Publish a Jekyll post to Substack as a draft'
  task :publish, [:post_path] do |t, args|
    require 'substack'
    
    post_path = args[:post_path]
    
    unless post_path && File.exist?(post_path)
      puts "❌ Error: Please provide a valid post path"
      puts "Usage: rake \"substack:publish[_posts/2025-01-01-my-post.markdown]\""
      puts "Run 'rake substack:list' to see available posts."
      exit 1
    end
    
    content = File.read(post_path)
    frontmatter = parse_frontmatter(content)
    body = parse_body(content)
    
    # Skip newsletter posts (they came FROM Substack)
    categories = frontmatter['categories'].to_s.downcase
    if categories.include?('newsletter')
      puts "❌ This post has category 'newsletter' and originated from Substack. Skipping."
      exit 1
    end
    
    title = frontmatter['title'] || 'Untitled'
    subtitle = frontmatter['description'] || ''
    
    puts "📝 Publishing to Substack: #{title}"
    
    # Initialize client - uses saved cookies, or authenticates with .env credentials
    client = nil
    begin
      client = Substack::Client.new
    rescue => e
      # Try to authenticate with .env credentials
      email = ENV['SUBSTACK_EMAIL']
      password = ENV['SUBSTACK_PASSWORD']
      
      unless email && password
        puts "❌ No saved session and missing credentials in .env"
        puts "Add SUBSTACK_EMAIL and SUBSTACK_PASSWORD to your .env file"
        puts "See .env.example for reference"
        exit 1
      end
      
      puts "🔐 Authenticating with Substack..."
      client = Substack::Client.new(email: email, password: password)
    end
    
    user_id = client.get_user_id
    post = Substack::Post.new(title: title, subtitle: subtitle, user_id: user_id)
    
    # Parse markdown and build Substack post
    markdown_to_substack(body, post)
    
    draft = post.get_draft
    client.post_draft(draft)
    
    # Update sync tracking
    sync_data = load_sync_data
    basename = File.basename(post_path)
    sync_data[basename] = post_content_hash(post_path)
    save_sync_data(sync_data)
    
    puts "✅ Draft created on Substack!"
    puts "   Review and publish at: https://interessant3.substack.com/publish/posts"
  end
end

# Helper: Parse YAML frontmatter from Jekyll post
def parse_frontmatter(content)
  if content =~ /\A---\s*\n(.*?\n?)^---\s*$/m
    YAML.safe_load($1, permitted_classes: [Date, Time]) || {}
  else
    {}
  end
end

# Helper: Extract body content (everything after frontmatter)
def parse_body(content)
  content =~ /\A---\s*\n.*?\n?^---\s*\n(.*)$/m ? $1.strip : content.strip
end

# Helper: Convert local image paths to full URLs
def localize_image_url(url)
  base_url = 'https://www.santiago-martins.com'
  url.start_with?('/') ? "#{base_url}#{url}" : url
end

# Helper: Convert markdown to Substack post format
def markdown_to_substack(markdown, post)
  lines = markdown.lines
  i = 0
  
  while i < lines.length
    line = lines[i].rstrip
    
    # Skip empty lines
    if line.strip.empty?
      i += 1
      next
    end
    
    # Horizontal rule
    if line =~ /^(-{3,}|\*{3,}|_{3,})$/
      post.horizontal_rule
      i += 1
      next
    end
    
    # Headings
    if line =~ /^(\#{1,6})\s+(.+)$/
      level = $1.length
      heading_text = $2.strip
      post.heading(heading_text, level: level)
      i += 1
      next
    end
    
    # Standalone images
    if line =~ /^!\[([^\]]*)\]\(([^)\s]+)/
      alt = $1
      url = $2
      post.captioned_image(attrs: { src: localize_image_url(url), alt: alt })
      i += 1
      next
    end
    
    # Collect paragraph text (may span multiple lines until blank line)
    para_lines = []
    while i < lines.length && !lines[i].strip.empty? && 
          lines[i] !~ /^\#{1,6}\s/ && 
          lines[i] !~ /^(-{3,}|\*{3,}|_{3,})$/ &&
          lines[i] !~ /^!\[/
      para_lines << lines[i].rstrip
      i += 1
    end
    
    if para_lines.any?
      para_text = para_lines.join(' ').strip
      # Handle inline images within paragraphs
      if para_text =~ /!\[([^\]]*)\]\(([^)\s]+)/
        para_text.scan(/!\[([^\]]*)\]\(([^)\s]+)/) do |alt, url|
          post.captioned_image(attrs: { src: localize_image_url(url), alt: alt })
        end
        remaining = para_text.gsub(/!\[[^\]]*\]\([^)]+\)/, '').strip
        post.paragraph(remaining) unless remaining.empty?
      else
        post.paragraph(para_text)
      end
    end
  end
end

# Helper: Extract clean image URL from Substack img element
def extract_substack_image_url(img)
  # First try to get the clean URL from data-attrs JSON
  if img['data-attrs']
    begin
      data_attrs = JSON.parse(img['data-attrs'])
      if data_attrs['src'] && !data_attrs['src'].empty?
        return data_attrs['src']
      end
    rescue JSON::ParserError
      # Fall through to other methods
    end
  end
  
  # Fall back to src attribute, but clean it up
  src = img['src'] || ''
  
  # Extract the S3 URL from Substack CDN proxy URL
  if src =~ /substack-post-media\.s3\.amazonaws\.com[^\s")]+/
    match = src.match(/(https?:\/\/substack-post-media\.s3\.amazonaws\.com\/public\/images\/[a-f0-9-]+_\d+x\d+\.\w+)/)
    return match[1] if match
    
    # Try URL-encoded version
    if src =~ /%2F/
      decoded = URI.decode_www_form_component(src)
      match = decoded.match(/(https?:\/\/substack-post-media\.s3\.amazonaws\.com\/public\/images\/[a-f0-9-]+_\d+x\d+\.\w+)/)
      return match[1] if match
    end
  end
  
  # Return original src if we can't extract a cleaner URL
  src
end

# Helper: Strip Substack boilerplate and normalize HTML structure
def strip_substack_boilerplate(html)
  require 'nokogiri'
  
  doc = Nokogiri::HTML::DocumentFragment.parse(html)
  
  # Remove subscription widgets
  doc.css('.subscription-widget-wrap-editor, .subscription-widget').each(&:remove)
  
  # Remove sponsor sections (look for "Sponsored by" headers)
  doc.css('h4, h3, h2').each do |header|
    if header.text =~ /sponsored by/i
      # Remove the header and following siblings until next header or hr
      current = header
      while current
        next_el = current.next_sibling
        current.remove
        break if next_el.nil? || (next_el.name =~ /^h[1-6]$/) || next_el.name == 'hr'
        current = next_el
      end
    end
  end
  
  # Remove "Thanks for reading" sections
  doc.css('p').each do |p|
    if p.text =~ /thanks for reading|join us next week/i
      p.remove
    end
  end
  
  # Remove trailing hrs
  while doc.children.last&.name == 'hr' || (doc.children.last&.text&.strip || '').empty?
    break if doc.children.empty?
    doc.children.last.remove
  end
  
  # NORMALIZE HTML STRUCTURE FOR REVERSE_MARKDOWN
  
  # Remove SVG elements (buttons, icons) 
  doc.css('svg, button').each(&:remove)
  
  # Remove mention-wrap spans but keep the text
  doc.css('span.mention-wrap').each do |span|
    span.replace(span.inner_text)
  end
  
  # Process images FIRST before other transformations
  # Handle captioned-image-container and figure elements that contain images
  doc.css('.captioned-image-container, figure').each do |container|
    # Find the img, which may be nested in picture, div, a, etc.
    img = container.at_css('img')
    if img
      src = extract_substack_image_url(img)
      alt = img['alt'] || ''
      
      # Create a simple paragraph with just the image
      new_p = Nokogiri::XML::Node.new('p', doc)
      new_img = Nokogiri::XML::Node.new('img', doc)
      new_img['src'] = src
      new_img['alt'] = alt.empty? ? 'Image' : alt
      new_p.add_child(new_img)
      
      container.replace(new_p)
    else
      container.remove
    end
  end
  
  # Handle any remaining standalone picture elements
  doc.css('picture').each do |picture|
    img = picture.at_css('img')
    if img
      src = extract_substack_image_url(img)
      alt = img['alt'] || ''
      new_img = Nokogiri::XML::Node.new('img', doc)
      new_img['src'] = src
      new_img['alt'] = alt.empty? ? 'Image' : alt
      picture.replace(new_img)
    else
      picture.remove
    end
  end
  
  # Handle YouTube embeds - convert to links
  doc.css('.youtube-wrap, [data-component-name="Youtube2ToDOM"]').each do |yt|
    video_id = nil
    if yt['data-attrs']
      begin
        data = JSON.parse(yt['data-attrs'])
        video_id = data['videoId']
      rescue
      end
    end
    # Also try to find iframe src
    iframe = yt.at_css('iframe')
    if iframe && iframe['src'] =~ /youtube.*embed\/([^?&]+)/
      video_id ||= $1
    end
    
    if video_id
      link = Nokogiri::XML::Node.new('p', doc)
      a = Nokogiri::XML::Node.new('a', doc)
      a['href'] = "https://www.youtube.com/watch?v=#{video_id}"
      a.content = "Watch on YouTube"
      link.add_child(a)
      yt.replace(link)
    else
      yt.remove
    end
  end
  
  # Remove image expansion buttons/wrappers
  doc.css('.image-link-expand, .image2-inset').each do |el|
    img = el.at_css('img')
    if img
      el.replace(img)
    end
  end
  
  # Unwrap unnecessary div wrappers that might confuse the converter
  doc.css('div.pencraft, div.image2-inset').each do |div|
    div.replace(div.children)
  end
  
  # Clean up empty divs
  doc.css('div').each do |div|
    if div.text.strip.empty? && div.at_css('img, a, p, ul, ol, h1, h2, h3, h4, h5, h6').nil?
      div.remove
    end
  end
  
  # CRITICAL FIX: Unwrap <p> tags inside <li> elements
  # reverse_markdown doesn't handle nested lists when <p> tags are present inside <li>
  doc.css('li > p').each do |p|
    # Replace the p tag with its children, preserving content
    p.replace(p.children)
  end
  
  doc.to_html
end

# Helper: Clean up converted markdown
def clean_markdown(md)
  # Remove empty links
  md = md.gsub(/\[\s*\]\([^)]*\)/, '')
  
  # Remove broken/empty image tags (![](url), ![], or standalone !)
  md = md.gsub(/!\[\]\([^)]*\)/, '')  # ![](url)
  md = md.gsub(/!\[\]\s*$/, '')       # ![] at end of line
  md = md.gsub(/^\s*!\s*$/m, '')      # Standalone ! on a line
  
  # Remove excessive blank lines
  md = md.gsub(/\n{4,}/, "\n\n\n")
  
  # Remove Substack-specific artifacts
  md = md.gsub(/\[Subscribe\]\([^)]*\)/i, '')
  md = md.gsub(/\[Share\]\([^)]*\)/i, '')
  
  # Clean up image markdown (remove srcset artifacts)
  md = md.gsub(/\s*\d+w,?\s*/, ' ')
  
  # Remove all emojis
  md = md.gsub(/[\u{1F600}-\u{1F64F}]/, '')  # Emoticons
  md = md.gsub(/[\u{1F300}-\u{1F5FF}]/, '')  # Misc Symbols and Pictographs
  md = md.gsub(/[\u{1F680}-\u{1F6FF}]/, '')  # Transport and Map
  md = md.gsub(/[\u{1F700}-\u{1F77F}]/, '')  # Alchemical Symbols
  md = md.gsub(/[\u{1F780}-\u{1F7FF}]/, '')  # Geometric Shapes Extended
  md = md.gsub(/[\u{1F800}-\u{1F8FF}]/, '')  # Supplemental Arrows-C
  md = md.gsub(/[\u{1F900}-\u{1F9FF}]/, '')  # Supplemental Symbols and Pictographs
  md = md.gsub(/[\u{1FA00}-\u{1FA6F}]/, '')  # Chess Symbols
  md = md.gsub(/[\u{1FA70}-\u{1FAFF}]/, '')  # Symbols and Pictographs Extended-A
  md = md.gsub(/[\u{2600}-\u{26FF}]/, '')    # Misc symbols (sun, moon, stars, etc)
  md = md.gsub(/[\u{2700}-\u{27BF}]/, '')    # Dingbats
  md = md.gsub(/[\u{FE00}-\u{FE0F}]/, '')    # Variation Selectors
  md = md.gsub(/[\u{1F1E0}-\u{1F1FF}]/, '')  # Flags
  
  # Trim whitespace
  md.strip
end

# Helper: Strip emojis from a string
def strip_emojis(str)
  str.gsub(/[\u{1F600}-\u{1F64F}]/, '')  # Emoticons
     .gsub(/[\u{1F300}-\u{1F5FF}]/, '')  # Misc Symbols and Pictographs
     .gsub(/[\u{1F680}-\u{1F6FF}]/, '')  # Transport and Map
     .gsub(/[\u{1F700}-\u{1F77F}]/, '')  # Alchemical Symbols
     .gsub(/[\u{1F780}-\u{1F7FF}]/, '')  # Geometric Shapes Extended
     .gsub(/[\u{1F800}-\u{1F8FF}]/, '')  # Supplemental Arrows-C
     .gsub(/[\u{1F900}-\u{1F9FF}]/, '')  # Supplemental Symbols and Pictographs
     .gsub(/[\u{1FA00}-\u{1FA6F}]/, '')  # Chess Symbols
     .gsub(/[\u{1FA70}-\u{1FAFF}]/, '')  # Symbols and Pictographs Extended-A
     .gsub(/[\u{2600}-\u{26FF}]/, '')    # Misc symbols (sun, moon, stars, etc)
     .gsub(/[\u{2700}-\u{27BF}]/, '')    # Dingbats
     .gsub(/[\u{FE00}-\u{FE0F}]/, '')    # Variation Selectors
     .gsub(/[\u{1F1E0}-\u{1F1FF}]/, '')  # Flags
     .gsub(/\s+/, ' ')                    # Collapse multiple spaces
     .strip
end

# Helper: Download images and rewrite URLs
def download_and_rewrite_images(markdown, images_dir, post_id)
  # Find all Substack image URLs
  substack_pattern = /https?:\/\/substackcdn\.com[^\s\)]+|https?:\/\/substack-post-media\.s3\.amazonaws\.com[^\s\)]+/
  
  markdown.gsub(substack_pattern) do |url|
    begin
      # Clean URL (remove sizing params for download)
      clean_url = url.split('?').first
      clean_url = clean_url.gsub(/\$s_![^!]+!,/, '') # Remove Substack sizing
      
      # Generate local filename
      url_hash = Digest::MD5.hexdigest(clean_url)[0..7]
      extension = File.extname(URI.parse(clean_url).path).downcase
      extension = '.png' if extension.empty? || extension.length > 5
      local_filename = "#{post_id}-#{url_hash}#{extension}"
      local_path = File.join(images_dir, local_filename)
      
      # Download if not already exists
      unless File.exist?(local_path)
        puts "   📥 Downloading image: #{local_filename}"
        download_image(url, local_path)
      end
      
      # Return new URL
      "/#{local_path}"
    rescue => e
      puts "   ⚠️  Image download failed: #{e.message}"
      url # Keep original URL on failure
    end
  end
end

# Helper: Download image file
def download_image(url, local_path)
  uri = URI.parse(url)
  
  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = (uri.scheme == 'https')
  http.open_timeout = 10
  http.read_timeout = 30
  
  request = Net::HTTP::Get.new(uri.request_uri)
  request['User-Agent'] = 'Mozilla/5.0 (compatible; Jekyll Import)'
  
  response = http.request(request)
  
  case response
  when Net::HTTPSuccess
    File.open(local_path, 'wb') { |f| f.write(response.body) }
  when Net::HTTPRedirection
    download_image(response['location'], local_path)
  else
    raise "HTTP #{response.code}: #{response.message}"
  end
end
