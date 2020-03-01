# Main Process API
> Electron API （Electron API 有三种）
- Main Process （主进进程）
- Renderer Process（渲染进程）
- Share Modules（共享模块）

## App
### 事件
#### ready: 

> 当 Electron 完成初始化时被触发。

- 两种使用方法

```
app.on('ready', createWindow)
app.on('ready', () => {
  console.log('App is ready!')
  createWindow()
})
```

- 检查应用是否登录：app.isReady()

- 如果应用没有Ready，app.isReady()的值为 false

```js
console.log('应用是否登录：' + app.isReady())

```
- 此时应用应该已经Ready

```js
setTimeout(() => {
  console.log('应用是否登录：' + app.isReady())
}, 2000)
```

#### before-quit
> 在应用程序开始关闭窗口之前触发。

```js
app.on('before-quit', (e) => {
  console.log('App is quiting')
  e.preventDefault()
})
```

#### browser-window-blur
> 在 browserWindow 失去焦点时发出

```js
app.on('browser-window-blur', (e) => {
  console.log('App unfocused')
})
```

#### browser-window-focus
> 在 browserWindow 获得焦点时发出

```js
app.on('browser-window-focus', (e) => {
  console.log('App focused')
})
```

### 方法
#### app.quit()

```js
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

```js
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

```js
mainWindow.loadURL('https://www.baidu.com')
```

### 优雅的显示窗口
- 使用ready-to-show事件

```js
let mainWindow = new BrowserWindow({ show: false })
mainWindow.once('ready-to-show', () => {
  mainWindow.show()
})
```

- 设置 backgroundColor

```js
let win = new BrowserWindow({ backgroundColor: '#2e2c29' })
```

### 父子窗口

- 窗口定义

```js
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

```js
secondaryWindow = new BrowserWindow({
  parent: mainWindon, // 定义父窗口
  modal: true // 锁定在主窗口
})
```

- 子窗口显示和隐藏

```js
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

```js
mainWindow = new BrowserWindow({
  frame: false
})
```

让页面可拖拽

```html
<body style="user-select: none; -webkit-app-region:drag;">
```

no-drag 修复下面控件的bug

```html
<input style="-webkit-app-region: no-drag;" type="range" name="range" min="0" max="10">
```

显示红绿灯

```js
mainWindow = new BrowserWindow({
  titleBarStyle: 'hidden' // or hiddenInset 距离红绿灯更近
})
```

### 属性与方法
#### minWidth && minHeight

```js
mainWindow = new BrowserWindow({
  minWidth: 300,
  minHeight: 300
})
```

更多详见：https://electronjs.org/docs/api/browser-window#new-browserwindowoptions

#### 窗口焦点事件

```js
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

```js
let allWindows = BrowserWindow.getAllWindows()
console.log(allWindows)
```

更多详见: https://electronjs.org/docs/api/browser-window#%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95

#### 实例属性

- id

```js
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

```html
<div>
   <img src="https://placekitten.com/500/500" alt="">
</div>
<script>
let wc = mainWindow.webContents
wc.on('did-finish-load', () => {
  console.log('Conent fully loaded')
})
wc.on('dom-ready', () => {
  console.log('DOM Ready')
})
</script>
```

- new-window

```html
<div>
  <a target="_blank" href="https://placekitten.com/500/500"><h3>Kitten</h3></a>
</div>

<script>
wc.on('new-window', (e, url) => {
  e.preventDefault()
  console.log('DOM Ready')
})
</script>
```

- before-input-event

```js
wc.on('before-input-event', (e, input) => {
  console.log(`${input.key} : ${input.type}`)
})
```

- login
- did-navigate

```js
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

```html
<div>
  <video src="./mgm.mp4" controls width="400"></video>
</div>
<script>
wc.on('media-started-playing', () => {
  console.log('Video Started')
})
wc.on('media-paused', () => {
  console.log('Video Paused')
})
</script>
```

- context-menu : 右键上下文信息

```js
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

```js
wc.on('context-menu', (e, params) => {
  wc.executeJavaScript(`alert('${params.selectionText}')`)
})
```

## Session
> 管理浏览器会话、cookie、缓存、代理设置等。


### 起步
- 创建session对象

```js
let session = mainWindow.webContents.session
console.log(session) // {}
```

- 在chromium 创建localStorage，然后创建两个窗口，两个session共享

```js
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

```js
const {app, BrowserWindow, session} = require('electron')
let ses = mainWindow.webContents.session
console.log(Object.is(session.defaultSession, ses)) // true
```

- 自定义session

```js
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

```js
ses.clearStorageData() // 删除主窗口的的storage
```

### cookie

> 查询和修改一个会话的cookies

```js
// 查询所有 cookies
session.defaultSession.cookies.get({})
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })
```

```js
// 查询所有与设置的 URL 相关的所有 cookies
session.defaultSession.cookies.get({ url: 'http://www.github.com' })
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })
```

```js
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

```html

<h2><a href="https://picsum.photos/5000/5000/" download>Download Image</a></h2>
<progress value="0" max="100" id="progress"></progress>

<script>
  window.progress = document.getElementById('progress')
</script>
```

```js
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

```js
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

```js
dialog.showSaveDialog({}, filename => {
  console.log(filename)
})
```

```js
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

```js
const {app, BrowserWindow, globalShortcut} = require('electron')

globalShortcut.register('G', () => {
  console.log('User pressed G')
})
```

```js
globalShortcut.register('CommandOrControl+Y', () => {
  console.log('User pressed G with a combination key')
  globalShortcut.unregister('CommandOrControl+G')
})
```

## Menu

##### 1、index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <textarea name="name" rows="8" cols="80"></textarea>

    <!-- All of the Node.js APIs are available in this renderer process. -->
    We are using Node.js <strong><script>document.write( process.versions.node)</script></strong>,
    and Electron <strong><script>document.write( process.versions.electron )</script></strong>.

    <script>
      // You can also require other files to run in this process
      require('./renderer.js')
    </script>
  </body>
</html>
```

##### 2、main.js

```js
// Modules
const {app, BrowserWindow, Menu, MenuItem} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

let mainMenu = Menu.buildFromTemplate( require('./mainMenu') )


// Create a new BrowserWindow when `app` is ready
function createWindow () {

  mainWindow = new BrowserWindow({
    width: 1000, height: 800,
    webPreferences: { nodeIntegration: true }
  })

  // Load index.html into the new BrowserWindow
  mainWindow.loadFile('index.html')

  // Open DevTools - Remove for PRODUCTION!
  mainWindow.webContents.openDevTools();

  Menu.setApplicationMenu(mainMenu)

  // Listen for window being closed
  mainWindow.on('closed',  () => {
    mainWindow = null
  })
}

// Electron `app` is ready
app.on('ready', createWindow)

// Quit when all windows are closed - (Not macOS - Darwin)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// When app icon is clicked and app is running, (macOS) recreate the BrowserWindow
app.on('activate', () => {
  if (mainWindow === null) createWindow()
})

```

##### 3、mainMenu.js

```js
module.exports = [
  {
    label: 'Electron',
    submenu: [
      { label: 'Item 1'},
      { label: 'Item 2', submenu: [ { label: 'Sub Item 1'} ]},
      { label: 'Item 3'},
    ]
  },
  {
    label: 'Edit',
    submenu: [
      { role: 'undo'},
      { role: 'redo'},
      { role: 'copy'},
      { role: 'paste'},
    ]
  },
  {
    label: 'Actions',
    submenu: [
      {
        label: 'DevTools',
        role: 'toggleDevTools'
      },
      {
        role: 'toggleFullScreen'
      },
      {
        label: 'Greet',
        click: () => { console.log('Hello from Main Menu') },
        accelerator: 'Shift+Alt+G'
      }
    ]
  }
]
```

## Context Menus

##### 1、index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <textarea name="name" rows="8" cols="80"></textarea>

    <!-- All of the Node.js APIs are available in this renderer process. -->
    We are using Node.js <strong><script>document.write( process.versions.node)</script></strong>,
    and Electron <strong><script>document.write( process.versions.electron )</script></strong>.

    <script>
      // You can also require other files to run in this process
      require('./renderer.js')
    </script>
  </body>
</html>
```

##### 2、main.js

```js
// Modules
const {app, BrowserWindow, Menu} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

let contextMenu = Menu.buildFromTemplate([
  { label: 'Item 1' },
  { role: 'editMenu' }
])

// Create a new BrowserWindow when `app` is ready
function createWindow () {

  mainWindow = new BrowserWindow({
    width: 1000, height: 800,
    webPreferences: { nodeIntegration: true }
  })

  // Load index.html into the new BrowserWindow
  mainWindow.loadFile('index.html')

  // Open DevTools - Remove for PRODUCTION!
  mainWindow.webContents.openDevTools();

  mainWindow.webContents.on('context-menu', e => {
    contextMenu.popup()
  })

  // Listen for window being closed
  mainWindow.on('closed',  () => {
    mainWindow = null
  })
}

// Electron `app` is ready
app.on('ready', createWindow)

// Quit when all windows are closed - (Not macOS - Darwin)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// When app icon is clicked and app is running, (macOS) recreate the BrowserWindow
app.on('activate', () => {
  if (mainWindow === null) createWindow()
})
```

## Tray (托盘)

##### 1、main.js

```js
// Modules
const {app, BrowserWindow, Tray, Menu} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow, tray

let trayMenu = Menu.buildFromTemplate([
  { label: 'Item 1' },
  { role: 'quit' }
])

function createTray() {

  tray = new Tray('trayTemplate@2x.png')
  tray.setToolTip('Tray details')

  tray.on('click', e => {

    if (e.shiftKey) {
      app.quit()
    } else {
      mainWindow.isVisible() ? mainWindow.hide() : mainWindow.show()
    }
  })

  tray.setContextMenu(trayMenu)
}

// Create a new BrowserWindow when `app` is ready
function createWindow () {

  createTray()

  mainWindow = new BrowserWindow({
    width: 1000, height: 800,
    webPreferences: { nodeIntegration: true }
  })

  // Load index.html into the new BrowserWindow
  mainWindow.loadFile('index.html')

  // Open DevTools - Remove for PRODUCTION!
  mainWindow.webContents.openDevTools();

  // Listen for window being closed
  mainWindow.on('closed',  () => {
    mainWindow = null
  })
}

// Electron `app` is ready
app.on('ready', createWindow)

// Quit when all windows are closed - (Not macOS - Darwin)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// When app icon is clicked and app is running, (macOS) recreate the BrowserWindow
app.on('activate', () => {
  if (mainWindow === null) createWindow()
})
```

## powerMonitor (电源指示器)

```js
// Modules
const electron = require('electron')
const {app, BrowserWindow} = electron

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

// Create a new BrowserWindow when `app` is ready
function createWindow () {

  mainWindow = new BrowserWindow({
    width: 1000, height: 800,
    webPreferences: { nodeIntegration: true }
  })

  // Load index.html into the new BrowserWindow
  mainWindow.loadFile('index.html')

  // Open DevTools - Remove for PRODUCTION!
  mainWindow.webContents.openDevTools();

  // Listen for window being closed
  mainWindow.on('closed',  () => {
    mainWindow = null
  })

  electron.powerMonitor.on('resume', e => {
    if(!mainWindow) createWindow()
  })

  electron.powerMonitor.on('suspend', e => {
    console.log('Saving some data')
  })
}

// Electron `app` is ready
app.on('ready', createWindow)

// Quit when all windows are closed - (Not macOS - Darwin)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// When app icon is clicked and app is running, (macOS) recreate the BrowserWindow
app.on('activate', () => {
  if (mainWindow === null) createWindow()
})
```