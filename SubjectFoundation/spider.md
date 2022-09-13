[TOC]
### 分布式与并行
>分布式指服务分配在多台机器上，且机器之间保持一定的协作关系。当任务到达时，将任务划分成小块分配给其他机器，然后综合这些结果得到最终的结果。
>实现一个分布式的应用，并不是单纯的将多个程序并行在多个机器上启动，而且要合理利用节点里的每台机器，完成大量的计算任务。
### selenium

```
https://www.jianshu.com/p/1531e12f8852

```

```
browser = webdriver.Chrome(executable_path=r'\chromedriver.exe',chrome_options=chrome_options)
browser.get('https://www.baidu.com')
logging.info(browser.page_source)
```

pdf下载

>https://www.cnblogs.com/lingwang3/p/14440087.html

```
chrome_options = webdriver.ChromeOptions()
chrome_options.add_experimental_option("excludeSwitches", ['enable-automation'])
chrome_options.add_experimental_option('prefs', {
    "download.default_directory": pdf_directory,
    "download.prompt_for_download": False,
    "download.directory_upgrade": True,
    "plugins.always_open_pdf_externally": True  # 这句配置很重要
    }
                                   )
browser = webdriver.Chrome(executable_path=r'\chromedriver.exe',chrome_options=chrome_options)
browser.get('https://www.caida.org/publications/papers/2012/analysis_slash_zero/analysis_slash_zero.pdf')
time.sleep(10)
logging.info(browser.page_source)
```



### beautifulsoup4

##### 使用find连续调用获取子级元素div即可

```
bs=BeautifulSoup(response,"html.parser")
allList=bs.findAll(id='enterprise-body')
enterprise_body=allList[0]
for technique_sidenav in enterprise_body.findAll(attrs={'class':'sidenav'}):
        technique_body=technique_sidenav.find(name='div',attrs={'class':'sidenav-body'})
        for subtechnique_sidenav in technique_body.findAll(attrs={'class':'sidenav'}):
            subtechnique_body = subtechnique_sidenav.find(name='div',attrs={'class':'sidenav-body'})
            for sub_subtechnique_sidenav in subtechnique_body.findAll(attrs={'class':'sidenav'}):
                sub_subtechnique_header = sub_subtechnique_sidenav.find(attrs={'class':'sidenav-head'})
```



##### find_all函数检索所有的子孙

```
查看findAll函数，其具有参数recursive可设置是否递归。
```



