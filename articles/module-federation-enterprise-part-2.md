---
title: Module Federation for Enterprise (Part 2)
published: true
description: A guide for supporting a fully dynamic, multi-environment Module Federation configuration
tags: modulefederation, webpack, microfrontends, javascript
series: Module Federation for Enterprise
cover_image: https://miro.medium.com/max/1400/0*cdV9cuy_k9J2_kiO.png
---

# Module Federation for Enterprise (Part 2)

[Part 1](https://dev.to/waldronmatt/tutorial-a-guide-to-module-federation-for-enterprise-n5) in my series **Module Federation for Enterprise** covered an approach to support multiple testing environments and eliminate hard-coded URLs from webpack configurations.

Despite the benefits, the approach we covered introduces a lot of complexity. It requires a lot of scaffolding, knowledge of Webpack internals, and an understanding of the project that is counterintuitive to more streamlined approaches.

But what if there was an easier way? Luckily, there is! In this guide, we'll use a new technique to simplify a multi-environment module federation setup.

## Before We Begin

It is important to mention current options first that might suit your requirements better:

Module Federation supports promise-based logic to infer the remotes at runtime. This is useful for dynamically inferring the path and versioned remotes. You can see an example in the [Webpack documentation](https://webpack.js.org/concepts/module-federation/#promise-based-dynamic-remotes) and additionally via [Zack Jackson's comment](https://dev.to/scriptedalchemy/comment/1i1h1) on my blog post.

Another common approach is to use `.env` variables and/or plugins like `EnvironmentPlugin` for different environments. This is a great option for many use-cases. An example of dynamic remotes with runtime environment variables can be found on the [Module Federation examples repository](https://github.com/module-federation/module-federation-examples/tree/master/advanced-api/dynamic-remotes-runtime-environment-variables#module-federation-dynamic-remotes-with-runtime-environment-variables).

## High Level Overview

This guide will use Webpack's [NormalModuleReplacementPlugin](https://webpack.js.org/plugins/normal-module-replacement-plugin/) to infer the remote URL at runtime for multiple environments. We'll cover what this plugin is, the advantages of using it, and go over code to help us set up our remotes at runtime.

**Note**: This guide will include only essential code snippets relevant to the discussion of this approach. A complete working example of what we'll cover can be found in my **[module-federation-template repository](https://github.com/waldronmatt/module-federation-template)**.

## Project Structure

Here is a basic overview of our project structure:

The `public/` and `src/` directories will hold our code and assets. The `configs/` directory will hold our webpack configurations. And lastly, the `environments/` directory will hold our module federation URLs. More on that below.

```
| Module Federation Monorepo
| -----------
| packages/
|   host/
|       configs/
|       environments/
|       public/
|       src/
|           bootstrap.js
|           app.js
|           init-remote.js
|   remote/
|       configs/
|       public/
|       src/
|           bootstrap.js
|           app.js
```

## The `NormalModuleReplacementPlugin`

According to the Webpack docs:

> The NormalModuleReplacementPlugin allows you to replace resources that match resourceRegExp with newResource. If newResource is relative, it is resolved relative to the previous resource. If newResource is a function, it is expected to overwrite the request attribute of the supplied resource.

In other words, we can replace a module with another module if our regex condition matches. In our case, we want to use this plugin to swap files dependent on a webpack variable we supply to our script in `package.json`.

First we'll create two files in the `environments/` directory called `dev` and `prod`. `FormApp` is the scope of our remote app and the value is the remote URL.

`dev.js`

```js
export default {
  FormApp: "http://localhost:3001",
};
```

`prod.js`

```js
export default {
  FormApp: "https://module-federation-template-remote.netlify.app",
};
```

Next, we'll create a file called `init-remotes.js` under `src/` with the following import and log out the URL of `FormApp`:

`init-remotes.js`

```js
import config from "../environments/TARGET_ENV";

console.log(config.FormApp);
```

Next, we'll pass in a Webpack variable called `TARGET_ENV` and assign it a value that matches the same name of the files we created in the previous step (`TARGET_ENV=dev` and `TARGET_ENV=prod`):

`package.json`

```json
  "scripts": {
    "dev": "webpack serve --env development TARGET_ENV=dev --config config/webpack.dev.js",
    "build": "webpack --env production TARGET_ENV=prod --config config/webpack.prod.js"
  },
```

In our Webpack configuration, we can access this variable and assign a default fallback value like so:

`webpack.common.js`

```js
const targetEnv = env.TARGET_ENV || "prod";
```

Lastly, we'll import and use `NormalModuleReplacementPlugin` to use the appropriate environment module.

`webpack.common.js`

```js
const webpack = require("webpack");

const commonConfig = (isProduction, env) => {
  const targetEnv = env.TARGET_ENV || "prod";

  return merge([
    {
      plugins: [
        new webpack.NormalModuleReplacementPlugin(
          /(.*)TARGET_ENV(\.*)/,
          (resource) => {
            resource.request = resource.request.replace(
              /TARGET_ENV/,
              `${targetEnv}`
            );
          }
        ),
      ],
    },
  ]);
};

module.exports = commonConfig;
```

What we're doing here is using regex to match `TARGET_ENV`. During runtime, `NormalModuleReplacementPlugin` will match this resource via the import `import config from '../environments/TARGET_ENV';` we added in `init-remotes.js` and will replace the `TARGET_ENV` portion of the import with `targetEnv` which is the value of the environment variable we passed in via our scripts in `package.json`.

For example, for development builds at runtime, the import in `init-remote.js` will become:

```js
import config from "../environments/dev";
```

And will successfully log the `dev` URL:

```js
http://localhost:3001
```

That's pretty cool! With this method we can easily add in more remotes and add additional files to support more testing environments. I personally like this method over `.env` files and associated plugins. It gives me a clean way to support many environments and reduces conditional logic in my webpack configurations.

## Code for Initializing Dynamic Remotes

Now that we have multi-environment support, we'll need to add in some code to initialize our remotes dynamically for us to use in the host app.

Let's break this down into several steps:

1. Create `<script>` tags that will bootstrap our remotes.
2. Connect to the remote containers dynamically at runtime.
3. Load the default module of our remote.
4. Create a function to handle the previous three steps.

So far we have our environment setup in `init-remote.js`:

```js
import config from "../environments/TARGET_ENV";
```

We'll add code to create the script tags. We resolve a promise if the script tag was successfully created after `onload`.

The `module` parameter is the remote URL that will be passed in like this: `setRemoteScript(config['FormApp'])`. We're using the `config` from our environment import and the remote scope `FormApp` as a key/value pair to retrieve the corresponding URL from `environments/dev.js` file for development builds, etc.

```js
const setRemoteScript = (module) =>
  new Promise((resolve) => {
    const remoteUrlWithVersion = `${module}/remoteEntry.js`;
    const script = document.createElement("script");
    script.src = remoteUrlWithVersion;
    document.head.appendChild(script);
    script.onload = () => resolve();
  }).catch((err) => {
    console.log(err, `Error setting script tag for ${module}.`);
  });
```

**Note**: If you have versioned remotes, you can substitute the `remoteUrlWithVersion` variable with the following code and modify to suit your needs:

```js
const setRemoteScript = (module) =>
  new Promise((resolve) => {
    const urlParams = new URLSearchParams(window.location.search);
    const version = urlParams.get("appVersionParam");
    const remoteUrlWithVersion = `${module}${version}/remoteEntry.js`;
    const script = document.createElement("script");
    script.src = remoteUrlWithVersion;
    document.head.appendChild(script);
    script.onload = () => resolve();
  }).catch((err) => {
    console.log(err, `Error setting script tag for ${module}.`);
  });
```

Next we'll need to dynamically connect to the remote containers. We'll copy/paste the code found on the [Webpack docs](https://webpack.js.org/concepts/module-federation/#dynamic-remote-containers).

The `scope` and `module` will be passed in like so: `loadComponent('FormApp', './initContactForm')`

```js
const loadComponent = (scope, module) => {
  return async () => {
    await __webpack_init_sharing__("default");
    const container = window[scope];
    await container.init(__webpack_share_scopes__.default);
    const factory = await window[scope].get(module);
    const Module = factory();
    return Module;
  };
};
```

Next we'll write a small function to get us the default module of our remote.

The `remote` parameter will be the returned value from `loadComponent`.

```js
const loadModuleFrom = (remote) => {
  remote()
    .then((module) => module.default())
    .catch((err) => {
      console.log(err, `Error loading default module from ${remote}.`);
    });
};
```

And finally, we'll create a function to handle all the steps to get our remotes.

```js
const initRemote = (remoteScope, remoteModule) => {
  setRemoteScript(config[remoteScope])
    .then(() => {
      const loadedComponent = loadComponent(remoteScope, remoteModule);
      loadModuleFrom(loadedComponent);
    })
    .catch((err) => {
      console.log(
        err,
        `Error initializing ${remoteModule} from ${remoteScope}.`
      );
    });
};
```

Altogether our code should look like the following:

`init-remote.js`

```js
import config from "../environments/TARGET_ENV";

const setRemoteScript = (module) =>
  new Promise((resolve) => {
    const remoteUrlWithVersion = `${module}/remoteEntry.js`;
    const script = document.createElement("script");
    script.src = remoteUrlWithVersion;
    document.head.appendChild(script);
    script.onload = () => resolve();
  }).catch((err) => {
    console.log(err, `Error setting script tag for ${module}.`);
  });

const loadComponent = (scope, module) => {
  return async () => {
    await __webpack_init_sharing__("default");
    const container = window[scope];
    await container.init(__webpack_share_scopes__.default);
    const factory = await window[scope].get(module);
    const Module = factory();
    return Module;
  };
};

const loadModuleFrom = (remote) => {
  remote()
    .then((module) => module.default())
    .catch((err) => {
      console.log(err, `Error loading default module from ${remote}.`);
    });
};

const initRemote = (remoteScope, remoteModule) => {
  setRemoteScript(config[remoteScope])
    .then(() => {
      const loadedComponent = loadComponent(remoteScope, remoteModule);
      loadModuleFrom(loadedComponent);
    })
    .catch((err) => {
      console.log(
        err,
        `Error initializing ${remoteModule} from ${remoteScope}.`
      );
    });
};

export default () => initRemote;
```

Now it's time to start using our remote app! In `app.js`, we can use the following code to lazy load it:

`app.js`

```js
import(/* webpackChunkName: "FormApp" */ "./init-remote")
  .then((module) => {
    const initRemote = module.default();
    initRemote("FormApp", "./initContactForm");
  })
  .catch((err) => {
    // eslint-disable-next-line no-console
    console.log(err, "Error initializing lazy loaded remote.");
  });
```

We're loading the default module of `init-remote.js` which is `initRemote`; the function we created to handle all the steps for loading our remotes. Supply the remote scope name and the remote module key value to `initRemote` and now you're all set to start using the remote app!

A complete working example of this setup can be found on my **[module-federation-template repository](https://github.com/waldronmatt/module-federation-template)**.

Details on my personal setup:

- In `remote`, I'm using the Module Federation Live Reloading Plugin to hot reload my remote app via `@module-federation/fmr`.
- In `host`, I'm using the Automatic Vendor Federation Plugin to easily manage shared dependencies via `@module-federation/automatic-vendor-federation`.

Important configuration notes on my personal setup:

- Use an asynchronous boundary to load all your code by dynamically importing your `app.js` in a separate file and then reference that file as your main entrypoint.
- If you're using Webpack `SplitChunks`, make sure your chunks are set to `async` and your `cacheGroups` names are unique between host and remote(s).

## Conclusion

Thank you for reading! I hope you find this approach of supporting multiple environments for Module Federation useful.

To summarize, we used `NormalModuleReplacementPlugin` to load in the appropriate environment configuration file at runtime. We also added code to dynamically load our remotes to use in our host app.

Advantages:

- `NormalModuleReplacementPlugin` helps reduce conditional logic over methods using `.env` files for setups with many environments
- `NormalModuleReplacementPlugin` is built into the Webpack core library, so no reliance on third-party plugins
- Can be easily integrated into your build pipeline
- Project can be easily scaled to support multiple remotes and environments
- Project setup is less complex than the original implementation I proposed in Part 1 of this blog series

A complete working example of this setup can be found on my **[module-federation-template repository](https://github.com/waldronmatt/module-federation-template)**.

Please like and share if you found this article useful. Also please leave a comment with your thoughts and what your favorite approach is for setting up your Module Federation projects!
