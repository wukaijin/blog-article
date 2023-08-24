笔记： 用 git hooks 来规范代码提交,提高代码质量，记录目前个人最佳实践  
以 react + vite 为例，包含 `prettier` `eslint` 和 `commitlint`

# husky

Install husky

```bash
$ pnpm i husky pre-commit -D
```

Activate hooks

```bash
$ npx husky install
```

- Need to install the following packages:  
-  husky-init  
- Ok to proceed? (y) y  
- husky-init updating package.json  
-  setting prepare script to command "husky install"  
- husky - Git hooks installed  
- husky - created .husky/pre-commit

修改 pre-commit 钩子执行的 script

```bash
npx husky add .husky/pre-commit "npm run eslint"
```

注意： 如果手动添加，钩子文件要有执行权限，不然后面执行会忽略  
hint: The '.husky/commit-msg' hook was ignored because it's not set as executable.
hint: You can disable this warning with `git config advice.ignoredHook false`.

```bash
$ chmod 777 ./husky/*
```

# lint-staged

```bash
$ pnpm i lint-staged -D
```

改动 package.json

```json
// package.json
"gitHooks": {
  "pre-commit": "lint-staged"
},
"lint-staged": {
  "src/**/*.{js,jsx,ts,tsx}": [
    "npm run eslint",
    "prettier --write"
  ],
  "*.{css,scss}": [
    "prettier --write"
  ]
},
```

# commitlint

```bash
$ pnpm i @commitlint/config-conventional @commitlint/cli -D
$ npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
```

根目录创建`commitlint.config.js`文件，配置 commitlint

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
}
```
package.json 配置
```json
// package.json
"husky": {
  "hooks": {
    "commit-msg": "commitlint -e $HUSKY_GIT_PARAMS"
  }
},
```
到这里 commit 时的校验和修复都已经好了。
增加一个 commitizen 来提示 commit 类型  
```bash
$ pnpm i -D commitizen
$ npx commitizen init cz-conventional-changelog --save-dev --save-exact
```
package.json 配置
```json
// package.json
"config": {
  "commitizen": {
    "path": "node_modules/cz-conventional-changelog"
  }
},
```
`commitizen` 可以再全局安装，项目内用要 `npx cz`  
[规范 github 源码](https://github.com/commitizen/cz-conventional-changelog/blob/master/engine.js)  
`cz-customizable`包可以自定义规则, 略
# end
prettier 配置比较简单，不写了，最后贴一版合并的 package.json 方便抄
```json
{
  "name": "demo",
  "scripts": {
    // ...
    "prettier": "prettier --write '**/*.{js,jsx,tsx,ts,less,md,json}'",
    "eslint": "eslint --ext .js,.jsx,.ts,.tsx ./src/",
    "eslint:fix": "npm run eslint -- --fix"
  },
  "gitHooks": {
    "pre-commit": "lint-staged"
  },
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -e $HUSKY_GIT_PARAMS"
    }
  },
  "config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,ts,tsx}": [
      "npm run eslint:fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "prettier --write"
    ]
  },
  // ...
}
```

