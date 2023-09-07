# Development

This document aims to provide as much context as possible to aid in the development of examples in in this monorepo.

## Getting Started

Please refer to [the Getting Started section of the `README.md`](README.md#getting-started) for a general-purpose guide on getting started. The rest of this document will assume that you've installed all of the prequisites and setup described there.

## WordPress Local Development Environment

This project recommends the use the [`@wordpress/env`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/) package to get a local development environment. 

> **Warning**
> Having  [Docker](https://docs.docker.com/get-docker/) installed is one of the [prerequisites of `wp-env`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/#prerequisites).

To start the local WordPress instance with one of two commands:

1. `npm run env:start` - Starts the instance normally.
2. `npm run env:start:debug` - Starts the instance with debugging enabled.

> **Note**
> See ["Quick and easy local WordPress development with wp-env"](https://developer.wordpress.org/news/2023/03/quick-and-easy-local-wordpress-development-with-wp-env/) and [`wp-env` package reference](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/) to learn more about `wp-env`

The WordPress instance will be available at http://localhost:8888/. You can login with the username and password `admin` and the password `password` at http://localhost:8888/wp-login.php. 

<details>
  <summary>The plugins at <code>.wp-env.json</code> should be automatically activated.</summary>
<br>  
<em>You can edit the property <code>"plugins"</code> at <code>.wp-env.json</code> to include just the examples you're interested in. To apply these changes after having started your instance, run <code>npm run env:update</code> from the root folder</em>
</details>
<br>  
To stop this local WordPress instance later run:

```
npm run env:stop
```

<details>
  <summary>If you get some errors you can try running <code>npm run env:restart</code></summary>
<br>  
<em>If you continue having errors go to Docker, remove all containers and then run <code>npm run env:start</code></em>
</details>


## Build process

<p><em>ESNext and JSX Syntax are not directly supported by browsers. The examples that make use of this (recommended) syntax require a <a href="https://developer.wordpress.org/block-editor/how-to-guides/javascript/js-build-setup/">build process</a> to get a regular JavaScript version that browsers can understand.</em></p>


> ESNext is a dynamic name that refers to whatever the next version is at the time of writing. ESNext features are more correctly called proposals, because, by definition, the specification has not been finalized yet.

> https://legacy.reactjs.org/docs/faq-build.html
> https://legacy.reactjs.org/docs/react-without-es6.html
> https://legacy.reactjs.org/docs/react-without-jsx.html
> https://react.dev/reference/react/createElement#creating-an-element-without-jsx

## Turborepo Commands

Our repository uses [Turborepo](https://turborepo.org) for `build` and `test` commands. This tool ensures that all dependencies of a plugin, package, or tool are prepared before running a command. This is done transparently when running these commands. When using `pnpm run {command}` without any options, it will execute that command against every project in the repository. You can view a list of the commands Turborepo supports in [our turbo.json file](turbo.json).

### Plugin, Package, and Tool Filtering

If you are interested in running a `turbo` command against a single plugin, package, or tool, you can do so with the `--filter` flag. This flag supports the `"name"` option in `package.json` files, paths, and globs.

If you would like to read more about the syntax, please check out [the Turborepo filtering documentation](https://turborepo.org/docs/core-concepts/filtering).

### Examples

Here are some examples of the ways you can use Turborepo / pnpm commands:

```bash
# Lint and build all plugins, packages, and tools. Note the use of `-r` for lint,
# turbo does not run the lint at this time.
pnpm run -r lint && pnpm run build

# Build WooCommerce Core and all of its dependencies
pnpm run --filter='woocommerce' build

# Lint the @woocommerce/components package - note the different argument order, turbo scripts
# are not running lints at this point in time.
pnpm run -r --filter='@woocommerce/components' lint

# Test all of the @woocommerce scoped packages
pnpm run --filter='@woocommerce/*' test

# Build all of the JavaScript packages
pnpm run --filter='./packages/js/*' build

# Build everything except WooCommerce Core
pnpm run --filter='!woocommerce' build

# Build everything that has changed since the last commit
pnpm run --filter='[HEAD^1]' build
```

### Cache busting Turbo

In the event that you need to force turbo not to cache a command you can set the env variable `TURBO_FORCE=true`.

e.g.

```bash
# Force an uncached build of WooCommerce Core and all of its dependencies
TURBO_FORCE=true pnpm run --filter='woocommerce' build
```

## Other Commands

Outside of the commands in [our turbo.json file](turbo.json), each plugin, package, and tool may have unique scripts in their `package.json` files. In these cases, you can execute those commands using `pnpm {script}` and the same `--filter` syntax as Turborepo.

### Examples

Here are some examples of the commands you will make use of.

```bash
# Add a changelog entry for WooCommerce Core
pnpm --filter=woocommerce run changelog add

# Create the woocommerce.zip file
pnpm --filter=woocommerce run build:zip
```

## Plugin Development Environments

The plugins in our repository make use of [the `@wordpress/env` package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/). This supplies convenient commands for creating, destroying, cleaning, and testing WordPress environments.

```bash
# Make sure you are in the working directory of the plugin you are interested in setting up the environment for
cd plugins/woocommerce
# Start will create the environment if necessary or start an existing one
pnpm -- wp-env start
# Stop will, well, stop the environment
pnpm -- wp-env stop
# Destroy will remove all of the environment's files.
pnpm -- wp-env destroy
```

Each of the [plugins in our repository](plugins) support using this tool to spin up a development environment. Note that rather than having a single top-level environment, each plugin has its own. This is done in order to prevent conflicts between them.

Please check out [the official documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/) if you would like to learn more about this tool.

## Troubleshooting

### Installing PHP in Unix (e.g. Ubuntu)

Many unix systems such as Ubuntu will have PHP already installed. Sometimes without the extra packages you need to run WordPress and this will cause you to run into troubles.

Use your package manager to add the extra PHP packages you'll need.
e.g. in Ubuntu you can run:

```
sudo apt update
sudo apt install php-bcmath \
                 php-curl \
                 php-imagick \
                 php-intl \
                 php-json \
                 php-mbstring \
                 php-mysql \
                 php-xml \
                 php-zip
```