# Strapi Configuration Skill

A Claude Code skill that scaffolds fully configured Strapi projects with content types, seed data, public API access, and auto-populate middleware — ready to use out of the box.

## Installation

Claude Code discovers skills automatically from `.claude/skills/` directories. This skill should be placed at:

```
<project-root>/.claude/skills/strapi-configuration/
```

**For this project (strapi-core monorepo)**, the skill is already installed at:
```
.claude/skills/strapi-configuration/SKILL.md
```

**For other projects**, copy the entire `strapi-configuration/` directory into `.claude/skills/`:
```bash
cp -r .claude/skills/strapi-configuration /path/to/other-project/.claude/skills/
```

**For global access** (available across all projects), place it in your home directory:
```bash
cp -r .claude/skills/strapi-configuration ~/.claude/skills/
```

After placing the skill, restart Claude Code (or start a new session) for it to be discovered. You can verify by typing `/strapi-configuration` in Claude Code.

## Quick Start

```
/strapi-configuration blog /tmp/my-blog
```

This creates a complete Strapi blog project with articles, authors, categories, tags, and pages — all with seed data and public API endpoints.

## Usage

### Using a Preset

Pick from one of the built-in presets:

```
/strapi-configuration <preset> <output-path>
```

| Preset | What You Get |
|--------|-------------|
| `blog` | Articles, Authors, Categories, Tags, Homepage, About, Global |
| `ecommerce` | Products, Categories, Reviews, Homepage, Global |
| `portfolio` | Projects, Skills, Testimonials, Homepage, About, Contact, Global |
| `restaurant` | Menu Items, Categories, Reservations, Locations, Homepage, About, Global |

### Using a Custom Description

Don't see a preset that fits? Describe your project instead:

```
/strapi-configuration "online learning platform with courses, instructors, and student reviews" /tmp/my-lms
```

```
/strapi-configuration "real estate listing site with properties, agents, and neighborhoods" /tmp/my-realty
```

The skill will design appropriate content types, components, and seed data based on your description. It can also save the result as a reusable template for next time.

## What Gets Generated

For each project, the skill creates:

### Content Types
Collection types and single types with full API structure:
```
src/api/<name>/
├── content-types/<name>/schema.json    # Schema definition
├── controllers/<name>.ts               # Core controller
├── routes/<name>.ts                    # Core router (with middleware config)
├── middlewares/default-populate.ts      # Auto-populate middleware
└── services/<name>.ts                  # Core service
```

### Shared Components
Reusable components like SEO metadata, media blocks, rich text, and social links:
```
src/components/
├── shared/
│   ├── seo.json
│   ├── media.json
│   ├── rich-text.json
│   └── social-link.json
└── <preset>/
    └── <domain-specific>.json
```

### Route-Based Middleware
Each content type gets a `default-populate` middleware that automatically populates relations and media fields on API responses — no `?populate=*` needed from the frontend.

### Seed Script
A `scripts/seed.ts` that:
- Creates sample entries for every content type
- Sets public permissions so the API works immediately
- Only runs once (tracks first-run state)

### Public API Access
All content type endpoints are configured as publicly accessible, so you can start fetching data right away.

## After Generation

```bash
cd <output-path>
npm run develop       # Start Strapi (creates admin on first run)
npm run seed          # Populate with sample data
```

Then try the API:

```bash
# Blog example
curl http://localhost:1337/api/articles
curl http://localhost:1337/api/articles/1

# Restaurant example
curl http://localhost:1337/api/menu-items
curl http://localhost:1337/api/menu-categories

# E-commerce example
curl http://localhost:1337/api/products
curl http://localhost:1337/api/categories
```

Relations and media are auto-populated thanks to the middleware — no query params needed.

## Examples

### Create a blog
```
/strapi-configuration blog /tmp/my-blog
```
Gives you 3 sample articles, 2 authors, 3 categories, 3 tags, and configured pages (homepage, about, global settings).

### Create an e-commerce store
```
/strapi-configuration ecommerce /tmp/my-store
```
Gives you 5 products with features, 3 categories, 4 reviews, and storefront pages with hero banners.

### Create a portfolio site
```
/strapi-configuration portfolio /tmp/my-portfolio
```
Gives you 4 projects with technology tags, 6 skills, 3 testimonials, and pages for homepage, about, contact.

### Create a restaurant website
```
/strapi-configuration restaurant /tmp/my-restaurant
```
Gives you 8 menu items with allergens and dietary flags, 4 menu categories, a location with opening hours, 3 sample reservations, and restaurant pages. The reservation endpoint also allows public `create` so customers can book via the API.

### Create a custom project
```
/strapi-configuration "fitness app with workouts, exercises, and training plans" /tmp/my-fitness
```
The skill designs content types, relations, components, and seed data based on your description.

## Adding Your Own Templates

Create a JSON file in `templates/` following the structure of existing presets. See `SKILL.md` for the full template schema reference.

```
templates/my-template.json
```

Then use it:
```
/strapi-configuration my-template /tmp/my-project
```
