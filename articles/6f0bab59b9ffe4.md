---
title: "Nuxt3+ESLint+Prettier+TailwindCSS+daisyUI+Vitest+Storybookで環境構築"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nuxt3, tailwindcss, daisyui, vitest, storybook]
published: true
---

# 環境

```zsh
yarn -v
1.22.18
```

# 構築

## Nuxt3

### プロジェクト作成

```zsh
mkdir nuxt-app
cd nuxt-app
```

```zsh
npx nuxi init .
```

```zsh
.
├── .gitignore
├── .npmrc
├── .nuxt
├── README.md
├── app.vue
├── node_modules
├── nuxt.config.ts
├── package.json
├── tsconfig.json
└── yarn.lock
```

## ESLint + Prettier

### インストール

```zsh
yarn add -D typescript eslint prettier eslint-config-prettier eslint-plugin-import eslint-plugin-vue @typescript-eslint/eslint-plugin @typescript-eslint/parser @nuxtjs/eslint-config-typescript
```

### 設定ファイルの作成

```js:.eslintrc.cjs
module.exports = {
  root: true,
  env: {
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
    '@nuxtjs/eslint-config-typescript',
    'prettier',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  },
  plugins: ['vue', '@typescript-eslint'],
  rules: {
    /* typescript */
    'dot-notation': 'off',
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          '../*',
          '~/*',
          '~~/*',
          './assets/*',
          './components/*',
          './pages/*',
          './plugins/*',
          './router/*',
          './composables/*',
          './server/*',
          './store/*',
          './types/*',
          './utils/*',
          './libs/*',
          './*.vue',
        ],
      },
    ],
    'import/order': [
      'error',
      {
        'groups': [
          'builtin',
          'external',
          'parent',
          'sibling',
          'index',
          'object',
          'type',
        ],
        'pathGroups': [
          {
            pattern: '{vue,vue-router,vite,@vitejs/plugin-vue}',
            group: 'builtin',
            position: 'before',
          },
          {
            pattern: '@src/**',
            group: 'parent',
            position: 'before',
          },
        ],
        'pathGroupsExcludedImportTypes': ['builtin'],
        'alphabetize': {
          order: 'asc',
        },
        'newlines-between': 'always',
      },
    ],
    '@typescript-eslint/consistent-type-imports': [
      'error',
      { prefer: 'type-imports' },
    ],

    /* nuxt */
    'vue/multi-word-component-names': 'off',
    'vue/require-v-for-key': 'off',
  },
};
```

```json:.prettierrc
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "quoteProps": "consistent",
  "trailingComma": "es5",
  "vueIndentScriptAndStyle": true
}
```

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .nuxt
├── .prettierrc
├── README.md
├── app.vue
├── node_modules
├── nuxt.config.ts
├── package.json
├── tsconfig.json
└── yarn.lock
```

## サーバの設定

```ts:nuxt.config.ts
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  nitro: {
    preset: 'node',
  },
  devServer: {
    host: '0.0.0.0',
  },
});
```

## tsconfigの設定

### インストール

```zsh
yarn add -D @tsconfig/strictest
```

### tsconfigの編集

:::message
`tsconfig.json`を直接編集するのではなく、`nuxt.config.ts`を編集する
:::

```diff ts:nuxt.config.ts
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  nitro: {
    preset: 'node',
  },
  devServer: {
    host: '0.0.0.0',
  },
+ typescript: {
+   tsConfig: {
+     extends: '@tsconfig/strictest/tsconfig.json',
+     compilerOptions: {
+       noImplicitReturns: false, // For middleware
+     },
+   },
+ },
});
```

## srcDirの変更

```diff ts:nuxt.config.ts
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  nitro: {
    preset: 'node',
  },
  devServer: {
    host: '0.0.0.0',
  },
  typescript: {
    tsConfig: {
      extends: '@tsconfig/strictest/tsconfig.json',
      compilerOptions: {
        noImplicitReturns: false, // For middleware
      },
    },
  },
+ srcDir: 'src',
});
```

### `app.vue`の移動

※これ以降、`node_modules`と`.nuxt`の表記は省略する

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .prettierrc
├── README.md
├── nuxt.config.ts
├── package.json
├── src
│   └── app.vue
├── tsconfig.json
└── yarn.lock
```

## TailwindCSS

### インストール

```zsh
yarn add -D @nuxtjs/tailwindcss
```

### 設定ファイルの作成

```js:tailwind.config.cjs
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.html', './src/**/*.vue', './src/**/*.jsx'],
  /* 以下は個人的な設定 */
  theme: {
    extend: {
      colors: {
        'light-black': '#333333',
      },
    },
  },
};
```

```diff ts:nuxt.config.ts
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  nitro: {
    preset: 'node',
  },
  devServer: {
    host: '0.0.0.0',
  },
  typescript: {
    tsConfig: {
      extends: '@tsconfig/strictest/tsconfig.json',
      compilerOptions: {
        noImplicitReturns: false, // For middleware
      },
    },
  },
  srcDir: 'src',
+ modules: ['@nuxtjs/tailwindcss'],
+ tailwindcss: {
+   exposeConfig: true,
+   configPath: 'tailwind.config',
+ },
});
```

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .prettierrc
├── README.md
├── nuxt.config.ts
├── package.json
├── src
│   └── app.vue
├── tailwind.config.cjs
├── tsconfig.json
└── yarn.lock
```

## TailwindCSS用のESLintとPrettierの設定

### インストール

```zsh
yarn add -D eslint-plugin-tailwindcss prettier-plugin-tailwindcss
```

### 設定ファイルの編集

```diff js:.eslintrc.cjs
module.exports = {
  root: true,
  env: {
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
+   'plugin:tailwindcss/recommended',
    '@nuxtjs/eslint-config-typescript',
    'prettier',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  },
- plugins: ['vue', '@typescript-eslint'],
+ plugins: ['vue', '@typescript-eslint', 'tailwindcss'],
  rules: {
    /* typescript */
    'dot-notation': 'off',
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          '../*',
          '~/*',
          '~~/*',
          './assets/*',
          './components/*',
          './pages/*',
          './plugins/*',
          './router/*',
          './composables/*',
          './server/*',
          './store/*',
          './types/*',
          './utils/*',
          './libs/*',
          './*.vue',
        ],
      },
    ],
    'import/order': [
      'error',
      {
        'groups': [
          'builtin',
          'external',
          'parent',
          'sibling',
          'index',
          'object',
          'type',
        ],
        'pathGroups': [
          {
            pattern: '{vue,vue-router,vite,@vitejs/plugin-vue}',
            group: 'builtin',
            position: 'before',
          },
          {
            pattern: '@src/**',
            group: 'parent',
            position: 'before',
          },
        ],
        'pathGroupsExcludedImportTypes': ['builtin'],
        'alphabetize': {
          order: 'asc',
        },
        'newlines-between': 'always',
      },
    ],
    '@typescript-eslint/consistent-type-imports': [
      'error',
      { prefer: 'type-imports' },
    ],

    /* nuxt */
    'vue/multi-word-component-names': 'off',
    'vue/require-v-for-key': 'off',

+   /* tailwindcss */
+   'tailwindcss/no-custom-classname': [
+     'warn',
+     {
+       config: 'tailwind.config.cjs',
+     },
+   ],
+   'tailwindcss/classnames-order': 'off',
  },
};
```

## daisyUI

### インストール

```zsh
yarn add daisyui
```

### 型定義の追加

```ts:src/types/global.d.ts
declare module 'daisyui';
```

### 設定ファイルの編集

```diff js:tailwind.config.cjs
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.html', './src/**/*.vue', './src/**/*.jsx'],
  theme: {
    extend: {
      colors: {
        'light-black': '#333333',
      },
    },
  },
+ plugins: [require('daisyui')],
};
```

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .prettierrc
├── README.md
├── nuxt.config.ts
├── package.json
├── src
│   ├── app.vue
│   └── types
│       └── global.d.ts
├── tailwind.config.cjs
├── tsconfig.json
└── yarn.lock
```

## Vitest

### インストール

```zsh
yarn add -D vitest @vue/test-utils unplugin-auto-import
```

### 設定ファイルの作成

```ts:vitest.config.ts
/// <reference types="vitest" />

import Vue from '@vitejs/plugin-vue';
import { defineConfig } from 'vite';
import type { UserConfig } from 'vite';

import AutoImport from 'unplugin-auto-import/vite';

import type { InlineConfig } from 'vitest';

interface VitestConfigExport extends UserConfig {
  test: InlineConfig;
}

export default defineConfig({
  plugins: [
    Vue(),
    AutoImport({ imports: ['vue'], dts: 'src/types/auto-imports.d.ts' }),
  ],
  test: {
    global: true,
    environment: 'jsdom',
  },
  resolve: {
    alias: {
      '@': '/src',
    },
  },
} as VitestConfigExport);
```

#### 解説

##### `VitestConfigExport`について

ここでは、`VitestConfigExport`というインタフェースを定義している。これは、もともとの`defineConfig`の引数である`UserConfig`型には`test`というプロパティはなく、`test`プロパティを設定した際に出るエラーを回避するために定義している

@[card](https://stackoverflow.com/questions/72146352/vitest-defineconfig-test-does-not-exist-in-type-userconfigexport)

##### `AutoImport`について

`AutoImport`は、`unplugin-auto-import`というプラグインを利用している。これは、`import`文を自動で追加してくれるプラグインである。もともとNuxtではVue由来の関数などは自動でimportされるようになっているが、Vitestでテストを実行する際は自動でimportされないため、`Reference Error`を回避するためにこのプラグインを利用している

### `scripts`への追加

```diff json:package.json
{
  "scripts": {
    "build": "nuxt build",
    "dev": "nuxt dev",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare",
+   "test": "vitest"
  },
}
```

### テスト実行

```zsh
yarn test
```

```zsh
 MISSING DEP  Can not find dependency 'jsdom'

? Do you want to install jsdom? › (y/N) y
```

`jsdom`がないというエラーが出るので、`y`を入力してインストールする

コンポーネントもテストも作成していないのでテスト自体は失敗する

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .prettierrc
├── README.md
├── nuxt.config.ts
├── package.json
├── src
│   ├── app.vue
│   └── types
│       ├── auto-imports.d.ts
│       └── global.d.ts
├── tailwind.config.cjs
├── tsconfig.json
├── vitest.config.ts
└── yarn.lock
```

## Vitest用のESLintの設定

### インストール

```zsh
yarn add -D eslint-plugin-vitest
```

### 設定ファイルの編集

```diff js:.eslintrc.cjs
module.exports = {
  root: true,
  env: {
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:tailwindcss/recommended',
    '@nuxtjs/eslint-config-typescript',
    'prettier',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  },
- plugins: ['vue', '@typescript-eslint', 'tailwindcss'],
+ plugins: ['vue', '@typescript-eslint', 'tailwindcss', 'vitest'],
  rules: {
    /* typescript */
    'dot-notation': 'off',
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          '../*',
          '~/*',
          '~~/*',
          './assets/*',
          './components/*',
          './pages/*',
          './plugins/*',
          './router/*',
          './composables/*',
          './server/*',
          './store/*',
          './types/*',
          './utils/*',
          './libs/*',
          './*.vue',
        ],
      },
    ],
    'import/order': [
      'error',
      {
        'groups': [
          'builtin',
          'external',
          'parent',
          'sibling',
          'index',
          'object',
          'type',
        ],
        'pathGroups': [
          {
            pattern: '{vue,vue-router,vite,@vitejs/plugin-vue}',
            group: 'builtin',
            position: 'before',
          },
          {
            pattern: '@src/**',
            group: 'parent',
            position: 'before',
          },
        ],
        'pathGroupsExcludedImportTypes': ['builtin'],
        'alphabetize': {
          order: 'asc',
        },
        'newlines-between': 'always',
      },
    ],
    '@typescript-eslint/consistent-type-imports': [
      'error',
      { prefer: 'type-imports' },
    ],

    /* nuxt */
    'vue/multi-word-component-names': 'off',
    'vue/require-v-for-key': 'off',

    /* tailwindcss */
    'tailwindcss/no-custom-classname': [
      'warn',
      {
        config: 'tailwind.config.cjs',
      },
    ],
    'tailwindcss/classnames-order': 'off',

+   /* vitest */
+   'vitest/consistent-test-it': [
+     'error',
+     {
+       fn: 'test',
+     },
+   ],
+   'vitest/expect-expect': 'warn',
+   'vitest/lower-case-title': 'off',
+   'vitest/max-nested-describe': [
+     'error',
+     {
+       max: 1,
+     },
+   ],
+   'vitest/no-conditional-tests': 'error',
+   'vitest/no-focused-tests': 'warn',
+   'vitest/no-identical-title': 'error',
+   'vitest/no-skipped-tests': 'warn',
  },
};
```

## Storybook

### インストール

```zsh
npx sb init --type vue3 --builder @storybook/builder-vite
```

```zsh
✔ Do you want to run the 'eslintPlugin' migration on your project? … yes
```

eslintのplugin入れるか？ と聞かれるので同意する

`.storybook`と`stories`ディレクトリが作成される。しかし、`stories`に関しては不要なので削除する

### 設定ファイルの編集

```js:.storybook/main.js
const AutoImport = require('unplugin-auto-import/vite');

module.exports = {
  stories: ['../src/**/*.stories.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
  ],
  framework: '@storybook/vue3',
  core: {
    builder: '@storybook/builder-vite',
  },
  features: {
    storyStoreV7: true,
  },
  async viteFinal(config) {
    config.plugins.push(
      AutoImport({ imports: ['vue'], dts: 'src/types/auto-imports.d.ts' })
    );

    return config;
  },
};
```

### ESLintの設定

eslintのプラグインを入れた影響で`.eslintrc.cjs`が崩されているので修正する

```diff js:.eslintrc.cjs
module.exports = {
  root: true,
  env: {
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:tailwindcss/recommended',
    '@nuxtjs/eslint-config-typescript',
+   'plugin:storybook/recommended',
    'prettier',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
  },
  plugins: ['vue', '@typescript-eslint', 'tailwindcss', 'vitest'],
  rules: {
    /* typescript */
    'dot-notation': 'off',
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          '../*',
          '~/*',
          '~~/*',
          './assets/*',
          './components/*',
          './pages/*',
          './plugins/*',
          './router/*',
          './composables/*',
          './server/*',
          './store/*',
          './types/*',
          './utils/*',
          './libs/*',
          './*.vue',
        ],
      },
    ],
    'import/order': [
      'error',
      {
        'groups': [
          'builtin',
          'external',
          'parent',
          'sibling',
          'index',
          'object',
          'type',
        ],
        'pathGroups': [
          {
            pattern: '{vue,vue-router,vite,@vitejs/plugin-vue}',
            group: 'builtin',
            position: 'before',
          },
          {
            pattern: '@src/**',
            group: 'parent',
            position: 'before',
          },
        ],
        'pathGroupsExcludedImportTypes': ['builtin'],
        'alphabetize': {
          order: 'asc',
        },
        'newlines-between': 'always',
      },
    ],
    '@typescript-eslint/consistent-type-imports': [
      'error',
      { prefer: 'type-imports' },
    ],

    /* nuxt */
    'vue/multi-word-component-names': 'off',
    'vue/require-v-for-key': 'off',

    /* tailwindcss */
    'tailwindcss/no-custom-classname': [
      'warn',
      {
        config: 'tailwind.config.cjs',
      },
    ],
    'tailwindcss/classnames-order': 'off',

    /* vitest */
    'vitest/consistent-test-it': [
      'error',
      {
        fn: 'test',
      },
    ],
    'vitest/expect-expect': 'warn',
    'vitest/lower-case-title': 'off',
    'vitest/max-nested-describe': [
      'error',
      {
        max: 1,
      },
    ],
    'vitest/no-conditional-tests': 'error',
    'vitest/no-focused-tests': 'warn',
    'vitest/no-identical-title': 'error',
    'vitest/no-skipped-tests': 'warn',
  },
};
```

### `htmlhintrc`の設定

`.storybook/preview-head.html`にて`doctype-first`エラーが出ているのでそれを消す

```json:.storybook/.htmlhintrc
{
  "doctype-first": false
}
```

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .prettierrc
├── .storybook
│   ├── .htmlhintrc
│   ├── main.js
│   ├── preview-head.html
│   └── preview.js
├── README.md
├── nuxt.config.ts
├── package.json
├── src
│   ├── app.vue
│   └── types
│       ├── auto-imports.d.ts
│       └── global.d.ts
├── tailwind.config.cjs
├── tsconfig.json
├── vitest.config.ts
└── yarn.lock
```

## TailwindCSSをStorybookに適用する

### インストール

```zsh
yarn add -D tailwindcss @tailwindcss/postcss7-compat
```

`@nuxtjs/tailwindcss`をインストールしているのに別に`tailwindcss`をインストールしなければならないのが少々不満だが、postcssのバージョン違いの影響によりこのような処置をとる必要があるらしい

### 設定ファイルの編集

```diff js:tailwind.config.cjs
/** @type {import('tailwindcss').Config} */
module.exports = {
- content: ['./src/**/*.html', './src/**/*.vue', './src/**/*.jsx'],
+ content: ['./src/**/*.{html,js,jsx,ts,tsx,vue}'],
  theme: {
    extend: {
      colors: {
        'light-black': '#333333',
      },
    },
  },
  plugins: [require('daisyui')],
};
```

### CSSファイルの作成

```css:src/assets/css/tailwind.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### PostCSSの設定

```js:postcss.config.cjs
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### Storybookの設定

```diff js:.storybook/preview.js
+import '../src/assets/css/tailwind.css';

export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};
```

```zsh
.
├── .eslintrc.cjs
├── .gitignore
├── .npmrc
├── .prettierrc
├── .storybook
│   ├── .htmlhintrc
│   ├── main.js
│   ├── preview-head.html
│   └── preview.js
├── README.md
├── nuxt.config.ts
├── package.json
├── postcss.config.cjs
├── src
│   ├── app.vue
│   ├── assets
│   │   └── css
│   │       └── tailwind.css
│   └── types
│       ├── auto-imports.d.ts
│       └── global.d.ts
├── tailwind.config.cjs
├── tsconfig.json
├── vitest.config.ts
└── yarn.lock
```

## aliasをStorybookに適用する

### 設定ファイルの編集

```diff js:.storybook/main.js
+const path = require('path');

const AutoImport = require('unplugin-auto-import/vite');

module.exports = {
  stories: ['../src/**/*.stories.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
  ],
  framework: '@storybook/vue3',
  core: {
    builder: '@storybook/builder-vite',
  },
  features: {
    storyStoreV7: true,
  },
  async viteFinal(config) {
    config.plugins.push(
      AutoImport({ imports: ['vue'], dts: 'src/types/auto-imports.d.ts' })
    );

+   config.resolve.alias['@'] = path.resolve(__dirname, '..', 'src');

    return config;
  },
};
```

```diff js:.storybook/preview.js
-import '../src/assets/css/tailwind.css';
+import '@/assets/css/tailwind.css';

export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};
```

# 最終的な`package.json`

```json:package.json
{
  "private": true,
  "scripts": {
    "build": "nuxt build",
    "dev": "nuxt dev",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare",
    "test": "vitest",
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook"
  },
  "devDependencies": {
    "@babel/core": "^7.20.12",
    "@nuxtjs/eslint-config-typescript": "^12.0.0",
    "@nuxtjs/tailwindcss": "^6.2.0",
    "@storybook/addon-actions": "^6.5.15",
    "@storybook/addon-essentials": "^6.5.15",
    "@storybook/addon-interactions": "^6.5.15",
    "@storybook/addon-links": "^6.5.15",
    "@storybook/builder-vite": "^0.2.7",
    "@storybook/testing-library": "^0.0.13",
    "@storybook/vue3": "^6.5.15",
    "@tailwindcss/postcss7-compat": "^2.2.17",
    "@tsconfig/strictest": "^1.0.2",
    "@typescript-eslint/eslint-plugin": "^5.48.1",
    "@typescript-eslint/parser": "^5.48.1",
    "@vue/test-utils": "^2.2.7",
    "babel-loader": "^8.3.0",
    "eslint": "^8.31.0",
    "eslint-config-prettier": "^8.6.0",
    "eslint-plugin-import": "^2.27.4",
    "eslint-plugin-storybook": "^0.6.10",
    "eslint-plugin-tailwindcss": "^3.8.0",
    "eslint-plugin-vitest": "^0.0.29",
    "eslint-plugin-vue": "^9.9.0",
    "jsdom": "^21.0.0",
    "nuxt": "^3.0.0",
    "prettier": "^2.8.3",
    "prettier-plugin-tailwindcss": "^0.2.1",
    "tailwindcss": "^3.2.4",
    "typescript": "^4.9.4",
    "unplugin-auto-import": "^0.12.1",
    "vitest": "^0.27.1",
    "vue-loader": "^16.8.3"
  },
  "dependencies": {
    "daisyui": "^2.46.1"
  }
}
```

# まとめ

今回の構成のテンプレートリポジトリを作成しました。

@[card](https://github.com/airRnot1106/nuxt-template-v3)
