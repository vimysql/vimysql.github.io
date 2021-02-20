---
layout: post
title: Python获取年报数据1-会计师事务所
category: [python]
tags: [web scraping]
---

##### 爬取新浪财经年报数据1-会计师事务所
作为大A股多家上市公司的“股东”（韭菜），不仅需要关注上市公司的质量，还需要关注为其服务的会计师事务所。
###### 以下为本实验的缘由
你说你好好的注册会计师不好好干，来掺和这*ST是为了什么？
![image](/img/2021-02-20-python-stockndbg/stockndbg_1.png)

```
import xlrd #（excel read）来读取Excel文件
import xlwt #（excel write）来生成Excel文件
import urllib.request
import re
import os
import time
import random
import pandas as pd

workbook = xlwt.Workbook()  # 新建一个工作簿
sheet = workbook.add_sheet('sheet_name',cell_overwrite_ok=True)  # 在工作簿中新建一个表格
file_folder= "E:/pyproject/stockndbg/filepath/"
file_source="E:/pyproject/stockndbg/stock.txt"  # 证券代码清单txt
f=open(file_source)
stock = []
for line in f.readlines():
    line = line.replace('\n','')
    stock.append(line)
#print(stock)
f.close()


def write_excel_xls(path,value,inum):
    index = len(value)  # 获取需要写入数据的行数
    # print("index is",index)
    for num in range(0, index):
        sheet.write(inum,num,value[num])
    # for i in range(0, index):
    #     for j in range(0, len(value[i])):
    #         sheet.write(i, j, value[i][j])  # 像表格中写入数据（对应的行和列）
    print("xls格式表格写入数据成功！")

def requested(url):
    import requests
    headers = {'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'}
    print(url)
    response = requests.get(url, headers=headers)
    response.encoding = 'gbk'  # 解决乱码问题
    html_text = response.text
    return html_text

def xpathed(html_text):
    import lxml
    from lxml import etree
    selector = etree.HTML(html_text)
    title = selector.xpath('//tbody/tr/td//text()')
    # 去除其中的冒号
    title_1 = []
    for i in title:
        i = i.strip()
        i = i.strip('：')
        # title_1.append(i)
        for j in i.split():
            print("--------------",j)
            title_1.append(j)
        title_1.append(i)
    print("title_1", title_1[00:1500])
    return title_1

def pachong(url):
    req.add_header('User-Agent','Mozilla/5.0 (Windows NT 6.2; rv:16.0) Gecko/20100101 Firefox/16.0')
    page = urllib.request.urlopen(req)
    time.sleep(random.random() * 3)
    try:
        html = page.read().decode('gbk')
        target = r'&id=[_0-9_]{7}'
        # target = r'&id=[_0-9_]{6}'
        target_list = re.findall(target,html)
        sid = each1
        if len(target_list)>0:
            year=2019
            each2=target_list[0]
            target_url='http://vip.stock.finance.sina.com.cn/corp/view/vCB_AllBulletinDetail.php?stockid='+sid+each2
            url=target_url
            # 开始爬虫
            html_text=requested(url)
            title_1=xpathed(html_text)

            time_now=time.strftime("%Y%m%d", time.localtime())
            file_name=each1+"_workbook.xls"
            path1 =file_folder+time_now+"_"+str(year)+"_"+file_name
            iinum = 1
            print(path1)
            for title in range(len(title_1)):
                if title_1[title].find('境内会计师事务所名称')>-1:
                    for i in range(20):
                        #print(i,"境内会计师事务所名称",title_1[title+i])
                        # 存入excel
                        write_excel_xls(path1, [each[0:7], each[8:], target_url, title_1[title],title_1[title+1],title_1[title+2],title_1[title+3],title_1[title+4],title_1[title+5],title_1[title+6],title_1[title+7],title_1[title+8],title_1[title+9],title_1[title+10],title_1[title+11],title_1[title+12]], iinum)
                        workbook.save(path1)  # 保存工作簿
                        iinum = iinum + 1

    except ExceptionType:
        print('年报列表页面编码错误;',path1,str(year)+title_1[0])

for each in stock:
    each1=each[2:8]
    url='http://vip.stock.finance.sina.com.cn/corp/go.php/vCB_Bulletin/stockid/'+each1+'/page_type/ndbg.phtml'
    req = urllib.request.Request(url)
    pachong(url)

#构建新的表格名称
new_filename = file_folder + time.strftime("%Y%m%d%H%M%S", time.localtime())+'_newfile.xls'
#找到文件路径下的所有表格名称
file_list = os.walk(file_folder)
new_list = []

for dir_path,dirs,files in file_list:
    for file in files:
         #重构文件路径
        file_path = os.path.join(dir_path,file)
        #将excel转换成DataFrame
        df = pd.read_excel(file_path)
        new_list.append(df)

#多个DataFrame合并为一个
df = pd.concat(new_list)
#写入到一个新excel表中
df.to_excel(new_filename,index=False)


```

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
