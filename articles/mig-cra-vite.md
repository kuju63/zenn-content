---
title: "Migrate Create-React-App to Vite"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cra", "vite", "react"]
published: true
---

## Motivation

1. Fix vulnerabilities

    Create-React-App is not maintained and has some issues including vulnerabilities. For example, webpack 5.64.4 has the vulnerabilities of ReDos([CVE-2022-37603](https://github.com/advisories/GHSA-3rfm-jhwj-7488)).

    I expect to fix these vulnerabilities by migrating to Vite.

2. Improve Development Experience

    Create-React-App is too slowly in dev mode, because it uses webpack and more unused loaders.

3. Improve Production build time

    The big project takes a long time to production build(generating static site) on Create-React-App. My project takes 10 minutes over to build. I expect to reduce the build time by migrating to Vite.

## Migration Steps

Migration base sample is this repository. It is a simple SPA with React and Redux / TypeScript using Create-React-App.

@[card](https://github.com/kuju63/crt-vite-sample/tree/main/base)

### Install Vite

First step is to install Vite and add `vite.config.ts` file. Vite is need to add `@vitejs/plugin-react` for bundling React application.

```bash
npm install -D vite @vitejs/plugin-react
```

After installing Vite, create `vite.config.ts` file in the root directory and create `vite-env.d.ts`.

https://github.com/kuju63/crt-vite-sample/blob/main/vite/vite.config.ts

https://github.com/kuju63/crt-vite-sample/blob/main/vite/src/vite-env.d.ts

And update tsconfig for using bundler.

https://github.com/kuju63/crt-vite-sample/blob/main/vite/tsconfig.json

https://github.com/kuju63/crt-vite-sample/blob/main/vite/tsconfig.node.json

Next, move `public/index.html` to the root directory and remove `%PUBLIC_URL%`.

Finally, add build script into `package.json`.

```json
{
  "scripts": {
    "build:vite": "tsc && vite build",
    "preview": "vite preview",
  }
}
```

### Add eslint

Create-React-App has eslint configuration by default. So, I need to add eslint configuration.

```bash
npm install -D @typescript-eslint/eslint-plugin \
    @typescript-eslint/parser eslint-plugin-import \
    eslint-plugin-jest eslint-plugin-jsx-a11y \
    eslint-plugin-react eslint-plugin-react-hooks \
    eslint-plugin-testing-library
```

Add eslint configuration file `.eslintrc.js` in the root directory for improving maintainability and remove `eslint` section on `package.json`.

`.eslintrc.js` is to add many rules for matching Create-React-App's eslint configuration.

https://github.com/kuju63/crt-vite-shttps://github.com/kuju63/crt-vite-sample/blob/main/vite/babel.config.cjsample/blob/main/vite/.eslintrc.js

Add `.eslintignore` file for ignoring build files and node_modules.

https://github.com/kuju63/crt-vite-sample/blob/main/vite/.eslintignore

### Add Jest

As with eslint, Jest is also configured in Create-React-App. The settings are need to migrate these too.

First, install Jest and related packages including babel. babel is to need to transform TypeScript to JavaScript.

``` bash
npm install -D @babel/preset-env @babel/preset-react @babel/preset-typescript \
    @types/jest jest react-test-renderer jest-environment-jsdom
```

#### Configure Jest

Jest with TypeScript needs to configure jest using babel.

First, create `jest.config.cjs` file in the root directory.

https://github.com/kuju63/crt-vite-sample/blob/main/vite/jest.config.cjs

Next, create `babel.config.cjs` file in the root directory for transforming TypeScript.

https://github.com/kuju63/crt-vite-sample/blob/main/vite/babel.config.cjs

Next, create mock file for CSS/SCSS/LESS and SVG files.

If you use CSS in js, you need to create `styleMock.js` file in `__mocks__` directory. Create-React-App default configuration can use `import 'style.css'` without any configuration. But Vite needs to configure it.

https://github.com/kuju63/crt-vite-sample/blob/main/vite/__mocks__/styleMock.js

Finally, add `test` script into `package.json`.

```json
{
  "scripts": {
    "test": "jest",
  }
}
```

### Remove Create-React-App

The last step is to remove Create-React-App. And update build script to use Vite.

First, remove `react-scripts` from `package.json`.

```bash
npm unistall react-scripts
```

Finally, update build script to use Vite.

https://github.com/kuju63/crt-vite-sample/blob/f5ee8c21671798b3ce2a8385c189183fa6e8bfe9/vite/package.json#L14-L22
