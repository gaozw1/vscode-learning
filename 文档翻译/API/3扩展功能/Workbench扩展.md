# 扩展Workbench

“Workbench"指的是整个Visual Studio代码UI，它包含以下UI组件:

* Title Bar
* Activity Bar
* Side Bar
* Panel
* Editor Group
* Status Bar

VS Code提供了各种api，允许你把自己的组件添加到Workbench中。例如，如下图所示:

![workbench-contribution](https://code.visualstudio.com/assets/api/extension-capabilities/extending-workbench/workbench-contribution.png)

* Activity Bar: The[Azure App Service extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) 添加了一个[视图容器](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#view-container)
* Side Bar: 内置[NPM extension](https://github.com/microsoft/vscode/tree/master/extensions/npm) 在浏览器视图中添加了一个[Tree View](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#tree-view)
* Editor Group: 内置[Markdown extension](https://github.com/microsoft/vscode/tree/master/extensions/markdown-language-features) 在编辑组中的其他编辑器旁边添加了一个[Webview](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#webview)
* Status Bar: [VSCodeVim extension](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim) 在状态栏中增加了一个[Status Bar Item](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#status-bar-item)

## 视图容器[#](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#views-container)

通过[`contributes.viewsContainers`](https://code.visualstudio.com/api/references/contribution-points#contributes.viewsContainers)贡献点，您可以添加新的视图容器，这些视图容器显示在五个内置视图容器旁边。在树形视图主题中了解更多信息。

## 树视图[#](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#tree-view)

通过[`contributes.views`](https://code.visualstudio.com/api/references/contribution-points#contributes.views)贡献点，您可以添加新视图使其在任意视图视图容器中显示。在[树视图](https://code.visualstudio.com/api/extension-guides/tree-view) 主题中了解更多信息。

## Web视图[#](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#webview)

web视图是用HTML/CSS/JavaScript构建的高可定制度的视图。它们显示在编辑器组区域的文本编辑器旁边。在[Webview指南](https://code.visualstudio.com/api/extension-guides/webview)中阅读更多关于Webview的内容。

## 状态栏项目[#](https://code.visualstudio.com/api/extension-capabilities/extending-workbench#status-bar-item)

扩展可以创建显示在状态栏的自定义[`StatusBarItem`](https://code.visualstudio.com/api/references/vscode-api#StatusBarItem)。状态栏项目可以显示文本和图标，并在单击事件上运行命令。

- 显示文本和图标
- 单击运行命令

状态栏扩展示例可以在[这里](https://github.com/microsoft/vscode-extension-samples/tree/master/statusbar-sample)找到。
