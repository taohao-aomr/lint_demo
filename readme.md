# lint_demo

## 配置eslint prettier editorconfig stylelint ts

## 配置提交 lint-staged&husky

### lint-staged

lint-staged 是一个在提交代码之前运行linter或其他工具的工具。这意味着当开发人员尝试提交代码到版本控制系统（如git）时，lint-staged 会只对暂存区（staged files）的文件运行配置的命令，这通常是代码风格检查器（如ESLint、Prettier）、代码格式化工具或测试套件。
使用lint-staged可以确保只有符合项目规定代码质量标准的代码被提交，减少了不必要的错误和风格问题被引入代码库的可能性。
在一个项目中使用lint-staged通常包括以下步骤：
添加lint-staged到项目的依赖中。
在package.json中配置lint-staged，指定哪些文件与相对应的校验命令。
��交代码时，lint-staged会自动运行只对staged文件的检查。

### husky

husky 是一个用于简化Git钩子（hooks）的设置的工具，允许开发者轻松地在各种Git事件触发时运行脚本。例如，在提交之前（pre-commit）、推送之前（pre-push）、或者在提交信息被写入后（commit-msg）等。
husky的使用可以提高项目团队的工作效率，确保代码库中的代码符合特定的质量标准。它通常与lint-staged一起使用，以在提交前自动执行代码的静态检查。
使用husky包括以下简单步骤：
添加husky到项目依赖。
配置Git钩子，使用husky的配置。
当相应的Git事件被触发时，定义的脚本就会自动执行。

yarn add husky@9.0.11 lint-staged@15.2.2 -D

启用husky，执行如下命令会自动在package中增加命令
npx husky install

husky prepare 命令，自动加入，有时候也没法自动加入，手动写也是可以的
{
  "scripts": {
    "prepare": "husky install"
  }
}

在 package.json 中添加以下代码
{
 "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
    }
  },
  "lint-staged": {
    "*.{ts,tsx,js}": [
      "eslint --config .eslintrc.js"
    ],
    "*.{css,less,scss}": [
      "stylelint --config .stylelintrc.js"
    ],
    "*.{ts,tsx,js,json,html,yml,css,less,scss,md}": [
      "prettier --write"
    ]
  },
}

## commitlint

配置提交校验，commitlint 可以帮助我们进行 git commit 时的 message 格式是否符合规范
yarn add @commitlint/cli@19.2.0 @commitlint/config-conventional@19.1.0 -D
新增配置文件.commitlintrc.js
module.exports = {
 extends: ['@commitlint/config-conventional'],
 rules: {
  'type-enum': [
   2,
   'always',
   [
    'build',
    'ci',
    'chore',
    'docs',
    'feat',
    'fix',
    'perf',
    'refactor',
    'revert',
    'style',
    'test',
    'addLog',
   ],
  ],
 },
};

随后回到 package.json 的 husky 配置，增加一个钩子，并且改写husky的commit-msg钩子
"husky": {
  "hooks": {
   "pre-commit": "lint-staged",
   "commit-msg": "commitlint --config .commitlintrc.js -E HUSKY_GIT_PARAMS"
  }
 },

## 配置可视化的提交提示

yarn add commitizen@4.3.0 cz-conventional-changelog@3.3.0 -D
在pageage.json中增加更改commitizen的配置，并增加脚本
"config":{
    "commitizen":{
        "path":"node_modules/cz-conventional-changelog"
    }
}

"scripts":{
    commit:"git-cz"
}

## 配置自定义提交规范

yarn add cz-customizable@7.0.0 commitlint-config-cz@0.13.3 -D
变更 commitlint.config.js
module.exports = {
 extends: ['cz'],
 rules: {},
};

增加 .cz-config.js
'use strict';
module.exports = {
 types: [
  {value: 'feat', name: '✨新增:    新的内容'},
  {value: 'fix', name: '🐛修复:    修复一个Bug'},
  {value: 'docs', name: '📝文档:    变更的只有文档'},
  {value: 'style', name: '💄格式:    空格, 分号等格式修复'},
  {value: 'refactor', name: '️♻️重构:    代码重构，注意和特性、修复区分开'},
  {value: 'perf', name: '️️⚡️性能:    提升性能'},
  {value: 'test', name: '✅测试:    添加一个测试'},
  {value: 'build', name: '🔧工具:    开发工具变动(构建、脚手架工具等)'},
  {value: 'rollback', name: '⏪回滚:    代码回退'},
  {value: 'addLog', name: '👨🏻‍💻添加log:    代码回退'},
 ],
 scopes: [
  {name: 'leetcode'},
  {name: 'javascript'},
  {name: 'typescript'},
  {name: 'Vue'},
  {name: 'node'},
 ],
 // it needs to match the value for field type. Eg.: 'fix'
 /*scopeOverrides: {
 fix: [
   {name: 'merge'},
   {name: 'style'},
   {name: 'e2eTest'},
   {name: 'unitTest'}
 ]
  },*/
 // override the messages, defaults are as follows
 messages: {
  type: '选择一种你的提交类型:',
  scope: '选择一个scope (可选):',
  // used if allowCustomScopes is true
  customScope: 'Denote the SCOPE of this change:',
  subject: '短说明:\n',
  body: '长说明，使用"|"换行(可选)：\n',
  breaking: '非兼容性说明 (可选):\n',
  footer: '关联关闭的issue，例如：#31, #34(可选):\n',
  confirmCommit: '确定提交说明?(yes/no)',
 },
 allowCustomScopes: true,
 allowBreakingChanges: ['特性', '修复'],
 // limit subject length
 subjectLimit: 100,
};
package.json 中,将原来commit配置,变更为自定义配置
"config": {
  "commitizen": {
   "path": "node_modules/cz-customizable"
  }
 }

## 配置使用git commit走自定义配置

更改husky的prepare-commit-msg钩子，如下
exec < /dev/tty && node_modules/.bin/cz --hook || true
