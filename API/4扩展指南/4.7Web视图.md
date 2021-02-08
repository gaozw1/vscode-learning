# Webview API

webview API允许扩展在Visual Studio代码中创建完全可定制的视图。例如，内置的Markdown扩展使用webview来呈现Markdown预览。webview还可以用来构建复杂的用户界面，其甚至可以超出了VS Code原生api的支持范围。

可以把webview想成你的扩展控制的VS Code中的`iframe`。webview可以在这个框架中呈现几乎所有的HTML内容，并且它使用消息传递与扩展进行通信。这种自由让webview变得异常强大，并开启了一系列全新的扩展可能性。

webview在几个VS Code api中被使用:

- 使用`createWebviewPanel`创建Webview面板。在这种情况下，Webview面板在VS Code中显示为独特的编辑器。这使得它们对于显示自定义UI和自定义可视化非常有用。
- 作为[自定义编辑器](https://code.visualstudio.com/api/extension-guides/custom-editors)的视图。自定义编辑器允许扩展程序为编辑工作区中的任何文件提供自定义UI。自定义编辑器API还允许您的扩展接入编辑器事件，如撤消和重做，以及文件事件，如保存。
- 在侧边栏或面板区域中渲染的[Webview 视图](https://code.visualstudio.com/api/references/vscode-api#WebviewView)中。更多细节请参见[webview 视图示例扩展](https://github.com/microsoft/vscode-extension-samples/tree/master/webview-view-sample)。

本页面主要关注基本的webview面板API，尽管这里所涵盖的几乎所有内容都适用于自定义编辑器和webview视图中使用的webview视图。即使你对这些api更感兴趣，我们也建议你先阅读这篇文章，熟悉webview的基础知识。

## 链接#

* [Webview Sample](https://github.com/microsoft/vscode-extension-samples/blob/master/webview-sample/README.md)
* [Custom Editors Documentation](https://code.visualstudio.com/api/extension-guides/custom-editors)
* [Webview View Sample](https://github.com/microsoft/vscode-extension-samples/tree/master/webview-view-sample)

## VS Code API使用

* [`window.createWebviewPanel`](https://code.visualstudio.com/api/references/vscode-api#window.createWebviewPanel)
* [`window.registerWebviewPanelSerializer`](https://code.visualstudio.com/api/references/vscode-api#window.registerWebviewPanelSerializer)

## 我应该使用webview吗

webview是非常神奇的，但也应该少用，并且只在VS Code的原生API不够充分的时候使用。webview是资源密集型的，并且运行在独立于普通扩展的上下文中。设计糟糕的webview也很容易让人觉得与VS Code格格不入。

在使用webview之前，请考虑以下事项:

- 这个功能真的需要存在于VS Code中吗?作为一个单独的应用程序或网站会更好吗?
- webview是实现特性的唯一方法吗?你可以使用常规的VS Code api来代替吗?
- 你的webview是否能增加足够的用户价值，以证明它的高资源成本是合理的?

记住：仅仅因为你可以用webview做一些事情，并不意味着你应该这么做。然而，如果您确信自己需要使用webviews，那么本文档将提供帮助。让我们开始吧。

## webview API基础

为了解释webview API，我们将构建一个简单的扩展，称为**Cat Coding**。这个扩展将使用webview来显示猫在写代码(假定是在VS Code中)的gif。当我们使用这个API时，我们将继续向扩展添加功能，包括跟踪我们的猫写了多少行源代码的计数器，以及当猫引入bug时通知用户的通知。

这个`package.json`是**Cat Coding**扩展的第一个版本。您可以在[这里](https://github.com/microsoft/vscode-extension-samples/blob/master/webview-sample/README.md)找到示例应用程序的完整代码。我们的扩展的第一个版本[提供了一个命令](https://code.visualstudio.com/api/references/contribution-points#contributes.commands)，名为`catCoding.start`。当用户调用这个命令时，我们将显示一个包含我们的猫的简单webview。用户将能够从**命令面板**调用这个命令**Cat Coding: Start new cat coding session**，如果他们愿意的话，甚至可以为它创建一个快捷键。

```json
{
  "name": "cat-coding",
  "description": "Cat Coding",
  "version": "0.0.1",
  "publisher": "bierner",
  "engines": {
    "vscode": "^1.23.0"
  },
  "activationEvents": ["onCommand:catCoding.start"],
  "main": "./out/src/extension",
  "contributes": {
    "commands": [
      {
        "command": "catCoding.start",
        "title": "Start new cat coding session",
        "category": "Cat Coding"
      }
    ]
  },
  "scripts": {
    "vscode:prepublish": "tsc -p ./",
    "compile": "tsc -watch -p ./",
    "postinstall": "node ./node_modules/vscode/bin/install"
  },
  "dependencies": {
    "vscode": "*"
  },
  "devDependencies": {
    "@types/node": "^9.4.6",
    "typescript": "^2.8.3"
  }
}
```

现在让我们实现`catCoding.start`命令。在扩展名的主文件中，我们注册了`catCoding.start`命令，并使用它来显示基本的webview:

```ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      // Create and show a new webview
      const panel = vscode.window.createWebviewPanel(
        'catCoding', // Identifies the type of the webview. Used internally
        'Cat Coding', // Title of the panel displayed to the user
        vscode.ViewColumn.One, // Editor column to show the new webview panel in.
        {} // Webview options. More on these later.
      );
    })
  );
}
```

`vscode.window.createWebviewPanel`函数在编辑器中创建并显示一个webview。如果你尝试在当前状态下运行`catCoding.start`命令，你会看到以下内容：

![catcoding](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-no_content.png)

我们的命令打开了一个新的有正确的标题webview面板，但没有内容！要将我们的猫添加到新面板，我们还需要使用`webview.html`设置webview的HTML内容:

```ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      // Create and show panel
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {}
      );

      // And set its HTML content
      panel.webview.html = getWebviewContent();
    })
  );
}

function getWebviewContent() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
</body>
</html>`;
}
```

如果你再次运行指令，现在webview如下图：

![likethis](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-html.png)

Progress!

`webview.html`应该始终是一个完整的HTML文档。HTML片段或畸形的HTML可能会导致意外的行为。



### 更新webview内容

`webview.html`也可以在webview内容被创建之后更新它。让我们通过引入猫的旋转来让**Cat Coding**更加动态:

```ts
import * as vscode from 'vscode';

const cats = {
  'Coding Cat': 'https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif',
  'Compiling Cat': 'https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif'
};

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {}
      );

      let iteration = 0;
      const updateWebview = () => {
        const cat = iteration++ % 2 ? 'Compiling Cat' : 'Coding Cat';
        panel.title = cat;
        panel.webview.html = getWebviewContent(cat);
      };

      // Set initial content
      updateWebview();

      // And schedule updates to the content every second
      setInterval(updateWebview, 1000);
    })
  );
}

function getWebviewContent(cat: keyof typeof cats) {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="${cats[cat]}" width="300" />
</body>
</html>`;
}
```

设置`webview.html`替换整个webview内容，类似于重新加载iframe。一旦你开始在webview中使用脚本，记住这一点很重要，因为这意味着设置`webview.html`也会重置脚本的状态。

上面的例子也使用了`webview.title`更改在编辑器中显示的文档的标题。设置标题不会导致webview被重新加载。

### 生命周期#

Webview面板属于创建它们的扩展。扩展必须保持从`createWebviewPanel`返回的webview。如果你的扩展失去了这个引用，它不能重新获得对webview的访问，即使webview将继续在VS Code中显示。

与文本编辑器一样，用户也可以随时关闭webview面板。当一个webview面板被用户关闭时，webview本身就会被销毁。试图使用已销毁的webview会抛出异常。这意味着上面使用`setInterval`的例子实际上有一个严重的bug：如果用户关闭面板，`setInterval`将继续触发，它将尝试更新`panel.webview.html`，这当然会抛出异常。猫讨厌例外。让我们解决这个问题!

当一个webview被销毁时，`onDidDispose`事件被触发。我们可以使用这个事件来取消进一步的更新和清理webview的资源:

```js
import * as vscode from 'vscode';

const cats = {
  'Coding Cat': 'https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif',
  'Compiling Cat': 'https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif'
};

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {}
      );

      let iteration = 0;
      const updateWebview = () => {
        const cat = iteration++ % 2 ? 'Compiling Cat' : 'Coding Cat';
        panel.title = cat;
        panel.webview.html = getWebviewContent(cat);
      };

      updateWebview();
      const interval = setInterval(updateWebview, 1000);

      panel.onDidDispose(
        () => {
          // When the panel is closed, cancel any future updates to the webview content
          clearInterval(interval);
        },
        null,
        context.subscriptions
      );
    })
  );
}
```

扩展也可以通过调用 `dispose() `以编程方式关闭webview。例如，如果我们想把猫咪的工作时间限制在5秒内:

```ts
export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {}
      );

      panel.webview.html = getWebviewContent(cats['Coding Cat']);

      // After 5sec, pragmatically close the webview panel
      const timeout = setTimeout(() => panel.dispose(), 5000);

      panel.onDidDispose(
        () => {
          // Handle user closing panel before the 5sec have passed
          clearTimeout(timeout);
        },
        null,
        context.subscriptions
      );
    })
  );
}


```

### 可见性和移动

当一个webview面板移动到背景标签时，它会被隐藏。然而，它并没有被摧毁。当面板再次被移到前台时，VS Code会自动从`webview.html`中恢复webview的内容:

![visibility](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-restore.gif)

`.visible`属性告诉你webview面板当前是否可见。

扩展可以通过调用`reveal()`以编程方式将webview面板带到前台。此方法采用一个可选的目标视图列来显示面板。一个webview面板一次只能显示在一个编辑器列中。调用`reveal()`或拖动一个webview面板到一个新的编辑器列，将webview移动到那个新的列。

![reveal](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-drag.gif)

让我们更新我们的扩展，同一时间只允许一个webview存在。如果面板是在后台，那么`catCoding.start`命令会把它带到前台:

```ts
export function activate(context: vscode.ExtensionContext) {
  // Track currently webview panel
  let currentPanel: vscode.WebviewPanel | undefined = undefined;

  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const columnToShowIn = vscode.window.activeTextEditor
        ? vscode.window.activeTextEditor.viewColumn
        : undefined;

      if (currentPanel) {
        // If we already have a panel, show it in the target column
        currentPanel.reveal(columnToShowIn);
      } else {
        // Otherwise, create a new panel
        currentPanel = vscode.window.createWebviewPanel(
          'catCoding',
          'Cat Coding',
          columnToShowIn,
          {}
        );
        currentPanel.webview.html = getWebviewContent(cats['Coding Cat']);

        // Reset when the current panel is closed
        currentPanel.onDidDispose(
          () => {
            currentPanel = undefined;
          },
          null,
          context.subscriptions
        );
      }
    })
  );
}
```

现在运行的是新的扩展：

![new](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-single_panel.gif)

每当一个webview的可见性改变，或者当一个webview被移动到一个新的列，`onDidChangeViewState`事件被触发。我们的扩展可以使用这个事件来改变猫基于哪个列的webview显示:

```ts
const cats = {
  'Coding Cat': 'https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif',
  'Compiling Cat': 'https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif',
  'Testing Cat': 'https://media.giphy.com/media/3oriO0OEd9QIDdllqo/giphy.gif'
};

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {}
      );
      panel.webview.html = getWebviewContent(cats['Coding Cat']);

      // Update contents based on view state changes
      panel.onDidChangeViewState(
        e => {
          const panel = e.webviewPanel;
          switch (panel.viewColumn) {
            case vscode.ViewColumn.One:
              updateWebviewForCat(panel, 'Coding Cat');
              return;

            case vscode.ViewColumn.Two:
              updateWebviewForCat(panel, 'Compiling Cat');
              return;

            case vscode.ViewColumn.Three:
              updateWebviewForCat(panel, 'Testing Cat');
              return;
          }
        },
        null,
        context.subscriptions
      );
    })
  );
}

function updateWebviewForCat(panel: vscode.WebviewPanel, catName: keyof typeof cats) {
  panel.title = catName;
  panel.webview.html = getWebviewContent(cats[catName]);
}
```

![ondidchange](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-ondidchangeviewstate.gif)

### 检查和调试webviews

**Developer: Open Webview Developer Tools** VS Code命令让你调试Webview。运行该命令会为当前可见的webview启动一个Developer Tools实例:

![developer](https://code.visualstudio.com/assets/api/extension-guides/webview/basics-developer_tools.png)

webview的内容在webview文档的iframe中。您可以使用开发人员工具来检查和修改webview的DOM，并调试在webview中运行的脚本。

如果您使用webview Developer Tools控制台，请确保从控制台面板左上角的下拉菜单中选择**active frame**环境:

![activeframe](https://code.visualstudio.com/assets/api/extension-guides/webview/debug-active-frame.png)


**active frame**环境是webview脚本本身被执行的地方。

此外，**Developer: Reload Webview**命令会重新加载所有活动的Webview。如果你需要重置webview的状态，或者磁盘上的一些webview内容发生了变化，而你想要加载新内容，这可能会很有帮助。

## 加载本地内容#

webview运行在独立的上下文中，不能直接访问本地资源。这样做是出于安全原因。这意味着为了从扩展加载图像、样式表和其他资源，或者从用户当前工作区加载任何内容，您必须使用`Webview.asWebviewUri`函数将一个本地`file:`URI转换成一个特殊的URI, VS Code可以使用它来加载本地资源的子集。

想象一下，我们想要开始将猫的gif打包到我们的扩展中，而不是从Giphy中将它们拉出来。为此，我们首先创建一个URI到磁盘上的文件，然后通过`asWebviewUri`函数传递这些URI:

```ts
import * as vscode from 'vscode';
import * as path from 'path';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {}
      );

      // Get path to resource on disk
      const onDiskPath = vscode.Uri.file(
        path.join(context.extensionPath, 'media', 'cat.gif')
      );

      // And get the special URI to use with the webview
      const catGifSrc = panel.webview.asWebviewUri(onDiskPath);

      panel.webview.html = getWebviewContent(catGifSrc);
    })
  );
}
```

如果我们调试这段代码，我们会看到`catGifSrc`的实际值是这样的:

```
vscode-resource:/Users/toonces/projects/vscode-cat-coding/media/cat.gif
````

VS Code理解这个特殊的URI，并将使用它从磁盘加载我们的gif !

默认情况下，webview只能访问以下位置的资源:

- 在扩展的安装目录中。

- 在用户当前活动的工作区中。

使用`WebviewOptions.localResourceRoots`允许访问额外的本地资源。

还可以使用数据uri直接在webview中嵌入资源。

### 控制对本地资源的访问

webview可以通过`localResourceRoots`选项控制哪些资源可以从用户的机器上加载。`localresourceroot`定义了一组根uri，可以从中加载本地内容。

我们可以使用`localResourceRoots`来限制**Cat Coding**的webview只从我们扩展的`media`目录中加载资源:

```ts
import * as vscode from 'vscode';
import * as path from 'path';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {
          // Only allow the webview to access resources in our extension's media directory
          localResourceRoots: [vscode.Uri.file(path.join(context.extensionPath, 'media'))]
        }
      );

      const onDiskPath = vscode.Uri.file(
        path.join(context.extensionPath, 'media', 'cat.gif')
      );
      const catGifSrc = panel.webview.asWebviewUri(onDiskPath);

      panel.webview.html = getWebviewContent(catGifSrc);
    })
  );
}
```

如果要禁止所有本地资源，只需将`localresourceroot`设置为`[]`。

一般来说，webview在加载本地资源时应该尽可能限制。但是，请记住，`localresourceroot`本身并不提供完整的安全保护。确保您的webview也遵循[安全最佳实践](https://code.visualstudio.com/api/extension-guides/webview#security)，并添加一个[内容安全策略](https://code.visualstudio.com/api/extension-guides/webview#content-security-policy)来进一步限制可以加载的内容。

### 主题webview内容

Webview可以使用CSS来根据VS Code的当前主题改变它们的外观。VS Code将主题分为三类，并在`body`元素中添加一个特殊的类来指示当前的主题:

- `vscode-light`-亮主题。
- `vscode-dark`-暗主题。
- `vscode-high-contrast`-高对比主题。

下面的CSS根据用户当前的主题改变了webview的文本颜色:

```css
body.vscode-light {
  color: black;
}

body.vscode-dark {
  color: white;
}

body.vscode-high-contrast {
  color: red;
}
```

在开发webview应用程序时，确保它适用于这三种类型的主题。总是在高对比度模式下测试你的webview，以确保它对有视觉障碍的人可用。

webview也可以使用[CSS变量](https://developer.mozilla.org/docs/Web/CSS/Using_CSS_variables)访问VS Code主题颜色。这些变量名的前缀是`vscode`，并使用`.`替换`-`。例如`editor.foreground`变成`var(--vscode-editor-foreground)`：

```css
code {
  color: var(--vscode-editor-foreground);
}
```

在[主题颜色参考](https://code.visualstudio.com/api/references/theme-color)中查看可用的主题变量。[这个扩展](https://marketplace.visualstudio.com/items?itemName=connor4312.css-theme-completions)是可用的，它为变量提供智能感知建议。

以下与字体相关的变量也被定义:

- `--vscode-editor-font-family` - 编辑器字体样式(来自`editor.fontFamily`设置)。
- `--vscode-editor-font-weight` - 编辑器字体粗细(来自`editor.fontWeight`设置)。
- `--vscode-editor-font-size` -编辑器字体大小(来自`editor.fontSize`设置)。

最后，对于需要编写针对单个主题的CSS的特殊情况，webviews的body元素有一个名为`vcode -theme-name`的新数据属性，它存储当前活动主题的全名。这让你可以为webview编写特定于主题的CSS:

```css
body[data-vscode-theme-name="One Dark Pro"] {
    background: hotpink;
}


```

## 脚本和消息传递#

webview就像iframes一样，这意味着它们也可以运行脚本。JavaScript在webview中默认是禁用的，但它可以通过传入`enableScripts: true`选项轻松重新启用。

让我们使用一个脚本添加一个计数器来跟踪我们的猫所编写的源代码行。运行一个基本脚本非常简单，但请注意，这个示例仅用于演示目的。在实践中，你的webview应该使用[内容安全策略](https://code.visualstudio.com/api/extension-guides/webview#content-security-policy)禁用内联脚本:

```ts
import * as path from 'path';
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {
          // Enable scripts in the webview
          enableScripts: true
        }
      );

      panel.webview.html = getWebviewContent();
    })
  );
}

function getWebviewContent() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;
        }, 100);
    </script>
</body>
</html>`;
}
```

![script](https://code.visualstudio.com/assets/api/extension-guides/webview/scripts-basic.gif)

哇!那是效率很高的猫。

Webview脚本可以做任何普通网页上的脚本所能做的事情。请记住，webview存在于它们自己的上下文中，所以webview中的脚本不能访问VS Code API。于是消息传递的用武之地就来了!

### 将消息从扩展传递到webview

扩展可以使用`webview.postMessage()`将数据发送到它的webview。该方法将任何可序列化的JSON数据发送到webview。消息在webview中通过标准`message`事件接收。

为了演示这一点，让我们向**Cat Coding**添加一个新命令，指示当前正在编码的猫重构它们的代码(从而减少总行数)。新`catCoding.doRefactor`命令使用`postMessage`将指令发送到当前webview和`window.addEventListener('message', event => { ... })`在webview内部处理消息:

```ts
export function activate(context: vscode.ExtensionContext) {
  // Only allow a single Cat Coder
  let currentPanel: vscode.WebviewPanel | undefined = undefined;

  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      if (currentPanel) {
        currentPanel.reveal(vscode.ViewColumn.One);
      } else {
        currentPanel = vscode.window.createWebviewPanel(
          'catCoding',
          'Cat Coding',
          vscode.ViewColumn.One,
          {
            enableScripts: true
          }
        );
        currentPanel.webview.html = getWebviewContent();
        currentPanel.onDidDispose(
          () => {
            currentPanel = undefined;
          },
          undefined,
          context.subscriptions
        );
      }
    })
  );

  // Our new command
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.doRefactor', () => {
      if (!currentPanel) {
        return;
      }

      // Send a message to our webview.
      // You can send any JSON serializable data.
      currentPanel.webview.postMessage({ command: 'refactor' });
    })
  );
}

function getWebviewContent() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;
        }, 100);

        // Handle the message inside the webview
        window.addEventListener('message', event => {

            const message = event.data; // The JSON data our extension sent

            switch (message.command) {
                case 'refactor':
                    count = Math.ceil(count * 0.5);
                    counter.textContent = count;
                    break;
            }
        });
    </script>
</body>
</html>`;
}
```

![refactor](https://code.visualstudio.com/assets/api/extension-guides/webview/scripts-extension_to_webview.gif)

### 将消息从webview传递到扩展#

webview也可以将消息传递回它们的扩展。这是通过webview中一个特殊的VS Code API对象的`postMessage`函数实现的。要访问VS Code API对象，在webview中调用`acquireVsCodeApi`。这个函数在每个会话中只能被调用一次。你必须挂起这个方法返回的VS Code API实例，并把它交给任何其他想要使用它的函数。

我们可以在我们的**Cat Coding**webview中使用VS Code API和`postMessage`来提醒扩展，当我们的猫在代码中引入bug时:

```ts
export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {
          enableScripts: true
        }
      );

      panel.webview.html = getWebviewContent();

      // Handle messages from the webview
      panel.webview.onDidReceiveMessage(
        message => {
          switch (message.command) {
            case 'alert':
              vscode.window.showErrorMessage(message.text);
              return;
          }
        },
        undefined,
        context.subscriptions
      );
    })
  );
}

function getWebviewContent() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        (function() {
            const vscode = acquireVsCodeApi();
            const counter = document.getElementById('lines-of-code-counter');

            let count = 0;
            setInterval(() => {
                counter.textContent = count++;

                // Alert the extension when our cat introduces a bug
                if (Math.random() < 0.001 * count) {
                    vscode.postMessage({
                        command: 'alert',
                        text: '🐛  on line ' + count
                    })
                }
            }, 100);
        }())
    </script>
</body>
</html>`;
}
```

![bug](https://code.visualstudio.com/assets/api/extension-guides/webview/scripts-webview_to_extension.gif)

出于安全原因，你必须保持VS Code API对象的私有，并确保它不会泄露到全局作用域。

## 安全#

与任何网页一样，在创建webview时，您必须遵循一些基本的安全性最佳实践。

### 限制功能#

webview应该具有它所需要的最小功能集。例如，如果你的webview不需要运行脚本，不要设置`enablesscripts: true`。如果你的webview不需要从用户的工作空间加载资源，设置`localResourceRoots`为`[vscode.Uri.file(extensionContext.extensionPath)]`甚至`[]`来禁止访问所有本地资源。

### 内容安全政策#

[内容安全策略](https://developers.google.com/web/fundamentals/security/csp/)进一步限制了可以在webview中加载和执行的内容。例如，一个内容安全策略可以确保在webview中只能运行允许的脚本列表，或者甚至告诉webview只能通过`https`加载图像。

要添加一个内容安全策略，在webview的`<head>`放一个`<meta http-equiv="Content-Security-Policy">`指令

```ts
function getWebviewContent() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">

    <meta http-equiv="Content-Security-Policy" content="default-src 'none';">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Cat Coding</title>
</head>
<body>
    ...
</body>
</html>`;
}
```

`default-src 'none';`策略禁止所有的内容。然后，我们可以返回扩展功能所需的最小数量的内容。这里有一个内容安全策略，允许加载本地脚本和样式表，并通过`https`加载图像:

```ts
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'none'; img-src ${webview.cspSource} https:; script-src ${webview.cspSource}; style-src ${webview.cspSource};"
/>
```

`${webview.cspSource}`值是一个来自webview对象本身的值的占位符。关于如何使用这个值的完整示例，请参阅[webview示例](https://github.com/microsoft/vscode-extension-samples/blob/master/webview-sample)。

此内容安全策略还隐式地禁用内联脚本和样式。最好的做法是将所有内联样式和脚本提取到外部文件中，这样就可以在不违背内容安全策略的情况下正确加载它们。

### 仅通过https加载内容

如果你的webview允许加载外部资源，强烈建议你只允许通过`https`加载这些资源，而不允许通过http加载。上面的示例内容安全策略已经做到了这一点，它只允许图像通过`https:`加载。

### 清除所有用户输入#

与普通网页一样，在为webview构建HTML时，必须清除所有用户输入。如果不能正确地清理输入，可能会导致内容注入，这可能会给用户带来安全风险。

需要清理的取值示例:

- 文件内容。

- 文件和文件夹路径。

- 用户和工作区设置。

考虑使用一个助手库来构造HTML字符串，或者至少确保用户工作区中的所有内容都得到了适当的处理。

永远不要仅仅依靠清理来保证安全。确保遵循其他安全最佳实践，例如使用[内容安全策略](https://code.visualstudio.com/api/extension-guides/webview#content-security-policy)来最小化任何潜在内容注入的影响。

## 持久性#

在标准的webview[生命周期](https://code.visualstudio.com/api/extension-guides/webview#lifecycle)中，webview由`createWebviewPanel`创建，并在用户关闭它们或调用`.dispose()`时销毁。然而，webview的内容是在webview变为可见时创建的，当webview被移到后台时销毁。当webview被移动到背景标签时，webview中的任何状态都会丢失。

解决这个问题的最好方法是让你的webview无状态。使用[消息传递](https://code.visualstudio.com/api/extension-guides/webview#passing-messages-from-a-webview-to-an-extension)保存webview的状态，然后当webview再次变为可见时恢复状态。

### getState和设置状态#

运行在webview中的脚本可以使用`getState`和`setState`方法来保存和恢复一个可序列化的JSON状态对象。即使当一个webview面板被隐藏时，webview内容本身被销毁，这种状态也会持续存在。当webview面板被销毁时，状态也被销毁。

```ts
// Inside a webview script
const vscode = acquireVsCodeApi();

const counter = document.getElementById('lines-of-code-counter');

// Check if we have an old state to restore from
const previousState = vscode.getState();
let count = previousState ? previousState.count : 0;
counter.textContent = count;

setInterval(() => {
  counter.textContent = count++;
  // Update the saved state
  vscode.setState({ count });
}, 100);
```

`getState`和`setState`是保存状态的首选方式，因为它们的性能开销比`retainContextWhenHidden`要低得多。

### 序列化#

通过实现一个`WebviewPanelSerializer`，你的webview可以在VS Code重启时自动恢复。序列化建立在`getState`和`setState`上，并且只有当你的扩展为你的webview注册了`WebviewPanelSerializer`时才启用。

为了让我们的编码猫在VS Code重启时保持不变，首先添加一个`onWebviewPanel`激活事件到扩展的`package.json`:

```json
"activationEvents": [
    ...,
    "onWebviewPanel:catCoding"
]
```

无论VS Code何时需要恢复一个视图标签为 `catCoding`的webview，这个激活事件确保我们的扩展将被激活。

然后，在扩展的`activate`方法中，调用`registerWebviewPanelSerializer`来注册一个新的`WebviewPanelSerializer`。这个`WebviewPanelSerializer`负责从它的持久化状态恢复webview的内容。这个状态是webview内容使用`setState`设置的JSON blob。

```ts
export function activate(context: vscode.ExtensionContext) {
  // Normal setup...

  // And make sure we register a serializer for our webview type
  vscode.window.registerWebviewPanelSerializer('catCoding', new CatCodingSerializer());
}

class CatCodingSerializer implements vscode.WebviewPanelSerializer {
  async deserializeWebviewPanel(webviewPanel: vscode.WebviewPanel, state: any) {
    // `state` is the state persisted using `setState` inside the webview
    console.log(`Got state: ${state}`);

    // Restore the content of our webview.
    //
    // Make sure we hold on to the `webviewPanel` passed in here and
    // also restore any event listeners we need on it.
    webviewPanel.webview.html = getWebviewContent();
  }
}
```

现在如果你重启VS Code，打开一个cat coding面板，面板将自动恢复到相同的编辑器位置。

### retainContextWhenHidden #

对于具有非常复杂的UI或状态且不能快速保存和恢复的webview，你可以使用`retainContextWhenHidden`选项代替。这个选项使webview保持其内容在周围，但处于隐藏状态，即使webview本身不再在前台。

虽然**Cat Coding**很难说有复杂的状态，让我们尝试启用`retainContextWhenHidden`来看看这个选项是如何改变webview的行为的:

```ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',
        'Cat Coding',
        vscode.ViewColumn.One,
        {
          enableScripts: true,
          retainContextWhenHidden: true
        }
      );
      panel.webview.html = getWebviewContent();
    })
  );
}

function getWebviewContent() {
  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;
        }, 100);
    </script>
</body>
</html>`;
}
```

![retainContextWhenHidden](https://code.visualstudio.com/assets/api/extension-guides/webview/persistence-retrain.gif)

注意，当webview被隐藏然后恢复时，计数器不会重置。不需要额外的代码!使用`retainContextWhenHidden`, webview的行为类似于web浏览器中的后台标签。脚本和其他动态内容会被挂起，但一旦webview再次可见，就会立即恢复。即使启用了`retainContextWhenHidden`，你也不能向隐藏的webview发送消息。

尽管`retainContextWhenHidden`可能很吸引人，但请记住，这有很高的内存开销，应该只在其他持久性技术不起作用时使用。

## 下一步#

如果你想了解更多关于VS Code可扩展性的知识，试试这些主题:

- [扩展API](../1概述/概述.md) -了解完整的VS Code扩展API。
- [扩展功能](../3扩展功能/概述.md)——看看其他扩展VS Code的方法。
