# 爬虫日记：批量抓取花瓣网高清美图并保存

## 使用

先上代码：

~~~~python
__author__ = 'haohao'

import os
import lxml.html
import requests
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait

# executable_path的路径为phantomjs的phantomjs.exe的路径，需要先下载，解压，然后指明路径（http://phantomjs.org/）
browser = webdriver.PhantomJS(executable_path=r"D:\python软件\phantomjs-2.1.1-windows\bin\phantomjs.exe")
# browser = webdriver.Firefox()
wait = WebDriverWait(browser, 5)
browser.set_window_size(1400, 900)


def parser(url, param):
    # 解析模块
    browser.get(url)
    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, param)))
    html = browser.page_source
    doc = lxml.html.fromstring(html)
    return doc


def get_main_url():
    print('打开主页搜寻链接中...')
    try:
        doc = parser('http://huaban.com/boards/favorite/beauty/', '#waterfall')
        name = doc.xpath('//*[@id="waterfall"]/div/a[1]/div[2]/h3/text()')
        u = doc.xpath('//*[@id="waterfall"]/div/a[1]/@href')
        for item, fileName in zip(u, name):
            main_url = 'http://huaban.com' + item
            print('主链接已找到' + main_url)
            if '*' in fileName:
                fileName = fileName.replace('*', '')
            download(main_url, fileName)
    except Exception as e:
        print(e)


def download(main_url, fileName):
    print('-------准备下载中-------')
    try:
        doc = parser(main_url, '#waterfall')
        if not os.path.exists('image\\' + fileName):
            print('创建文件夹...')
            os.makedirs('image\\' + fileName)
        link = doc.xpath('//*[@id="waterfall"]/div/a/@href')
        # print(link)
        i = 0
        for item in link:
            i += 1
            minor_url = 'http://huaban.com' + item
            doc = parser(minor_url, '#pin_view_page')
            img_url = doc.xpath('//*[@id="baidu_image_holder"]/a/img/@src')
            img_url2 = doc.xpath('//*[@id="baidu_image_holder"]/img/@src')
            img_url += img_url2
            try:
                url = 'http:' + str(img_url[0])
                print('正在下载第' + str(i) + '张图片，地址：' + url)
                r = requests.get(url)
                filename = 'image\\{}\\'.format(fileName) + str(i) + '.jpg'
                with open(filename, 'wb') as fo:
                    fo.write(r.content)
            except Exception:
                print('出错了！')
    except Exception:
        print('出错啦!')


if __name__ == '__main__':
    get_main_url()

~~~~

如果本地没有引入的依赖包，需下载。

例如下载selenium包，cmd运行：

~~~~
pip install selenium
~~~~

或者选择Anaconda Prompt，右键以管理员身份运行：

~~~~
conda install pymongo
~~~~

由于项目使用的selenium和phantomjs，所以需要先下载phantomjs的压缩包，地址：http://phantomjs.org。然后在文件中指定`phantomjs.exe`文件夹的路径。

环境集成好，直接运行就好了。



## 学习小记

~~~~python
doc = parser('http://huaban.com/boards/favorite/beauty/', '#waterfall')
#item的名字
name = doc.xpath('//*[@id="waterfall"]/div/a[1]/div[2]/h3/text()')
#item的链接地址
u = doc.xpath('//*[@id="waterfall"]/div/a[1]/@href')
~~~~

其中有一点XPath的使用，具体请看：http://www.w3school.com.cn/xpath/index.asp



![1](C:\Users\Administrator\Desktop\LearnPython\爬虫日记：批量抓取花瓣网高清美图并保存\1.png)

![2](C:\Users\Administrator\Desktop\LearnPython\爬虫日记：批量抓取花瓣网高清美图并保存\2.png)

我们根据网页代码中的元素一层层找值：

![3](C:\Users\Administrator\Desktop\LearnPython\爬虫日记：批量抓取花瓣网高清美图并保存\3.png)