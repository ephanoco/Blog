## 概述

- JupyterLab 应用程序由一个核心应用程序对象和一组扩展组成。
- JupyterLab 扩展是一个包含多个 JupyterLab 插件的包。

## 模板

extension-template：使用 copier 创建 JupyterLab 扩展。

## 分发方式

### 预构建扩展

- 分发从源扩展预构建的包含非 JupyterLab JavaScript 依赖项的 JavaScript 代码包到例如 Python pip 或 conda 包中。
- 该代码包可以加载到 JupyterLab 中而无需重建 JupyterLab 。
- 可将预构建包直接复制到适当的目录来安装预构建扩展，从而绕过创建 Python 包的需要。

建议在 Python 包中发布预构建扩展，以方便用户。

## 插件

- 应用程序插件：包括主菜单系统、文件浏览器以及笔记本、控制台和文件编辑器组件。
- Mime 渲染器插件：包括 pdf 查看器、JSON 查看器和 Vega 查看器。
- 主题插件：JupyterLab 附带浅色和深色主题插件。

### 应用程序插件

应用程序插件是一个具有多个元数据字段的 JavaScript 对象。

```typescript
const plugin: JupyterFrontEndPlugin<MyToken> = {
  id: "my-extension:plugin", // 必需的唯一字符串。约定使用 NPM 扩展包名称:标识扩展内插件的字符串。
  description: "Provides a new service.", // 可选字符串。允许记录插件的目的。
  autoStart: true, // 指示插件是否应该在应用程序启动时激活。
  requires: [ILabShell, ITranslator], // 对应其他插件所提供服务的令牌列表。这些服务将在插件激活时作为参数传递给 activate 函数。
  optional: [ICommandPalette], // 同 requires 。
  provides: MyToken, // 与插件提供给系统的服务关联的令牌。
  activate: activateFunction, // 插件激活时调用的函数。参数依次为应用程序对象、与 requires 、optional 令牌对应的服务。
};
```

#### 应用程序对象

应用程序对象具有许多用于与应用程序交互的属性和方法，包括

- commands：一个可扩展的注册表，用于在应用程序中添加和执行命令。
- docRegistry：一个可扩展的注册表，包含应用程序能够读取和呈现的文档类型。
- restored：当应用程序完成加载时解析的 Promise 。
- serviceManager：用于与 Jupyter REST API 交互的低级管理器。
- shell：一个通用的 Jupyter 前端 shell 实例，它包含应用程序的用户界面。

### MIME 渲染器插件

MIME 渲染器插件是一个对象，其字段列在 rendermime-interfaces IExtension 对象中。

## 源扩展

源扩展在其 package.json 文件的 jupyterlab 字段中具有元数据。

- extension：应用程序插件
- mimeExtension：MIME 渲染器插件
- themePath：主题路径
- schemaDir：插件设置
- disabledExtensions：禁用其他扩展
- sharedPackages：依赖项的去重
- discovery：配套包

JupyterLab 扩展必须至少设置 jupyterlab.extension 或 jupyterlab.mimeExtension 之一。

## 预构建扩展

预构建扩展具有额外的 jupyterlab 元数据。

- outputDir：输出目录
- webpackConfig：自定义 webpack 配置

### 开发预构建扩展

使用 jupyter labextension build 命令构建预构建扩展。

构成预构建扩展的文件包括

- 一个主入口点 remoteEntry.<hash>.js
- 捆绑到 JavaScript 文件中的依赖项
- package.json （包含一些额外的构建元数据）
- 插件设置
- 主题目录结构（如果需要）

使用 labextension develop 命令创建指向预构建输出目录的链接，类似于 pip install -e

```bash
jupyter labextension develop . --overwrite
```

然后重新构建扩展并在浏览器中刷新 JupyterLab 应该会拾取预构建扩展源代码中的更改。

### 分发预构建扩展

将其输出目录复制到 JupyterLab 可以找到的位置，通常是 <sys-prefix>/share/jupyter/labextensions/<package-name> ，其中 <package-name> 是 package.json 中的 JavaScript 包名称。

## 源扩展的开发工作流程

在编写源扩展时，可以使用命令

```bash
jlpm install # install npm package dependencies
jlpm run build # optional build step if using TypeScript, babel, etc.
jupyter labextension install # install the current directory as an extension
```

这会导致构建器在构建应用程序文件之前重新安装源文件夹。可以使用 jupyter lab build 重新构建，它将重新安装这些包。

可以使用 jupyter labextension link 链接正在同时处理的其他本地 npm 包，它们将被重新安装，但不会被视为扩展。本地扩展和链接的包包含在 jupyter labextension list 中。

当使用本地扩展和链接的包时，可以运行命令

```bash
jupyter lab --watch
```

当链接的包之一发生更改时，这将导致应用程序增量重建。如果扩展是用 TypeScript 编写的，需要运行 jlpm run build 才能使更改反映在 JupyterLab 中。为了避免此步骤，也可以监视扩展中的 TypeScript 源代码，这通常分配给 tsc -w 快捷方式。
