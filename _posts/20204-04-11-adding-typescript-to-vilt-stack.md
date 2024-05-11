
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
sail npm -D @types/lodash @types/node @types/ziggy-js typescript ziggy-js
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