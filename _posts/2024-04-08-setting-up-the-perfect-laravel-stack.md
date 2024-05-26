---
title: Creating the perfect Laravel stack with VSCode DevContainer
categories: [Laravel, Web Development]
tags: [Laravel, DevContainer, VSCode, VILT, Inertia.js, Tailwind CSS]
image:
  path: assets/img/setting-up-the-perfect-laravel-stack/VILT-logo-meme.jpg
---


# Introduction
Welcome to this comprehensive guide on setting up a VILT stack in VSCode. Whether you are an experienced web developer or just beginning to explore these technologies, this guide will streamline your development process, enabling you to focus more on creating robust web applications and less on setup and configuration. We will also leverage VSCode's DevContainer feature to ensure a consistent development environment across all project contributors. All you need to begin is Docker and VSCode installed on your machine. Let’s dive into what makes the VILT stack a powerful choice for modern web development.

# What is the VILT Stack?
VILT stands for Vue.js, Inertia.js, Laravel, and Tailwind CSS. Each component plays a crucial role in web development:

- **Vue.js**: A progressive JavaScript framework used for building user interfaces. Vue enables dynamic content updating within web pages, making it a popular choice for single-page applications.

- **Inertia.js**: This library allows developers to create single-page applications using classic server-side routing and controllers. It bridges the gap between traditional server-side applications and modern API-based client-side rendering.

- **Laravel**: A comprehensive PHP framework that makes handling business logic and data management simple and elegant. Laravel’s rich ecosystem supports rapid development and excellent security practices.

- **Tailwind CSS**: A utility-first CSS framework that allows developers to style applications directly in their HTML. Tailwind's approach speeds up the styling process and reduces CSS clutter, making it easier to maintain and adjust styles as applications evolve.

Think of Vue.js, Inertia.js, Laravel, and Tailwind CSS as a superhero squad that cuts through web development chaos like a hot knife through butter. Toss in VSCode's DevContainer, and you’ve got a secret lair where every developer syncs up perfectly—making 'it works on my machine' a thing of the past!


# Table of Contents
1. [Prerequisites](#prerequisites)
2. [Creating DevContainer](#creating-devcontainer)
3. [Setting up Laravel](#setting-up-laravel)
4. [Installing Inertia.js](#installing-inertiajs)
5. [Installing Tailwind CSS](#installing-tailwind-css) (Optional)
6. [Installing Ziggy](#installing-ziggy) (Optional)
7. [Adding Debugging Support](#adding-debugging-support) (Optional)
8. [The cherry on top](#the-cherry-on-top) (Optional)

# Prerequisites
- [VSCode](https://code.visualstudio.com/)
  - [Dev Containers Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Docker](https://www.docker.com/)


*Yes that's it! You don't need to install PHP, Composer, Node.js, or any other dependencies on your machine. Everything will be installed in the DevContainer.*

## Creating DevContainer

### Step 1: Create devcontainer directory

In the root of our project directory, create a new directory called `.devcontainer`. This directory will contain all the configuration files for our devcontainer.

```bash
mkdir .devcontainer
touch .devcontainer/devcontainer.json
touch .devcontainer/Dockerfile
```

### Step 2: Configure devcontainer.json
This configuration will tell VSCode how to build and run our devcontainer.

```jsonc
{
  "name": "Debian",
  "build": {
    "dockerfile": "Dockerfile",
    "args": {
      "INSTALL_ZSH": "true",
      "USER_UID": "1000",
      "USER_GID": "1000"
    }
  },
  "workspaceFolder": "/home/vscode/project",
  "workspaceMount": "source=${localWorkspaceFolder},target=/home/vscode/project,type=bind,consistency=delegated",
  "runArgs": [
    "--init",
    "--privileged"
  ],
  "forwardPorts": [
    8080,
  ],
  "customizations": {
    "settings": {
      "terminal.integrated.defaultProfile.linux": "zsh",
    },
    "vscode": {
      "extensions": [
        // Helps with tailwind css class completion
        "bradlc.vscode-tailwindcss",

        // Nice quality of life when it comes to vue
        "znck.vue",
        "Vue.volar",

        // Used for PHP debugging
        "xdebug.php-debug"
      ]
    }
  },
  "remoteUser": "vscode",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/php:1": {
      "installComposer": true
    },
    "ghcr.io/devcontainers/features/node:1": {},
    },
  "containerEnv": {
    "APP_PORT": "8080"
  }
}
```
{: file='.devcontainer/devcontainer.json'}

### Step 3: Configure Dockerfile

```Dockerfile
FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

RUN mkdir -p /home/vscode/project
WORKDIR /home/vscode/project

RUN git clone https://github.com/zsh-users/zsh-completions.git /home/vscode/.oh-my-zsh/custom/plugins/zsh-completions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git /home/vscode/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/zsh-users/zsh-autosuggestions.git /home/vscode/.oh-my-zsh/custom/plugins/zsh-autosuggestions

RUN cp /home/vscode/.zshrc /home/vscode/.zshrc.bak

RUN echo "$(cat /home/vscode/.zshrc)" | awk '{gsub(/plugins=\(git\)/, "plugins=(git zsh-completions zsh-syntax-highlighting zsh-autosuggestions)")}1' > /home/vscode/.zshrc.replaced && mv /home/vscode/.zshrc.replaced /home/vscode/.zshrc

# Create alias for sail
RUN echo "alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'" >> /home/vscode/.zshrc
RUN echo 'export PATH="/workspaces/dht-webapp/vendor/bin/:$PATH"' >> /home/vscode/.zshrc

# Define the location of the npm cache. This is needed because permission problems
# will occur if the cache is stored in the default location (/root/tmp/.npm)
RUN echo 'export npm_config_cache=/home/vscode/tmp/npm-cache' >> /home/vscode/.zshrc

RUN chown -R vscode:vscode /home/vscode/project
RUN chmod -R 700 /home/vscode/project
```
{: file='.devcontainer/Dockerfile'}

### Step 4: Starting the DevContainer

To start the devcontainer, open the command palette in VSCode by pressing `Ctrl+Shift+P` and type `Dev Containers: Rebuild and Reopen in Container`.
This will build and start the devcontainer.

> Building the devcontainer for the first time might take a while as it needs to download all the necessary dependencies and tools.
{: .prompt-info }

That's it! You now have a your very own DevContainer with all your development tools and dependencies set up. Don't believe me? Try running
`composer --version` or `npm --version` in the terminal.

Best thing is, everyone in your team can now have the same development environment. No more "It works on my machine" excuses.


## Setting up Laravel

Now that we have our DevContainer set up, let's install Laravel. We will be using Laravel Sail which is a light-weight command-line interface for interacting with Laravel's default Docker development environment. Sail is a great way to get started with Laravel as it creates a docker-compose with all the necessary services like MySQL, Redis, etc. for you. This will save you a lot of time and effort in setting up your development environment.


### Step 1: Install Laravel
You have multiple options to install along with Laravel Sail such as `mysql`, `pgsql`, `redis`, `selenium`. You can find the full list of options [here](https://laravel.com/docs/11.x/installation#choosing-your-sail-services). These services will be added to the `docker-compose.yml` file. You can add these services by passing them as a query parameter in the URL.

In this example, I will be installing Laravel with `pgsql` and `redis`.

```bash
curl -s https://laravel.build/example-app?with=pgsql,redis | bash
```

> You can add services later by running `sail php artisan sail:install`
{: .prompt-info }


This will create a new Laravel application in the `example-app` directory. However lets move the files to the root of the project directory.

```bash
# Moves all files from example-app to the root of the project directory
# including hidden files
mv example-app/{.,}* .
```

We can now also safely remove the `example-app` directory as we no longer need it.

```bash
rm -rf example-app
```

### Step 2: Starting the development server
Great! There is nothing like the smell of a fresh installation of Laravel. However before we start the development server, lets configure the `APP_PORT` within our `.env` to `8080` so that we can access the application from our browser.
This will simply change the default port to 8080 instead of port 80 which is often used by other services.

```bash
# Configures the APP_PORT in .env to 8080
echo "APP_PORT=8080" >> .env
```

Start the development server by running the following command in the terminal.

```bash
sail up
```

### Step 3: Preparing the environment

Next lets just quickly run our migrations to create the necessary tables in our database.

In a new terminal run:
```bash
sail artisan migrate
```


You can now access your Laravel application by visiting [http://localhost:8080](http://localhost:8080) in your browser.

From now on, you can run all your Laravel commands using `sail` in your terminal. eg.
- `sail artisan migrate`
- `sail artisan test`
- `sail composer install`

*If you are purely interested in a Laravel setup, you can stop here. However, if you want to take it to the next level and set up Inertia.js and Tailwind CSS, keep reading.*

## Installing Inertia.js

Now that we have our Laravel application set up, let's install Inertia.js. We will be following the official [documentation](https://inertiajs.com/server-side-setup) to install Inertia.js.

### Step 1: Install dependencies
```bash
sail composer require inertiajs/inertia-laravel
```

### Step 2: Creating Root Template

Create a new file called `app.blade.php` in the `resources/views` directory. This file will be the root template for all our Inertia.js pages.



```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    @vite('resources/js/app.js')
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```
{: file='resources/views/app.blade.php'}

### Step 3: Create inertia middleware
```bash
sail php artisan inertia:middleware
```

This will create a new middleware called `HandleInertiaRequests` in the `app/Http/Middleware` directory.


Next lets add the middleware to the `web` middleware group in the `bootstrap/app.php` file.

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

// ----------------- Add the following -----------------
use App\Http\Middleware\HandleInertiaRequests;
// --------------------------------------------------

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
      // ------------- Also add the following -------------
      $middleware->web(append: [
        HandleInertiaRequests::class,
      ]);
      // --------------------------------------------------
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```
{: file='bootstrap/app.php'}

This will add the `HandleInertiaRequests` middleware to the `web` middleware group.

*Why do we need this? Well this is the middleware that you will be using to provided data that is available to all your Inertia.js pages. Such as the username always visible in your navigationbar. You can read more about it at [inertiajs.com/shared-data](https://inertiajs.com/shared-data).*

### Step 4: Setup Inertia.js Client-side
Now that our server-side setup is complete, let's set up the client-side. We will be using Vite.js as our build tool for this project.

First, let's install the necessary plugins.
```bash
sail npm install @vitejs/plugin-vue
```

Next lets update `vite.config.js` with the following changes.
```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

// ----------------- Add the following --------------
import vue from '@vitejs/plugin-vue';
// --------------------------------------------------

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        // ----------------- Add the following -----------
        vue({
          template: {
            transformAssetUrls: {
              base: null,
              includeAbsolute: false,
            },
          },
        }),
        // -----------------------------------------------
    ],
    // ----------------- Add the following ---------------
    server: {
      hmr: {
        host: 'localhost',
      },
      watch: {
        usePolling: false,
      },
    },
    // ---------------------------------------------------
});
```
{: file='vite.config.js'}



### Step 5: Create our main Vue App



Install the Inertia.js client-side adapter.
```bash
sail npm install @inertiajs/vue3
```


Add the following code to the `resources/js/app.js` file.
```js
import './bootstrap'; // Already exists in the file

// ----------------- Add the following -----------------
import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
    return pages[`./Pages/${name}.vue`]
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
// --------------------------------------------------
```
{: file='resources/js/app.js'}

That is it! You have now successfully set up Inertia.js in your Laravel application. You can now start building your application using Inertia.js.


### Step 6: Creating our very first Inertia.js page

Now that we have Inertia.js set up, let's create our first page. We will create a new page called `Dashboard` in the `resources/js/Pages` directory.

```bash
mkdir -p resources/js/Pages
touch resources/js/Pages/Dashboard.vue
```

```html
<script setup>
import { defineProps } from 'vue'

// Properties provided by the controller
defineProps({
  message: String
})

</script>

<template>
    <div>
        <h1>Dashboard</h1>
        <p>{% raw %} {{ message }} {% endraw %}</p>
    </div>
</template>
```
{: file='resources/js/Pages/Dashboard.vue'}

Next, let's create a new controller called `DashboardController`. This will create a new controller in the `app/Http/Controllers` directory named `DashboardController`.

```bash
sail php artisan make:controller DashboardController
```

Lets add a new method called `index` to the `DashboardController`.

```php
<?php

namespace App\Http\Controllers;

class DashboardController extends Controller
{
    // Add the following method
    public function index()
    {
        return inertia('Dashboard', [
            'message' => 'Hello, World!' // This will be passed to the defined props in the Vue component
        ]);
    }
}
```
{: file='app/Http/Controllers/DashboardController.php'}

Now that we have our very first controller, let's add a new route to our `routes/web.php` file so that we can access our new page.

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});


// Add the following line
Route::get('/dashboard', [\App\Http\Controllers\DashboardController::class, 'index']);
```
{: file='routes/web.php'}

This will route all requests to the `/dashboard` URL to the `index` method in the `DashboardController`.

### Step 7: Profit (Accessing the page)

Before we can access the page, we need to start our frontend server. This can be done by running the following command in the terminal.

```bash
sail npm run dev
```

Lets test if everything is working by visiting [http://localhost:8080/dashboard](http://localhost:8080/dashboard) in your browser.

You should now see our very first Inertia.js page with the message `Hello, World!` provided by our controller. How cool is that?

![Dashboard](assets/img/setting-up-the-perfect-laravel-stack/dashboard-without-tailwindcss.png)


You have now successfully set up Inertia.js in your Laravel application. You can now start building your application using Inertia.js, and the best part is that you can do it all in VSCode!


## Installing Tailwind CSS

Now that we have Inertia.js set up, let's install Tailwind CSS.

### Step 1: Installing dependencies

Lets install all the necessary dependencies for Tailwind CSS.

```bash
sail npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```


### Step 2: Create tailwind.config.js and postcss.config.js

Next lets initialize Tailwind CSS by running the following command.

```bash
sail npx tailwindcss init
```


This will create a new `tailwind.config.js` file in the root of our project directory. Lets start by configuring that.

Override the whole file with the following:
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
    './storage/framework/views/*.php',
    './resources/views/**/*.blade.php',
    './resources/js/**/*.vue',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
{: file='tailwind.config.js'}

Next, lets create a file named `postcss.config.js` file and paste the following:

```bash
code postcss.config.js
```

```js
export default {
  plugins: {
      tailwindcss: {},
      autoprefixer: {},
  },
};
```
{: file='postcss.config.js'}


Next, lets update our `resources/css/app.css` file.

Add the following three lines to the top of the file.
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
{: file='resources/css/app.css'}


Finally, lets make sure that our `app.css` file is importing the `resources/js/app.js` file.

Add the following line to the top of the file.
```js
import '../css/app.css' // Add this line

import './bootstrap';

import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
    return pages[`./Pages/${name}.vue`]
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```
{: file='resources/js/app.js'}


### Step 3: Adding Tailwind CSS to our Vue components
That's it! You now have Tailwind CSS set up in your Laravel application. You can now start using Tailwind CSS classes in your Vue components.

Let's test it by adding some Tailwind CSS classes to our `Dashboard.vue` file.


```vue
... // Existing code
<template>
    <div>
        <!-- Add tailwind class to our header -->
        <h1 class="bg-red-500">Dashboard</h1>
        <p>{% raw %} {{ message }} {% endraw %}</p>
    </div>
</template>
```
{: file='resources/js/Pages/Dashboard.vue'}


### Step 4: Restarting the frontend server

Before we can see the changes, we need to restart our frontend server. This can be done by restarting the `npm run dev` command in the terminal.

```bash
sail npm run dev
```

### Step 5: Profit (Accessing the page)

There we go! You have now successfully set up Tailwind CSS in your Laravel application. You can now start using Tailwind CSS classes in your Vue components.

![Tailwind CSS](assets/img/setting-up-the-perfect-laravel-stack/dashboard-with-tailwindcss.png)

## Installing Ziggy


### Step 1: Installing dependencies


First lets install Ziggy
```bash
sail composer require tightenco/ziggy
```


### Step 2: Make route() available globally
Since we will be using Ziggy's `route()` function A LOT, lets make it accessible everywhere.

We do this by adding the Blade directive `@routes` to our `app.blade.php` file within `<head>`.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    @routes
    @vite('resources/js/app.js')
    @inertiaHead
  </head>
  ...
</html>
```
{: file='resources/views/app.blade.php'}

### Step 3: Make `ziggy-js` resolveable

Before we can actually import `ziggy-js` in our frontend we need to tell the system where to locate it

```js
import path from 'path';      // <--- Add this

export default defineConfig({
    ...
    // Add the following
    resolve: {
        alias: {
            'ziggy-js': path.resolve('vendor/tightenco/ziggy'),
        },
    },
    ...
});
```
{: file='vite.config.js'}

This tells the system `ziggy-js` can be found in our `vendor/tightenco/ziggy` directory, which was created previously when we installed the composer package: `tightenco/ziggy`.
The neat thing about that is that we don't need to keep any extra node packages updated. Pretty cool right?


### Step 4: Include Ziggy vue pligin

Next we need to update our `resources/js/app.js` to make use of the Vue plugin which Ziggy provides out of the box.

```js
import './bootstrap';
import '../css/app.css'

import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'

import { ZiggyVue } from 'ziggy-js';        // <--- Import ZiggyVue Plugin

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
    return pages[`./Pages/${name}.vue`]
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .use(ZiggyVue)                         // <--- Tell Vue to use it
      .mount(el)
  },
})
```


### Step 5: Example

Well done, you have added Ziggy to your perfect Laravel project. Lets quickly dive into how we can use it!

Ziggy lets us do awesome stuff such as easily generate urls to other parts of our application. By using `route()` with the name of a route it will return the complete url.

First we need to make sure we have a route to test with. Lets add a name to our `/dashboard` route. To do this navigate to `routes/web.php`.

This is easily done by adding `->name('dashboard')` to our route entry.

```php
Route::get('/dashboard', [\App\Http\Controllers\DashboardController::class, 'index']);                    # Change from this
Route::get('/dashboard', [\App\Http\Controllers\DashboardController::class, 'index'])->name('dashboard'); # To this
```
{: file='routes/web.php'}

We can now generate urls to this route by simply using `route('dashboard')`.

```vue
<template>
  <a :href="route('dashboard')">Take me to the dashboard!</a>
</template>
```

To read more about how to use Ziggy, go to the official documentation here: [github.com/tighten/ziggy](https://github.com/tighten/ziggy?tab=readme-ov-file#route-function).

## Adding Debugging Support

Obviously, you will run into errors while developing your application. To make it easier to debug these errors, we can add Xdebug support in our project. This will allow us to set breakpoints and step through our code to find the root cause of the issue.

I got to admit, setting up Xdebug can be a bit tricky. My team and I have previously spent hours trying to get it to work, and it kept forgetting how we did it... So hopefully this guide will save you some time and effort and frustration.

### Step 1: Update our `docker-compose.yml` file
We need to configure our `laravel.test` service to connect to our Xdebug server, running within our DevContainer. To do this we need to modify the `XDEBUG_CONFIG` environment variable in our `docker-compose.yml` file.

```yml
services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.3
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.3/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'

            # -------- Change from this --------
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
            # -------- To this --------
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal client_port=9003 start_with_request=default idekey=VSCODE}'

        ... # Other configurations
```
{: file='docker-compose.yml'}

### Step 2: Create a `launch.json` file

Next, we need to create a `./vscode/launch.json` file which will tell VSCode how to start our Xdebug server.

```jsonc
{
  "version": "0.2.0",
  "configurations": [
      {
          "name": "🪲 Listen for Xdebug",
          "type": "php",
          "request": "launch",
          "log":false,
          "port": 9003,
          "pathMappings": {
              // The path where the project files are mounted within the containers running xdebug.
              // we need to tell xdebug that the files are located in a different path
              // than within the vscode workspace
              "/var/www/html": "${workspaceFolder}"
          }
      }
  ]
}
```
{: file='.vscode/launch.json'}

### Step 3: Update our .env file

Finally, we need to update our `.env` file to enable Xdebug support.
This can be done by adding the following line to the `.env` file.

This line tells XDebug to enable the `develop`, `debug`, `trace`, and `coverage` modes.

```bash
echo "SAIL_XDEBUG_MODE=\"develop,debug,trace,coverage\"" >> .env
```

I would also highly suggest that you add this to your `.env.example` file, so when other developers copy the .env.example file to .env they will also have Xdebug enabled by default. This will save you the poke on your shoulder asking why Xdebug is not working, so do yourself a favor and add it to the `.env.example` file.

```bash
# OPTIONAL
echo "SAIL_XDEBUG_MODE=\"develop,debug,trace,coverage\"" >> .env.example
```

### Step 4: Restart sail

Finally, restart the development server by running the following command in the terminal.

```bash
sail down
sail up
```


### Step 5: Start debugging

Press `F5` in VSCode to start the Xdebug server. You can now set breakpoints and step through your code to find the root cause of the issue.

Navigate to your `DashboardController` within `app/Http/Controllers` and set a breakpoint on the `index` method. Like so:

![Xdebug](assets/img/setting-up-the-perfect-laravel-stack/dashboard-controller-break-point.png)

Now visit [http://localhost:8080/dashboard](http://localhost:8080/dashboard) in your browser.

Success! You should see that the execution stops at the breakpoint you set in the `index` method.

![Xdebug](assets/img/setting-up-the-perfect-laravel-stack/dashboard-controller-break-point-triggered.png)





## The cherry on top

You have now successfully set up a VILT stack in VSCode using DevContainer.


### Creating a bootstrap script

When working with laravel and Inertia.js you will quickly forget to run the `npm run dev` command after you have ran `sail up`. To make this easier lets create a bootstrap script that will run both the commands.

Also while we are at it, lets also run a few more commands that we will be using frequently to make sure that our development environment is as consistent as possible.

Create a new new directory called `scripts` and add a script named `bootstrap.sh`.

```bash
code scripts/bootstrap.sh
```

```bash
#!/bin/bash
#
# ------- bootstrap.sh
# This script bootstraps the project and all its dependencies, including:
# Sail, NPM, databases and other external services.
#
# You can use it to initialize, reboot or otherwise restart the stack.
# -------

# Set sail path to variable
sail=vendor/bin/sail

# Determine which npm command to run later.
npm_command="${1:-dev}"

# Navigate to the script dir, basically using this as a baseline location.
pushd $(dirname "$0") >/dev/null

# Navigate to the root folder of the project, in order for sail to run succesfully.
pushd ../ >/dev/null

# Install sail
if [ ! -f $sail ]; then
    docker run --rm -u 1000:1000 -v ${PWD}:/app -w /app laravelsail/php83-composer composer install && npm install
fi

# Stop any running sail instances and remove volumes
$sail down --volumes

# Start sail in detached mode
$sail up --detach

# Update PHP dependencies
$sail composer install

# Install node modules
$sail npm install

# Run migrations and seed the database
$sail artisan migrate:fresh --seed

# BEGIN: scripts that we think fixes unimaginable errors.
# Make sure that cache is cleared and reset.
$sail artisan optimize:clear
# END: scripts that we think fixes unimaginable errors.

# Compile assets
$sail npm run $npm_command
```


Lastly, make sure that the script is executable by running the following command.

```bash
chmod +x scripts/bootstrap.sh
```

Now all you have to do after you have started your DevContainer is to run the following command in the terminal.

```bash
./scripts/bootstrap.sh
```

This will start the sail server, run the migrations, seed the database, and compile the assets all in one go. This will save you a lot of time and effort in setting up your development environment.

After this you are all set to start building your application. You can now start building your application using the VILT stack in VSCode.



## Last Words

Congratulations! You have successfully set up a VILT stack in VSCode using DevContainer. You can now start building your application using the VILT stack in VSCode. This setup will save you a lot of time and effort in setting up your development environment. You can now focus on building your application and not worry about setting up your development environment.

The result of this guide can be found in this [GitHub repository](https://github.com/RasmusGodske/laravel-vilt-stack-template) - feel free to clone it and start building your application.

I hope you found this guide helpful. If you have any questions or feedback, feel free to reach out to me either on my [LinkedIn](https://www.linkedin.com/in/rasmusgodske/) or in the comment section below. I would love to hear your thoughts on this guide!


## What's next?

- Fancy adding typescript support? If yes, you can find a guide here: [here](/posts/perfecting-laravel-typescript-for-better-bug-free-code).

**Happy coding!**



**Changelog**:
- Added a section on creating a bootstrap script to streamline the setup process
- Updated `.devcontainer.json` with additional extensions
- Added a section on setting up Xdebug for debugging purposes
- Added Last Words section
- Added link to github repository
- Added a a section about installing Ziggy