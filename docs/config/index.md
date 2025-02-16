[installation]: /guide/#installation-💿
[config reference]: https://vitejs.dev/config/
[plugins]: https://vitejs.dev/plugins/
[entrypoints]: /guide/development.html#entrypoints-⤵%EF%B8%8F
[json config]: /config/#shared-configuration-file-%F0%9F%93%84
[sourceCodeDir]: /config/#sourcecodedir
[autoBuild]: /config/#autobuild
[entrypointsDir]: /config/#entrypointsDir
[publicOutputDir]: /config/#publicoutputdir
[additionalEntrypoints]: /config/#additionalentrypoints
[watchAdditionalPaths]: /config/#watchadditionalpaths
[publicDir]: /config/#publicdir
[root]: /config/#root
[alignment with Rails defaults]: https://github.com/rails/webpacker/issues/769
[source maps]: https://vitejs.dev/config/#build-sourcemap
[import aliases]: /guide/development.html#import-aliases-👉
[reference these files]: https://github.com/ElMassimo/vite_ruby/blob/main/vite-plugin-ruby/example/app/frontend/entrypoints/main.ts#L4
[resolve.alias]: https://vitejs.dev/config/#resolve-alias
[tag helpers]: /guide/development.html#tag-helpers-🏷
[vite-plugin-ruby]: https://github.com/ElMassimo/vite_ruby/tree/main/vite-plugin-ruby
[Vite config file]: /config/#configuring-vite-⚡
[runtime env var]: https://github.com/ElMassimo/vite_ruby/discussions/159#discussioncomment-1841817

# Configuring Vite Ruby

The following section is an overview of basic configuration for _Vite Ruby_.

Most of the options discussed are specific to _Vite Ruby_, for the rest of the
available configuration options please check Vite's [config reference].

## Configuring Vite ⚡

When running <kbd>bin/vite</kbd> from the command line, Vite will use your `vite.config.ts`.

If you followed the [installation] section, it should look similar to:

```js
// vite.config.ts
import { defineConfig } from 'vite'
import RubyPlugin from 'vite-plugin-ruby'

export default defineConfig({
  plugins: [
    RubyPlugin(),
  ],
})
```

You can customize this file as needed, check Vite's [plugins] and [config reference] for more info.

## Shared Configuration File 📄

_Vite Ruby_ leverages a simple `config/vite.json` configuration file, which is
read both from Ruby and JavaScript, and allows you to easily configure options
such as the host and port of the Vite development server.

If you followed the [installation] section, it should look similar to:

```json
{
  "all": {
    "watchAdditionalPaths": []
  },
  "development": {
    "autoBuild": true,
    "publicOutputDir": "vite-dev",
    "port": 3036
  },
  "test": {
    "autoBuild": true,
    "publicOutputDir": "vite-test",
    "port": 3037
  }
}
```

The `all` section is applied to all environments, including production. You can override specific options in an environment-specific group.

The following options can all be specified in your `config/vite.json` file, or
overriden with environment variables.

## Ruby Configuration File 💎

To set environment variables that need to be resolved at [__runtime__][runtime env var], use `config/vite.rb`.

This will be loaded once `ViteRuby.config` is resolved, and you may configure
`ViteRuby.env` as needed or call `ViteRuby.reload_with(**options)` to override the configuration.

```ruby
# config/vite.rb
ViteRuby.env['ADMINISTRATOR_ASSETS_PATH'] =
  "#{ Gem.loaded_specs['administrator'].full_gem_path }/app/frontend"
```

Then, you can access those environment variables in `process.env` in the [Vite config file]:

```js
// vite.config.ts
const adminAssetsPath = process.env.ADMINISTRATOR_ASSETS_PATH
```

## Source Maps 🗺

The only notable difference with Vite.js config defaults, is that [source maps]
are enabled in production to be in [alignment with Rails defaults].

You may skip source map generation by explicitly configuring <kbd>[sourcemap][source maps]</kbd>:

```js
// vite.config.ts
export default defineConfig({
  build: { sourcemap: false },
```


## Development Options

### autoBuild

- **Default:** `false` in <kbd>production</kbd>, `true` in <kbd>test</kbd> and <kbd>development</kbd>
- **Env Var:** `VITE_RUBY_AUTO_BUILD`

  When enabled, Vite Ruby will track changes to <kbd>[sourceCodeDir]</kbd>,
  and trigger a Vite build on demand if files have changed. Additional files can be specified with <kbd>[watchAdditionalPaths]</kbd>.

  This is very convenient when running integration tests, or when a developer
  does not want to start the Vite development server (although it's highly recommended).

### host

- **Default:** `"localhost"`
- **Env Var:** `VITE_RUBY_HOST`

  Specify the hostname for the Vite development server.

### port

- **Default:** `3036`
- **Env Var:** `VITE_RUBY_PORT`

  Specify the port for the Vite development server.

### https

- **Type:** `boolean`
- **Env Var:** `VITE_RUBY_HTTPS`

  Enable TLS + HTTP/2 for the Vite development server.

## Build Options

The default configuration expects your code to be in `app/frontend`, and your
entrypoint files to be in the `app/frontend/entrypoints` directory.

The compiled assets will be outputed to the `public/vite/assets` directory.

You can customize this behavior using the following options.

### buildCacheDir

- **Default:** `tmp/cache/vite`
- **Env Var:** `VITE_RUBY_BUILD_CACHE_DIR`

  Specify the directory where the <kbd>[autoBuild]</kbd> cache should be stored, used to
  detect if a build is needed when the development server is not running.

### publicDir

- **Default:** `public`
- **Env Var:** `VITE_RUBY_PUBLIC_DIR`

  Specify the public directory (relative to <kbd>[root]</kbd>).

  It's expected for this directory to be served by Apache/NGINX, or loaded into a CDN.

  Check Rails `public_file_server.enabled` for more information.

### publicOutputDir

- **Default:** `vite`
- **Env Var:** `VITE_RUBY_PUBLIC_OUTPUT_DIR`

  Specify the output directory (relative to <kbd>[publicDir]</kbd>).

### assetsDir

- **Default:** `assets`
- **Env Var:** `VITE_RUBY_ASSETS_DIR`

  Specify the directory to nest generated assets under (relative to <kbd>[publicOutputDir]</kbd>).

### sourceCodeDir

- **Default:** `app/frontend`
- **Env Var:** `VITE_RUBY_SOURCE_CODE_DIR`

  Specify the directory where your source code will be located (relative to <kbd>[root]</kbd>).

  Vite Ruby will alias this directory as `~/` and `@/`, allowing you to make absolute imports which are more convenient.

  Files in this directory will be tracked for changes when using <kbd>[autoBuild]</kbd>.

### entrypointsDir

- **Default:** `entrypoints`
- **Env Var:** `VITE_RUBY_ENTRYPOINTS_DIR`

  Specify the directory where the [entrypoints] will be located (relative to <kbd>[sourceCodeDir]</kbd>).

## Other Options

### additionalEntrypoints

- **Version Added:** `3.0.0`
- **Default:** `["~/{assets,fonts,icons,images}/**/*"]`

  Specify additional [entrypoints], which can be referenced using [tag helpers]
  without having to place them inside <kbd>[entrypointsDir]</kbd>.

  `~/` is an alias for the <kbd>[sourceCodeDir]</kbd>, `["~/images/**/*"]`:

  ```ruby
  vite_asset_path 'images/logo.svg' # app/frontend/images/logo.svg
  ```

  Otherwise, globs are relative to <kbd>[root]</kbd>, such as `["app/components/**/*.js"]`.

  ```ruby
  vite_asset_path '/app/components/header.js' # leading slash is required
  ```

### assetHost

- **Default:** `Rails.application.config.action_controller.asset_host`
- **Env Var:** `VITE_RUBY_ASSET_HOST`

  Specify the asset host when using a CDN. Usually there's no need to explicitly configure it.

### base

- **Default:** `''`
- **Env Var:** `VITE_RUBY_BASE`

  Necessary when you are hosting your Ruby app in a nested path. Example:

  ```json
  "base": "/nested_path"
  ```

### configPath

- **Default:** `config/vite.json`
- **Env Var:** `VITE_RUBY_CONFIG_PATH`

  Specify where the [JSON config] file is located (relative to <kbd>[root]</kbd>).

  Not supported in the JSON file.

### devServerConnectTimeout

- **Default:** `0.01` (seconds)

  Timeout when attempting to connect to the dev server (in seconds).

  You can increase this timeout if the fallback compilation is being triggered
  even though the dev server is running.

### hideBuildConsoleOutput

- **Default:** `false`
- **Env Var:** `VITE_RUBY_HIDE_BUILD_CONSOLE_OUTPUT`

  Allows to skip Vite build output from logs, to keep the noise down.

### root

- **Default:** `Rails.root`
- **Env Var:** `VITE_RUBY_ROOT`

  Specify the project root.

### skipCompatibilityCheck

- **Version Added:** `3.0.0`
- **Default:** `false`
- **Env Var:** `VITE_RUBY_SKIP_COMPATIBILITY_CHECK`

  The version of <kbd>[vite-plugin-ruby]</kbd> is checked on initialization to
  fail early if an incompatible version is suspected, as it could lead to
  hard-to-debug errors such as missing entrypoints in the build.

  Running <kbd>bin/vite upgrade</kbd> will install a compatible
  version, which should resolve this error.

  Otherwise, this setting allows to skip that check. Use it responsibly.

### viteBinPath

- **Default:** `node_modules/.bin/vite`
- **Env Var:** `VITE_RUBY_VITE_BIN_PATH`

  The path where the Vite.js binary is installed. It will be used to execute the `dev` and `build` commands.

### watchAdditionalPaths

- **Default:** `[]`

  Specify which other paths should be tracked for changes when using <kbd>[autoBuild]</kbd>.

  You may provide globs such as `["app/components/**/*"]`, paths should be relative to <kbd>[root]</kbd>.

  The <kbd>[sourceCodeDir]</kbd> and any <kbd>[additionalEntrypoints]</kbd> are included by default.

  <hr/>

  In order to [reference these files] it's highly recommended to [define][resolve.alias] your own [import aliases]:

  ```js{7}
  import { resolve } from 'path'
  import { defineConfig } from 'vite'

  export default defineConfig({
    resolve: {
      alias: {
        '@assets': resolve(__dirname, 'app/assets'),
      },
    },
  })
  ```

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
