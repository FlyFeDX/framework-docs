我们希望通过以下方式为您的项目节省时间：

- **无须配置规则** - 统一的代码风格，无须配置规则，轻松拥有。
- **自动代码格式化** - 只需运行 `pnpm lint-fix` 轻松整理全部代码格式。
- **提前发现风格及程序问题** - 减少 Code Review 过程中的人工审查，简单的事情交给工具做，节约时间。
- **统一团队编码规范** - 一个团队, 一类项目, 一套规则.

再也不用维护 `.eslintrc` 了，开箱即用。

## 安装

```bash
pnpm add @FlyFeDX/lint-config -D
```

## 使用

1. 在 `.eslintrc.js`、`.prettierrc.js`、`.stylelintrc.js` 文件中:

```js
module.exports = {
  extends: [require.resolve('@FlyFeDX/lint-config/.eslintrc')],
}
```

```js
const prettier = require('@FlyFeDX/lint-config/.prettierrc')

module.exports = {
  ...prettier,
}
```

```js
module.exports = {
  extends: [require.resolve('@FlyFeDX/lint-config/.stylelintrc')],
}
```

我们的目标是统一代码风格，所以 `@FlyFeDX/lint-config` 不应该被自定义规则覆盖。

1. 在 `package.json` 文件中添加 script:

```json
"scripts": {
    "lint": "npx eslint \"./src/**/*.{ts, tsx, js, jsx}\"",
    "lint-fix": "npx eslint \"./src/**/*.{ts, tsx, js, jsx}\" --fix",
    "lint-css": "npx stylelint \"./src/**/*.{css,less}\"",
    "lint-css-fix": "npx stylelint \"./src/**/*.{css,less}\" --fix"
},
```

可以使用 `pnpm lint-fix` 来纠正大部分的代码风格问题。
