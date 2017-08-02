# heroku-buildpack-ember-cli

**NOTE**: This is not an officially supported Heroku buildpack.  It is used by the Heroku Dashboard and provided as open source for transparency. Use at your own risk.

**heroku-buildpack-ember-cli** enables you to run your Ember CLI application on Heroku. It provides the ability for changing ENV variables on-the-fly without modifying your codebase or redeploying your application. Primary features of this buildpack include:
+ builds the Ember application for production
+ automatically provides a properly configured `static.json` file to drive [heroku-buildpack-static](https://github.com/hone/heroku-buildpack-static)
+ provides caching headers for your `/assets` folder
+ regenerates your application's configuration each time an environment variable is changed on Heroku
  + but this doesn't pull in any addons' `environment.js` files. See [this issue](https://github.com/heroku/heroku-buildpack-ember-cli/issues/18)
+ updates your application's asset prefix each time the `CDN_HOST` variable is changed on Heroku

## dependencies
+ If you don't already have the Heroku CLI installed, go ahead and [fetch it from https://toolbelt.heroku.com](https://toolbelt.heroku.com).
+ Your application's config must contain `storeConfigInMeta: true` (which is also the default value).

Rather than duplicating code, heroku-buildpack-ember-cli builds on top of other buildpacks, namely [heroku-buildpack-nodejs](https://github.com/heroku/heroku-buildpack-nodejs) for [node.js](https://nodejs.org/en/) support as well as [heroku-buildpack-static](https://github.com/hone/heroku-buildpack-static) to actually serve the assets to your users. If you need support for private dependencies, check out [heroku-buildpack-github-netrc](https://github.com/timshadel/heroku-buildpack-github-netrc).

Here's a quick cheatsheet for adding the required buildpacks to your application; simply copy and paste the following:
```sh
heroku buildpacks:clear
heroku buildpacks:add https://github.com/heroku/heroku-buildpack-nodejs
heroku buildpacks:add https://github.com/heroku/heroku-buildpack-ember-cli
heroku buildpacks:add https://github.com/hone/heroku-buildpack-static
```

By default, the nodejs buildpack only installed `dependencies` and not `devDependencies`. And by default, Ember CLI puts everything in `devDependencies`. Doh! Fortunately, there's a simple fix:

```
heroku config:set NPM_CONFIG_PRODUCTION=false
```

Be sure to run this _before_ deploying.

### app.json
If your project includes an `app.json` file (used by Heroku for Pipelines and Heroku Button) be sure to update it to reflect the changes above. In particular, you'll want to set the `NPM_CONFIG_PRODUCTION` variable as well as specifying the correct buildpacks

## static.json
This buildpack injects a default [static.json](https://github.com/hone/heroku-buildpack-static#configuration) file into the application slug *unless one is provided by your application*. **Warning** The provided `static.json` sets cache headers on `/assets/**` which is only desirable _if_ you are leveraging [fingerprinting](http://ember-cli.com/asset-compilation/#fingerprinting-and-cdn-urls) in your project!

## Example Application
[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://dashboard.heroku.com/new?template=https://github.com/heroku/heroku-static-ember)

## Bower
For optimal performance, add the following (or similar) to your project's `package.json`:
```js
"postinstall": "bower install"
```

and ensure `bower` is included your `package.json`'s `dependencies`:
```js
"dependencies": {
  "bower": "*"
}
```

This allows the nodejs buildpack to cache `bower_components`  yielding faster build times 🙌
