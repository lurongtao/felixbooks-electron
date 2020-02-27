# IPC 通信

### 1、index.html

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

    <button type="button" id="talk">Talk to main process</button><br>

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

### 2、renderer.js

```js
// This file is required by the index.html file and will
// be executed in the renderer process for that window.
// All of the Node.js APIs are available in this process.

const { ipcRenderer } = require('electron')

let i = 1
setInterval( () => {
  console.log(i)
  i++
}, 1000)

document.getElementById('talk').addEventListener('click', e => {

  // ipcRenderer.send( 'channel1', 'Hello from main window')

  let response = ipcRenderer.sendSync( 'sync-message', 'Waiting for response')
  console.log(response)

})

ipcRenderer.on( 'channel1-response', (e, args) => {
  console.log(args)
})

ipcRenderer.on( 'mailbox', (e, args) => {
  console.log(args)
})
```

### 3、main.js

```js
// Modules
const {app, BrowserWindow, ipcMain} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

// Create a new BrowserWindow when `app` is ready
function createWindow () {

  mainWindow = new BrowserWindow({
    width: 1000, height: 800, x: 100, y:140,
    webPreferences: { nodeIntegration: true }
  })

  // Load index.html into the new BrowserWindow
  mainWindow.loadFile('index.html')

  // Open DevTools - Remove for PRODUCTION!
  mainWindow.webContents.openDevTools();

  mainWindow.webContents.on( 'did-finish-load', e => {

    // mainWindow.webContents.send( 'mailbox', {
    //   from: 'Ray',
    //   email: 'ray@stackacademy.tv',
    //   priority: 1
    // })
  })

  // Listen for window being closed
  mainWindow.on('closed',  () => {
    mainWindow = null
  })
}

ipcMain.on( 'sync-message', (e, args) => {
  console.log(args)

  setTimeout( () => {
    e.returnValue = 'A sync response from the main process'
  }, 4000)

})

ipcMain.on( 'channel1', (e, args) => {
  console.log(args)
  e.sender.send( 'channel1-response', 'Message received on "channel1". Thank you!')
})

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