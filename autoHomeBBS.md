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

#获取论坛首页所有帖子的标题和地址
def getList(pn):
	url = 'http://club.autohome.com.cn/bbs/forum-c-3788-%d.html'%pn
	req = urllib2.Request(
   			url = url,
    		#data = postdata,
   			headers = headers
		)
	opener = urllib2.urlopen(req,timeout = 10)#打开论坛
	content = opener.read()#访问源代码
	text = content.decode('gbk','ignore').encode('utf-8')#格式转换ignore忽略错误
	text = bs_preprocess.bs_preprocess(text)
	reg = re.compile(r'<a class="a_topic" target="_blank" href="(.+?)">(.+?)</a>')#通过正则表达式获取每个论坛明细页url和标题
	return re.findall(reg,text)
	#print text
	#print div

#访问每个帖子发帖内容
def getContent(url):
    url1 = domain+url
    s = requests.session()
    r = s.get(url1, headers=headers)
    text = r.text
    """
    req = urllib2.Request(
        url = url1,
        #data = postdata,
        headers = headers
    )
    opener = urllib2.urlopen(req,timeout = 10)#打开每个帖子
    text = opener.read()#访问源代码
    print text
    """
    #print chardet.detect(text)['encoding']
    #text = text.decode('gbk','ignore').encode('utf-8')#格式转换,ignore用来忽略错误
    #print text
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
    #print text
    #text = text.decode('gbk','ignore').encode('utf-8')#格式转换,ignore用来忽略错误
    soup = BeautifulSoup(bs_preprocess.bs_preprocess(text),'html.parser')
    title = soup.find('h1',attrs={'class':'rtitle'}).text#帖子标题
    title1 = repr(title)
    title2 = title1.decode('unicode-escape').encode('utf-8')
    print title2
    auto = soup.find('li',attrs={'class':'txtcenter fw'}).text#发帖作者
    print auto
    pageNumber = soup.find('span',attrs={'class':'fs'}).text#回帖页数
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


try:
    for i in getList(1000):
        url = i[0]
        title = i[1]
        print url,title
        getContent(url)
        getReplay(url)
        #break
except:
    print '空'

#getReplay('/bbs/thread-c-2123-62898919-1.html')
