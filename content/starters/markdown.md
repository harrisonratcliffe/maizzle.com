---
title: "Markdown"
repository: https://github.com/maizzle/starter-markdown.git
description: "Create emails from markdown files."
image: https://res.cloudinary.com/maizzle/image/upload/starters/markdown.jpg
date: 2022-12-05
---

# Markdown starter

This starter allows you to create emails from markdown files.

Simply add your markdown files to `src/content`, run the build command, and they will be converted to HTML emails using a predefined layout.

[View on GitHub &rarr;](https://github.com/maizzle/starter-markdown.git)

## Getting started

Scaffold a new project based on this starter:

```sh
npx degit maizzle/starter-markdown my-project
```

Install dependencies:

```sh no-copy
cd my-project

npm install
```

Start local development:

```sh
npm run dev
```

Build emails for production:

```sh
npm run build
```

## Custom layouts

The starter supports custom layouts, which you can add to `src/layouts`.

The default layout is `src/layouts/main.html`, but if you want to use a different layout for a specific markdown file, you can add a `layout` property to its front matter:

```md [src/content/example.md]
---
layout: secondary
---

## Custom layout

This email uses a custom layout, defined in `src/layouts/secondary.html`.
```

## Customization

See the detailed guide for the Markdown starter [here](/guides/markdown-emails/).
