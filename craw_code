# coding:utf8
import urllib
import re
from urllib.parse import urljoin
from bs4 import BeautifulSoup  # from beautifulsoup4 import BeautifulSoup

class UrlManager(object): #建立URL管理类，实现对未爬URL和已爬URL的管理
    def __init__(self):  #初始化UrlManager属性，创建未爬集合和已爬集合
        self.new_urls = set()
        self.old_urls = set()

    def add_new_url(self, url): #定义新加url是否为未读url方法
        if url is None:
            return
        if url not in self.new_urls and url not in self.old_urls:
            self.new_urls.add(url)

    def add_new_urls(self, urls): #定义获取新url方法
        if urls is None or len(urls) == 0:
            return
        for url in urls:
            self.add_new_url(url)

    def has_new_url(self): #返回是否还有未爬取url
        return len(self.new_urls) != 0

    def get_new_url(self): #将未爬取url逐个弹出
        new_url = self.new_urls.pop()
        self.old_urls.add(new_url)
        return new_url

class HtmlDownloader(object): #创建html下载类,实现对html的下载
    def download(self, new_url):  
        if new_url is None:
            return None
        response = urllib.request.urlopen(new_url)
        if response.getcode() == 200:
            return response.read()
        else:
            return None

class HtmlParser(object): #创建html解析类，实现对html的解析
    def parse(self, new_url, html_cont):
        if new_url is None or html_cont is None:
            return
        soup = BeautifulSoup(html_cont, "html.parser", from_encoding='utf-8')
        new_urls = self.get_new_urls(new_url, soup)
        new_data = self.get_new_data(new_url, soup)
        return new_urls, new_data

    def get_new_urls(self, new_url, soup): #获取解析后的新url
        #<a target="_blank" href="/item/%E8%A7%A3%E9%87%8A%E5%99%A8">解释器</a>
        new_urls = set()
        links = soup.find_all('a', href=re.compile(r"/item/"))
        for link in links:
            new_parse_url = link['href']
            new_full_url = urljoin(new_url, new_parse_url)
            new_urls.add(new_full_url)
        return new_urls


    def get_new_data(self, new_url, soup): #获取解析后新数据
        #<dd class="lemmaWgt-lemmaTitle-title"><h1>Python</h1><h2>（计算机程序设计语言）</h2>
        res_data = {}
        res_data['url'] = new_url
        title_node = soup.find('dd', class_="lemmaWgt-lemmaTitle-title").find("h1")
        # try:
        #     title_node_2 = soup.find('dd', class_="lemmaWgt-lemmaTitle-title").find("h2")
        #     res_data['title'] = title_node.get_text() + title_node_2.get_text()
        # except:
        #     res_data['title'] = title_node.get_text()
        res_data['title'] = title_node.get_text()
        #<div class="lemma-summary" label-module="lemmaSummary">
        summary_node = soup.find('div', class_="lemma-summary")
        res_data['summary'] = summary_node.get_text()
        return res_data

class HtmlOutputer(object): #将数据以html格式输出
    def __init__(self):
        self.datas = []
    def collect_data(self, new_data):
        if new_data is None:
            return
        self.datas.append(new_data)

    def output_html(self):
        fout = open('output.html', 'w',encoding='utf-8')
        fout.write('<html>')
        fout.write('<body>')
        fout.write('<table>')
        #ascii
        for data in self.datas:
            fout.write('<tr>')
            fout.write("<td>%s</td>"%data['url'])
            fout.write("<td>%s</td>"%data['title'])
            fout.write("<td>%s</td>"%data['summary'])
            fout.write('</tr>')
        fout.write('</table>')
        fout.write('</body>')
        fout.write('</html>')
        fout.close()

class SpiderMain(object): #主调度类
    def __init__(self):
        self.urls = UrlManager()
        self.downloader = HtmlDownloader()
        self.parser = HtmlParser()
        self.outputer = HtmlOutputer()

    def craw(self, root_url):
        count = 1
        self.urls.add_new_url(root_url)
        while self.urls.has_new_url():
            try:
                new_url = self.urls.get_new_url()
                print("craw %d : %s" % (count, new_url))
                html_cont = self.downloader.download(new_url)
                new_urls, new_data = self.parser.parse(new_url, html_cont)
                self.urls.add_new_urls(new_urls)
                self.outputer.collect_data(new_data)
                if count == 1000:
                    break
                count += 1
            except:
                print("craw failed")

        self.outputer.output_html()

if __name__ == "__main__":
    root_url = "https://baike.baidu.com/item/Python/407313?fr=aladdin" #入口url
    obj_spider = SpiderMain()
    obj_spider.craw(root_url)
