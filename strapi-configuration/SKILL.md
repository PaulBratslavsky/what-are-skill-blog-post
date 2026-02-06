---
name: strapi-configuration
description: >-
  Create a fully configured Strapi project with preconfigured content types,
  seed data, public API access, and route-based middleware for default
  population. Use when the user wants to scaffold a new Strapi project for
  a specific use case such as blog, e-commerce, portfolio, or restaurant.
  Supports preset templates with collection types, single types, components,
  seed data, public permissions, and auto-populate middleware. Also supports
  custom project descriptions where the user describes their website and the
  skill generates appropriate content types, components, and seed data.
compatibility: Requires Node.js and access to the strapi-core monorepo.
allowed-tools: Bash Read Write Edit Glob Grep
metadata:
  author: strapi
  version: "0.1.0"
---

# Strapi Configuration Skill

Create a fully configured Strapi project with preconfigured content types, seed data, public API access, and route-based middleware for default population.

## Usage

```
/strapi-configuration /path/to/my-project
/strapi-configuration
```

The skill will always present a selection menu showing available presets and a custom description option. If no output path is provided, it will ask for one.

## Available Presets

| Preset       | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `blog`       | Articles, Authors, Categories, Tags, Homepage, About, Global |
| `ecommerce`  | Products, Categories, Reviews, Homepage, Global              |
| `portfolio`  | Projects, Skills, Testimonials, Homepage, About, Contact     |
| `restaurant` | Menu Items, Categories, Reservations, Locations, Homepage    |

Template definitions are located in: `.claude/skills/strapi-configuration/templates/<preset>.json`

## Execution Steps

When the user invokes this skill, follow these steps **in order**:

### Step 1: Parse Arguments and Select Template

Extract `<output-path>` from the user's input. Any path-like argument should be treated as the output path.

**IMPORTANT: ALWAYS present a selection menu to the user.** Never auto-detect or assume a preset from the arguments. Even if the argument text contains a word like "blog" or "restaurant", treat it as an output path — do NOT silently select a matching preset.

1. If no output path was provided, ask the user for the desired project directory.

2. List all available template JSON files by scanning `.claude/skills/strapi-configuration/templates/*.json`. Read each template's `name`, `displayName`, and `description` fields.

3. Present a numbered selection menu to the user showing all available presets, with the last option always being the custom description option:

   ```
   Which type of project would you like to create?

   1. Blog — Articles, Authors, Categories, Tags, Homepage, About, Global
   2. E-commerce — Products, Categories, Reviews, Homepage, Global
   3. Portfolio — Projects, Skills, Testimonials, Homepage, About, Contact
   4. Restaurant — Menu Items, Categories, Reservations, Locations, Homepage
   5. Describe what you want to build (custom)
   ```

   Use the `AskUserQuestion` tool to present this selection.

4. Based on the user's selection:
   - If they chose a preset (options 1-4 or any template name), proceed to **Step 2** (Read the Template).
   - If they chose the last option (custom), proceed to **Step 2b** (Custom Project Generation).

### Step 2: Read the Template

Read the template JSON file at `.claude/skills/strapi-configuration/templates/<preset>.json`.

The template contains:
- `contentTypes` - Collection types and single types (schemas)
- `components` - Reusable component definitions
- `middlewares` - Route-based populate middleware configs per content type
- `seedData` - Sample data entries for each content type
- `publicPermissions` - Which API endpoints to make publicly accessible

### Step 2b: Custom Project Generation

When the user provides a description instead of a preset name, generate the template data dynamically.

**1. Gather requirements:**

If the user provided a short description (e.g., "online learning platform"), ask 1-2 clarifying questions to understand:
- What are the main entities/content types? (e.g., courses, lessons, instructors, students)
- What pages should exist? (e.g., homepage, about)
- Any special features? (e.g., reviews, categories, ratings)

If the user provided a detailed description, skip clarification and proceed.

**2. Design the content model:**

Based on the description, design content types following these patterns from existing templates:

- **Collection Types**: The main entities (e.g., `course`, `instructor`, `lesson`). Always include:
  - A `name` or `title` field (string, required)
  - A `slug` field (uid, targetField pointing to the name/title)
  - A `description` field (text or richtext)
  - Relevant relations between types (use manyToOne/oneToMany with inversedBy/mappedBy)
  - A `seo` component (shared.seo) on public-facing types

- **Single Types**: Always include `homepage` and `global`. Add others as needed (about, contact, etc.)
  - `global`: always has `siteName` (string), `siteDescription` (text), `favicon` (media), `defaultSeo` (shared.seo)
  - `homepage`: always has a hero section and `seo` (shared.seo)

- **Components**: Always include the shared set (`seo`, `media`, `rich-text`). Add custom components for the domain (e.g., `learning.lesson-resource`, `learning.curriculum-item`).

**3. Generate all template data in memory:**

Construct the same data structure as the preset JSON templates:
```
contentTypes, components, middlewares, seedData, publicPermissions
```

Follow the exact same Strapi schema format as the existing templates. Reference `.claude/skills/strapi-configuration/templates/blog.json` for the correct structure if needed.

**4. Generate seed data:**

Create 2-5 realistic sample entries per collection type with:
- Realistic names, descriptions, and content
- Component data inline (e.g., SEO metadata, technology tags)
- No media fields (skip image/file uploads in seed data)
- No relation fields in seed data (these require a two-pass approach)

**5. Design middleware populate configs:**

For each content type that has relations, media, or components, create a populate config that auto-fetches:
- Media fields: `{ fields: ["name", "alternativeText", "url", "formats"] }`
- Relations: `{ fields: ["name", "slug"] }` (or relevant identifying fields)
- Components with media: nested populate to reach media fields
- SEO components: `{ populate: { shareImage: { fields: [...] } } }`

**6. Set public permissions:**

- Collection types: `["find", "findOne"]`
- Single types: `["find"]`
- If the type accepts user submissions (like reservations or contact forms): add `"create"`

After constructing all data in memory, proceed to **Step 3** using the generated data exactly as if it came from a template file.

**7. Save as reusable template (optional):**

After generating the project, offer to save the generated template as a new JSON file:
```
Would you like me to save this as a reusable template at
.claude/skills/strapi-configuration/templates/<name>.json?
```

If the user agrees, write the template file so it can be reused with a preset name next time.

### Step 3: Create the Strapi Project

Run the CLI to create a vanilla Strapi project:

```bash
node packages/cli/create-strapi-app/bin/index.js <output-path> --non-interactive
```

Wait for this to complete before proceeding.

### Step 4: Generate Content Type Schemas

For each content type in the template, create the full API structure:

```
<output-path>/src/api/<name>/
├── content-types/
│   └── <name>/
│       └── schema.json
├── controllers/
│   └── <name>.ts
├── routes/
│   └── <name>.ts
├── middlewares/
│   └── default-populate.ts    (if middleware config exists for this type)
└── services/
    └── <name>.ts
```

**Schema file** (`schema.json`): Use the schema directly from the template's `contentTypes.<name>` object.

**Controller** (`<name>.ts`):
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreController('api::<name>.<name>');
```

**Service** (`<name>.ts`):
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreService('api::<name>.<name>');
```

**Route** (`<name>.ts`) - if NO middleware config exists for this type:
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreRouter('api::<name>.<name>');
```

**Route** (`<name>.ts`) - if middleware config EXISTS for this type:
```typescript
import { factories } from '@strapi/strapi';

export default factories.createCoreRouter('api::<name>.<name>', {
  config: {
    find: {
      middlewares: ['api::<name>.default-populate'],
    },
    findOne: {
      middlewares: ['api::<name>.default-populate'],
    },
  },
});
```

### Step 5: Generate Middleware Files

For each content type that has a middleware populate config in the template, create:

**`<output-path>/src/api/<name>/middlewares/default-populate.ts`**:
```typescript
export default (config, { strapi }) => {
  return async (ctx, next) => {
    // Set default populate if not already specified in the request
    if (!ctx.query.populate) {
      ctx.query.populate = <POPULATE_CONFIG_FROM_TEMPLATE>;
    }
    await next();
  };
};
```

Replace `<POPULATE_CONFIG_FROM_TEMPLATE>` with the actual populate object from the template's `middlewares.<name>.populate` value, serialized as a JavaScript object literal (not JSON.stringify).

### Step 6: Generate Component Files

For each component in the template, create:

```
<output-path>/src/components/<namespace>/<component-name>.json
```

Use the component definition directly from the template's `components.<namespace>.<component-name>` object.

### Step 7: Generate Seed Script

Create `<output-path>/scripts/seed.js` with the following structure:

```javascript
const { createStrapi, compileStrapi } = require('@strapi/strapi');

let strapi;

async function seedExampleApp() {
  const shouldImportSeedData = await isFirstRun();
  if (shouldImportSeedData) {
    try {
      console.log('Setting up the template...');
      await importSeedData();
      console.log('Ready to go');
    } catch (error) {
      console.log('Could not import seed data');
      console.error(error);
    }
  }
}

async function isFirstRun() {
  const pluginStore = strapi.store({
    environment: strapi.config.environment,
    type: 'type',
    name: 'setup',
  });
  const initHasRun = await pluginStore.get({ key: 'initHasRun' });
  await pluginStore.set({ key: 'initHasRun', value: true });
  return !initHasRun;
}

async function setPublicPermissions(newPermissions) {
  const publicRole = await strapi.query('plugin::users-permissions.role').findOne({
    where: { type: 'public' },
  });

  const allPermissionsToCreate = [];
  Object.keys(newPermissions).map((controller) => {
    const actions = newPermissions[controller];
    const permissionsToCreate = actions.map((action) => {
      return strapi.query('plugin::users-permissions.permission').create({
        data: {
          action: `api::${controller}.${controller}.${action}`,
          role: publicRole.id,
        },
      });
    });
    allPermissionsToCreate.push(...permissionsToCreate);
  });
  await Promise.all(allPermissionsToCreate);
}

async function createEntry({ model, entry }) {
  try {
    await strapi.documents(`api::${model}.${model}`).create({
      data: entry,
      status: 'published',
    });
  } catch (error) {
    console.error({ model, entry, error });
  }
}

async function importSeedData() {
  // Set public permissions
  await setPublicPermissions(<PUBLIC_PERMISSIONS_FROM_TEMPLATE>);

  // Import seed data for each content type
  <SEED_IMPORT_CALLS>
}

async function main() {
  const appContext = await compileStrapi();
  const app = await createStrapi(appContext).load();
  strapi = app;
  app.log.level = 'error';
  await seedExampleApp();
  await app.destroy();
  process.exit(0);
}

main().catch((error) => {
  console.error(error);
  process.exit(1);
});
```

Replace `<PUBLIC_PERMISSIONS_FROM_TEMPLATE>` with the `publicPermissions` object from the template.

Replace `<SEED_IMPORT_CALLS>` with calls to `createEntry()` for each item in `seedData`, iterating collection types and creating entries for single types.

### Step 8: Add Seed Script to package.json

Read `<output-path>/package.json` and add a seed script:

```json
{
  "scripts": {
    "seed": "node scripts/seed.js"
  }
}
```

### Step 9: Summary

After all files are generated, print a summary:

```
Project created at: <output-path>
Preset: <preset>

Content Types:
  - <list each content type with kind>

Components:
  - <list each component>

Middleware (default populate):
  - <list each content type with middleware>

Public API Endpoints:
  - GET /api/<pluralName> (find)
  - GET /api/<pluralName>/:id (findOne)
  ...

Next steps:
  1. cd <output-path>
  2. npm run develop    # Start the Strapi server
  3. npm run seed       # Seed the database with sample data
  4. Visit http://localhost:1337/admin to create an admin user
  5. Try the public API: curl http://localhost:1337/api/<first-plural-name>
```

## Adding New Templates

To add a new preset, create a new JSON file in `.claude/skills/strapi-configuration/templates/` following this structure:

```json
{
  "name": "my-preset",
  "displayName": "My Preset",
  "description": "Description of what this preset provides",
  "contentTypes": {
    "<singular-name>": {
      "kind": "collectionType | singleType",
      "collectionName": "<plural_name_with_underscores>",
      "info": {
        "singularName": "<singular-name>",
        "pluralName": "<plural-name>",
        "displayName": "<Display Name>",
        "description": "<description>"
      },
      "options": {
        "draftAndPublish": true
      },
      "pluginOptions": {},
      "attributes": {
        "<fieldName>": {
          "type": "<type>",
          ...constraints
        }
      }
    }
  },
  "components": {
    "<namespace>": {
      "<component-name>": {
        "collectionName": "components_<namespace>_<component_name>",
        "info": {
          "displayName": "<Display Name>",
          "icon": "<icon-name>",
          "description": ""
        },
        "options": {},
        "attributes": { ... }
      }
    }
  },
  "middlewares": {
    "<content-type-singular-name>": {
      "populate": {
        "<relation-field>": {
          "fields": ["field1", "field2"],
          "populate": { ... }
        }
      }
    }
  },
  "seedData": {
    "<content-type-singular-name>": [
      { "field1": "value1", ... },
      ...
    ]
  },
  "publicPermissions": {
    "<content-type-singular-name>": ["find", "findOne"]
  }
}
```

### Attribute Types Reference

| Type          | Example                                                                          |
| ------------- | -------------------------------------------------------------------------------- |
| `string`      | `{ "type": "string" }`                                                           |
| `text`        | `{ "type": "text", "maxLength": 500 }`                                           |
| `richtext`    | `{ "type": "richtext" }`                                                         |
| `email`       | `{ "type": "email" }`                                                            |
| `integer`     | `{ "type": "integer" }`                                                          |
| `float`       | `{ "type": "float" }`                                                            |
| `decimal`     | `{ "type": "decimal" }`                                                          |
| `boolean`     | `{ "type": "boolean", "default": false }`                                        |
| `date`        | `{ "type": "date" }`                                                             |
| `datetime`    | `{ "type": "datetime" }`                                                         |
| `uid`         | `{ "type": "uid", "targetField": "title" }`                                     |
| `enumeration` | `{ "type": "enumeration", "enum": ["a", "b", "c"] }`                            |
| `media`       | `{ "type": "media", "multiple": false, "allowedTypes": ["images"] }`            |
| `relation`    | `{ "type": "relation", "relation": "manyToOne", "target": "api::x.x" }`        |
| `component`   | `{ "type": "component", "repeatable": false, "component": "shared.seo" }`       |
| `dynamiczone` | `{ "type": "dynamiczone", "components": ["shared.media", "shared.rich-text"] }` |
| `json`        | `{ "type": "json" }`                                                            |
