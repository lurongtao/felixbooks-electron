# Renderer Process API

Renderer API 主要包括 `remote、Browser window proxy、desktop Capture`

> Renderer Process API
- remote
- Browser Window Proxy
- desktop Capture

## 1、remote (服务端对象)

##### 1.1 index.html

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

    <button type="button" name="button" id="test-button">Fullscreen</button><br>

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

##### 1.2 renderer.js

```js
// This file is required by the index.html file and will
// be executed in the renderer process for that window.
// All of the Node.js APIs are available in this process.

const remote = require('electron').remote

const { app, dialog, BrowserWindow } = remote

const button = document.getElementById('test-button')

button.addEventListener('click', e => {

  // dialog.showMessageBox({ message: 'Dialog invoked from Renderer process' })

  // let secWin = new BrowserWindow({
  //   width: 400, height: 350
  // })
  // secWin.loadFile('index.html')

  // console.log( remote.getGlobal('myglob') )

  // app.quit()

  let win = remote.getCurrentWindow()
  win.maximize()

})
```

##### 1.3 main.js

```js
// Modules
const {app, BrowserWindow} = require('electron')

global['myglob'] = 'A var set in main.js'

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

## 2、Browser Window Proxy （浏览器窗口代理）

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

    <h3><a href="#" onclick="newWin()">New Window</a></h3>
    <h3><a href="#" onclick="closeWin()">Close Window</a></h3>
    <h3><a href="#" onclick="styleWin()">Bad Fonts</a></h3>

    <script>

      let win

      const newWin = () => {
        win = window.open('https://electronjs.org', '_blank', 'width=500,height=450,alwaysOnTop=1')
      }

      const closeWin = () => {
        win.close()
      }

      const styleWin = () => {
        win.eval("document.getElementsByTagName('body')[0].style.fontFamily = 'Comic Sans MS'")
      }

    </script>
  </body>
</html>
```

## 3、webFrame

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

    <img src="https://placekitten.com/450/300" alt=""><br>

    <button onclick="zoomUp()">Increase Zoom</button>
    <button onclick="zoomDown()">Decrease Zoom</button>
    <button onclick="zoomReset()">Reset Zoom</button>

    <script>

      const { webFrame } = require('electron')

      const zoomUp = () => {
        webFrame.setZoomLevel( webFrame.getZoomLevel() + 1 )
      }
      const zoomDown = () => {
        webFrame.setZoomLevel( webFrame.getZoomLevel() - 1 )
      }
      const zoomReset = () => {
        webFrame.setZoomLevel( 1 )
      }

      console.log( webFrame.getResourceUsage() )

    </script>
  </body>
</html>
```

## 4、desktopCapturer（桌面快照）

##### 4.1 index.html

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

    <img width="100%" src="" id="screenshot"><br>
    <button id="screenshot-button">Get Screenshot</button>

    <script>
      // You can also require other files to run in this process
      require('./renderer.js')
    </script>
  </body>
</html>
```

##### 4.2 renderer.js

```js
const { desktopCapturer } = require('electron')

document.getElementById('screenshot-button').addEventListener('click', () => {

  desktopCapturer.getSources({ types:['window'], thumbnailSize: {width:1920, height:1080} }, (error, sources) => {

    console.log(sources)

    document.getElementById('screenshot').src = sources[0].thumbnail.toDataURL()
  })

})
```