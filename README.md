我来帮你详细介绍如何使用Electron打包HTML项目。你已经安装了Node.js和npm，并且已经安装了Electron模块，接下来需要配置项目并打包。

## 1. 基本项目结构准备

首先，确保你的项目结构如下：

```
your-project/
├── index.html          # 你的HTML主文件
├── main.js            # Electron主进程文件
├── package.json       # npm配置文件
├── css/               # 样式文件
├── js/                # JavaScript文件
└── assets/            # 图片等资源文件
```

## 2. 创建或更新 package.json

如果你的项目还没有 `package.json`，先创建一个：

```bash
npm init -y
```

然后修改 `package.json`，添加必要的配置：

```json
{
  "name": "your-app-name",
  "version": "1.0.0",
  "description": "Your app description",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "pack": "electron-builder --dir",
    "dist": "electron-builder"
  },
  "keywords": [],
  "author": "Your Name",
  "license": "MIT",
  "devDependencies": {
    "electron": "^latest",
    "electron-builder": "^latest"
  }
}
```

## 3. 创建主进程文件 (main.js)

创建 `main.js` 文件：

```javascript
const { app, BrowserWindow, Menu } = require('electron')
const path = require('path')

// 创建窗口函数
function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false,
      enableRemoteModule: true
    },
    icon: path.join(__dirname, 'assets/icon.png') // 可选：应用图标
  })

  // 加载你的HTML文件
  mainWindow.loadFile('index.html')
  
  // 开发模式下打开开发者工具
  // mainWindow.webContents.openDevTools()
}

// 应用准备就绪后创建窗口
app.whenReady().then(() => {
  createWindow()
  
  // macOS特殊处理
  app.on('activate', function () {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// 所有窗口关闭时退出应用（macOS除外）
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

// 可选：创建自定义菜单
const isMac = process.platform === 'darwin'
const template = [
  ...(isMac ? [{
    label: app.name,
    submenu: [
      { role: 'about' },
      { type: 'separator' },
      { role: 'services' },
      { type: 'separator' },
      { role: 'hide' },
      { role: 'hideothers' },
      { role: 'unhide' },
      { type: 'separator' },
      { role: 'quit' }
    ]
  }] : []),
  {
    label: 'File',
    submenu: [
      isMac ? { role: 'close' } : { role: 'quit' }
    ]
  },
  {
    label: 'View',
    submenu: [
      { role: 'reload' },
      { role: 'forcereload' },
      { role: 'toggledevtools' },
      { type: 'separator' },
      { role: 'resetzoom' },
      { role: 'zoomin' },
      { role: 'zoomout' },
      { type: 'separator' },
      { role: 'togglefullscreen' }
    ]
  }
]

const menu = Menu.buildFromTemplate(template)
Menu.setApplicationMenu(menu)
```

## 4. 安装 Electron Builder（打包工具）

```bash
# 安装 electron-builder
npm install --save-dev electron-builder
```

## 5. 配置构建选项

在 `package.json` 中添加 `build` 配置：

```json
"build": {
  "appId": "com.yourcompany.yourapp",
  "productName": "Your App Name",
  "directories": {
    "output": "dist"
  },
  "files": [
    "**/*",
    "!**/node_modules/*/{CHANGELOG.md,README.md,README,readme.md,readme}",
    "!**/node_modules/*/{test,__tests__,tests,powered-test,example,examples}",
    "!**/node_modules/*.d.ts",
    "!**/*.iml",
    "!**/node_modules/.bin",
    "!**/*.{o,hprof,orig,pyc,pyo,rbc,swp,csproj,sln,xproj}",
    "!.editorconfig",
    "!**/._*",
    "!**/{.DS_Store,.git,.hg,.svn,CVS,RCS,SCCS,__pycache__,thumbs.db,.gitignore,.gitattributes,.flowconfig,.yarn-integrity,.idea,appveyor.yml,.travis.yml,circle.yml,npm-debug.log,yarn.lock,.yarn-metadata.json}"
  ],
  "win": {
    "target": [
      "nsis",
      "portable"
    ],
    "icon": "assets/icon.ico"
  },
  "mac": {
    "category": "public.app-category.utilities",
    "target": [
      "dmg",
      "zip"
    ],
    "icon": "assets/icon.icns"
  },
  "linux": {
    "target": [
      "AppImage",
      "deb",
      "rpm"
    ],
    "icon": "assets/icon.png"
  },
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true,
    "createDesktopShortcut": true,
    "createStartMenuShortcut": true
  }
}
```

## 6. 测试应用

首先测试应用是否能正常运行：

```bash
npm start
```

如果能看到你的HTML页面在Electron窗口中打开，说明配置成功。

## 7. 打包应用

### 方法一：创建可分发的应用

```bash
# 构建应用
npm run dist
```

### 方法二：只生成可执行文件（不打包安装程序）

```bash
npm run pack
```

## 8. 跨平台打包（可选）

默认情况下，electron-builder会在当前操作系统上构建对应的应用。如果需要为其他平台打包：

```bash
# 在 package.json 的 scripts 中添加
"scripts": {
  "build:win": "electron-builder --win --x64",
  "build:mac": "electron-builder --mac --x64",
  "build:linux": "electron-builder --linux --x64"
}

# 然后运行
npm run build:win   # 构建Windows应用
npm run build:mac   # 构建macOS应用
npm run build:linux # 构建Linux应用
```

## 9. 解决常见问题

### 问题1：打包后白屏或找不到文件
确保 `main.js` 中正确加载文件路径：

```javascript
// 使用绝对路径
mainWindow.loadFile(path.join(__dirname, 'index.html'))
```

### 问题2：资源文件丢失
确保在 `package.json` 的 `build.files` 中包含所有必要文件。

### 问题3：应用图标不显示
准备不同格式的图标：
- Windows: `.ico` 文件（至少 256x256）
- macOS: `.icns` 文件
- Linux: `.png` 文件（至少 256x256）

可以使用在线工具转换，如：https://convertio.co/zh/

## 10. 简化方案：使用 electron-forge

如果你觉得 electron-builder 配置复杂，可以尝试 electron-forge：

```bash
# 安装 electron-forge
npm install --save-dev @electron-forge/cli

# 初始化
npx electron-forge import

# 打包
npm run make
```

## 打包结果

打包完成后，你会在 `dist` 文件夹中找到：
- Windows: `.exe` 安装程序或便携版
- macOS: `.dmg` 或 `.app` 文件
- Linux: `.AppImage` 或 `.deb` 文件

这样你就可以分发你的HTML应用了！
