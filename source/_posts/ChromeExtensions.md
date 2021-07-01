---
title: Chrome Extensions
comments: true
tags: 
    - Chrome Extensions
categories: 浏览器插件
thumbnail: https://img0.baidu.com/it/u=1754948549,4053210497&fm=26&fmt=auto&gp=0.jpg
banner: https://img0.baidu.com/it/u=1754948549,4053210497&fm=26&fmt=auto&gp=0.jpg
---

![Untitled.png](Untitled.png)

# 1. 概览:

![Untitled.png](Untitled.png)

# 2. **入门**

首先我们看看的浏览器插件的定义:

浏览器插件是基于Web技术（例如HTML，JavaScript和CSS）构建的可以定制浏览体验的小型软件程序。它们使用户可以根据个人需要或偏好来定制Chrome功能和行为。

要想开发一款浏览器插件,我们只需要有一个manifest.json文件即可, 为了快速上手浏览器插件开发,我们需要把浏览器开发者工具打开, 具体步骤如下:

1. **在谷歌浏览器中输入chrome://extensions/**
2. **将开发者模式启动**

![https://pic3.zhimg.com/80/v2-a557be3300d7012d25f89d00038b7e22_720w.jpg](https://pic3.zhimg.com/80/v2-a557be3300d7012d25f89d00038b7e22_720w.jpg)

## 2.1 **导入自己的浏览器插件包**

通过以上三个步骤我们就可以开启浏览器插件开发之旅了.浏览器插件一般放在浏览器地址栏右侧,我们可以在manifest.json文件配置插件的icon,并配置一定的规则,就能看到我们的浏览器插件图标了,如下图:

![https://pic4.zhimg.com/80/v2-023f0963017950f69fee85bfaf35bd3b_720w.jpg](https://pic4.zhimg.com/80/v2-023f0963017950f69fee85bfaf35bd3b_720w.jpg)

下面我们来具体讲解一下浏览器插件开发的核心概念.

## 2.2 **核心知识点**

浏览器插件一般涉及以下几个核心文件:

- **manifest.json** 用来配置所有和插件相关的配置(必须放在根目录)
- **background.js** 后台脚本(后台页面),生命周期和浏览器一致,一般放置全局代码
- **content-scripts** 插件向页面注入脚本的一种形式,我们可以通过content-scripts向页面注入js和css资源,并可控制允许注入的范围
- **popup** 点击插件图标后打开的自定义窗口, 用来处理用户交互

笔者画了一张简图来大致表示一下它们之间的关系:

![https://pic4.zhimg.com/80/v2-bcebb2fc6d562fefac9de971f6e0ee9f_720w.jpg](https://pic4.zhimg.com/80/v2-bcebb2fc6d562fefac9de971f6e0ee9f_720w.jpg)

接下来我们来具体了解一下一上几个核心知识点.

### 2**.2.1 manifest.json**

谷歌官网简单的配置,如下:

```docker
{
    "name": "My Extension",
    "version": "2.1",
    "description": "Gets information from Google.",
    "icons": {
      "128": "icon_16.png",
      "128": "icon_32.png",
      "128": "icon_48.png",
      "128": "icon_128.png"
    },
    "background": {
      "persistent": false,
      "scripts": ["background_script.js"]
    },
    "permissions": ["https://*.google.com/", "activeTab"],
    "browser_action": {
      "default_icon": "icon_16.png",
      "default_popup": "popup.html"
    }
 }
```

各字段含义介绍如下:

- **name** 浏览器插件名称, 将会在插件列表中显示
- **description** 浏览器插件简介, 方便告诉开发者插件的功能和作用, 将会在插件列表中显示
- **version** 浏览器插件版本
- **icon** 浏览器插件图标

![Untitled%201.png](Untitled%201.png)

- **background** 背景页的脚本路径,一般为插件目录的相对地址
- **permissions** 允许使用的浏览器API的权限,比如contextMenus(右键菜单), tabs(操作标签), webRequest(使用web请求), storage(允许使用本地存储), “**http://**”(可以通过executeScript或者insertCSS访问的网站)
- **browser_action** 浏览器右上角图标设置(包括popup页面, 鼠标悬停时的标题, icon等)
- **content_scripts** 需要直接注入页面的javascript脚本
- **web_accessible_resources** 普通页面能够直接访问的插件资源列表，如果不设置是无法直接访问的
- **chrome_url_overrides** 覆盖浏览器默认页面(经常用来做浏览器的自定义桌面)
- **omnibox** 向地址栏注册一个关键字以提供搜索建议，只能设置一个关键字(多用于自定义搜索拦截)
- **default_locale** 默认语言(比如"zh_CN")

### 2**.2.2 background.js**

background页面主要用来提供一些全局配置, 事件监听, 业务转发等.举几个常用案例:

**1.定义右键菜单**

```docker
// background.js
const systems = {
  a: '趣谈前端',
  b: '掘金',
  c: '微信'
}

chrome.runtime.onInstalled.addListener(function() {
  // 上下文菜单
  for (let key of Object.keys(systems)) {
    chrome.contextMenus.create({
      id: key,
      title: systems[key],
      type: 'normal',
      contexts: ['selection'],
    });
  }
});

// manifest.json
{
    "permissions": ["contextMenus"]
}
```

效果如下:

![Untitled%202.png](Untitled%202.png)

**2.设置只有.com后缀的页面才会激活插件**

```docker
chrome.runtime.onInstalled.addListener(function() {
  // 类似于什么时候激活浏览器插件图标这种感觉
  chrome.declarativeContent.onPageChanged.removeRules(undefined, function() {
    chrome.declarativeContent.onPageChanged.addRules([{
      conditions: [new chrome.declarativeContent.PageStateMatcher({
        pageUrl: {hostSuffix: '.com'},
      })
      ],
      actions: [new chrome.declarativeContent.ShowPageAction()]
    }]);
  });
});
```

如下图所示,当页面地址的后缀不等于.com时,插件icon将不被激活:

![https://pic3.zhimg.com/80/v2-e71bb0259baff0138a3ed613d3636342_720w.jpg](https://pic3.zhimg.com/80/v2-e71bb0259baff0138a3ed613d3636342_720w.jpg)

**3.和content_script或者popup页面进行消息通信**

```docker
chrome.runtime.onMessage.addListener(
	function(request, sender, sendResponse) {
	  console.log(sender.tab ?
	              "from a content script:" + sender.tab.url :
	              "from the extension");
	  if (request.greeting == "hello")
	    sendResponse({farewell: "goodbye"});
	});
```

### 2.**2.3 content-scripts**

内容脚本一般植入会被植入到页面中, 并且可以控制页面中的dom. 我们可以利用它实现屏蔽网页广告, 定制页面皮肤等操作. 在manifest.json中的基本配置如下:

```docker
{
    "content_scripts": [{
    "matches": [
        "http://*/*",
        "https://*/*"
    ],
    "js": [
        "lib/jquery3.4.min.js",
        "content_script.js"
    ],
    "css": ["base.css"]
  }],
}
```

以上代码中我们定义了content_scripts允许注入的页面范围, 插入页面的js以及css, 这样我们就能轻松改变某一个页面的样式.比如我们可以在页面中注入一个按钮:

![Untitled%203.png](Untitled%203.png)

后面的浏览器插件案例中会详细介绍content_scripts的用法.

### **2.2.4 popup**

popup是用户点击插件图标时打开的一个小窗口，当失去焦点后窗口就立即关闭，我们一般用它来处理一些简单的用户交互和插件说明。

由于popup窗口也是一个网页,所以我们一般会建立一个popup.html和popup.js用来控制popup的页面展示和交互.我们在manifest.json中配置如下:

```docker
{
    "page_action": {
        "default_title": "小夕图片提取插件",
        "default_popup": "popup.html"
  },
}
```

这里要注意一点的是,我们在popup.html中不能直接使用script脚本,需要用引入脚本文件的方式.如下:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>在线图片提取工具</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
    <div class="pop-wrap">
    </div>
    <script src="lib/jquery3.4.min.js"></script>
    <script src="popup.js"></script>
  </body>
</html>
```

以下是笔者写的一个插件的popup页面:

![Untitled%204.png](Untitled%204.png)

## 2.3.通信机制

对于一个相对复杂的浏览器插件来说,我们不仅仅只操作dom或者提供基本的功能就行了,我们还需要向第三方或者自己的服务器抓取有用的页面数据,这个时候就需要用到插件的通信机制了.

因为content_script脚本存在于当前页面,受同源策略影响,导致我们无法将捕获到的数据传给第三方平台或者自己的服务器, 所以我们需要基于浏览器的通信API.一下是谷歌浏览器插件的通信流程:

![https://pic2.zhimg.com/80/v2-5a05979498bf90fe0443290623e1c8cd_720w.jpg](https://pic2.zhimg.com/80/v2-5a05979498bf90fe0443290623e1c8cd_720w.jpg)

### 2.**3.1 popup和background相互通信**

由官方文档可知popup可以直接访问background页的方法,所以popup可以直接与其通信:

```html
// background.js
var getData = (data) => { console.log('拿到数据:' + data) }
// popup.js
let bgObj = chrome.extension.getBackgroundPage();
bgObj.getData(); // 访问bg的函数
```

### 2.**3.2 popup或者background页和content_script通信**

这里我们使用chrome的tabs API,如下:

```html
// popup.js
// 发送消息给content_script
chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    chrome.tabs.sendMessage(tabs[0].id, "activeBtn", function(response) {
      console.log(response);
    });
  });
  
// 接收消息
chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {
    console.log(sender.tab ?
                "from a content script:" + sender.tab.url :
                "from the extension");
    if (request.greeting == "hello")
      sendResponse({farewell: "goodbye"});
  });
```

content_script接收和发送消息:

```html
// 接收消息
chrome.runtime.onMessage.addListener(
  function(message, sender, sendResponse) {
    if (message == "activeBtn"){
      // ...
      sendResponse({farewell: "激活成功"});
    }
 });
 
 // 主动发送消息
 chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
   console.log(response, document.body);
   // document.body.style.backgroundColor="orange"
});
```

有关消息的长连接,在谷歌官网也写的很清楚:

![https://pic2.zhimg.com/80/v2-8064fff68ca7ab8e0d40631b6b7155fd_720w.jpg](https://pic2.zhimg.com/80/v2-8064fff68ca7ab8e0d40631b6b7155fd_720w.jpg)

我们可以采用如下方式进行长连接:

```html
// content_script.js
var port = chrome.runtime.connect({name: "徐小夕"});
port.postMessage({Ling: "你好"});
port.onMessage.addListener(function(msg) {
  if (msg.question == "你是做什么滴?")
    port.postMessage({answer: "搬砖"});
  else if (msg.question == "搬砖有钱吗?")
    port.postMessage({answer: "木有"});
});

// popup.js
chrome.runtime.onConnect.addListener(function(port) {
  port.onMessage.addListener(function(msg) {
    if (msg.Ling == "你好")
      port.postMessage({question: "你是做什么滴?"});
    else if (msg.answer == "搬砖")
      port.postMessage({question: "搬砖有钱吗?"});
    else if (msg.answer == "木有")
      port.postMessage({question: "太难了."});
  });
});
```

## **4.数据存储**

chrome.storage用来针对插件全局进行数据存储,我们在任何一个页面(popup或content_script或background)下存储了数据,我们在以上三个页面都可以获取到, 具体用法如下:

```html
获取数据
chrome.storage.sync.get('imgArr', function(data) {
  console.log(data)
});
// 保存数据
chrome.storage.sync.set({'imgArr': imgArr}, function() {
    console.log('保存成功');
  });
  
// 另一种方式
chrome.storage.local.set({key: value}, function() {
  console.log('Value is set to ' + value);
});
```

## **5.应用场景**

谷歌浏览器的插件应用场景很多,正如文章开头的思维导图中写的.以下是笔者总结的一些应用场景，大家感兴趣可以尝试去实现：

- 谷歌浏览器自定义桌面
- 网页性能分析工具
- 网页爬虫
- 埋点工具
- 网页热力图生成工具
- 安全拦截插件
- 广告过滤插件
- 网站动态换肤
- 第三方数据导入
- 代码格式化工具
- 在线协作工具
- 防作弊插件

还有很多实用工具可以开发，大家可以好好把玩。接下来就来通过实现一个网页图片提取插件，来总结以下浏览器插件开发流程。

## 6.开发一款抓取网站图片资源的浏览器插件

首先还是按照笔者的风格，在开发任何一种工具之前都要明确需求，所以我们来看看该插件的功能点：

- 能植入网页按钮，通过点击按钮捕获网页图片
- 能在用户端展示捕获的图片
- 点击插件能预览捕获的图片

基本上就这几个功能，接下来我会展示核心代码，在介绍代码之前我们先预览一下插件的实现效果：

![https://pic3.zhimg.com/80/v2-0c79ea16b92fb0619cd70248a8090462_720w.jpg](https://pic3.zhimg.com/80/v2-0c79ea16b92fb0619cd70248a8090462_720w.jpg)

![https://pic2.zhimg.com/80/v2-ecee95b625f566a9595d73d55768c211_720w.jpg](https://pic2.zhimg.com/80/v2-ecee95b625f566a9595d73d55768c211_720w.jpg)

插件目录结构如下：

![https://pic1.zhimg.com/80/v2-79f003a0e9a21ec8566a37a3a4820e50_720w.jpg](https://pic1.zhimg.com/80/v2-79f003a0e9a21ec8566a37a3a4820e50_720w.jpg)

因为插件的开发比较简单，所以我直接用jquery开发。这里我们主要关注popup.js和content_script.js, popup.js中主要用来获取从content_script页传过来的图片数据，并展示在popup.html中，另外又一个需要注意的是当页面没有注入生成按钮时，popupu需要发送信息给content页面，主动让其生成按钮，代码如下：

```jsx
chrome.storage.sync.get('imgArr', function(data) {
  data.imgArr && data.imgArr.forEach(item => {
    var imgWrap = $("<div class='img-box'></div>")
    var img = $("<img src='" + item + "' alt='" + item + "' />")
    imgWrap.append(img);
    $('#content').append(imgWrap);
    $('.empty').hide();
  })
});

$('#activeBtn').click(function(element) {
  chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    chrome.tabs.sendMessage(tabs[0].id, "activeBtn", function(response) {
      console.log(response);
    });
  });
});
```

对于content页面，我们需要实现的是动态生成按钮，并且在页面中植入弹窗来展示获取到的图片，另一方面需要将图片数据传递给storage，以便popup页面可以获取图片数据。

由于页面比较简单，笔者就不用过多的第三方库了，笔者先简单手写一个modal组件，代码如下：

```jsx
// 弹窗
~function Modal() {
  var modal;
  
  if(this instanceof Modal) {
    this.init = function(opt) {
      modal = $("<div class='modal'></div>");
      var title = $("<div class='modal-title'>" + opt.title + "</div>");
      var close_btn = $("<span class='modal-close-btn'>X</span>");
      var content = $("<div class='modal-content'></div>");
      var mask = $("<div class='modal-mask'></div>");
      close_btn.click(function(){
        modal.hide()
      })
      title.append(close_btn);
      content.append(title);
      content.append(opt.content);
      modal.append(content);
      modal.append(mask);
      $('body').append(modal);
    }
    this.show = function(opt) {
      if(modal) {
        modal.show();
      }else {
        var options = {
          title: opt.title || '标题',
          content: opt.content || ''
        }
        this.init(options)
        modal.show();
      }
    }
    this.hide = function() {
      modal.hide();
    }
  }else {
    window.Modal = new Modal()
  }
}()
```

**第一步，我们先批量获取页面图片数据：**

```jsx
var imgArr = []
$('img').each(function(i) {
  var src = $(this).attr('src');
  var realSrc = /^(http|https)/.test(src) ? src : location.protocol+ '//' + location.host + src;
  imgArr.push(realSrc)
})
```

因为图片的src路径可能是相对地址，所以笔者在这里用正则简单处理以下，当然我们可以进行更细粒度的控制。

**第二步，将图片数据存储到storage中：**

```jsx
chrome.storage.sync.set({'imgArr': imgArr}, function() {
  console.log('保存成功');
});
```

**第三步，生成预览图片的弹窗，这里用笔者上面实现的modal组件**：

```jsx
Modal.show({
  title: '提取结果',
  content: imgBox
})
```

**第四步，当popup发送激活按钮的通知时，我们要在网页中动态插入生成按钮：**

```jsx
chrome.runtime.onMessage.addListener(
  function(message, sender, sendResponse) {
    if (message == "activeBtn"){
      if(!$('.crawl-btn')) {
        $('body').append("<div class='crawl-btn'>提取</div>")
      }else {
        $('.crawl-btn').css("background-color","orange");
        setTimeout(() => {
          $('.crawl-btn').css("background-color","#06c");
        }, 3000);
      }
      sendResponse({farewell: "激活成功"});
    }
 });
```

setTimeout那段纯属是为了吸引用户视线，当然我们可以用更优雅的方式来处理。插件核心代码主要是这些，当然还有很多细节要考虑，我把配置文件和一些细节放到github了，如果感兴趣的朋友可以安装感受一下。

github地址：[https://github.com/MrXujiang/fetchImg](https://github.com/MrXujiang/fetchImg)