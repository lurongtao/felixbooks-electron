# 共享API (Shared API)

本节重点讲解 `process`、`screen`、`shell`、`nativeImage`、`clipboard` 几个部分内容。

## 1、process (进程)

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

    <!-- All of the Node.js APIs are available in this renderer process. -->
    We are using Node.js <strong><script>document.write( process.versions.node)</script></strong>,
    and Electron <strong><script>document.write( process.versions.electron )</script></strong>.

    <br><button type="button" onclick="process.hang()">Hang Renderer</button>
    <br><button type="button" onclick="process.crash()">Crash Renderer</button>

    <script>
      // let i = 1
      // setInterval(() => {
      //   console.log(i)
      //   i++
      // }, 500)

    </script>
  </body>
</html>
```

##### 1.2 main.js

```js
// Modules
const {app, BrowserWindow} = require('electron')

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

  mainWindow.webContents.on( 'crashed', mainWindow.reload )

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

## 2、screen (屏幕)

##### 1.1 main.js

```js
// Modules
const electron = require('electron')
const {app, BrowserWindow} = electron

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

// Create a new BrowserWindow when `app` is ready
function createWindow () {

  let primaryDisplay = electron.screen.getPrimaryDisplay()

  mainWindow = new BrowserWindow({
    x: primaryDisplay.bounds.x, y: primaryDisplay.bounds.y,
    width: primaryDisplay.size.width/2, height: primaryDisplay.size.height,
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

##### 1.2 renderer.js

```js
const electron = require('electron')

const displays = electron.screen.getAllDisplays()

console.log( `${displays[0].size.width} x ${displays[0].size.height}` )
console.log( `${displays[0].bounds.x}, ${displays[0].bounds.y}` )
console.log( `${displays[1].size.width} x ${displays[1].size.height}` )
console.log( `${displays[1].bounds.x}, ${displays[1].bounds.y}` )


electron.screen.on( 'display-metrics-changed', (e, display, metricsChanged) => {
  console.log( metricsChanged )
})

document.getElementsByTagName('body')[0].addEventListener( 'click', e => {
  console.log( electron.screen.getCursorScreenPoint() )
})
```

## 3、shell

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

    <button onclick="showSite()">Launch Electron.js Site</button><br>
    <button onclick="openSplash()">Open Splash.png</button><br>
    <button onclick="showSplashFile()">Show Splash.png</button><br>
    <button onclick="deleteSplashFile()">Delete Splash.png</button><br>

    <script>

      const { shell } = require('electron')

      const showSite = e => {
        shell.openExternal('https://electronjs.org')
      }


      const splashPath = `${__dirname}/splash.png`

      const openSplash = e => {
        shell.openItem(splashPath)
      }
      const showSplashFile = e => {
        shell.showItemInFolder(splashPath)
      }
      const deleteSplashFile = e => {
        shell.moveItemToTrash(splashPath)
      }

    </script>
  </body>
</html>
```

## 4、nativeImage (本地图片)

```html 
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Convert splash.png:</h1>

    <button onclick="toPng()">PNG</button>
    <button onclick="toJpg()">JPG</button>
    <button onclick="toTag()">Show</button>
    <br><img src="" id="preview">


    <script>

      const fs = require('fs')
      const { nativeImage, remote } = require('electron')

      const splash = nativeImage.createFromPath(`${__dirname}/splash.png`)

      const saveToDesktop = (data, ext) => {

        let desktopPath = remote.app.getPath('desktop')
        fs.writeFile( `${desktopPath}/splash.${ext}`, data, console.log )
      }

      const toTag = e => {

        let size = splash.getSize()

        let splashURL = splash.resize({ width: size.width/4, height: size.height/4 }).toDataURL()
        document.getElementById('preview').src = splashURL
      }
      const toPng = e => {
        let pngSplash = splash.toPNG()
        saveToDesktop( pngSplash, 'png' )
      }
      const toJpg = e => {
        let jpgSplash = splash.toJPEG(100)
        saveToDesktop( jpgSplash, 'jpg' )
      }

    </script>
  </body>
</html>
```

## 5、clipboard（剪贴板）

##### 5.1 index.html

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

    <!-- All of the Node.js APIs are available in this renderer process. -->
    We are using Node.js <strong><script>document.write( process.versions.node)</script></strong>,
    and Electron <strong><script>document.write( process.versions.electron )</script></strong>.

    <br><button onclick="makeUpper()">Make clipboard uppercase</button>
    <br><button onclick="showImage()">Show clipboard image</button>

    <br><img src="" id="cbImage">

    <script>

      const { clipboard } = require('electron')

      console.log( clipboard.readText() )

      const showImage = e => {
        let image = clipboard.readImage()
        document.getElementById('cbImage').src = image.toDataURL()
      }

      const makeUpper = e => {
        let cbText = clipboard.readText()
        clipboard.writeText( cbText.toUpperCase() )
      }

    </script>
  </body>
</html>
```

##### 5.2 main.js

```js
// Modules
const {app, BrowserWindow, clipboard} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

// Create a new BrowserWindow when `app` is ready
function createWindow () {

  clipboard.writeText('Hello from the main process!')

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