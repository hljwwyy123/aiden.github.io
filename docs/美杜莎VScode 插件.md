# 美杜莎VScode 插件

# 背景
奥运票务相关系统(ATP/CTP/GP)是一个国际化的系统，涉及到多语言的文案需求。国际化方案选型集团的美杜莎，而团队内部80% 小伙伴使用VScode，所以在为了提高在开发过程中对文案创建、编辑、查看的效率，抽空开发了一个VScode的插件。传送门：[https://marketplace.visualstudio.com/items?itemName=Aiden.mdsi18n](https://marketplace.visualstudio.com/items?itemName=Aiden.mdsi18n)
为什么想要在**开发时**做这样一个工具，是因为在没有这个工具之前的链路是这样的：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/218124/1612773424861-ac932661-f3c9-4630-87b5-6cde9f57114b.png#align=left&display=inline&height=433&margin=%5Bobject%20Object%5D&name=image.png&originHeight=866&originWidth=1460&size=209231&status=done&style=none&width=730)
如果有这样一个VScode 插件可以做到：


![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/218124/1612773436778-80b22040-5509-46ab-88fc-9b4964ea30b6.png#align=left&display=inline&height=404&margin=%5Bobject%20Object%5D&name=image.png&originHeight=808&originWidth=1070&size=143124&status=done&style=none&width=535)
这样就节省了切换vscode/chrome以及copy文案、在美杜莎页面里搜索文案的时间。
如果创建/编辑一个key能节省1分钟，创建100个key就能节省一个多小时的时间 🙃，每用一次mdsi18n插件就能延长1min的寿命。。。


# 能力
注：对文案的创建、编辑、查找都是基于美杜莎现有的OpenApi能力，全都是异步请求，没有对本地 i18n.json /i18n excel等files的读写。
#### 1. Hover到美杜莎Key上查看所对应的翻译结果
![vscode-hover-plugin.gif](https://intranetproxy.alipay.com/skylark/lark/0/2021/gif/218124/1612770873212-8f2bf63c-3742-4a16-b11a-2eab60f1613f.gif#align=left&display=inline&height=362&margin=%5Bobject%20Object%5D&name=vscode-hover-plugin.gif&originHeight=362&originWidth=916&size=53289&status=done&style=none&width=916)
#### 2. 创建新的美杜莎Key
![newmdskey.gif](https://intranetproxy.alipay.com/skylark/lark/0/2021/gif/218124/1612770909083-370e8a77-e204-46ea-a5a9-d540c2c8e78d.gif#align=left&display=inline&height=759&margin=%5Bobject%20Object%5D&name=newmdskey.gif&originHeight=759&originWidth=922&size=264280&status=done&style=none&width=922)
#### 3. 编辑已有的美杜莎文案
![editKey.gif](https://intranetproxy.alipay.com/skylark/lark/0/2021/gif/218124/1612770933493-2b30e7c7-5a9e-42a5-ac12-a93632739927.gif#align=left&display=inline&height=757&margin=%5Bobject%20Object%5D&name=editKey.gif&originHeight=757&originWidth=925&size=407140&status=done&style=none&width=925)
#### 4. 选中文字查询对应的美杜莎Key
![mds-getKeyByValue.gif](https://intranetproxy.alipay.com/skylark/lark/0/2021/gif/218124/1612770955721-3c67f79e-5b8b-41f1-8e7a-4b8171a27cae.gif#align=left&display=inline&height=700&margin=%5Bobject%20Object%5D&name=mds-getKeyByValue.gif&originHeight=700&originWidth=1020&size=389007&status=done&style=none&width=1020)


# 安装及使用
可通过上面传送门链接通过 微软的marketplace下载。
VScode 插件面板内搜索 **mdsi18n **安装。


插件提供了四个配置项：
```json
"properties": {
  "mdsi18n.appName": {
    "type": "string",
    "default": "vscode-mdsi18n",
    "description": "i18n美杜莎应用名称"	
  },
  "mdsi18n.keyPrefix": {
    "type": "string",
    "default": "xx.x",
    "description": "i18n美杜莎Key前缀"
  },
  "mdsi18n.userId": {
    "type": "number",
    "default": "",
    "description": "鉴权id, 增加、修改文案必填"
  },
  "mdsi18n.defaultTagMapToKeyPrefix": {
    "type": "string",
    "default": "[{\"prefix\":\"fe.gp\",\"tag\":[\"GP\",\"Test\"]}]",
    "description": "匹配特定前缀下默认标签. 如：[{ \"prefix\":\"fe.gp\", \"tag\": [\"GP\"] }, { \"prefix\":\"atp.fe.seat\", \"tag\": [\"ATP\",\"Seat\"] }]"
  }
}
```
该份配置可配置到两个作用域中：User全局 / 当前 workespace。所以当切换到其他美杜莎应用的项目中时，可将插件的配置 配置到workspace中。
配置完成之后刷新当前workespace 或者重新启动VScode，因为插件的挂载是在VScode onLanuch 的生命周期内挂载。


# VScode插件开发指南
在[**mdsi18n**](https://marketplace.visualstudio.com/items?itemName=Aiden.mdsi18n)插件中共使用到了 vscode extension提供的以下能力：

- Command
   - 展示创建/编辑美杜莎文案的Webview
   - 查找选中文案对应的美杜莎key
- HoverProvider - hover到美杜莎key展示value
- Menu - 选中文案后右键菜单
- webview - 创建、编辑美杜莎web页面



根据上述插件能力 贴一下核心代码逻辑：
#### 1.Hover到美杜莎Key上查看所对应的翻译结果
```javascript
// 声明HoverProvider
const hoverDisposable = languages.registerHoverProvider(
  ['typescript', 'javascript', 'typescriptreact', 'javascriptreact', 'json'], 	// registerHoverProvider第一个参数为当前provider生效的文件类型： ts、js、tsx、jsx、json
  {
    async provideHover(document: TextDocument, position: Position, token: CancellationToken ) {
      const line = document.lineAt(position).text; // 光标所在的行
      const keyRegExp = new RegExp(`[\`|\'|\"]${KEY_PREFIX}[^\'|\'|\"]*[\`|\'|\"]`, 'g');	// 匹配符合前缀的内容
      let positionMatch: RegExpMatchArray | null = line.match(keyRegExp);
      if (positionMatch && positionMatch.length) {
        let positionWord = document.getText(document.getWordRangeAtPosition(position, keyRegExp));
        positionWord = positionWord.replace(REPLACE_SPACE_REG, '$1');
        const mcm: MarkdownString = await getValueFromHoverText(positionWord);	// 这里返回的MarkdownString就是Hover之后展示的内容
        return new Hover(mcm);
      }
    }
  }
);

// -  然后在 vscode extension的统一入口 extension.ts中挂载 HoverProvider
// this method is called when your extension is activated
// your extension is activated the very first time the command is executed
export function activate(context: vscode.ExtensionContext) {
	// This line of code will only be executed once when your extension is activated
  // ...
	context.subscriptions.push(hoverDisposable);
  // ...
}
```
#### 2.创建/编辑美杜莎Key
```javascript
// 还是基于上面HoverProvider，因为创建/编辑的入口都在Hover上，只是getValueFromHoverText这里需要处理返回内容
// 如果当前Hover的Key没有找到美杜莎文案
export async function getValueFromHoverText(mdsKey: string): Promise<MarkdownString> {
  // ...
  const webViewArgs = { webviewType: "Create", data: { mdsKey }};	// command 携带参数传递给webview，参数必须经过如下处理
  const commandUri = Uri.parse(`command:${Command.showWebview}?${encodeURIComponent(JSON.stringify(webViewArgs))}`);	// 你细品
  let noKeyTip = `*没有找到对应美杜莎文案* \n\n [创建Key](${commandUri} "创建美杜莎key")`;
  const markdown = new MarkdownString(noKeyTip, true);
  markdown.isTrusted = true;	// 如果想让Hover执行command 一定要设置 isTrusted，否则vscode将不识别命令
  return markdown;
}

// 如果当前Hover的Key 已经存在美杜莎文案，提供编辑入口
export async function getValueFromHoverText(mdsKey: string): Promise<MarkdownString> {
  const markdown: MarkdownString = new MarkdownString();
  // ...
  const webViewArgs = {
    webviewType: "Edit",
    data: {
      mdsKey,
      zhCn: valueMap.get('Simplified Chinese'),
      enUs: valueMap.get('english'),
      tagName: handleTagName(data[0].tagName || ""),
      description: data[0].description || ""
    }
	};
  const commandUri = Uri.parse(`command:${Command.showWebview}?${encodeURIComponent(JSON.stringify(webViewArgs))}`);
  let noKeyTip = `&nbsp;&nbsp;[编辑文案](${commandUri} "编辑美杜莎文案")`;
  markdown.appendMarkdown(noKeyTip);
  markdown.isTrusted = true;
  return markdown;
}

```
#### 3.选中文字查询对应的美杜莎Key
```javascript
// 查找美杜莎Key 实际是一个Command，可以通过右键菜单，或者command + shift + p 唤起
export default class FindSelectionCommand {
  public registerCommand(context: ExtensionContext) {
    const editor: TextEditor | undefined = window.activeTextEditor;	// 当前vscode激活的编辑窗口，即光标所在的编辑器窗口
    context.subscriptions.push(commands.registerCommand(Command.findSelection, async (args: FindSelectionCommandArgument) => {
      if (!args || !editor) { return; }
      const text = editor.document.getText(editor.selection);	// 可以通过 editor.selection 直接获取当前所在编辑器选中的内容
      if (!text) {
        return;
      }
      await getMdsKeyByValue(text);	// 这里处理美杜莎openApi请求, 然后通过quickPick的形式展示获取到的美杜莎Key
    }));
  }
}

export default async function getMdsKeyByValue(value: string): Promise<void> {
  const itemList: QuickPickItem[] = [];
  // ... 
	// window.showQuickPick 返回的 promise中能获取到选中的option
  const selectedItem: any = await window.showQuickPick(itemList, {canPickMany: false, placeHolder: "未找到匹配的美杜莎Key"});
  const editor: TextEditor | undefined = window.activeTextEditor;
  editor?.edit(editBuilder => {
    editBuilder.replace(editor.selection, selectedItem.label);
  });
}


// -最后别忘了在 vscode extension的统一入口 extension.ts中注册命令
export function activate(context: vscode.ExtensionContext) {
  // ...
	new FindSelectionCommand().registerCommand(context);
  // ...
}
```
# Thanks

# Repository： 
[**http://gitlab.alibaba-inc.com/og-fe/vscode-plugin-mdsi18n**](http://gitlab.alibaba-inc.com/og-fe/vscode-plugin-mdsi18n)






