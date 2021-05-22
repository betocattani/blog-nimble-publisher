%{
  title: "Tailwind CSS - Install and Config",
  author: "Luiz Cattani",
  tags: ~w(elixir phoenix process),
  description: "My notes explaning the basic concepts of TailwindCSS"
}
---

Tailwind Course - Adam Wathan
https://www.youtube.com/watch?v=21HuwjmuS7A&list=PL7CcGwsqRpSM3w9BT_21tUU8JN2SnyckR&index=2


Install and config Tailwind project

create a project with package.json
```bash
& npm init -y
```

install tailwdind and postcss
```bash
$ npm install tailwindcss postcss-cli autoprefixer
```

Create `tailwind.config.js` file
```bash
$ npx tailwind init
```

create a new postcss config file in the root of the project
```
$ touch postcss.config.js
```

```js
// postcss.config.js

module.exports = {
  plugins: [
    require('tailwindcss'),
    require('autoprefixer')
  ]
}
```

Create a tailwind.css file
```bash
$ touch css/tailwind.css
```

Custom markers - when compile find the base, components and utilities
```css
/* css/tailwind.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Change the file package.json, add the path to the tailwind.css file and define a path to save the compiled file
```js
{
  "name": "tailwindcss_course",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "postcss css/tailwind.css -o public/build/tailwind.css"
  },
// ...
```

Build the application
```bash
npm run build
```

Install Live Server (Auto refresh)
```bash
$ npm install -g live-server
```

Using Live Server point to a folder
```bash
$ live-server public/
```

Create a simple index html file `/public/index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- link the tailwind css file -->
    <link rel="stylesheet" href="/build/tailwind.css">
  </head>
  <body class="bg>
    <!-- Add a hello_world with tailwind classes -->
    <h1 class="text-4xl font-bold text-center text-blue-500">Hello World</h1>
  </body>
</html>
```

Create a new directory to images of /public
```bash
$ mkdir /public/img
```
