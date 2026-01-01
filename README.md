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

## Substack Import (Legacy)

For bulk importing from a Substack export ZIP file (useful for initial migration):

### Export from Substack

1. Go to your Substack dashboard → Settings → Exports
2. Download your posts export (ZIP file)
3. Extract to `substack_posts/` directory in the project root

### Run the import

```bash
# Dry run (preview)
bundle exec rake "substack:import[true]"

# Full import
bundle exec rake "substack:import[false]"
```

### What it does

- Reads `posts.csv` manifest and HTML files from export
- Converts HTML to Markdown
- Downloads images to `assets/images/substack/`
- Creates Jekyll posts with proper frontmatter
- Strips Substack boilerplate (subscription widgets, sponsor sections)
- Removes emojis from titles

> **Note**: For ongoing sync, use `substack:pull` instead - it fetches directly from the API without needing an export.

## Substack Sync

The blog includes a bidirectional sync system between Jekyll markdown posts and Substack:

- **Push**: Publish Jekyll posts to Substack as drafts
- **Pull**: Import Substack posts into the Jekyll blog

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

3. **Login to Substack** (required due to CAPTCHA):
   ```bash
   bundle exec rake substack:login
   ```
   This opens a browser window where you complete the login (including any CAPTCHA). Your session is saved to `~/.substack_cookies.yml` and reused for future commands.

### Commands

| Command | Description |
|---------|-------------|
| `bundle exec rake substack:login` | Interactive login - opens browser for CAPTCHA |
| `bundle exec rake substack:status` | Show sync status - which posts are synced, new, or modified |
| `bundle exec rake substack:sync` | Sync all new/modified posts to Substack as drafts |
| `bundle exec rake "substack:sync[true]"` | Dry run - preview what would be synced |
| `bundle exec rake substack:pull` | Pull latest posts from Substack API into the blog |
| `bundle exec rake "substack:pull[true]"` | Dry run - preview which posts would be pulled |
| `bundle exec rake substack:list` | List posts available for Substack publishing with sync status |
| `bundle exec rake "substack:publish[_posts/file.md]"` | Publish a single post to Substack |
| `bundle exec rake substack:mark_all_synced` | Mark all existing posts as synced (useful for initial setup) |
| `bundle exec rake "substack:mark_synced[_posts/file.md]"` | Mark a specific post as synced without publishing |
| `bundle exec rake substack:reset_sync` | Reset sync tracking (will re-sync all on next sync) |

### Authentication

Substack uses Cloudflare CAPTCHA protection, so automated login doesn't work. The `substack:login` task:

1. Opens a Chrome browser window
2. Pre-fills your email and password from `.env`
3. You solve the CAPTCHA manually
4. Cookies are saved to `~/.substack_cookies.yml`

Sessions typically last several weeks. If you get authentication errors, run `substack:login` again.

### Workflow: Jekyll → Substack

```bash
# 1. Create a new post
bundle exec rake post

# 2. Write your content in _posts/

# 3. Check what will be synced
bundle exec rake substack:status

# 4. Push to Substack as draft
bundle exec rake substack:sync

# 5. Review and publish at https://interessant3.substack.com/publish/posts
```

### Workflow: Substack → Jekyll

```bash
# Pull new Substack posts into _posts/
bundle exec rake substack:pull
```

### Features

- **Change Detection**: Tracks content hashes in `.substack_synced.yml` to detect new and modified posts
- **Smart Filtering**: Automatically excludes newsletter imports, drafts, posts published before 2026, and non-English posts
- **Frontmatter Control**: Use `substack_sync: false` to skip a post or `substack_sync: true` to force sync
- **Rate Limiting**: Includes delays between posts to be nice to Substack's API
- **Image Handling**: Downloads images from Substack CDN when pulling, uploads local images when pushing

### Troubleshooting

| Problem | Solution |
|---------|----------|
| "No saved session found" | Run `bundle exec rake substack:login` |
| "Session invalid or expired" | Run `bundle exec rake substack:login` |
| CAPTCHA blocking login | Use `substack:login` task (manual CAPTCHA solving) |
| Post not syncing | Check frontmatter for `substack_sync: false` or `published: false` |

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
