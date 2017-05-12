#-*- coding: utf-8 -*-
import urllib2#访问网页用到的模块
import re
import requests
import public.bs_preprocess as bs_preprocess
from bs4 import BeautifulSoup
from lxml import html
from fake_useragent import UserAgent
import chardet
import sys
import gzip, StringIO

#reload(sys)
#sys.setdefaultencoding('utf-8')#设置默认编码格式，必须先reload
#type = sys.getdefaultencoding()
#print type
#print sys.getdefaultencoding()
domain = 'http://club.autohome.com.cn'#拼接用
ua = UserAgent()#使用随机header，模拟人类
#伪装成浏览器
headers = {'User-Agent':'ua.random'}#使用随机header，模拟人类
targetUrl ='http://club.autohome.com.cn/bbs/forum-c-359-1.html'#此处放入想爬取的车型论坛网址
targetUrl1 = targetUrl[0:targetUrl.rfind('-')+1]
print targetUrl1
a_list = []
#获取该车型论坛总共帖子页数
def getUrl():
    url =targetUrl
    s = requests.session()
    r = s.get(url, headers=headers)
    text = r.text
    soup = BeautifulSoup(bs_preprocess.bs_preprocess(text),'html.parser')
    pageNumber = soup.find('span',attrs={'class':'fs'}).text
    return int(re.findall(r"\d+\.?\d*",pageNumber)[0])

#获取论坛所有帖子的标题和地址
def getList(pn):
    url = targetUrl1+str(pn)+'.html'
    s = requests.session()
    r = s.get(url, headers=headers)
    text = r.text
    soup = BeautifulSoup(bs_preprocess.bs_preprocess(text),'html.parser')
    div = soup.find('div',attrs={'id':'subcontent'})
    #print div
    for a in div.find_all('a',attrs={'class':'a_topic'}):
        a_list.append(a['href'])
        #return a_list['href']
    #return div.find_all('a',attrs={'class':'a_topic'})
    #print str(div)

    #reg = re.compile(r'<a class="a_topic" target="_blank" href="(.+?)">(.+?)</a>',re.S)#通过正则表达式获取每个论坛明细页url和标题
    #return re.findall(reg,div)
    #result = reg.search(div)


#访问每个帖子发帖内容
def getContent(url):
    url1 = domain+url
    s = requests.session()
    r = s.get(url1, headers=headers)
    text = r.text
    soup = BeautifulSoup(bs_preprocess.bs_preprocess(text),'html.parser')
    title = soup.find('h1',attrs={'class':'rtitle'}).text#帖子标题
    title1 = repr(title)
    title2 = title1.decode('unicode-escape').encode('utf-8')
    print title2
    auto = soup.find('li',attrs={'class':'txtcenter fw'}).text#发帖作者
    print auto
    content = soup.find('div',attrs={'class':'conright fr'})#帖子内容div
    date = content.find('span',attrs={'xname':'date'}).text#发帖时间
    print date
    for conttxt in content.find_all('div',attrs={'class':'tz-paragraph'}):
        conttxt = conttxt.text#帖子文字内容
        print conttxt
    for contimg in content.find_all('div',attrs={'class':'tz-figure'}):
        contimg = contimg.div.img['src']#帖子图片链接
        print contimg


#访问每个帖子回帖内容
def getReplay(url):
    url = domain+url
    url1 = url[0:url.rfind('-')+1]#用来拼接回复地址
    #print url
    s = requests.session()
    r = s.get(url, headers=headers)
    text = r.text
    soup = BeautifulSoup(bs_preprocess.bs_preprocess(text),'html.parser')
    title = soup.find('h1',attrs={'class':'rtitle'}).text#帖子标题
    title1 = repr(title)
    title2 = title1.decode('unicode-escape').encode('utf-8')
    print title2
    auto = soup.find('li',attrs={'class':'txtcenter fw'}).text#发帖作者
    print auto
    pageNumber = soup.find('span',attrs={'class':'fs'}).text#获取回帖页数
    pageNumber =  int(re.findall(r"\d+\.?\d*",pageNumber)[0])
    #print pageNumber
    pn = 1
    while pn <= pageNumber:
        url2 = url1+str(pn)+'.html'
        print url2
        r = s.get(url2, headers=headers)
        text = r.text
        soup = BeautifulSoup(bs_preprocess.bs_preprocess(text),'html.parser')
        reply = soup.find('div',attrs={'id':'maxwrap-reply'})#找到回复的div
        for replyDiv in reply.find_all('div',attrs={'class':'clearfix contstxt outer-section'}):
            replyPeople = replyDiv.find('li',attrs={'class':'txtcenter fw'}).text
            date1 = replyDiv.find('span',attrs={'xname':'date'}).text#回帖时间
            text = replyDiv.find('div',attrs={'xname':'content'}).text#回帖内容
            print replyPeople
            print date1
            print text
        #print url2
        pn = pn +1


#调度
try:
    num = 1
    total = getUrl()
    while num <= total:
        getList(num)
        for url in a_list:
            print url
            #getContent(url)
            #getReplay(url)
            #break
        print num
        num = num+1
except:
    print '空'

#getReplay('/bbs/thread-c-2123-62898919-1.html')
