---
layout: article
title:  "如何使用Selenium来下载PDF文件"
date:   2022-05-10 10:00:00 +0800
tags: Blockchain ZKP Groth16
categories: Python Selenium
---

## 如何使用Selenium来下载PDF？

今天帮老婆写了个爬虫，遇到了这么一个需求: 从给定的页面上下载一个PDF文件。

但是我们遇到的问题是：不管怎么处理，PDF文件都会在模拟下载后使用Chrome打开，而不是另存为到目标文件夹。搜了一圈，发现不管中英文的结果都是如何把一个html页面保存成PDF。大概是有将页面保存成PDF的需求的人更多。

这里分享一下结论，希望能帮助到遇到之后相同问题的开发人员:

- 在创建`chromedriver`的时候，同时传入一个`webdriver.chrome.options.Options`, 如下:

```python
    options = webdriver.chrome.options.Options()
    # 这里是自定义的下载文件夹地址
    download_dir = "" 
    options.add_experimental_option('prefs',  {
        // 目标文件夹
        "download.default_directory": download_dir,
        // 关闭用Chrome原生的打开PDF
        "download.prompt_for_download": False,
        "download.directory_upgrade": True,
        "plugins.always_open_pdf_externally": True
        }
    )
    // 创建webdriver的时候额外传入options
    driver = webdriver.Chrome(chrome_options=options)
```

上述的options可以使得Chrome在遇到一个目标为PDF类型的地址时，不会用原生浏览的方式打开它，而是将其保存(下载)到options中指定的目录中。