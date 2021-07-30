---
layout: post
author: jean-romain krupa
title: "Setup Tailwind Css for rails project with webpack"
date: 2021-07-26 11:21:51 +0100
tags: rails tailwind webpack
---

Here we go for a quick tailwind css setup, with rails and turbolinks. 

```bash
$ yarn add tailwindcss@latest postcss@latest autoprefixer@latest
```

if you have an error 👇 then fix it here [here](https://tailwindcss.com/docs/installation#post-css-7-compatibility-build)

```
Error: PostCSS plugin tailwindcss requires PostCSS 8.
```

Now we need to import tailwindcss in `postcss.config.js` that is in the root of your directory.

```javascript
// postcss.config.js

module.exports = {
  plugins: [
    require("tailwindcss")("./app/javascript/stylesheets/tailwind.config.js"),
    require("postcss-import"),
    require("postcss-flexbugs-fixes"),
    require("postcss-preset-env")({
      autoprefixer: {
        flexbox: "no-2009",
      },
      stage: 3,
    }),
  ],
};

```

You can create the tailwind configuration file with the command.

```
$ npx tailwindcss init

```

it should generate a file `tailwind.config.js` at the root of the folder. like 👇 

```javascript
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

it is in this file, where you will be able to "pimp" the tailwind configuration. For example, you can add colors or fonts, ask tailwind to purge the unused classes in production (to reduce the file size)... here is an example of a pimped config file:

```javascript
module.exports = {
  purge: {
    enabled: ["production", "staging"].includes(process.env.NODE_ENV),
    content: [
      "./**/*.html.erb",
      "./**/**/*.html.erb",
      "./**/*.html+phone.erb",
      "./app/helpers/**/*.rb",
      "./app/javascript/**/*.js",
    ],
  },
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {
      colors: {
        'primary': {
          50: '#F6F7FF',
          100: '#EDF0FE',
          200: '#D2D9FD',
          300: '#B6C1FB',
          400: '#8093F9',
          500: '#4965F6',
          600: '#425BDD',
          700: '#2C3D94',
          800: '#212D6F',
          900: '#161E4A',
        },
        'secondary': {
          50: '#F5FDF9',
          100: '#ECFAF4',
          200: '#CFF3E3',
          300: '#B2ECD2',
          400: '#78DDB0',
          500: '#3ECF8E',
          600: '#38BA80',
          700: '#257C55',
          800: '#1C5D40',
          900: '#133E2B',
        },
        'tertiary': {
          50: '#F7F7F8',
          100: '#EEEEF1',
          200: '#D5D5DB',
          300: '#BCBCC5',
          400: '#898A9A',
          500: '#57586E',
          600: '#4E4F63',
          700: '#343542',
          800: '#272832',
          900: '#1A1A21',
        },
        'danger': {
          50: '#FEF8F8',
          100: '#FEF2F2',
          200: '#FCDEDE',
          300: '#FACACA',
          400: '#F7A3A3',
          500: '#F37B7B',
          600: '#DB6F6F',
          700: '#924A4A',
          800: '#6D3737',
          900: '#492525',
        },
        "code-400": "#fefcf9",
        "code-600": "#3c455b",
      }
    },
  },
  variants: {
    extend: {},
  },
  plugins: [],
}


```

Now you can move this file into, `javascript/stylesheets` (you need to create the stylesheets folder).

Create the `application.scss` file inside the `javascript/stylesheets` folder. and import the tailwind lib

```scss
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";

// You can add your own custom components/styles below
```

let's now import it in our `application.js` by adding this line `import "stylesheets/application";` at the end of the file.

```javascript
// This file is automatically compiled by Webpack, along with any other files
// present in this directory. You're encouraged to place your actual application logic in
// a relevant structure within app/javascript and only use these pack files to reference
// that code so it'll be compiled.

import Rails from "@rails/ujs"
import Turbolinks from "turbolinks"
import * as ActiveStorage from "@rails/activestorage"
import "channels"

Rails.start()
Turbolinks.start()
ActiveStorage.start()

import "stylesheets/application"; // 👈 add this little boy

```

and now we need the stylesheet to be compiled by webpack, so add this line in `views/layouts/application.html.erb`

```erb
    <%= stylesheet_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
```

Et voilà ! try it out in a the view by adding something like `<h2 class="bg-green-500">Rails and tailwind are amazing!</h2>`

You can use this amazing [tailwind Ui](https://tailwindui.com/) kit if you want to go faster and have UI components ready to go in your app.

Feel free to create classes thanks to the `@apply keyword`. For example here is how you could create the famous good old bootstrap `btn btn-primary`!

```scss
// app/javascript/components/button.scss

.btn {
  @apply inline-flex text-sm font-semibold text-center px-4 py-0 rounded-lg no-underline items-center justify-between;

  height: 45px;
  line-height: 45px;
  transition: ease 0.3s background, ease 0.3s transform, ease 0.2s color;

  &:hover,
  &:focus {
    @apply cursor-pointer;
  }

  &:disabled {
    @apply bg-blue-200 text-white cursor-not-allowed;
  }
}

.btn-primary {
  @apply bg-blue-500 text-white;

  &:hover,
  &:focus {
    @apply bg-blue-600 text-white;
  }

  &.outline {
    @apply bg-transparent border border-blue-500 text-blue-500 border-blue-500 shadow-none;

    &:hover,
    &:focus {
      @apply border-blue-600 text-blue-600 underline;
    }
  }
}

```

remember to `@import "components/buttons";`in your `application.scss` 
