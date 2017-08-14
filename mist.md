# Mist 代码解析

## main.js代码入口

1. 引入实例化依赖对象

    - 设置global._ 为 underscore
    - 引入electron依赖，引入本地其他依赖

2. 初始化设置

    - Q(`promise`) config
    - Settting.init()

3. 引入其他实例化对象依赖

    - 设置[global.db](#dbjs)
    - 引入 `appMenu, ipcCommunicator, ipcProviderBackend, ethereumNode, swarmNode, nodeSync`
    - 设置 global 属性

4. 监听 app　事件

    - 监听 `window-all-close, open-url, uncaughtException, before-quit` 事件
    - app `ready` 回调 => db.init 回调 `onReady function()`

5. 注册自定义协议 `bzz`

6. onReady 函数

    - 设置 `global.config = db.getCollection('SYS_config')`
    - 同步 db 到 backend `dbSync.backendSyncInit()`
    - 初始化 Windows `Windows.init()`
    - 开启自定义协议 `bzz`
    - 检查更新 `UpdateChecker.run()`
    - initialize the web3 IPC provider backend [ipcProviderBackend.init](#ipcproviderbackendjs)
    - i18n.changeLanguage(Settings.language)
    - appMenu()
    - defaultWindow.manage(mainWindow.window) // 使用windowStateKeeper创建mainWindow
    - 创建 splashWindow
    - 检查时间同步 `check time sync`
    - spalshWindow ready => kickStart

7. kickStart 函数

    - 设置`ClinentBinaryManager, ethereumNode, swarmNode, nodeSync` 的各种监听事件，并通过Windows 广播到对应的事件
    - 检查旧链 `CHECK for legacy chain (FORK RELATED)`
    - `ClientBinaryManager.init()`
    - `ethereumNode.init()`
    - `swarmNode.init()`
    - update menu, to show node switching possibilities `appMenu()`
    - getAccounts `ethereumNode.send('eth_accounts', [])`
    - 如果没有设置账户，弹出账户board
    - 同步结束后，打开mainWindow，关闭 splashWindow

8. mainWindow 

    - 设置window 的监听事件
    - load global 设置的 URL
    - 设置 refreshMenu 的监听事件


***

## 依赖js

 - [Settings.js](#settingsjs)
 - [db.js](#dbjs)
 - [appMenu](#appmenujs)
 - [ipcProviderBackend](#ipcproviderbackendjs)
 - [ethereumNode](#ethereumnodejs)
 - [swarmNode](#swarmnodejs)
 - [nodeSync](#nodesyncjs)
 - [UpdateChecker](#updatecheckerjs)
 - [dbSync](#dbsyncjs)
 - [window](#windowjs)
 - [updateChecker](#windowjs)



*** 

### Settings.js

命令行参数，设置各种初始值，并提供get/set(部分)方法
  - 有一个`defaultConfig`用于设置默认运行格式，可以通过`../config.json`覆盖和添加其他配置
  - 使用`yargs`设置和读取命令行参数及其格式和初始值
  - `init`() 方法设置logger的logleveal，设置_log
  - get `dbFilePath`() userDataPath/mist.lokidb
  - get `appDataPath`() app.getPath('appdata')， 同样的还有homePath
  - get `rpcIpcPath`() mac: userHomePath//Library/Ethereum/geth.ipc || win: \\\\.\\pipe\\geth.ipc
  - initConfig(),saveConfig(),loadConfig() 设置global.config  (这里应该是以db的方式？？)
  - loadUserData(),saveUserData() 文件读取方式设置和读取userData/(path)

***

### db.js

    const db = global.db = require('./modules/db')

依赖 [loki.js](https://rawgit.com/techfort/LokiJS/master/jsdoc/index.html) 实现db功能
  - init() 设置dbfile (path => userData/mist.loki) 返回promise

      读取dbfile，返回db
  - getCollection(name) 查询name对应的collection
  - close()

***

### ipcCommunicator.js

    require('./modules/ipcCommunicator.js')

分为 UI Action 和 Mist Api 的监听事件

***

### appMenu.js

    appMenu = require('./modules/menuItems.js')

    // export
    module.exports = createMenu;
    createMenu(webview) // 调用electron Menu.buildFromTemplate

    // 用于 create 一个menu template，menu国际化，绑定菜单事件
    menuTmpl(webview)

***

### ipcProviderBackend.js

The IPC provider backend filter and tunnel all incoming request to the ethereum node.

    exports.init = () => {
        return new IpcProviderBackend();
    };

***

### ethereumNode.js

    class EthereumNode extends EventEmitter 

***

### swarmNode.js

    class SwarmNode extends EventEmitter 

***

### nodeSync.js

checks the current node whether its synching or not and how much it kept up already.

    class NodeSync extends EventEmitter

***

### UpdateChecker.js

Check for updates to the app.

***

### dbSync.js

 Sync IPC calls received from given window into given db table.

 通过ipcMain 和 ipcRender 通信，实现前后端的db 数据同步，其中 `Tracker, collection.find.onceSync, observeChanges` 方法都是全局引入的 meteor 的api

    backendSyncInit() // 通过 ipcMain 对 backEnd 监听事件。做一些db同步操作
    syncDataFromBackend() // 通过 ipcRender 对同步数据做出监听后的数据和视图的变化
    frontendSyncInit() // 初始化前端同步数据操作，调用 syncDataFromBackend 方法

***

### Windows.js

包括继承自 `Window extends EventEmitter` 类和 `Windows` 类

其中 `class Window extends EventEmitter` 类是用来实现具体`window`的初始化过程及基本监听函数

`Windows` 类实现了全局`window`对象集合的管理，包括对单个`window`窗口对象的创建，load，消息的broadcast

    broadCast() // 通过 channel 发送异步消息给渲染进程，你也可发送任意的参数.参数应该在 JSON 内部序列化，并且此后没有函数或原形链被包括了,发送给当前激活的所有window对象
    init() // 生成loading window，ipc监听，监听show hide 事件

***

### updateChecker.js

    run() // 执行检查更新
    runVisibly() // 带更新窗口的检查过程
    check() // 具体更新过程 

根据 [https://api.github.com/repos/ethereum/mist/releases](https://api.github.com/repos/ethereum/mist/releases) 返回的json，检查当前最新的version与app.version的大小对比。如果有更新，则返回更新对象的url, version, name