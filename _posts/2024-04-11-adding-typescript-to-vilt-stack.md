---
title: "Perfecting Laravel Stack: Adding Typescript Support"
categories: [Laravel, Web Development]
tags: [Laravel, DevContainer, VSCode, VILT, Inertia.js, Typescript]
---


## Adding Typescript Support

Credit: https://tannercampbell.com/using_typescript_with_inertiajs_and_vue/


```bash
code tsconfig.json
```

Add the following:
```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "allowJs": true,
    "lib": ["esnext", "dom"],
    "types": ["@types/node", "@types/ziggy-js"],
    "paths": {
      "@/*": ["./resources/js/*"]
    },
    "outDir": "./public/build/assets"
  },
  "typeRoots": ["./node_modules/@types", "resources/js/types"],
  "include": [
    "resources/js/**/*.ts",
    "resources/js/**/*.d.ts",
    "resources/js/**/*.vue"
  ],
  "exclude": ["node_modules", "public"]
}
```
{: file='tsconfig.vue'}


Install typescript dependencies:
```bash
sail npm install -D @types/lodash @types/node @types/ziggy-js typescript ziggy-js
```


Rename `resources/js/app.js` to `resources/js/app.ts`.


```bash
mv resources/js/app.js resources/js/app.ts
```

Now lets also change update files referencing it:
```js
...
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.ts'], // From this
            input: ['resources/css/app.css', 'resources/js/app.ts'], // To this
            refresh: true,
        }),
```
{: file='vite.config.js'}

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    @vite('resources/js/app.js') // From this
    @vite('resources/js/app.ts') // To this
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```
{: file='resources/views/app.blade.php'}


### Renaming `bootstrap.js` to `bootstrap.ts`
Next we need to rename `resources/js/bootstrap.js` to `resources/js/bootstrap.ts`.

```bash
mv resources/js/bootstrap.js resources/js/bootstrap.ts
```

Since `bootstrap.js` is only used within `app.ts`, and it doesn't include the extension, we are good to go.


### Utililzing Typescript Within Vue Components

Now that we have added typescript support ot our project, we can now utilize typescript within our Vue components. Here is an example of how we can do this:

First of, to make our Vue components typescript aware, we need to add the `lang="ts"` attribute to our script tag. Here is an example:

So that our `<script setup>` becomes like this: `<script setup lang="ts">`

```vue
<script setup lang="ts"> // <------ Remember to add lang="ts" to script tag
import { defineProps } from 'vue'

// Import PropType from vue
import type { PropType } from 'vue'


// Properties provided by the controller
defineProps({
  message: String // Change from this
  message: String as PropType<string> // To this
})

</script>

<template>
  <div>
      <!-- Add tailwind class to our header -->
      <h1 class="bg-red-500">Dashboard</h1>
      <p> {% raw %} {{ message }} {% endraw %} </p>
  </div>
</template>
```

We can even go more crazy with components with custom types like this:
```vue
<script setup lang="ts">
import { defineProps } from 'vue'
import type { PropType } from 'vue'

interface Book {
  title: string
  author: string
  year: number
}

defineProps({
  book: Object as PropType<Book>
})

</script>

<template>
  <div>
      <p> {% raw %} {{ book.title }} {% endraw %} </p>
      <p> {% raw %} {{ book.author }} {% endraw %} </p>
      <p> {% raw %} {{ book.year }} {% endraw %} </p>
  </div>
</template>
```


## Typing Inertia.js Page Props

Since we are using Inertia.js, we can also type our page props. Here is an example of how we can do this:

Lets start by creating a `types` directory in our `resources/js` directory. This is where we will store all our typescript declaration files.

```bash
mkdir resources/js/types
```

Next, let's create a declaration file called `inertia.d.ts` in the `resources/js/types` directory:

```bash
touch resources/js/types/inertia.d.ts
```

Add the following to the `inertia.d.ts` file:

```ts
import { PageProps as InertiaPageProps } from '@inertiajs/core';

// Globally page props provided by `./app/Http/Middleware/HandleInertiaRequests.php`
interface AppPageProps {
  user: {
    id: number;
    name: string;
    email: string;
  }
}

declare module '@inertiajs/core' {
  interface PageProps extends AppPageProps, InertiaPageProps {}
}
```

This code example would match a hypothetical `HandleInertiaRequests` middleware that adds a `user` object to every page. This is a great way to add global page props to every page in your application.

```php
class HandleInertiaRequests extends Middleware
{
    // ... Rest of the code
    public function share(Request $request): array
    {
        return array_merge(parent::share($request), [
            'user' => [
                'id' => "1", // OR $request->user()->id,
                'name' => "John Doe", // OR $request->user()->name,
                'email' => "johndoe@gmail.com", // OR $request->user()->email,
            ],
        ]);
    }
}
```
{: file='app/Http/Middleware/HandleInertiaRequests.php'}

Now, we can use the [usePage hook](https://inertiajs.com/shared-data) to access the page props in our Vue components. Here is an example of how we can do this:
```vue
<script setup lang="ts">
// We use the usePage hook to access the page props
import { usePage } from '@inertiajs/vue3';

const page = usePage();
</script>


<template>
  <div>
      <p>Id: {{ page.props.user.id }} </p>
      <p>Name: {{ page.props.user.name }} </p>
      <p>Email: {{ page.props.user.email }} </p>
  </div>
</template>
```
{: file='resources/js/Pages/Dashboard.vue(From previous guide)'}

## Adding types ziggy routes

Another thing we should do is to add types to our ziggy routes. This will help us to avoid typos and other errors when using the `route` helper in our Vue components.

To do this, we need to create a `ziggy.d.ts` file in the `resources/js/types` directory:

```bash
touch resources/js/types/ziggy.d.ts
```

Add the following to the `ziggy.d.ts` file:

```ts
import routeFn from 'ziggy-js';

declare global {
  const route: typeof routeFn;
}
```

And there we go, you now also have types for your ziggy routes!

![Component Output](assets/img/adding-typescript-to-vilt-stack/use-page-page-props-working.png)

How cool is that? We can now access the page props in our Vue components with typescript support.

Bare in mind that we obviously need to update our `resources/js/types/inertia.d.ts` every time we add a new page prop to our `HandleInertiaRequests` middleware. This is a small price to pay for the benefits of typescript.


### Installing VSCode Extensions

One of the main reasons why we are using typescript is to take advantage of the intellisense that it provides. To make the most of this, let's make sure we have the following extensions installed in our VSCode:

- [Vue - Official](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (Previously named Volar) `Vue.volar`


### Conclusion