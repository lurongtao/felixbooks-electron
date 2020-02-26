# Main Process API
> Electron API
- Main Process
- Renderer Process
- Share Modules

## App
### 事件
#### ready: 
> 当 Electron 完成初始化时被触发。
1. 两种使用方法
```
app.on('ready', createWindow)
app.on('ready', () => {
  console.log('App is ready!')
  createWindow()
})
```
2. 检查应用是否登录：app.isReady()

- 如果应用没有Ready，app.isReady()的值为 false
```
console.log('应用是否登录：' + app.isReady())
```
- 此时应用应该已经Ready
```
setTimeout(() => {
  console.log('应用是否登录：' + app.isReady())
}, 2000)
```

#### before-quit
> 在应用程序开始关闭窗口之前触发。

```
app.on('before-quit', (e) => {
  console.log('App is quiting')
  e.preventDefault()
})
```

#### browser-window-blur
> 在 browserWindow 失去焦点时发出

```
app.on('browser-window-blur', (e) => {
  console.log('App unfocused')
})
```

#### browser-window-focus
> 在 browserWindow 获得焦点时发出

```
app.on('browser-window-focus', (e) => {
  console.log('App focused')
})
```

### 方法
#### app.quit()
```
app.on('browser-window-blur', (e) => {
  setTimeout(() => {
    app.quit()
  }, 3000)
})

app.on('browser-window-blur', (e) => {
  setTimeout(app.quit, 3000)
})
```

#### app.getPath(name)
```
app.on('ready', () => {
  console.log(app.getPath('desktop'))
  console.log(app.getPath('music'))
  console.log(app.getPath('temp'))
  console.log(app.getPath('userData'))

  createWindow()
})
```

## BrowserWindow
> electron.BrowserWindow: 创建和控制浏览器窗口

### 实例方法
win.loadURL(url[, options]): 和 loadFile 互斥
```
mainWindow.loadURL('https://www.baidu.com')
```

### 优雅的显示窗口
- 使用ready-to-show事件
```
let mainWindow = new BrowserWindow({ show: false })
mainWindow.once('ready-to-show', () => {
  mainWindow.show()
})
```
- 设置 backgroundColor
```
let win = new BrowserWindow({ backgroundColor: '#2e2c29' })
```

### 父子窗口
- 窗口定义
```
secondaryWindow = new BrowserWindow({
  width: 600,
  height: 600,
  webPreferences: { nodeIntegration: true }
})

secondaryWindow.loadFile('index.html')

secondaryWindow.on('closed',  () => {
   mainWindow = null
})
```

- 窗口关系
```
secondaryWindow = new BrowserWindow({
  parent: mainWindon, // 定义父窗口
  modal: true // 锁定在主窗口
})
```

- 子窗口显示和隐藏
```
secondaryWindow = new BrowserWindow({
  show: false
})

setTimeout(() => {
  secondaryWindow.show()
  setTimeout(() => {
    secondaryWindow.hide()
  }, 3000)
}, 2000)
```

### 无边框窗口
> Frameless Window
```
mainWindow = new BrowserWindow({
  frame: false
})
```
让页面可拖拽
```
<body style="user-select: none; -webkit-app-region:drag;">
```
no-drag 修复下面控件的bug
```
<input style="-webkit-app-region: no-drag;" type="range" name="range" min="0" max="10">
```
显示红绿灯
```
mainWindow = new BrowserWindow({
  titleBarStyle: 'hidden' // or hiddenInset 距离红绿灯更近
})
```

### 属性与方法
#### minWidth && minHeight
```
mainWindow = new BrowserWindow({
  minWidth: 300,
  minHeight: 300
})
```
更多详见：https://electronjs.org/docs/api/browser-window#new-browserwindowoptions

#### 窗口焦点事件
```
secWindow = new BrowserWindow({
  width: 400, height: 300,
  webPreferences: { nodeIntegration: true },
})

mainWindow.on('focus', () => {
  console.log('mainWindow focused')
})

secWindow.on('focus', () => {
  console.log('secWindow focused')
})

app.on('browser-window-focus', () => {
  console.log('App focused')
})
```

#### 静态方法
- getAllWindows()
```
let allWindows = BrowserWindow.getAllWindows()
console.log(allWindows)
```
更多详见: https://electronjs.org/docs/api/browser-window#%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95

#### 实例属性
- id
```
console.log(secWindow.id)
```
更多详见：https://electronjs.org/docs/api/browser-window#%E5%AE%9E%E4%BE%8B%E5%B1%9E%E6%80%A7

#### 实例方法
- maximize()
```
secWindow.on('closed', () => {
  mainWindow.maximize()
})
```
更多详见：https://electronjs.org/docs/api/browser-window#%E5%AE%9E%E4%BE%8B%E6%96%B9%E6%B3%95

### state
> electron-window-state 保存窗口的状态
`npm install electron-window-state`

### webContents
> webContents 是 EventEmitter 的实例， 负责渲染和控制网页, 是 BrowserWindow 对象的一个属性。
```
let wc = mainWindow.webContents
console.log(wc)
```

#### 方法 getAllWebContents(）
- 返回 WebContents[] - 所有 WebContents 实例的数组。 包含所有Windows，webviews，opened devtools 和 devtools 扩展背景页的 web 内容
```
const {app, BrowserWindow, webContents} = require('electron')
console.log(webContents.getAllWebContents())
```

#### 实例事件
- did-finish-load
- dom-ready
```
<div>
   <img src="https://placekitten.com/500/500" alt="">
</div>

let wc = mainWindow.webContents
wc.on('did-finish-load', () => {
  console.log('Conent fully loaded')
})
wc.on('dom-ready', () => {
  console.log('DOM Ready')
})
```

- new-window
```
<div>
  <a target="_blank" href="https://placekitten.com/500/500"><h3>Kitten</h3></a>
</div>

wc.on('new-window', (e, url) => {
  e.preventDefault()
  console.log('DOM Ready')
})
```

- before-input-event
```
wc.on('before-input-event', (e, input) => {
  console.log(`${input.key} : ${input.type}`)
})
```

- login
- did-navigate
```
mainWindow.loadURL('https://httpbin.org/basic-auth/user/passwd')

wc.on('login', (e, request, authInfo, callback) => {
  console.log('Logging in:')
  callback('user', 'passwd')
})

wc.on('did-navigate', (e, url, statusCode, message) => {
  console.log(`Navigated to: ${url}, with response code: ${statusCode}`)
  console.log(message)
})
```

- media-started-playing
- media-paused
```
<div>
  <video src="./mgm.mp4" controls width="400"></video>
</div>

wc.on('media-started-playing', () => {
  console.log('Video Started')
})
wc.on('media-paused', () => {
  console.log('Video Paused')
})
```

- context-menu : 右键上下文信息
```
wc.on('context-menu', (e, params) => {
  console.log(`Context menu opened on: ${params.mediaType} at x:${params.x}, y:${params.y}`)
})

wc.on('context-menu', (e, params) => {
  console.log(`User seleted text: ${params.selectionText}`)
  console.log(`Selection can be copied: ${params.editFlags.canCopy}`)
})
```

#### 实例方法
- executeJavaScript()
```
wc.on('context-menu', (e, params) => {
  wc.executeJavaScript(`alert('${params.selectionText}')`)
})
```

## Session
> 管理浏览器会话、cookie、缓存、代理设置等。


### 起步
- 创建session对象
```
let session = mainWindow.webContents.session
console.log(session) // {}
```
- 在chromium 创建localStorage，然后创建两个窗口，两个session共享
```
mainWindow = new BrowserWindow({
  width: 1000, height: 800,
  webPreferences: { nodeIntegration: true }
})

secWindow = new BrowserWindow({
  width: 500, height: 400,
  webPreferences: { nodeIntegration: true }
})

let session = mainWindow.webContents.session
let session2 = mainWindow.webContents.session
console.log(Object.is(session, session2)) // true

// Load index.html into the new BrowserWindow
mainWindow.loadFile('index.html')
secWindow.loadFile('index.html')

// Open DevTools - Remove for PRODUCTION!
mainWindow.webContents.openDevTools();
secWindow.webContents.openDevTools();

// Listen for window being closed
mainWindow.on('closed',  () => {
  mainWindow = null
})
secWindow.on('closed',  () => {
  secWindow = null
})
```

- defaultSession
```
const {app, BrowserWindow, session} = require('electron')
let ses = mainWindow.webContents.session
console.log(Object.is(session.defaultSession, ses)) // true
```

- 自定义session
```
let customSes = session.fromPartition('part1')
console.log(Object.is(customSes, ses)) //false, 此时customSes 还是共享session

secWindow = new BrowserWindow({
  width: 500, height: 400,
  webPreferences: { 
    nodeIntegration: true,
    session: customSes // 定义session, 此时子窗口有自己的session
  }
})

// 在子窗口里创建localstorge: winName/secWindow
// 关闭所有窗口，发现创建的localstorage又消失了，因为此时的session存储在内存里，重新启动应用又消失了。可以加前缀persist，使其变为永久存储：

let customSes = session.fromPartition('persist:part1')

// 或者：

secWindow = new BrowserWindow({
  width: 500, height: 400,
  webPreferences: { 
    nodeIntegration: true,
    - session: customSes
    + partition: 'persist:part1'
  }
})

```

- 实例方法
```
ses.clearStorageData() // 删除主窗口的的storage
```

### cookie
> 查询和修改一个会话的cookies
```
// 查询所有 cookies
session.defaultSession.cookies.get({})
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })
```
```
// 查询所有与设置的 URL 相关的所有 cookies
session.defaultSession.cookies.get({ url: 'http://www.github.com' })
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })
```
```
// 设置一个 cookie，使用设置的名称；
// 如果存在，则会覆盖原先 cookie.
const cookie = { url: 'http://www.github.com', name: 'dummy_name', value: 'dummy' }
session.defaultSession.cookies.set(cookie)
  .then(() => {
    // success
  }, (error) => {
    console.error(error)
  })
```

### downloadItem
> 控制来自于远程资源的文件下载。

```
// html
<h2><a href="https://picsum.photos/5000/5000/" download>Download Image</a></h2>
<progress value="0" max="100" id="progress"></progress>

<script>
  window.progress = document.getElementById('progress')
</script>
```

```
// main.js
let ses = session.defaultSession

ses.on('will-download', (e, downloadItem, webContents) => {

  let fileName = downloadItem.getFilename()
  let fileSize = downloadItem.getTotalBytes()

  // Save to desktop
  downloadItem.setSavePath(app.getPath('desktop') + `/${fileName}`)

  downloadItem.on('updated', (e, state) => {

    let received = downloadItem.getReceivedBytes()

    if (state === 'progressing' && received) {
      let progress = Math.round((received/fileSize)*100)
      webContents.executeJavaScript(`window.progress.value = ${progress}`)
    }
  })
})
```

## dialog - 对话框
> 显示用于打开和保存文件、警报等的本机系统对话框

```
const {app, BrowserWindow, dialog} = require('electron')

mainWindow.webContents.on('did-finish-load', () => {
  dialog.showOpenDialog({
    buttonLabel: '选择',
    defaultPath: app.getPath('desktop'),
    properties: ['multiSelections', 'createDirectory', 'openFile', 'openDirectory']
  }, filepaths => {
    console.log(filepaths)
  })
})
```

```
dialog.showSaveDialog({}, filename => {
  console.log(filename)
})
```

```
const answers = ['Yes', 'No', 'Maybe']

dialog.showMessageBox({
  title: 'Message Box',
  message: 'Please select an option',
  detail: 'Message details.',
  buttons: answers
}, response => {
  console.log(`User selected: ${answers[response]}`)
})
```

## 快捷键+系统快捷键
> **快捷键**：定义键盘快捷键。
> **系统快捷键**：在应用程序没有键盘焦点时，监听键盘事件。

快捷键可以包含多个功能键和一个键码的字符串，由符号+结合，用来定义你应用中的键盘快捷键

示例：

+ CommandOrControl+A
+ CommandOrControl+Shift+Z

快捷方式使用 register 方法在 globalShortcut 模块中注册。

globalShortcut 模块可以在操作系统中注册/注销全局快捷键, 以便可以为操作定制各种快捷键。

注意: 快捷方式是全局的; 即使应用程序没有键盘焦点, 它也仍然在持续监听键盘事件。 在应用程序模块发出 ready 事件之前, 不应使用此模块。

```
const {app, BrowserWindow, globalShortcut} = require('electron')

globalShortcut.register('G', () => {
  console.log('User pressed G')
})
```

```
globalShortcut.register('CommandOrControl+Y', () => {
  console.log('User pressed G with a combination key')
  globalShortcut.unregister('CommandOrControl+G')
})
```