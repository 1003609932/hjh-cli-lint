## CLI 使用

### 安装

在终端执行：

```bash
npm install hjh-fe-lint -g
```

安装完成后，可执行 `hjh-fe-lint -h` 以验证安装成功。

### 使用

#### `hjh-fe-lint init`：一键接入

在项目根目录执行 `hjh-fe-lint init`，即可一键接入规范，为项目安装规范 `Lint` 所需的依赖和配置。

具体会做以下事情：

- 写入各种配置文件，包括：
  - `.eslintrc.js`、`.eslintignore`：ESLint 配置（继承 `eslint-config-hjh`）及黑名单文件
  - `.stylelintrc.js`、`.stylelintignore`：stylelint 配置（继承 `stylelint-config-hjh`）及黑名单文件
  - `commitlint.config.js`：commitlint 配置（继承 `commitlint-config-hjh`）
  - `.markdownlint.json`、`.markdownlintignore`：`markdownlint` 配置及黑名单文件
  - `.prettierrc.js`：符合规范的 [Prettier 配置](https://prettier.io/docs/en/configuration.html)
  - `.editorconfig`：符合规范的 [editorconfig](https://editorconfig.org/)
  - `.vscode/extensions.json`：写入规范相关的 [VSCode 插件推荐](https://code.visualstudio.com/docs/editor/extension-gallery#_workspace-recommended-extensions)，包括 `ESLint`、`stylelint`、`markdownlint`、`prettier` 等
  - `.vscode/settings.json`：写入规范相关的 [VSCode 设置](https://code.visualstudio.com/docs/getstarted/settings#_settings-file-locations)，设置 `ESLint` 和 `stylelint` 插件的 `validate` 及**保存时自动运行 fix**，如果选择使用 `Prettier`，会同时将 `prettier-vscode` 插件设置为各前端语言的 defaultFormatter，并配置**保存时自动格式化**
  - `hjh-fe-lint.config.js`hjh-fe-lint 包的一些配置，如启用的功能等
- 配置 git commit 卡口：使用 [husky](https://www.npmjs.com/package/husky) 设置代码提交卡口，在 git commit 时会运行 `hjh-fe-lint commit-file-scan` 和 `hjh-fe-lint commit-msg-scan` 分别对提交文件和提交信息进行规范检查。`hjh-fe-lint commit-file-scan` 默认仅对 error 问题卡口，如果你想对 warn 问题也卡口，可以增加 `--strict` 参数以开启严格模式

> 注 1：如果项目已经配置过 ESLint、stylelint 等 Linter，执行 `hjh-fe-lint init` 将会提示存在冲突的依赖和配置，并在得到确认后进行覆盖：
>
> 注 2：如果项目的 .vscode/ 目录被 .gitignore 忽略，可以在拉取项目后单独执行 `hjh-fe-lint init --vscode` 命令写入 `.vscode/extensions.json` 和 `.vscode/settings.json` 配置文件

#### `hjh-fe-lint scan`：一键扫描

在项目的根目录执行命令，即可扫描项目的规范问题：

支持下列参数：

- `-q` `--quiet` 仅报告 error 级别的问题
- `-o` `--output-report` 输出扫描出的规范问题日志
- `-i` `--include <dirpath>` 指定要进行规范扫描的目录
- `--no-ignore` 忽略 eslint 的 ignore 配置文件和 ignore 规则

> 注 1：事实上，你可以在任意目录执行 `hjh-fe-lint scan` `hjh-fe-lint` 会根据文件类型、JSON 等特征嗅探项目类型。但我们还是推荐在执行过 `hjh-fe-lint init` 的项目根目录执行 `hjh-fe-lint scan`，以得到最准确的扫描结果。
>
> 注 2: `hjh-fe-lint` 会根据项目内有无 eslint 和 stylelint 配置文件判断使用项目的配置文件还是 `hjh-fe-lint` 默认配置进行扫描。若使用项目的，在未安装依赖时会帮其安装（执行 npm i）。若使用项目配置扫描失败，则使用默认配置扫描

#### `hjh-fe-lint fix`：一键修复

在项目的根目录执行命令，即可修复部分规范问题：

支持下列参数：

- `-i` `--include <dirpath>` 指定要进行修复扫描的目录
- `--no-ignore` 忽略 eslint 的 ignore 配置文件和 ignore 规则

注意请 review 下修复前后的代码，以免工具误修的情况。

#### `hjh-fe-lint commit-file-scan` 提交文件扫描

在 git commit 时对提交文件进行规范问题扫描，需配合 git 的 pre-commit 钩子使用。

支持下列参数：

- `-s` `--strict` 严格模式，对 warn 和 error 问题都卡口，默认仅对 error 问题卡口

#### `hjh-fe-lint commit-msg-scan` 提交信息扫描

git commit 时对 commit message 的格式进行扫描（使用 commitlint），需配合 [husky](https://www.npmjs.com/package/husky) 的 commit-msg 钩子使用。

## Node.js API 使用

### 安装

```bash
npm install hjh-fe-lint --save
```

### API

#### init：初始化

- hjh-fe-lint.init(config)：将项目一键接入规范，效果等同于 `hjh-fe-lint init`

示例：

```js
(await hjh) -
  fe -
  lint.init({
    eslintType: 'react',
    enableESLint: true,
    enableStylelint: true,
    enableMarkdownlint: true,
    enablePrettier: true,
    disableNpmInstall: false,
  });
```

config 参数如下：

| 参数               | 类型       | 默认值 | 说明                                                                                                             |
| ------------------ | ---------- | ------ | ---------------------------------------------------------------------------------------------------------------- |
| cwd                | string     | -      | 项目绝对路径                                                                                                     |
| eslintType         | ESLintType | -      | 语言和框架类型，如果不配置，等同于 hjh-fe-lint init，控制台会出现选择器，如果配置，控制台就不会出现选择器        |
| enableESLint       | boolean    | true   | 是否启用 ESLint，如果不配置默认值为 true，即默认启用 ESLint                                                      |
| enableStylelint    | boolean    | -      | 是否启用 stylelint，如果不配置，等同于 hjh-fe-lint init，控制台会出现选择器，如果配置，控制台就不会出现选择器    |
| enableMarkdownlint | boolean    | -      | 是否启用 markdownlint，如果不配置，等同于 hjh-fe-lint init，控制台会出现选择器，如果配置，控制台就不会出现选择器 |
| enablePrettier     | boolean    | -      | 是否启用 Prettier                                                                                                |
| disableNpmInstall  | boolean    | false  | 是否禁用自动在初始化完成后安装依赖                                                                               |

##### ESLintType

- `default`: JavaScript 项目（未使用 React 和 Vue 的 JS 项目）
- `react`: JavaScript + React 项目
- `vue`: JavaScript + Vue 项目
- `typescript/default`: TypeScript 项目（未使用 React 和 Vue 的 TS 项目）
- `typescript/react`: TypeScript + React 项目
- `typescript/vue`: TypeScript + Vue 项目
- `es5`: ES5 及之前版本的 JavaScript 老项目

## 配置

`hjh-fe-lint` 基于一份配置进行扫描（但你也可以零配置使用），支持的配置参数有：

| 参数                | 类型                    | 默认值 | 说明                                                                                           |
| ------------------- | ----------------------- | ------ | ---------------------------------------------------------------------------------------------- |
| enableESLint        | boolean                 | true   | 是否启用 ESLint                                                                                |
| enableStylelint     | boolean                 | true   | 是否启用 stylelint                                                                             |
| enableMarkdownlint  | boolean                 | true   | 是否启用 markdownlint                                                                          |
| enablePrettier      | boolean                 | -      | 是否启用 Prettier                                                                              |
| eslintOptions       | ESLint.Options          | -      | ESLint 配置项，若未设置将使用执行目录下或内置的默认 eslintrc 和 eslintignore 进行扫描          |
| stylelintOptions    | stylelint.LinterOptions | -      | stylelint 配置项，若未设置将使用执行目录下或内置的默认 stylelintrc 和 stylelintignore 进行扫描 |
| markdownlintOptions | markdownlint.Options    | -      | markdownlint 配置项，若未设置将使用执行目录下或内置的默认 markdownlint 配置文件进行扫描        |

`hjh-fe-lint` 会读取执行目录下的 `hjh-fe-lint.config.js` 作为配置文件。`hjh-fe-lint init` 会在执行目录下新增如下的 `hjh-fe-lint.config.js` 文件：

```js
module.exports = {
  enableESLint: true,
  enableStylelint: true,
  enableMarkdownlint: true,
  enablePrettier: true,
};
```

## 常见问题

### TypeScript 项目扫描性能问题

如果你的 TS 项目 commit 卡口和 `hjh-fe-lint scan` 运行时间很长，可以通过如下在 `.eslintrc.js` 中增加以下配置提升性能：

```js
module.exports = {
  parserOptions: {
    project: [], // for lint performance
    createDefaultProgram: false, // for lint performance
  },
  rules: {
    '@typescript-eslint/dot-notation': 0, // for lint performance
    '@typescript-eslint/restrict-plus-operands': 0, // for lint performance
  },
};
```
