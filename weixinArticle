from typing import re
import re
import requests
from requests import ConnectionError
from  urllib.parse import urlencode
from pyquery import PyQuery as pq
from lxml.etree import XMLSyntaxError
#from spider.config import *
base_url = 'http://weixin.sogou.com/weixin?'

headers = {
    'Cookie': 'IPLOC=CN3401; SUID=B04320702320940A000000005C89F802; SUV=1552545793916385; ABTEST=5|1552545799|v1; SNUID=AC583C6C1C19981420C7053D1CC6BF4F; weixinIndexVisited=1; JSESSIONID=aaaodbxoWtJI7emiUXRLw; sct=2',
    'Host': 'weixin.sogou.com',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36'
}

keyword = '风景'
proxy_url = 'http://localhost:5000/get'
proxy = None
max_count = 5

# client = pymongo.MongoClient(MONGO_URI)
# db = client[MONGO_DB]

#获取代理
def get_proxy():
    try:
        response = requests.get(proxy_url)
        if response.status_code == 200:
            return response.content.decode()
        return None
    except ConnectionError:
        return None
#获取页面内容
def get_html(url, count=1):
    global proxy
    print('Crawing', url)
    print('Trying Count', count)
    if count >= max_count:
        print('Tries too many Counts')
        return None
    try:
        #判断是否有代理
        if proxy:
            proxies = {
                'http': 'http://' + proxy
            }
            response = requests.get(url, allow_redirects=False, headers=headers, proxies=proxies)
        else:
            response = requests.get(url, allow_redirects=False, headers=headers)
        if response.status_code == 200:
            return response.content.decode()
        if response.status_code == 302:
            #需要代理
            print('302')
            proxy = get_proxy()
            if proxy:
                print('User Proxy', proxy)
                return get_html(url)
            else:
                print("Get proxy Failed")
                return None
    except ConnectionError as e:
        print('Error Occurred', e.args)
        proxy = get_proxy()
        count += 1
        return get_html(url, count)
#1.获取索引页
def get_index_page(query,page):
    data = {
        'query': query,
        'type': 2,
        'page': page
    }
    queries = urlencode(data)
    url = base_url + queries
    html = get_html(url)
    return html

def parse_index(html):
    doc = pq(html)
    items = doc('.news-box .news-list li .txt-box h3 a').items()
    #print(items)
    for item in items:
        yield item.attr('href')


def get_detail(url):
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return response.text
        return None
    except ConnectionError:
        return None

def parse_detail(html):
    try:
        doc = pq(html)
        title = doc('.rich_media_title').text()
        # title2 = doc('#activity-name').text()
        # print("-----------" + title2)
        content = doc('.rich_media_content').text()
        #date = doc('#post-date').text()
        #date = doc('#publish_time').text()
        date_pattern = re.compile('var publish_time = \"(.*?)\"', re.S)
        date_search = re.search(date_pattern, html)
        if date_search:
            date = date_search.group(1)
        else:
            date = None
        #print("----------------"+date)
        nickname = doc('#js_profile_qrcode > div > strong').text()
        wechat = doc('#js_profile_qrcode > div > p:nth-child(3) > span').text()
        return {
            'title': title,
            'content': content,
            'date': date,
            'nickname': nickname,
            'wechat': wechat
        }
    except XMLSyntaxError:
        return None

# def save_to_mongo(data):
#     if db['articles'].update({'title': data['title']}, {'$set': data}, True):
#         print('Saved to Mongo', data['title'])
#     else:
#         print('Saved to Mongo Failed', data['title'])

def main():
    for page in range(1, 101):
        #print(page)
        html = get_index_page(keyword,page)
        #print(html)
        if html:
            article_urls = parse_index(html)
            for article_url in article_urls:
                print(article_url)
                article_html = get_detail(article_url)
                if article_html:
                    article_data = parse_detail(article_html)
                    print(article_data)
                    # if article_data:
                    #     save_to_mongo(article_data)

if __name__ == '__main__':
    main()
