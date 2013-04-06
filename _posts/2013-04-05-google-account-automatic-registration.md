---
layout: post
title: "google注册过程分析"
description: "google account automatic registration"
category: coding
tags: [google]
---
{% include JB/setup %}

### 前言
google账户的注册页面为[https://accounts.google.com/NewAccount](https://accounts.google.com/NewAccount),看了下google账户的注册机制，发现所有的POST变量都可以找到，只有`bgresponse`不是页面中直接可以得到的。`bgresponse`是专门验证是不是bot的。使用了google的`botguard`技术，如果不能正确的发送这个值的话，google就会要求进行手机验证。这个值的获取方法如下。

<!--more-->

### 程序

```html
<html>
<body>

<script type="text/javascript" src="https://www.google.com/js/bg/JnBnYvDSdG9SCf318u50U-J5uM1u66XDgtBIu4U1f7s.js"></script>
<script>
        document.bg = new botguard.bg('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');
        if (document.bg) {
  	        document.bg.invoke(function(response) {
 	                document.write(response);
 	        });
        }

</script>
</body>
</html>
```

使用`pywebkitgtk`处理，可以获取到javascript解析后的页面，方法为：

```python
#!/usr/bin/env python
import sys, thread
import gtk
import webkit
import warnings
from time import sleep
from optparse import OptionParser
 
warnings.filterwarnings('ignore')
 
class WebView(webkit.WebView):
    def get_html(self):
        self.execute_script('oldtitle=document.title;document.title=document.documentElement.innerHTML;')
        html = self.get_main_frame().get_title()
        self.execute_script('document.title=oldtitle;')
        return html
 
class Crawler(gtk.Window):
    def __init__(self, url, file):
        gtk.gdk.threads_init() # suggested by Nicholas Herriot for Ubuntu Koala
        gtk.Window.__init__(self)
        self._url = url
        self._file = file
 
    def crawl(self):
        view = WebView()
        view.open(self._url)
        view.connect('load-finished', self._finished_loading)
        self.add(view)
        gtk.main()
 
    def _finished_loading(self, view, frame):
        with open(self._file, 'w') as f:
            f.write(view.get_html())
        gtk.main_quit()
 
def main():
    options = get_cmd_options()
    crawler = Crawler(options.url, options.file)
    crawler.crawl()
 
def get_cmd_options():
    """
        gets and validates the input from the command line
    """
    usage = "usage: %prog [options] args"
    parser = OptionParser(usage)
    parser.add_option('-u', '--url', dest = 'url', help = 'URL to fetch data from')
    parser.add_option('-f', '--file', dest = 'file', help = 'Local file path to save data to')
 
    (options,args) = parser.parse_args()
 
    if not options.url:
        print 'You must specify an URL.',sys.argv[0],'--help for more details'
        exit(1)
    if not options.file:
        print 'You must specify a destination file.',sys.argv[0],'--help for more details'
        exit(1)
 
    return options
 
if __name__ == '__main__':
    main()
```

这样就可以得到`bgresponse`的值了。
