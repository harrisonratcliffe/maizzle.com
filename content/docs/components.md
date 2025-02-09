---
title: "Components"
description: "Import components into your HTML email templates and render them with custom slot content and data."
---

# Components

**👋 New syntax**

You are viewing the documentation for the new Components syntax introduced in `v4.4.0`. Not ready to switch yet? See the [legacy Components docs](https://v43x.maizzle.com/docs/components).

---

Components help you organize blocks of markup into files that can be referenced throughout your project with simple, declarative syntax.

## Create

To create a Component, add an HTML file in `src/components`:

```xml [src/components/alert.html]
<content />
```

The `<content />` tag will be replaced with the content passed to the Component.

<Alert type="info">You can safely omit the `<content />` tag if you want to use Components as includes, and don't actually need to pass any content to them.</Alert>

## Tags

There are two ways to use Components:

- through the x-tag syntax
- through the `<component>` tag

### x-tag

Component names are automatically registered and can be used without having to specify their file path, by using a Blade-like syntax: start with the string `x-` followed by the `kebab-case` name of the component file.

For example, let's use the `alert.html` Component we created earlier:

```xml [src/templates/example.html]
<x-alert>
  This text will replace the `content` tag in the Component.
</x-alert>
```

The following naming convention is used:

| Component file | x-tag syntax |
| -------------- | ----- |
| `alert.html` | `<x-alert>` |
| `alert_info.html` | `<x-alert_info>` |
| `AlertInfo.html` | `<x-alertinfo>` |

As you can see, the second and last examples are not very readable, which is why we recommend using a [nested file structure](#nested-file-structure) instead.

### `<component>` tag

Alternatively, you may use the `<component>` tag to insert a Component:

```xml [src/templates/example.html]
<component src="src/components/alert.html">
  This text will replace the `content` tag in the Component.
</component>
```

The `src` attribute is mandatory and it needs to point to the Component's file path, relative to the project root.

## Nested file structure

If a Component is nested deeper within your `src/components` directory, you can reference it through dot notation.

For example, consider the following Component:

```xml [src/components/button/alt.html]
<a href="https://maizzle.com">
  <content />
</a>
```

You may reference it like this:

```xml [src/templates/example.html]
<x-button.alt>
  Go to website
</x-button.alt>
```

The nested folder path comes after `x-`, and the file name comes after the dot.

### Index files

If you add an `index.html` Component inside a nested directory, you can also reference it without its file name part:

```xml [src/components/button/index.html]
<a href="https://maizzle.com">
  <content />
</a>
```

Both of these will work and use the same `button/index.html` Component:

```xml [src/templates/example.html]
<x-button>
  Go to website
</x-button>

<x-button.index>
  Go to website
</x-button.index>
```

## Slots

A Component may define slots, which act as placeholders that can be replaced (filled) with code when you use it.

For example, let's create a banner Component with a slot for a custom title:

```xml [src/components/banner.html]
<div role="banner">
  <slot:title />

  <content />
</div>
```

We can use it like this:

```xml [src/templates/example.html]
<x-banner>
  <fill:title>
    <h2>This is the title</h2>
  </fill:title>

  <p>This is the content</p>
</x-banner>
```

The result will be:

```xml [build_production/example.html]
<div role="banner">
  <h2>This is the title</h2>

  <p>This is the content</p>
</div>
```

### Default content

A slot may have default content, which will be used if it hasn't been filled.

```xml [src/components/banner.html]
<div role="banner">
  <h2>
    <slot:title>Default title</slot:title>
  </h2>

  <content />
</div>
```

### Prepend

You may prepend content to a slot by using the `prepend` attribute on the `<fill>` tag.

```xml [src/templates/example.html]
<x-banner>
  <fill:title prepend>Hello, </fill:title>

  <p>This is the content</p>
</x-banner>
```

With our default `slot:title` example, that would result in:

```xml [build_production/example.html]
<div role="banner">
  <h2>Hello, Default title</h2>

  <p>This is the content</p>
</div>
```

### Append

You may also append content to a slot by using the `append` attribute on the `<fill>` tag.

```xml [src/templates/example.html]
<x-banner>
  <fill:title append>, what a name!</fill:title>

  <p>This is the content</p>
</x-banner>
```

With our default `slot:title` example, that would result in:

```xml [build_production/example.html]
<div role="banner">
  <h2>Default title, what a name!</h2>

  <p>This is the content</p>
</div>
```

### Check if a slot is filled

You may check if a slot has been filled by using the `$slots` variable in a Component.

For example, let's create a `<x-footer>` Component that will pull in another Component based on whether a `copyright` slot has been filled or not:

```xml [src/components/footer.html]
<div>
  <content />

  <if condition="$slots.copyright?.filled">
    <!-- src/components/copyright.html -->
    <x-copyright />
  </if>
</div>
```

## Stacks

You may push content to named stacks that can be rendered in other Components.

For example, imagine you're coding a Shopify email template and need to add some Liquid code at the very top of the HTML, before the `doctype`.

You would modify your Layout to include a `stack` tag:

```hbs [src/layouts/layout.html]
<stack name="liquid-vars" />

<!doctype html>
<html>
<head>
  <style>{{{ page.css }}}</style>
</head>
<body>
  <slot:template />
</body>
```

<Alert type="danger">`stack` and `push` tags require a non-empty `name` attribute.</Alert>

You may then push content to that stack from a Template:

```xml [src/templates/example.html]
<push name="liquid-vars">
  {% capture email_title %} Your shopping cart is waiting for you {% endcapture %}
</push>

<x-layout>
  <fill:template>
    <!-- your email HTML... -->
  </fill:template>
</x-layout>
```

<Alert>You may also use the `<push>` tag inside the `<fill:template>` tag.</Alert>

Result:

```liquid [build_production/example.html]
{% capture email_title %} Your shopping cart is waiting for you {% endcapture %}

<!doctype html>
<!-- etc. -->
```

### once

You may use the `once` attribute on the `<push>` tag to only push content once in a rendering cycle. This is useful if you're rendering the Component in a loop and want to make sure the contents of `<push>` is only rendered once.

For example, imagine this Card Component:

```hbs [src/components/card.html]
<push name="head" once>
  <style tailwindcss>
    .card {
      @apply bg-white rounded-lg shadow-md;
    }
  </style>
</push>

<div class="card">
  <!-- ... -->
</div>
```

Looping over this Component will only push that CSS once to the `head` stack:

```xml [src/templates/example.html]
<x-layout>
  <fill:template>
    <each loop="[1,2,3]">
      <x-card />
    </each>
  </fill:template>
</x-layout>
```

## Props

Props are attributes that can be added to a Component. They can be used to pass data to the Component, or to configure its behavior.

To use props in a Component, you need to define them first. This is done by adding a `<script>` tag with the `props` attribute:

```hbs [src/components/alert.html]
<script props>
  module.exports = {
    title: props.title || 'Default title'
  }
</script>

<div>
  {{ title }}
</div>
```

Props that you pass to the Component will be available in the `<script>` tag as the `props` object. In this example we're getting the `title` prop from `props.title`, falling back to a default value if it's not provided.

The script uses `module.exports` to export an object with props as keys. You can use these keys inside the Component through the curly braces syntax, as shown above.

To pass the `title` prop to the Component, you would use the `title` attribute:

```xml [src/templates/example.html]
<x-alert title="Hello, world!" />
```

#### Encoding props data

When passing a props object to a Component, you need to encode the values.

For example, these won't work:

```xml
<x-alert props='{ "title": "Component's Title" }' />
```

```xml
<x-alert props='{ "title": "Component\'s Title" }' />
```

But this will:

```xml
<x-alert props='{ "title": "Component&#39;s Title" }' />
```

### Aware props

By default, props are scoped to the Component and are not available to nested Components. If you need to change this, you may use the `aware:` prefix when passing props to a Component:

Consider the following two Components:

```hbs [src/components/child.html]
<script props>
  module.exports = {
    title: props.title || 'Default child title'
  }
</script>

<div>
  Title in child: {{ title }}
</div>
```

```hbs [src/components/parent.html]
<script props>
  module.exports = {
    title: props.title || 'Default parent title'
  }
</script>

<div>
  Title in parent: {{ title }}
  <x-child />
</div>
```

If you pass the `title` to the `x-parent` Component:

```xml [src/templates/example.html]
<x-parent title="Hello, world!" />
```

... the result will be:

```html [build_production/example.html]
<div>
  Title in parent: Hello, world!
  <div>
    Title in child: Default child title
  </div>
</div>
```

As you can see, the `title` prop was not passed down to the `x-child` Component, and it used the default value instead.

To make sure a prop is passed down to all nested Components, use the `aware:` prefix:

```xml [src/templates/example.html]
<x-parent aware:title="Hello, world!" />
```

Now you'll see the result that you'd expect:

```html [build_production/example.html]
<div>
  Title in parent: Hello, world!
  <div>
    Title in child: Hello, world!
  </div>
</div>
```

## Attributes

You may pass HTML attributes to a Component and they will be added to the root element of the Component.

If you want to change the element to which the attributes are added, you can use the `attributes` attribute:

```jsx [src/components/example.html]
<table>
  <tr>
    <td attributes>
      <content />
    </td>
  </tr>
</table>
```

### Expressions in attributes

[Expressions](/docs/expressions) may be used in a Component's attribute:

```hbs [src/templates/example.html]
---
title: "Hello, world!"
---

<x-alert title="{{ page.title }}" />
```

### Attribute removal

The following attributes will be removed from the target element:

- unknown HTML attributes (see [valid-attributes.js](https://github.com/thewebartisan7/posthtml-components/blob/main/src/valid-attributes.js))
- attributes that are defined as props in the Component

### Attribute merging

`class` and `style` attribute values will be merged with existing ones from the target element. All other attributes will be overwritten.

### Safelist attributes

If you need to safelist or even block certain HTML attributes that you pass to a Component, see the [Component configuration docs](/docs/configuration/components#elementattributes).

## Variables

When creating a Component, you have access to global `page` variables:

```hbs [src/components/example.html]
<div>
  Building for: {{ page.env }}
</div>
```

Using this `<x-example />` Component in a Template will render:

```html [build_production/example.html]
<div>
  Building for: production
</div>
```
