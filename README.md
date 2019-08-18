# electron-as-browser

![version](https://img.shields.io/npm/v/electron-as-browser.svg?style=flat-square)![downloads](https://img.shields.io/npm/dt/electron-as-browser.svg?style=flat-square)![license](https://img.shields.io/npm/l/electron-as-browser.svg?style=flat-square)

A node module to help build	browser like windows in electron.

## Features

-   Made with [BrowserView](https://electronjs.org/docs/api/browser-view) instead of [<webview>](https://electronjs.org/docs/api/webview-tag)
-   Pluggable control panel(Navigation interface)
-   Customizable web contents and browser windows
-   Tricky auto resize just out of the box

## Install

`npm i electron-as-browser`

## Usage

### First, create BrowserLikeWindow in [Main](https://electronjs.org/docs/glossary#main-process) process

```javascript
const BrowserLikeWindow = require('electron-as-browser');
const fileUrl = require('file-url');

let browser;

browser = new BrowserLikeWindow({
	controlPanel: fileUrl('you-control-interface.html'),
	startPage: 'https://page-to-load-once-open',
	blankTitle: 'New tab',
	debug: true // will open controlPanel's devtools
});

// Trigger on new tab created
browser.on('new-tab', ({ webContents }) => {
	// Customize webContents if your like
});

browser.on('closed', () => {
	// Make it garbage collected
	browser = null;
});
```

### Second, style your own browser control interface(renderer).

To make the control interface works, there are two steps:

-   Setup ipc channels to receive tabs' informations
-   Send actions to control the behaviours

For react users, there is a custom hook `useConnect` to help you setup ipc channels.

```javascript
const useConnect = require('electron-as-browser/useConnect');

const ControlPanel = () => {
	const { tabs, tabIDs, activeID } = useConnect();
	return (
		<div>Use tabs informations to render your control panel</div>
	);
}
```

For non-react users, you have to setup ipc channels yourself, there are three steps:

-   `ipcRenderer.send('control-ready')` on dom ready
-   `ipcRenderer.on('tabs-update', (e, tabs) => { // update tabs })`
-   `ipcRenderer.on('active-update', (e, activeID) => { // update active tab's id })`

Don't forget to `removeListener` on `ipcRenderer` once control panel unmounted.

Once setup ipc channels, you'll get all your control panel needed informations:

-   `[tabs](#tabs)` an object contains all the opened tab's informations
-   `[tabIDs](#tabid)` array of opened tab's ids
-   `[activeID](#tabid)` current active tab's id

Construct and style your control interface as your like.

Then you can send actions to control the browser view, the actions can require from `control`:

```javascript
import {
	sendEnterURL, // sendEnterURL(url) to load url
	sendChangeURL, // sendChangeURL(url) on addressbar input change
	sendGoBack,
	sendGoForward,
	sendReload,
	sendStop,
	sendNewTab, // sendNewTab([url])
	sendSwitchTab, // sendSwitchTab(toID)
	sendCloseTab // sendCloseTab(id)
} from 'electron-as-browser/control';
```

See [examples](./examples) for a full control interface implementation.

**NOTE: make sure to setup local bundler to compile es6 code for `electron-as-browser/useConnect` and `electron-as-browser/control`, most bundler will not compile code in node_modules by default.**

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [TabID](#tabid)
-   [Tabs](#tabs)
-   [Bounds](#bounds)
    -   [Properties](#properties)
-   [BrowserLikeWindow](#browserlikewindow)
    -   [Parameters](#parameters)
    -   [getControlBounds](#getcontrolbounds)
    -   [newTab](#newtab)
        -   [Parameters](#parameters-1)
    -   [switchTab](#switchtab)
        -   [Parameters](#parameters-2)
    -   [destroyView](#destroyview)
        -   [Parameters](#parameters-3)
-   [Tab](#tab)
    -   [Properties](#properties-1)
-   [BrowserLikeWindow#closed](#browserlikewindowclosed)
-   [BrowserLikeWindow#new-tab](#browserlikewindownew-tab)

### TabID

BrowserView's id as tab id

Type: [number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)

### Tabs

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)&lt;[TabID](#tabid), [Tab](#tab)>

### Bounds

Type: [object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

#### Properties

-   `x` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 
-   `y` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 
-   `width` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 
-   `height` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 

### BrowserLikeWindow

**Extends EventEmitter**

A browser like window

#### Parameters

-   `options` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `options.width` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** browser window's width (optional, default `1024`)
    -   `options.height` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** browser window's height (optional, default `800`)
    -   `options.controlPanel` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** control interface path to load
    -   `options.controlHeight` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** control interface's height (optional, default `130`)
    -   `options.viewReferences` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** webReferences for every BrowserView
    -   `options.winOptions` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** options for BrowserWindow
    -   `options.startPage` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** start page to load on browser open (optional, default `''`)
    -   `options.blankPage` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** blank page to load on new tab (optional, default `''`)
    -   `options.blankTitle` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** blank page's title (optional, default `'about:blank'`)
    -   `options.debug` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** toggle debug

#### getControlBounds

Get control view's bounds

Returns **[Bounds](#bounds)** Bounds of control view(exclude window's frame)

#### newTab

Create a tab

##### Parameters

-   `url` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)**  (optional, default `this.options.blankPage`)
-   `appendTo` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?** add next to specified tab's id

#### switchTab

Swith to tab

##### Parameters

-   `viewId` **[TabID](#tabid)** 

#### destroyView

Destroy tab

##### Parameters

-   `viewId` **[TabID](#tabid)** 

### Tab

Type: [object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

#### Properties

-   `url` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** tab's url
-   `title` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** tab's title
-   `favicon` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** tab's favicon url
-   `isLoading` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 
-   `canGoBack` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 
-   `canGoForward` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

### BrowserLikeWindow#closed

closed event

### BrowserLikeWindow#new-tab

new-tab event.

Type: BrowserView

## Controls

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [sendEnterURL](#sendenterurl)
    -   [Parameters](#parameters)
-   [sendChangeURL](#sendchangeurl)
    -   [Parameters](#parameters-1)
-   [sendGoBack](#sendgoback)
-   [sendGoForward](#sendgoforward)
-   [sendCloseTab](#sendclosetab)
    -   [Parameters](#parameters-2)
-   [sendNewTab](#sendnewtab)
    -   [Parameters](#parameters-3)
-   [sendSwitchTab](#sendswitchtab)
    -   [Parameters](#parameters-4)

### sendEnterURL

Tell browser view to load url

#### Parameters

-   `url` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### sendChangeURL

Tell browser view url in address bar changed

#### Parameters

-   `url` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

### sendGoBack

Tell browser view to goBack

### sendGoForward

Tell browser view to goForward

### sendCloseTab

Tell browser view to close tab

#### Parameters

-   `id` **TabID** 

### sendNewTab

Create a new tab

#### Parameters

-   `url` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** 

### sendSwitchTab

Tell browser view to switch to specified tab

#### Parameters

-   `id` **TabID** 

## useConnect

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [useConnect](#useconnect)
    -   [Parameters](#parameters)

### useConnect

A custom hook to create ipc connection between BrowserView and ControlView

#### Parameters

-   `options` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `options.onTabsUpdate` **[function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)** trigger after tabs updated(title, favicon, loading etc.) (optional, default `noop`)
    -   `options.onTabActive` **[function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)** trigger after active tab changed (optional, default `noop`)