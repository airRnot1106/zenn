---
title: "Node + Hono + Prisma + Jestで環境構築"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["hono", "prisma", "jest"]
published: false
---

## はじめに

Node.jsでHonoとPrismaを使っての開発、Jestでのテストを行う環境を整えるのに苦労したので、備忘録として残す

## 環境構築

### プロジェクト作成

```bash
mkdir node-hono-prisma-jest
```

```bash
cd node-hono-prisma-jest
```

```bash
pnpm create hono@latest .
```

```other
create-hono version 0.2.6
✔ Which template do you want to use? › nodejs
cloned honojs/starter#main to /Users/.../node-hono-prisma-jest
✔ Copied project files
```

```bash
pnpm install
```

#### 動作確認

```bash
pnpm start
```

<http://localhost:3000/>にアクセスして`Hello Hono!`と表示されれば成功

### TypeScriptやLinter周りの設定

これはお好みで。

:::details 長いので省略

```bash
pnpm add -D typescript @types/node @tsconfig/strictest
```

```json:tsconfig.json
{
  "extends": "@tsconfig/strictest/tsconfig.json",
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "outDir": "dist",
    "noPropertyAccessFromIndexSignature": false,
    "noImplicitReturns": false,
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "~/*": ["./*"]
    },
    "esModuleInterop": true,
    "isolatedModules": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
  },
  "include": ["src/**/*", "__tests__/**/*", "scripts/**/*"]
}

```

```bash
pnpm add -D eslint eslint-config-airbnb-base eslint-import-resolver-alias eslint-import-resolver-typescript eslint-plugin-import eslint-plugin-jest eslint-plugin-unused-imports @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier eslint-config-prettier
```

```js:.eslintrc.cjs
module.exports = {
  root: true,
  extends: [
    'eslint:recommended',
    'airbnb-base',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking',
    'plugin:import/typescript',
    'plugin:jest/recommended',
    'plugin:jest/style',
    'prettier',
  ],
  plugins: ['import', 'unused-imports', '@typescript-eslint', 'jest'],
  env: {
    'jest/globals': true,
    'es6': true,
  },
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json',
  },
  settings: {
    'import/parsers': {
      '@typescript-eslint/parser': ['.ts', '.tsx'],
    },
    'import/resolver': {
      alias: {
        map: [
          ['@', './src'],
          ['~', './'],
        ],
        extensions: ['.ts', '.js', '.json'],
      },
    },
  },
  rules: {
    /* eslint */
    'consistent-return': 'off',
    'no-underscore-dangle': 'off',
    /* typescript */
    '@typescript-eslint/no-unused-vars': [
      'warn',
      {
        argsIgnorePattern: '^_',
        varsIgnorePattern: '^_',
        caughtErrorsIgnorePattern: '^_',
        destructuredArrayIgnorePattern: '^_',
      },
    ],
    /* import */
    'import/extensions': 'off',
    'import/prefer-default-export': 'off',
    'unused-imports/no-unused-imports': 'error',
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
        'pathGroupsExcludedImportTypes': ['builtin'],
        'newlines-between': 'always',
        'alphabetize': {
          order: 'asc',
        },
      },
    ],
    '@typescript-eslint/consistent-type-imports': [
      'error',
      {
        prefer: 'type-imports',
      },
    ],
    'no-restricted-imports': [
      'error',
      {
        patterns: ['./*', '../*'],
      },
    ],
    /* jest */
    'jest/consistent-test-it': ['error', { fn: 'it' }],
  },
};

```

```ignore:.eslintignore
# config
.eslintrc.cjs
.prettierrc
tsconfig.json
jest.config.js
jest.setup.js

# scripts
scripts/

# build dir
**/dist/

# generated dir
**/generated/

```

```json:.prettierrc
{
  "singleQuote": true,
  "semi": true,
  "tabWidth": 2,
  "quoteProps": "consistent",
  "trailingComma": "es5",
  "bracketSpacing": true,
  "arrowParens": "always"
}

```

```json:.vscode/settings.json
{
  "editor.formatOnSave": true,

  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },

  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],

  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": false
  },

  "javascript.format.enable": false,
  "typescript.format.enable": false,

  "eslint.workingDirectories": [
    {
      "mode": "auto"
    }
  ]
}

```

:::

### esbuildの設定

ビルドツールはesbuildを使用

```bash
pnpm add -D esbuild esbuild-plugin-alias-path
```

ビルドの用のスクリプトを作成

```ts:scripts/build.ts
import path from 'path';
import { argv } from 'process';

import { build } from 'esbuild';
import { aliasPath } from 'esbuild-plugin-alias-path';

import type { BuildOptions } from 'esbuild';

const options: BuildOptions = {
  entryPoints: [path.resolve(__dirname, '../src/index.ts')],
  minify: argv[2] === 'production',
  bundle: true,
  target: 'es2015',
  platform: 'node',
  external: [],
  outfile: path.resolve(__dirname, '../dist/index.js'),
  plugins: [
    aliasPath({
      alias: {
        '@': path.resolve(__dirname, '../src'),
      },
    }),
  ],
};

build(options).catch((err) => {
  process.stderr.write(err.stderr);
  process.exit(1);
});

```

`package.json`を修正

```json:package.json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsx scripts/build.ts production",
    "build-dev": "tsx scripts/build.ts development"
  },
  "dependencies": {
    "@hono/node-server": "^1.0.2",
    "hono": "^3.2.7"
  },
  "devDependencies": {
    "@tsconfig/strictest": "^2.0.1",
    "@types/node": "^20.4.1",
    "@typescript-eslint/eslint-plugin": "^5.61.0",
    "@typescript-eslint/parser": "^5.61.0",
    "esbuild": "^0.18.11",
    "esbuild-plugin-alias-path": "^2.0.2",
    "eslint": "^8.44.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-config-prettier": "^8.8.0",
    "eslint-import-resolver-alias": "^1.1.2",
    "eslint-import-resolver-typescript": "^3.5.5",
    "eslint-plugin-import": "^2.27.5",
    "eslint-plugin-jest": "^27.2.2",
    "eslint-plugin-unused-imports": "^2.0.0",
    "prettier": "^3.0.0",
    "tsx": "^3.12.2",
    "typescript": "^5.1.6"
  }
}

```

#### 動作確認

```bash
pnpm build
```

```bash
pnpm start
```

速い。esbuildすごい

### Prisma

まずはDB環境を作る

```yaml:compose.yaml
version: "3.9"
services:
  db:
    image: postgres:15-alpine
    restart: always
    env_file:
      - .env
    volumes:
      - ./db/tmp/db:/var/lib/postgresql/data
    ports:
      - "5432:5432"

```

```env:.env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/postgres?schema=public"

POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=postgres

```

```bash
docker compose up --build -d
```

```bash
pnpm add -D prisma
```

```bash
pnpm dlx prisma init
```

```bash
pnpm add zod zod-prisma-types
```

```prisma:prisma/schema.prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

generator zod {
  provider                  = "pnpm dlx zod-prisma-types"
  output                    = "../src/schema/generated/prisma"
  createRelationValuesTypes = true
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}

```

`package.json`にprisma用のscriptを追加

```json:package.json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsx scripts/build.ts production",
    "build-dev": "tsx scripts/build.ts development",
    "prisma:migrate": "prisma migrate dev",
    "prisma:generate": "prisma generate",
    "prisma:reset": "prisma migrate reset --force",
    "prisma:studio": "prisma studio"
  },
  "dependencies": {
    "@hono/node-server": "^1.0.2",
    "@prisma/client": "4.16.2",
    "hono": "^3.2.7",
    "zod": "^3.21.4",
    "zod-prisma-types": "^2.7.4"
  },
  "devDependencies": {
    "@tsconfig/strictest": "^2.0.1",
    "@types/node": "^20.4.1",
    "@typescript-eslint/eslint-plugin": "^5.61.0",
    "@typescript-eslint/parser": "^5.61.0",
    "esbuild": "^0.18.11",
    "esbuild-plugin-alias-path": "^2.0.2",
    "eslint": "^8.44.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-config-prettier": "^8.8.0",
    "eslint-import-resolver-alias": "^1.1.2",
    "eslint-import-resolver-typescript": "^3.5.5",
    "eslint-plugin-import": "^2.27.5",
    "eslint-plugin-jest": "^27.2.2",
    "eslint-plugin-unused-imports": "^2.0.0",
    "prettier": "^3.0.0",
    "prisma": "^4.16.2",
    "tsx": "^3.12.2",
    "typescript": "^5.1.6"
  }
}

```

```bash
pnpm prisma:migrate
```

```other
✔ Enter a name for the new migration: … init
```

prismaのインスタンス作成

```ts:src/libs/prisma.ts
import { PrismaClient } from '@prisma/client';

// eslint-disable-next-line import/no-mutable-exports
let prisma: PrismaClient;

if (process.env.NODE_ENV === 'production') {
  prisma = new PrismaClient();
} else {
  const globalWithPrisma = global as typeof globalThis & {
    prisma: PrismaClient;
  };
  if (!globalWithPrisma.prisma) {
    globalWithPrisma.prisma = new PrismaClient();
  }
  prisma = globalWithPrisma.prisma;
}

export default prisma;

```

## Jest

```bash
pnpm add -D jest @types/jest esbuild-jest @quramy/jest-prisma node-fetch@2
```

`tsconfig.json`に`types`を追加

```json:tsconfig.json
{
  "extends": "@tsconfig/strictest/tsconfig.json",
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "outDir": "dist",
    "noPropertyAccessFromIndexSignature": false,
    "noImplicitReturns": false,
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "~/*": ["./*"]
    },
    "esModuleInterop": true,
    "isolatedModules": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "types": ["@types/jest", "@quramy/jest-prisma"]
  },
  "include": ["src/**/*", "__tests__/**/*", "scripts/**/*"]
}

```

```js:jest.config.js
module.exports = {
  testMatch: ['**/*.test.ts'],
  transform: {
    '^.+\\.ts$': 'esbuild-jest',
  },
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^~/(.*)$': '<rootDir>/$1',
  },
  testEnvironment: '@quramy/jest-prisma/environment',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
};

```

`@quramy/jest-prisma`を使うと、jest環境でprismaを使う際に、自動的にロールバックしてくれるようになる

@[card](https://github.com/Quramy/jest-prisma)

```js:jest.setup.js
const nodeFetch = require('node-fetch');
// Mocking fetch Web API using node-fetch
if (typeof fetch === 'undefined') {
  global.fetch = nodeFetch;
  global.Request = nodeFetch.Request;
  global.Response = nodeFetch.Response;
}

jest.mock('./src/libs/prisma', () => {
  return {
    __esModule: true,
    default: jestPrisma.client,
  };
});

```

jest環境でHonoを使うと

```other
ReferenceError: request is not defined
ReferenceError: response is not defined
```

というエラーが出るので、`jest.setup.js`で対策をする

`package.json`にjest用のscriptを追加

```json:package.json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsx scripts/build.ts production",
    "build-dev": "tsx scripts/build.ts development",
    "prisma:migrate": "prisma migrate dev",
    "prisma:generate": "prisma generate",
    "prisma:reset": "prisma migrate reset --force",
    "prisma:studio": "prisma studio",
    "test": "DATABASE_URL='postgresql://postgres:postgres@localhost:5432/postgres?schema=test' jest --watchAll",
    "test:exec": "DATABASE_URL='postgresql://postgres:postgres@localhost:5432/postgres?schema=test' pnpm prisma:reset && pnpm test"
  },
  "dependencies": {
    "@hono/node-server": "^1.0.2",
    "@prisma/client": "4.16.2",
    "hono": "^3.2.7",
    "zod": "^3.21.4",
    "zod-prisma-types": "^2.7.4"
  },
  "devDependencies": {
    "@quramy/jest-prisma": "^1.5.0",
    "@tsconfig/strictest": "^2.0.1",
    "@types/jest": "^29.5.2",
    "@types/node": "^20.4.1",
    "@typescript-eslint/eslint-plugin": "^5.61.0",
    "@typescript-eslint/parser": "^5.61.0",
    "esbuild": "^0.18.11",
    "esbuild-jest": "^0.5.0",
    "esbuild-plugin-alias-path": "^2.0.2",
    "eslint": "^8.44.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-config-prettier": "^8.8.0",
    "eslint-import-resolver-alias": "^1.1.2",
    "eslint-import-resolver-typescript": "^3.5.5",
    "eslint-plugin-import": "^2.27.5",
    "eslint-plugin-jest": "^27.2.2",
    "eslint-plugin-unused-imports": "^2.0.0",
    "jest": "^29.6.1",
    "node-fetch": "2",
    "prettier": "^3.0.0",
    "prisma": "^4.16.2",
    "tsx": "^3.12.2",
    "typescript": "^5.1.6"
  }
}

```

jest環境ではDBを本番用と分けたいので、script内で環境変数を書き換える

#### 動作確認

試しにテストを書いてみる

```ts:__tests__/models/user.test.ts
import prisma from '@/libs/prisma';

describe('User', () => {
  describe('add user', () => {
    it('creates a new user with valid input', async () => {
      const email = 'test@example.com';
      const name = 'test';

      const user = await prisma.user.create({
        data: {
          email,
          name,
        },
      });

      expect(
        await prisma.user.findUnique({
          where: {
            email,
          },
        })
      ).toStrictEqual(user);
    });

    it('throws an error if email is already in use', async () => {
      const email = 'test@example.com';
      const name = 'test';

      await prisma.user.create({
        data: {
          email,
          name,
        },
      });

      await expect(
        prisma.user.create({
          data: {
            email,
            name: 'test2',
          },
        })
      ).rejects.toMatchObject({
        name: 'PrismaClientKnownRequestError',
      });
    });
  });
});

```

(prismaでcreateに失敗したときは、rejectはされるがerrorはthrowされないっぽい？)

### APIの実装とテスト

```bash
pnpm add @hono/zod-validator
```

```ts:src/api/user/route.ts
import { zValidator } from '@hono/zod-validator';
import { Hono } from 'hono';
import { z } from 'zod';

import prisma from '@/libs/prisma';

export const user = new Hono().post(
  '/',
  zValidator(
    'json',
    z.object({
      email: z.string().email(),
      name: z.string().max(10),
    }),
    (result, c) => {
      if (!result.success) {
        return c.json(
          {
            error: {
              message: result.error,
            },
          },
          400
        );
      }
    }
  ),
  async (c) => {
    const { email, name } = c.req.valid('json');

    const createUserResponse = await (async () => {
      try {
        const newUser = await prisma.user.create({
          data: {
            email,
            name,
          },
        });

        return {
          status: 200,
          data: newUser,
        } as const;
      } catch (e) {
        return {
          status: 500,
          error: {
            message: e instanceof Error ? e.message : 'Internal Server Error',
          },
        } as const;
      }
    })();

    return c.jsonT(createUserResponse, createUserResponse.status);
  }
);

```

```ts:src/index.ts
import { serve } from '@hono/node-server';
import { Hono } from 'hono';

import { user } from '@/api/user/route';

const app = new Hono().basePath('/api');

export const route = app.route('/user', user);

if (process.env.NODE_ENV !== 'test') {
  serve(app);
}

```

jest環境では`serve`しないようにすることで、portの競合を防ぐ

```ts:src/api/user/user.test.ts
import { route } from '@/index';

describe('User API', () => {
  describe('POST /user', () => {
    it('should return 200 with valid data', async () => {
      const response = await route.request('http://localhost:3000/api/user', {
        method: 'POST',
        body: JSON.stringify({
          email: 'test@example.com',
          name: 'test',
        }),
      });

      expect(response.status).toBe(200);
    });

    it('should return 400 with invalid data', async () => {
      const response = await route.request('http://localhost:3000/api/user', {
        method: 'POST',
        body: JSON.stringify({
          email: 'test',
          name: 'test',
        }),
      });

      expect(response.status).toBe(400);
    });
  });
});

```

ちゃんと通った

## まとめ

`@quramy/jest-prisma`が便利すぎて神
