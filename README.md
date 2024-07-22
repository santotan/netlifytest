# Netlify deployment test

Using Node version `18.18.0`.

## Setup

```shell
yarn
```

## Test in local development

```shell
cd packages/client
yarn netlify:dev
```

Note the terminal log:
> Loaded edge function hello

Go to `/test` route on the website to see "Hello world" from edge function.

## Deploy

```shell
cd packages/client
yarn netlify:deploy
```

Note the terminal log:
> Packaging Edge Functions from netlify/edge-functions directory:
> - hello

Note that a directory was created at path `packages/client/packages` that contains 
Netlify client build files.

Note that deployed website does not have `/test` route, and Netlify dashboard 
says no edge functions were deployed.

# Original repository setup

## Workspaces setup

Set up `package.json` with `private: true` and `workspaces`:

```text
  "private": true,
  "workspaces": [
    "packages/*"
  ],
```

## Expo package setup

```shell
mkdir packages
cd packages
yarn create expo client
rm -rf node_modules
cd ../..
yarn
```

Create `packages/client/.env` with contents:

```text
EXPO_USE_METRO_WORKSPACE_ROOT=1
```

Create `packages/client/metro.config.js` with:

```javascript
const { getDefaultConfig } = require('expo/metro-config');
const path = require('path');

// Find the project and workspace directories
const projectRoot = __dirname;
// This can be replaced with `find-yarn-workspace-root`
const monorepoRoot = path.resolve(projectRoot, '../..');

const config = getDefaultConfig(projectRoot);

// 1. Watch all files within the monorepo
config.watchFolders = [monorepoRoot];
// 2. Let Metro know where to resolve packages and in what order
config.resolver.nodeModulesPaths = [
    path.resolve(projectRoot, 'node_modules'),
    path.resolve(monorepoRoot, 'node_modules'),
];

module.exports = config;

```

Add `package/client/index.js` with:

```javascript
import { registerRootComponent } from 'expo';

import App from './App';

// registerRootComponent calls AppRegistry.registerComponent('main', () => App);
// It also ensures that whether you load the app in Expo Go or in a native build,
// the environment is set up appropriately
registerRootComponent(App);

```

## Netlify setup

Add `packages/client/netlify/edge-functions/hello.js` with:

```javascript
export default () => new Response('Hello world');

export const config = { path: '/test' };

```

Create `packages/client/netlify.toml`:

```text
[build]
publish = "dist"
command = "expo export -p web --clear"
```
