# Strapi Configuration Skill - Development Guide

## Overview

This skill scaffolds fully configured Strapi projects from preset templates or custom descriptions. It generates content types, components, route-based middleware for auto-population, seed scripts, and public API permissions.

## Directory Structure

```
skills/strapi-configuration/
├── CLAUDE.md                     # This file - development context
├── SKILL.md                      # Skill definition (agentskills.io spec)
├── README.md                     # User-facing documentation
├── scripts/                      # Helper scripts (future)
├── references/                    # Reference docs (future)
└── templates/                    # Preset template definitions
    ├── blog.json                 # Blog preset
    ├── ecommerce.json            # E-commerce preset
    ├── portfolio.json            # Portfolio preset
    └── restaurant.json           # Restaurant preset
```

## Key Conventions

### Template JSON Format

Each template file MUST follow the Strapi schema format exactly:

**Content Types** require: `kind`, `collectionName`, `info` (singularName, pluralName, displayName, description), `options` (draftAndPublish), `pluginOptions`, `attributes`

**Components** require: `collectionName` (format: `components_<namespace>_<name>`), `info` (displayName, icon, description), `options`, `attributes`

**Middlewares** require: each entry wrapped in `{ "populate": { ... } }`

**Seed Data**: all entries are arrays (even single types use `[{...}]` format)

### CLI Testing Command

```bash
node packages/cli/create-strapi-app/bin/index.js /tmp/test-<name> --non-interactive
```

### Strapi API Patterns

- Controllers: `factories.createCoreController('api::<name>.<name>')`
- Routes: `factories.createCoreRouter('api::<name>.<name>')`
- Services: `factories.createCoreService('api::<name>.<name>')`
- Documents API: `strapi.documents('api::<name>.<name>').create({ data, status: 'published' })`
- Public permissions: `strapi.query('plugin::users-permissions.permission').create({ data: { action: 'api::<name>.<name>.<action>', role: publicRole.id } })`

### Middleware Populate Pattern

```typescript
export default (config, { strapi }) => {
  return async (ctx, next) => {
    if (!ctx.query.populate) {
      ctx.query.populate = { /* populate config */ };
    }
    await next();
  };
};
```

Registered in routes with: `middlewares: ['api::<name>.default-populate']`

## Adding a New Preset

1. Create `templates/<preset-name>.json` following the structure of `blog.json`
2. Include all 5 top-level sections: `contentTypes`, `components`, `middlewares`, `seedData`, `publicPermissions`
3. Add the preset to the Available Presets table in `SKILL.md`
4. Ensure all schema fields follow Strapi format (test by running the skill)

## Common Pitfalls

- **Missing `collectionName`/`info`/`options`** on content types or components - Strapi will fail to register them
- **`draftAndPublish` as top-level key** instead of nested inside `options` - schema validation will fail
- **Middleware without `populate` wrapper** - the populate config must be inside a `populate` key
- **Relation `inversedBy`/`mappedBy` mismatch** - both sides of a relation must reference each other correctly
- **Single type seed data as objects** instead of arrays - keep everything as arrays for consistency

## Future Improvements

- [ ] Add more presets (SaaS landing page, event management, real estate, news/magazine)
- [ ] Support for i18n (internationalization) plugin configuration
- [ ] Generate TypeScript types from content type schemas
- [ ] Auto-run seed script after project creation
- [ ] Support for custom plugins configuration
- [ ] Add image/media seed data support (upload files during seeding)
- [ ] Validation script to check template JSON against Strapi schema rules
- [ ] Interactive mode: step-by-step content type builder
- [ ] Support for Strapi v5 document service relations in seed data (two-pass seeding)
