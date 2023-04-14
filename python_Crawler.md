# python 爬虫

## 环境

> python 3.7 +pycharm 可实现单独的版本控制
>
> requests **请求**   beautifulsoup4 **解析HTML**  Selenium **多线程**

### 分析网页源代码

* 右键查看源代码

  > 这个没有经过js渲染过的代码

* 右键检查

  > Elements **这个是经过渲染的源代码**
  >
  > > 这个还可以直接右键检查定位到源码的位置在哪

  > Network

  > >  展示加载时间
  >
  > > DIseable cache       不让它缓存
  >
  > > preserve log 	     让他去其他页面也要加载日志 
  >
  > > doc                          显示他加载的数据  **一般看这个**
  > >
  > > headers                  
  > >
  > > > 看请求的URL    
  > > >
  > > > 请求的方法
  > > >
  > > > 写程序的时候可以用返回的状态码来判断
  > > > content-Type 返回他的格式和编码
  > > >
  > > > cookice    **同下**
  > > >
  > > > user-Agent    **用来伪装成正常用户**
  > >
  > > Respose   显示的就是文档的HTML

  ## Request

  ``` request.get/post(url,params,data,header,timeout,verify,allow_redirects,cookies)``` 

  * url : 要下载的目标网页的url
  * params : 字典形式 ，设置URL后面的参数 ，比如？id=123&name=xaiozhang
  * data : 字典或者字符串， 一般用于post方法提交数据
  * headers : 设置urer-agent. refer等请求头
  * timeout : 超时时间，单位秒
  * verify : True/False 是否进行https证书的验证，默认是 需要自己设置证书地址
  * allow_redirects : True/Flase 默认是 : 是否让request做重定向处理
  * cookie ：附带本地的cookie 数据

  ```r = requests.get/post(url)```

  * r.status_code **查看当前状态码**
  * r.encoding   **查看编码以及变更编码 request会根据headers推测编码，推测不到就会设为Iso-8859-1可能会乱码**
  * r.text    **查看返回的网页类容**
  * r.headers **查看返回的http的headess**
  * r.url **查看实际访问的URL**
  * r.content **以字节的方式返回内容，比如用于下载图片**
  * r.cookies **服务端写入本地的cookie数据**

## Beautifulsoup解析Html 网页

* 语法

  > 创建一个beautifulsoup 对象
  >
  > ``` python
  > from bs4 import BeautifulSoup 
  > soup = BeautifulSoup(html_doc,  #HTML 文档字符串
  >                     "html.parser",  #HTML解析器
  >                     from_encoding="utf_8") #编码格式
  >  
  > ```
  >
  > **搜索节点 find_all find**
  >
  > > 按节点名称
  > >
  > > 按节点属性
  > >
  > > 按节点文字
  >
  > ```python
  > find_all(name, attrs, string)  			#查找所有节点
  > soup.find_all('a')         				#查找标签为a的节点
  > soup.find_all('a',href='/view/123.html')#找标签a,链接为/view/123.html
  > soup.fing_all('div',class_='abc',string='python')#找标签为div 类是abs 文字是python
  > #class 与python关键字冲突了
  > ```
  >
  > 
  >
  > 访问节点 **名称 属性 文字**
  >
  >   ```python
  >   <a href='1.html'>python</a>
  >   node.name  		#找标签的名称
  >   node['href']	#找节点的属性
  >   node.get_text() #找节点文字
  >   ```

### 爬虫通用的三大模块

* URL管理器

  ```python
  class UrlManage:
      def __init__(self):
          self.new_urls = set()
          self.old_urls = set()
      def add_new_url(self, url):
          if url is None or len(url) == 0:
              return
          if url in self.new_urls or url in self.old_urls:
              return
          self.new_urls.add(url)
      def add_new_urls(self, urls):
          if urls is None or len(urls) == 0:
              return
          for url in urls:
              self.add_new_url(url)
      def get_url(self):
          if self.has_new_url():
              url = self.new_urls.pop()
              self.old_urls.add(url)
              return url
          else:
              return None
      def has_new_url(self):  # 看有没有新的在爬取的url
          return len(self.new_urls) > 0
  
  ```

  

* 网页解析器

  ```python
  if __name__ == '__main__':
      root_url = 'http://www.crazyant.net/'
      manger = url_manager.UrlManage()
      manger.add_new_url(root_url)
      fout = open("url.txt", 'w')
      while manger.has_new_url():
          url = manger.get_url()
          r = requests.get(url, timeout=3)
          if r.status_code != 200:
              print("timeout 200")
              continue
          soup = BeautifulSoup(r.text, "html.parser")
          title = soup.title.string
          fout.write("%s\t%s\n" % (url, title))
          fout.flush()
          links = soup.findAll('a')
          for link in links:
              href = link.get("href")
              if href is None:
                 continue
              pattern = r'^http://www.crazyant.net/\d+.html$'
              if re.match(pattern, href):
                  manger.add_new_url(href)
          print("success: %s, %s, %s" % (url, title, len(manger.new_urls)))
  fout.close()
  ```

  

* 网页下载器

### bug

* 代理问题

  ```os.environ['NO_PROXY'] = url```

* 访问的链接要加http

* header 是字典