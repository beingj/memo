# Setup VS Code for TypeScript and Vue

## File organization

create 3 folders:

- `src`
  - typescript files
- `js`
  - javascript files
- `test`
  - test files

## TypeScript

create `tsconfig.json`

```bash
tsc --init
```

modify `tsconfig.json`

```json
{
    "compilerOptions": {
        "target": "ES2017",
        "module": "umd",
        "strict": true,
        "esModuleInterop": true,
        "outDir": "js"
    },
    "include": [
        "src/*"
    ],
    "exclude": [
        "test"
    ]
}
```

VS Code doesn’t support *compileOnSave* flag in `tsconfig.json`. refer to [issue](https://github.com/microsoft/vscode/issues/973)

we have to start it manually: press <kbd>Ctrl + Shift + B</kbd> to open a list of tasks in VS Code and select "`tsc: watch - tsconfig.json`"

## Mocha

create `package.json`

```bash
npm init
```

install mocha and chai

```bash
npm install --save-dev mocha chai
```

install ts-node, typescript and types

```bash
npm install --save-dev ts-node typescript
npm install --save-dev @types/chai @types/mocha
```

add test script to `package.json`

```json
{
  "scripts": {
     "test": "mocha --require ts-node/register test/**/*.ts"
  },
}
```

create a module: `src/a.ts`

```typescript
export class a {
    constructor() { }
    m() {
        return 'from a.m'
    }
}
```

create a test case for the module: `test/test.ts`

```typescript
import { expect } from 'chai';
import { a } from '../src/a';

describe('test', () => {
    it('class a method m', () => {
        let s = new a().m()
        expect(s).to.equal('from a.m');
    });
})
```

run the test

```bash
npm run test
```

## Mocha sidebar

install test coverage tool nyc

```bash
npm install --save-dev nyc
```

create `.nycrc.json`, add content as below to only check ts files in `src` folder, and not include `app.ts`. refer to [offical doc](https://github.com/istanbuljs/nyc#selecting-files-for-coverage)

```json
{
    "include": [
        "src"
    ],
    "exclude": [
        "src/app.ts"
    ]
}
```

install vscode extension mocha sidebar, and create/modify `.vscode/settings.json`

```json
{
    "mocha.files.glob": "test/**/*.ts",
    "mocha.requires": [
        "ts-node/register"
    ],
    "mocha.debugSettingsName": "Mocha Tests", // debug config name in `.vscode/launch.json`
}
```

mocha sidebar will create a `coverage` folder for every folder open by VS Code. we have to disable it.

<kbd>Ctrl + Shift + P</kbd>, input "`open user setting`" and click, input "`mocha.coverage`" to search, then click "`Edit in settings.json`". refer to [doc](https://marketplace.visualstudio.com/items?itemName=maty.vscode-mocha-sidebar)

```json
    "mocha.coverage": {
        "enable": false
    },
```

open `Activity Bar > Test > MOCHA`

- click the umbrella icon to run test coverage check. and check result in folder `coverage/index.html`
- select test and click triangle icon to run test

## Debug

> **Note 1**
> make sure node is update to date. old version node has issue to launch mocha debuger
>
> **Note 2**
> mocha version 5.x worked with the "-u tdd" option but 6.x works with "-u bdd". refer to [SO](https://stackoverflow.com/a/55570526)

create/modify `.vscode/launch.json`

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Mocha Tests",
            "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
            "args": [
                "--require",
                "ts-node/register",
                "-u",
                "bdd", // tdd => bdd
                "--timeout",
                "999999",
                "--colors",
                "--recursive",
                "${workspaceFolder}/test/**/*.ts"
            ],
            "internalConsoleOptions": "openOnSessionStart"
        },
    ]
}
```

set a breakpoint at the line start with `expect` in `test/test.ts`

```typescript
        expect(s).to.equal('from a.m');
```

open `Activity Bar > Run > MOCHA`, select "`Mocha Tests`", and click to start debug.

debuger should stop at the breakpoint. click `Continue` to continue.

## settings.json

modify `.vscode/settings.json` to hide some folders

```json
{
    "mocha.files.glob": "test/**/*.ts",
    "mocha.requires": [
        "ts-node/register"
    ],
    "mocha.debugSettingsName": "Mocha Tests", // debug config name in `.vscode/launch.json`

    "files.exclude": {
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "node_modules": true,
        ".nyc_output": true,
        "coverage": true,
        "js": true
    },
    "search.exclude": {
        "**/node_modules": true,
        "**/bower_components": true,
        "**/*.code-search": true,
        "js": true
    }
}
```

## vue

install vue and types

```bash
npm install --save-dev vue @types/vue
```

but there is no types definition files in node_modules/@types/vue. we have to copy them from node_modules/vue/types.

```bash
cp node_modules/vue/types node_modules/@types/vue -a
```

download `vue.js` and `require.js` and put them to `js` folder

## New project

create `index.html`

```html
<!DOCTYPE html>
<head>
    <title>my app</title>
    <meta charset="UTF-8">
    <meta name="description" content="my app">
    <meta name="keywords" content="beingj">
    <meta name="author" content="beingj@163.com">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            width: 1100px;
            margin: auto;
            margin-bottom: 3em;
        }
    </style>
</head>
<body>
    <h1>my app</h1>
    <div id='app'>
        <div v-if='loading' class="loading">loading...</div>
        <template>
            <p>
                {{ msg }}
            </p>
        </template>
    </div>
    <script data-main="js/app.js" src="js/require.js"></script>
</body>
```

create `src/app.ts`

```typescript
import Vue from 'vue'
import { a } from 'a';

const app = new Vue({
    el: '#app',
    data: {
        loading: false,
        msg: new a().m()
    },
})
```

double click `index.html` to open it in browser

## git

```bash
git init
```

create .gitignore

```text
node_modules
.nyc_output
coverage
```
