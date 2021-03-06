---
title: Using with Preprocessors
description: A guide to using Tailwind with common CSS preprocessors like Sass, Less, and Stylus.
---

Since Tailwind is a PostCSS plugin, there's nothing stopping you from using it with Sass, Less, Stylus, or other preprocessors, just like you can with other PostCSS plugins like [Autoprefixer](https://github.com/postcss/autoprefixer).

It's important to note that **you don't need to use a preprocessor with Tailwind** — you typically write very little CSS on a Tailwind project anyways so using a preprocessor just isn't as beneficial as it would be in a project where you write a lot of custom CSS.

This guide only exists as a reference for people who need to or would like to integrate Tailwind with a preprocessor for one reason or another.

---

## Using PostCSS as your preprocessor

If you're using Tailwind for a brand new project and don't need to integrate it with any existing Sass/Less/Stylus stylesheets, you should highly consider relying on other PostCSS plugins to add the preprocessor features you use instead of using a separate preprocessor.

This has a few benefits:

- **Your builds will be faster**. Since your CSS doesn't have to be parsed and processed by multiple tools, your CSS will compile much quicker using only PostCSS.
- **No quirks or workarounds.** Because Tailwind adds some new non-standard keywords to CSS (like `@tailwind`, `@apply`, `theme()`, etc.), you often have to write your CSS in annoying, unintuitive ways to get a preprocessor to give you the expected output. Working exclusively with PostCSS avoids this.

For a fairly comprehensive list of available PostCSS plugins see the [PostCSS GitHub repository](https://github.com/postcss/postcss/blob/master/docs/plugins.md), but here are a few important ones we use on our own projects and can recommend.

### Build-time imports

One of the most useful features preprocessors offer is the ability to organize your CSS into multiple files and combine them at build time by processing `@import` statements in advance, instead of in the browser.

The canonical plugin for handling this with PostCSS is [postcss-import](https://github.com/postcss/postcss-import).

To use it, install the plugin via npm:

```shell
# npm
npm install postcss-import

# yarn
yarn add postcss-import
```

Then add it as the very first plugin in your PostCSS configuration:

```js
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss'),
    require('autoprefixer'),
  ]
}
```

One important thing to note about `postcss-import` is that it strictly adheres to the CSS spec and disallows `@import` statements anywhere except at the very top of a file.

Won't work, `@import` statements must come first

```css
/* components.css */

.btn {
  @apply px-4 py-2 rounded font-semibold bg-gray-200 text-black;
}

/* Will not work */
@import "./components/card";
```

The easiest solution to this problem is to never mix regular CSS and imports in the same file. Instead, create one main entry-point file for your imports, and keep all of your actual CSS in separate files.

Use separate files for imports and actual CSS

```css
/* components.css */
@import "./components/buttons.css";
@import "./components/card.css";
```

```css
/* components/buttons.css */
.btn {
  @apply px-4 py-2 rounded font-semibold bg-gray-200 text-black;
}
```

```css
/* components/card.css */
.card {
  @apply p-4 bg-white shadow rounded;
}
```

The place you are most likely to run into this situation is in your main CSS file that includes your `@tailwind` declarations.

Won't work, `@import` statements must come first

```css
@tailwind base;
@import "./custom-base-styles.css";

@tailwind components;
@import "./custom-components.css";

@tailwind utilities;
@import "./custom-utilities.css";
```

You can solve this by creating separate files for each `@tailwind` declaration, and then importing those files in your main stylesheet. To make this easy, we provide separate files for each `@tailwind` declaration out of the box that you can import directly from `node_modules`.

The `postcss-import` plugin is smart enough to look for files in the `node_modules` folder automatically, so you don't need to provide the entire path — `"tailwindcss/base"` for example is enough.

Import our provided CSS files

```css
@import "tailwindcss/base";
@import "./custom-base-styles.css";

@import "tailwindcss/components";
@import "./custom-components.css";

@import "tailwindcss/utilities";
@import "./custom-utilities.css";
```

### Nesting

To add support for nested declarations, you have two options:

- [postcss-nested](https://github.com/postcss/postcss-nested), which uses a syntax much like Sass.

- [postcss-nesting](https://github.com/jonathantneal/postcss-nesting), which follows the [CSS Nesting](https://drafts.csswg.org/css-nesting-1/) specification that will hopefully be available directly in the browser in the future.

To use either of these plugins, install them via npm:

```shell
# npm
npm install postcss-nested  # or postcss-nesting

# yarn
yarn add postcss-nested  # or postcss-nesting
```

Then add them to your PostCSS configuration, somewhere after Tailwind itself but before Autoprefixer:

```js
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss'),
    require('postcss-nested'), // or require('postcss-nesting')
    require('autoprefixer'),
  ]
}
```

### Variables

These days CSS variables (officially known as custom properties) have really good [browser support](https://caniuse.com/#search=css%20custom%20properties), so you may not actually need a plugin for variables at all.

However if you need to support IE11, you can use the [postcss-custom-properties](https://github.com/postcss/postcss-custom-properties) plugin to automatically create fallbacks for your variables.

To use it, install it via npm:

```shell
# npm
npm install postcss-custom-properties

# yarn
yarn add postcss-custom-properties
```

Then add it to your PostCSS configuration, somewhere after Tailwind itself but before Autoprefixer:

```js
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss'),
    require('postcss-nested'),
    require('postcss-custom-properties'),
    require('autoprefixer'),
  ]
}
```

### Future CSS features

You can add support for dozens of upcoming CSS features to your project using the [postcss-preset-env](https://github.com/csstools/postcss-preset-env) plugin.

To use it, install it via npm:

```shell
# npm
npm install postcss-preset-env

# yarn
yarn add postcss-preset-env
```

Then add it to your PostCSS configuration somewhere after Tailwind itself:

```js
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss'),
    require('postcss-preset-env')({ stage: 1 }),
  ]
}
```

**It's important to note that CSS variables, nesting, and autoprefixer are included out-of-the-box**, so if you're using `postcss-preset-env`, you don't need to add separate plugins for those features.

---

## Using Sass, Less, or Stylus

To use Tailwind with a preprocessing tool like Sass, Less, or Stylus, you'll need to add an additional build step to your project that lets you run your preprocessed CSS through PostCSS. If you're using Autoprefixer in your project, you already have something like this set up.

The exact instructions will be different depending on which build tool you are using, so see our [installation documentation](/docs/installation#3-process-your-css-with-tailwind) to learn more about integrating Tailwind into your existing build process.

The most important thing to understand about using Tailwind with a preprocessor is that **preprocessors like Sass, Less, and Stylus run separately, before Tailwind**. This means that you can't feed output from Tailwind's `theme()` function into a Sass color function for example, because the `theme()` function isn't actually evaluated until your Sass has been compiled to CSS and fed into PostCSS.

Won't work, Sass is processed first

```css
.alert {
  background-color: darken(theme('colors.red.500'), 10%);
}
```

For the most cohesive development experience, it's recommended that you [use PostCSS exclusively](#using-post-css-as-your-preprocessor).

Aside from that, each preprocessor has its own quirk or two when used with Tailwind, which are outlined with workarounds below.

### Sass

When using Tailwind with Sass, using `!important` with `@apply` requires you to use interpolation to compile properly.

Won't work, Sass complains about !important

```css
.alert {
  @apply bg-red-500 !important;
}
```

Use interpolation as a workaround

```css
.alert {
  @apply bg-red-500 #{!important};
}
```

### Less

When using Tailwind with Less, you cannot nest Tailwind's `@screen` directive.

Won't work, Less doesn't realise it's a media query

```css
.card {
  @apply rounded-none;

  @screen sm {
    @apply rounded-lg;
  }
}
```

Instead, use a regular media query along with the `theme()` function to reference your screen sizes, or simply don't nest your `@screen` directives.

Use a regular media query and theme()

```css
.card {
  @apply rounded-none;

  @media (min-width: theme('screens.sm')) {
    @apply rounded-lg;
  }
}
```

Use the @screen directive at the top-level

```css
.card {
  @apply rounded-none;
}
@screen sm {
  .card {
    @apply rounded-lg;
  }
}
```

### Stylus

When using Tailwind with Stylus, you can't use Tailwind's `@apply` feature without wrapping the entire CSS rule in `@css` so that Stylus treats it as literal CSS:

Won't work, Stylus complains about @apply

```css
.card {
  @apply rounded-lg bg-white p-4
}
```

Use @css to avoid processing as Stylus

```css
@css {
  .card {
    @apply rounded-lg bg-white p-4
  }
}
```

This comes with a significant cost however, which is that **you cannot use any Stylus features inside a `@css` block**.

Another option is to use the `theme()` function instead of `@apply`, and write out the actual CSS properties in long form:

Use theme() instead of @apply

```css
.card {
  border-radius: theme('borderRadius.lg');
  background-color: theme('colors.white');
  padding: theme('spacing.4');
}
```

In addition to this, Stylus doesn't support nesting the `@screen` directive (just like Less).

Won't work, Stylus doesn't realise it's a media query

```css
.card {
  border-radius: 0;

  @screen sm {
    border-radius: theme('borderRadius.lg');
  }
}
```

Instead, use a regular media query along with the `theme()` function to reference your screen sizes, or simply don't nest your `@screen` directives.

Use a regular media query and theme()

```css
.card {
  border-radius: 0;

  @media (min-width: theme('screens.sm')) {
    border-radius: theme('borderRadius.lg');
  }
}
```

Use the @screen directive at the top-level

```css
.card {
  border-radius: 0;
}
@screen sm {
  .card {
    border-radius: theme('borderRadius.lg');
  }
}
```
