require 'Date'
require 'csv'
require 'fileutils'
require 'reverse_markdown'
require 'net/http'
require 'uri'
require 'digest'

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
  
  # Simplify <picture> elements: extract the <img> tag and promote it
  doc.css('picture').each do |picture|
    img = picture.at_css('img')
    if img
      # Get the src attribute (prefer data-attrs src or regular src)
      src = img['src']
      alt = img['alt'] || ''
      # Create simple img tag
      new_img = Nokogiri::XML::Node.new('img', doc)
      new_img['src'] = src
      new_img['alt'] = alt
      picture.replace(new_img)
    else
      picture.remove
    end
  end
  
  # Simplify figure/captioned-image-container: promote images
  doc.css('.captioned-image-container, figure').each do |container|
    img = container.at_css('img')
    if img
      src = img['src']
      alt = img['alt'] || ''
      new_img = Nokogiri::XML::Node.new('img', doc)
      new_img['src'] = src
      new_img['alt'] = alt
      # Wrap in paragraph for better markdown output
      p_tag = Nokogiri::XML::Node.new('p', doc)
      p_tag.add_child(new_img)
      container.replace(p_tag)
    else
      container.remove
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
