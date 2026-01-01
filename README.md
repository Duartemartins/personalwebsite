# Duarte's Blog

A personal blog about technology, society, and philosophy, built with Jekyll and hosted on GitHub Pages.

**Live site**: [santiago-martins.com](https://www.santiago-martins.com)

## About

This is where I explore ideas that capture my curiosity: technology, the intricacies of society, and the occasional philosophical musing. The blog takes a data-led approach to topics, with charts and visualizations as first-class citizens.

### Content

- **Tech Insights** - Programming, systems, and tech trends
- **Social Commentary** - Observations on the evolving dynamics of society
- **Philosophical Thoughts** - Reflections on life's big questions
- **Newsletter Archive** - Weekly "Interessant3" posts imported from [Substack](https://interessant3.substack.com)

## Brand Identity: The Editorial Workshop

The visual identity merges the authority and readability of high-end print journalism with the raw, functional aesthetic of a craftsman's workshop. It is intellectual, precise, and unapologetically digital-native while respecting print traditions.

### Visual Language

- **Metaphor**: The "Broadsheet" meets the "Workbench"
- **Key Elements**:
  - Structural borders framing content
  - Persistent sidebar acting as a workspace
  - Data-led storytelling with charts as first-class citizens
- **Atmosphere**: Authoritative, Warm, Intellectual, Urgent

### Typography

| Usage | Font | Style |
|-------|------|-------|
| Headlines | Instrument Serif | Serif, high-contrast |
| Body text | Libre Baskerville | Serif, optimized for reading |
| UI & Data | Inter | Sans-serif, modern/technical |

### Color Palette

| Name | Hex | Usage |
|------|-----|-------|
| Canvas | `#fcfbf9` | Warm newsprint background |
| Sidebar | `#f4f3f0` | Workspace chrome |
| Ink | `#1a1a1a` | Primary text |
| Ink Light | `#555555` | Secondary text |
| Accent | `#c02626` | Editorial red for emphasis |
| Border | `#e0ded9` | Structural lines |

Read the full brand identity guide at [/brand-identity/](https://www.santiago-martins.com/brand-identity/).

## Development

### Prerequisites

- Ruby 3.x
- Bundler

### Setup

```bash
bundle install
```

### Run locally

```bash
bundle exec jekyll serve --livereload
```

Site will be available at `http://localhost:4000`.

### Create a new post

```bash
bundle exec rake post
```

Follow the prompts to enter title, description, categories, tags, and language.

### List draft posts

```bash
bundle exec rake drafts
```

## Substack Import

The blog includes a Rake task to import posts from a Substack newsletter export.

### Prerequisites

Add these gems to your Gemfile:

```ruby
gem 'csv'
gem 'nokogiri'
gem 'reverse_markdown'
```

### Pull from Substack API

If you have new posts on Substack that you want to bring into your Jekyll blog:

```bash
# Preview what will be pulled
bundle exec rake "substack:pull[true]"

# Pull posts
bundle exec rake substack:pull
```

This command:
1. Fetches the latest 50 posts from the Substack API.
2. Skips posts that already exist in `_posts` (checks by filename and title).
3. Converts Substack HTML to Markdown.
4. Downloads images to `assets/images/substack/`.
5. Creates new Jekyll posts with `categories: newsletter`.

### Export from Substack

1. Go to your Substack dashboard → Settings → Exports
2. Download your posts export (ZIP file)
3. Extract to `substack_posts/` directory in the project root
4. Ensure `posts.csv` and all `.html` files are in `substack_posts/`

### Run the import

**Dry run** (preview without creating files):

```bash
bundle exec rake "substack:import[true]"
```

**Full import**:

```bash
bundle exec rake "substack:import[false]"
```

### What the import does

1. **Reads** the `posts.csv` manifest from Substack export
2. **Filters** out unpublished and paid-only posts
3. **Converts** HTML to Markdown using `reverse_markdown`
4. **Preprocesses** HTML to handle Substack's complex nested structure:
   - Unwraps `<p>` tags inside `<li>` elements (fixes nested list conversion)
   - Simplifies `<picture>` and `<figure>` elements
   - Removes SVG buttons and UI elements
5. **Strips** Substack boilerplate:
   - Subscription widgets
   - Sponsor sections
   - "Thanks for reading" footers
6. **Downloads** images from Substack CDN to `assets/images/substack/`
7. **Removes** all emojis from titles and content
8. **Creates** Jekyll posts with proper frontmatter:
   - `categories: newsletter`
   - `lang: en`
   - Original publish date preserved

### Handling duplicates

The import skips files that already exist. To re-import specific posts:

```bash
# Delete the post(s) you want to re-import
rm _posts/2025-01-01-post-title.markdown

# Run import again
bundle exec rake "substack:import[false]"
```

### Output

```
✅ Created: _posts/2025-01-15-interessant3-123.markdown
   📥 Downloading image: 123456789-abc12345.png
⏭️  Already exists: 2025-01-08-interessant3-122.markdown
⏭️  Skipping unpublished: Draft Post Title
⏭️  Skipping paid-only: Premium Content

==================================================
Import complete!
  Imported: 170
  Skipped: 63
  Errors: 0
```

## Substack Sync

The blog includes a system to automatically cross-post your Jekyll markdown posts to Substack as drafts.

### Setup

1. Create a `.env` file in the root directory (see `.env.example`):
   ```dotenv
   SUBSTACK_EMAIL=your_email@example.com
   SUBSTACK_PASSWORD=your_password
   ```
2. Install dependencies:
   ```bash
   bundle install
   ```

### Commands

| Command | Description |
|---------|-------------|
| `bundle exec rake substack:status` | Show sync status - which posts are synced, new, or modified |
| `bundle exec rake substack:pull` | Pull latest posts from Substack API into the blog |
| `bundle exec rake "substack:pull[true]"` | Dry run - preview which posts would be pulled from Substack |
| `bundle exec rake substack:list` | List posts available for Substack publishing with sync status |
| `bundle exec rake substack:sync` | Sync all new/modified posts to Substack as drafts |
| `bundle exec rake "substack:sync[true]"` | Dry run - preview what would be synced |
| `bundle exec rake "substack:publish[_posts/file.md]"` | Publish a single post to Substack |
| `bundle exec rake substack:mark_all_synced` | Mark all existing posts as synced (useful for initial setup) |
| `bundle exec rake "substack:mark_synced[_posts/file.md]"` | Mark a specific post as synced without publishing |
| `bundle exec rake substack:reset_sync` | Reset sync tracking (will re-sync all on next sync) |

### Features

- **Change Detection**: Tracks content hashes in `.substack_synced.yml` to detect new and modified posts.
- **Smart Filtering**: Automatically excludes newsletter imports, drafts, posts published before 2026, and non-English posts (unless explicitly enabled).
- **Frontmatter Control**: Use `substack_sync: false` to skip a post or `substack_sync: true` to force sync.
- **Rate Limiting**: Includes a delay between posts to comply with Substack's API.

## Project Structure

```
├── _config.yml          # Jekyll configuration
├── _data/
│   └── ui_text.yml      # Localized UI strings (EN/PT)
├── _includes/           # Reusable components
│   ├── chart_bar.html   # Bar chart component
│   ├── chart_line.html  # Line chart component
│   └── sidebar.html     # Navigation sidebar
├── _layouts/            # Page templates
│   ├── default.html
│   ├── home.html
│   ├── page.html
│   └── post.html
├── _posts/              # Blog posts (Markdown)
├── _sass/               # SCSS stylesheets
├── assets/
│   ├── css/
│   └── images/
│       └── substack/    # Downloaded newsletter images
├── brand-concepts/      # Brand identity documentation
├── pt/                  # Portuguese translations
├── substack_posts/      # Substack export (not committed)
├── Gemfile
├── Rakefile             # Automation tasks
└── README.md
```

## License

Content © Duarte Martins. All rights reserved.

## Connect

- **Newsletter**: [interessant3.substack.com](https://interessant3.substack.com)
- **GitHub**: [@duartemartins](https://github.com/duartemartins)
- **Twitter/X**: [@duarteosrm](https://twitter.com/duarteosrm)
- **Side Project**: [PopaDex](https://popadex.com) - Personal finance tracker
