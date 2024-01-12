---
layout:     post
title:      利用爬虫获取山西能源局提供的煤矿级数据
subtitle:   
date:       2023-08-11
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Python
---

本来就不怎么会用Python，再加上好久好久没用了，花了两天时间解决了科研过程遇到的实际问题。


# 总体思路
> 1. 使用爬虫获取[山西能源局](http://nyj.shanxi.gov.cn/mkscnldxgscysxxgg/ggl/)提供的煤矿级的数据（包括煤矿名称、所属城市、开采深度、瓦斯类型等）
> 
> 2. 根据煤矿名称，数量少的可以直接通过[坐标拾取系统](https://api.map.baidu.com/lbsapi/getpoint/)，数量多的可以利用[百度地图api](https://lbsyun.baidu.com/apiconsole/key#/home)，获得煤矿对应的经纬度（[百度坐标系下](http://lbs.baidu.com/index.php?title=coordinate)）
> 
> 3. 百度坐标系转为WGS84坐标系


# 一、爬虫

山西能源局一共提供了248个煤矿的详细数据，另外还提供了一个[689个煤矿的粗略数据](http://nyj.shanxi.gov.cn/mkscnldxgscysxxgg/ggl/202203/t20220311_5333766.html)（pdf格式，只有名称和年产量）。拿一个煤矿（1/248）的网页练练手，还算比较简单。

---

    import requests
    from bs4 import BeautifulSoup
    import pandas as pd
    
    url = "http://nyj.shanxi.gov.cn/mkscnldxgscysxxgg/ggl/202308/t20230808_9088665.html"
    resp = requests.get(url)
    resp.encoding = "utf-8"
    soup = BeautifulSoup(resp.text, 'html.parser')
    p_tags = soup.select("div#zhengwen div p")
    start = 5
    end = 17
    target_p_tags = p_tags[start:end]
    # 初始化两个列表来存储第一个span和第二个span的内容
    titles = []
    data = []
    
    # 遍历每个<p>标签
    for target_p_tag in target_p_tags:
        # 找到当前<p>标签下的所有<span>标签
        span_tags = target_p_tag.find_all('span')
        # 如果有两个<span>标签
        if len(span_tags) == 2:
            titles.append(span_tags[0].text.strip())
            data.append(span_tags[1].text.strip())
        if len(span_tags) == 1:
            split_contents = span_tags[0].text.split("：")
            titles.append(split_contents[0])
            data.append(split_contents[1])
    # 打印提取的数据
    for first_content, second_content in zip(titles, data):
        print(f"Title: {first_content}\tData: {second_content}")
    df = pd.DataFrame(columns=titles)
    df.loc[0] = data
    df.to_excel('数据.xlsx', index=False)

后面需要考虑自动翻页爬取，多个煤矿的数据的拼接，遇到了不少的问题。很大的原因出在这个网页的编写有点随意，一个有规律的网页是比较容易爬取的。根据爬取中的问题，逐一解决，有了以下代码

```
import requests
from bs4 import BeautifulSoup
import pandas as pd
url = "http://nyj.shanxi.gov.cn/mkscnldxgscysxxgg/ggl/index.html"
page = 1
total_data = pd.DataFrame()
titles = ['煤矿名称','隶属企业','所在地址','生产能力','开拓方式','井筒数量','开采水平','现采煤层','采煤工艺','瓦斯等级','水文地质类型','自燃倾向性']
while True:
    resp = requests.get(url)
    resp.encoding = "utf-8"
    soup = BeautifulSoup(resp.text, 'html.parser')
    links = []
    for li in soup.select("div#main1 ul li"):
        link = li.a.get("href")
        link = link.replace("./", "/")
        # 开始爬取第一页
        sub_url = "http://nyj.shanxi.gov.cn/mkscnldxgscysxxgg/ggl" + link
        sub_resp = requests.get(sub_url)
        sub_resp.encoding = "utf-8"
        sub_soup = BeautifulSoup(sub_resp.text, 'html.parser')
        # 开始爬取第一个煤矿
        p_tags = sub_soup.select("div#zhengwen div p")
        if len(p_tags) == 0:
            p_tags = sub_soup.select("div#zhengwen p")
        data = []
        single_mine_titles = []
        if len(p_tags) <= 6:
            break  # 跳出内层循环
        for title in titles:
            # print(title)
            for p in p_tags:
                # span_tags = p.find_all('span')
                # span_texts = [span.get_text() for span in span_tags]
                # span_texts = ''.join(span_texts)
                # 递归地提取<p>元素内所有子元素的文本内容并拼接起来
                def extract_text(element):
                    text = ''
                    for sub_element in element.contents:
                        if isinstance(sub_element, str):
                            text += sub_element
                        else:
                            text += extract_text(sub_element)
                    return text
                span_texts = extract_text(p)
                if title in span_texts and ("：" in span_texts or ":" in span_texts) and not("《" in span_texts):
                    single_mine_titles.append(title)
                    split_contents = span_texts.split("：")
                    data.append(split_contents[len(split_contents)-1])
                    # 如果有两个<span>标签
                    # if len(span_tags) >= 2:
                    #     single_mine_titles.append(title)
                    #     end_index = len(span_tags) - 1
                    #     data.append(span_tags[end_index].text.strip())
                    # if len(span_tags) == 1:
                    #     single_mine_titles.append(title)
                    #     split_contents = span_tags[0].text.split("：")
                    #     data.append(split_contents[1])
        print(data[0] + 'done')
        df = pd.DataFrame(columns=single_mine_titles)
        df.loc[0] = data
        total_data = pd.concat([total_data, df],axis=0, ignore_index=True,sort=False)
        # # 遍历每个<p>标签
        # for target_p_tag in target_p_tags:
        #     # 找到当前<p>标签下的所有<span>标签
        #     span_tags = target_p_tag.find_all('span')
        #     # 如果有两个<span>标签
        #     if len(span_tags) == 2:
        #         titles.append(span_tags[0].text.strip().replace("：", ""))
        #         print(span_tags[0].text.strip().replace("：", ""))
        #         data.append(span_tags[1].text.strip())
        #     if len(span_tags) == 1:
        #         split_contents = span_tags[0].text.split("：")
        #         titles.append(split_contents[0])
        #         data.append(split_contents[1])
        # df = pd.DataFrame(columns=titles)
        # df.loc[0] = data
        # total_data = pd.concat([total_data, df], axis=0)
    print("第" + str(page) + "页已爬取完毕")
    if page > 12:
        break
    # 获取下一页url
    next_url = "http://nyj.shanxi.gov.cn/mkscnldxgscysxxgg/ggl" + "/index_" + str(page) + ".html"
    page = page + 1
    url = next_url
    print(url)
total_data.to_excel('248mines.xlsx', index=False,header=False)
```

另外，689个煤矿的pdf需要转为表格

```
import pdfplumber
import pandas as pd
pdf = pdfplumber.open("F:\博士\山西煤矿调查\数据\山西能源局\全省生产煤矿生产能力情况.pdf")
total_table_data = pd.DataFrame()
pages = len(pdf.pages)
for i in range(0,pages):
    # 访问第i页
    print(i)
    ith_page = pdf.pages[i]
    # 自动读取表格信息,返回列表
    table = ith_page.extract_table()
    ith_table_data = pd.DataFrame(table[1:],columns=table[0])
    total_table_data = pd.concat([total_table_data, ith_table_data],sort=False)

# 保存为excel
total_table_data.to_excel('689mines.xlsx',index=False)
```

## 二、根据地址获取经纬度（WGS84）

网上关于根据地址获取百度坐标的教程很多，但是[地理编码API](https://lbsyun.baidu.com/faq/api?title=webapi/guide/webservice-geocoding-base)在于只能返回一个结果，但这个结果可能并不是我们想要的。这里就需要[地点输入提示API](https://lbsyun.baidu.com/faq/api?title=webapi/place-suggestion-api)，类似于模糊查询。例如，你在地图app搜索“武汉”，弹出来的结果会包括“武汉站”、“武汉长江大桥”、“武汉大学”等等相关的结果。

经过以上调研后，初步的思路是利用百度api获取多个可能的地址（单次请求最多返回10条结果，所以我采用循环5次来输出更多的结果），然后比较  **输入的地址（来自山西能源局）** 与  **输出的地址（来自百度API）** ，看哪两个之间字符差异最小，便输出它的百度坐标，然后再转换为 WGS84坐标。

**个人用户限额问题**：地点输入提示API（属于 地点检索）每天限额100次；地理编码API 每天限额5000次。

2.1 地理编码API

这个API的本质其实是将一段url放入浏览器中

```
https://api.map.baidu.com/geocoding/v3?address=山西阳煤集团南岭煤业有限公司&output=json&ak=gPQM8Q32ZlRqxOt3zY9kllE64CiUmW9A&city=太原
```

输出为

```
{"status":0,"result":{"location":{"lng":112.51487234998418,"lat":37.81868763461172},"precise":0,"confidence":50,"comprehension":24,"level":"NoClass"}}
```

## 2.1 地点输入提示API

```
# encoding:utf-8
import requests 
import json
# 接口地址
url = "https://api.map.baidu.com/place/v2/suggestion"
# 此处填写你在控制台-应用管理-创建应用后获取的AK
ak = "gPQM8Q32ZlRqxOt3zY9kllE64CiUmW9A"
params = {
    "query":    "天安门",
    "region":    "北京",
    "city_limit":    "true",
    "output":    "json",
    "ak":       ak,
}
response = requests.get(url=url, params=params)
if response:
    html = response.text
    temp = json.loads(html, strict=False)
    names = temp['result']
    for name in names:
        print(name['name'])
```

输出如下：

```
天安门广场
天安门
天安门广场-国旗
天安门东-地铁站
天安门-城楼检票处(入口)
天安门东地铁站-c口
天安门东地铁站-b口
天安门西-地铁站
天安门东-公交车站
天安门广场-出入口
```

## 2.2 比较字符的差异

```
import difflib
def stri_similar(s1,s2):
    return difflib.SequenceMatcher(None,s1,s2).quick_ratio()
s1 = '山西阳煤集团南岭煤业有限公司'
s2 = '山西焦煤西山煤电南岭煤业公司'
s3 = '阳煤集团'
print(stri_similar(s1,s2))
print(stri_similar(s1,s3))
```

输出如下：

```
0.6428571428571429
0.4444444444444444
```

但是在实际使用中，又出现了问题。比如我们输入的是 **山西阳煤集团南岭煤业有限公司** ,百度返回的多个地址包括{**山西焦煤西山煤电南岭煤业公司**，**山西普大煤业集团有限公司**}，直接比较字符会发现**山西普大煤业集团有限公司**的相似度更高，不难发现是 山西、煤业、有限公司 这些无关紧要的词干扰了重要信息的对比，在这里更重要的信息其实是 **南岭**，有了这个思路之后，想到的办法是将地址里无关紧要的字符删去，然后再对比。

## 2.3 address2coordinates_v2.py

```
# encoding:utf-8
import requests
import json
from stri_similar import stri_similar
import re
pattern = r'山西|煤业|有限|集团|公司|煤'
def address2coordinates(address,city): #从本地的xlsx文件中获取商圈名称，作为此函数的实参
    # 接口地址
    url = "https://api.map.baidu.com/place/v2/suggestion"
    # 此处填写你在控制台-应用管理-创建应用后获取的AK
    ak = "gPQM8Q32ZlRqxOt3zY9kllE64CiUmW9A"
    params = {
        "query": address,
        "region": city,
        "city_limit": "true",
        "output": "json",
        "ak": ak,
    }

    name = []
    index = 0
    max_ratio = 0
    for i in range(3):
        response = requests.get(url=url, params=params)
        html = response.text
        temp = json.loads(html, strict=False)
        result = temp['result']
        for result_element in result:
            result_element_name = result_element['name']
            result_element_lon = result_element['location']['lng']
            result_element_lat = result_element['location']['lat']
            key_result_element_name = re.sub(pattern, '', result_element_name)
            key_address = re.sub(pattern, '', address)
            simi_ratio = stri_similar(key_address, key_result_element_name)
            # print(result_element_name)
            name.append(result_element_name)
            max_ratio = max(max_ratio, simi_ratio)
            if max_ratio == simi_ratio:
                index_max = index
                best_add = result_element_name
                best_add_lon = result_element_lon
                best_add_lat = result_element_lat
            index = index + 1
    # print('最大相似度对应的index为：' + str(index_max))
    print('与《'+ str(address) + '》匹配最好的是'+ '《' + best_add + '》')
    return best_add,best_add_lat,best_add_lon                                  #纬度 latitude,经度 longitude
```

在test.py中调用写好的address2coordinates函数

```
from bd2wgs import bd09_to_wgs84
from address2coordinates_v2 import address2coordinates
best_address = address2coordinates('山西阳煤集团南岭煤业有限公司','太原')
print(best_address)
bd_lon = best_address[2]
bd_lat = best_address[1]
wgs = bd09_to_wgs84(bd_lon, bd_lat)
print(wgs)
```

输出如下

```
与《山西阳煤集团南岭煤业有限公司》匹配最好的是《山西焦煤西山煤电南岭煤业公司》
('山西焦煤西山煤电南岭煤业公司', 37.643285, 112.204221)
[112.19133777344152, 37.63623982956219]
```

经过与Google earth对比，坐标准确无误，接下来可以批量处理了。



## 2.4 批量处理

使用v1的main.py (每日配额5000)

```
import pandas as pd
from address2coordinates_v1 import address2coordinates
from bd2wgs import bd09_to_wgs84
data = pd.read_excel(r"F:\博士\山西煤矿调查\数据\山西能源局\689mines.xlsx")
test = data.序号
for index in data.序号:                             #index为data的序号，从0开始
    print(data.loc[index - 1, '煤矿名称'])
    address = data.loc[index - 1, '煤矿名称']
    city = data.loc[index - 1, '市']
    best_address = address2coordinates(address, city)  #通过 百度api ，返回address对应的经纬度
    bd_lon = best_address[1]
    bd_lat = best_address[0]
    # print(bd_lat,bd_lon)
    # 坐标转换
    # 百度转wgs84
    wgs = bd09_to_wgs84(bd_lon, bd_lat)
    wgs_lon = wgs[0]
    wgs_lat = wgs[1]
    # data.loc[index - 1, '百度搜索的地址'] = best_address[0]
    data.loc[index-1, '纬度'] = wgs_lat
    data.loc[index-1, '经度'] = wgs_lon
# 保存为excel
data.to_excel('689mines_with_wgs84_coordinates.xlsx',index=False)
```

使用v2的main.py (每日配额100)

```
import pandas as pd
from address2coordinates_v2 import address2coordinates
from bd2wgs import bd09_to_wgs84
data = pd.read_excel(r"F:\博士\山西煤矿调查\数据\山西能源局\689mines.xlsx")
test = data.序号
for index in data.序号:                             #index为data的序号，从0开始
    print(data.loc[index - 1, '煤矿名称'])
    address = data.loc[index - 1, '煤矿名称']
    city = data.loc[index - 1, '市']
    best_address = address2coordinates(address, city)  #通过 百度api ，返回address对应的经纬度
    bd_lon = best_address[2]
    bd_lat = best_address[1]
    # print(bd_lat,bd_lon)
    # 坐标转换
    # 百度转wgs84
    wgs = bd09_to_wgs84(bd_lon, bd_lat)
    wgs_lon = wgs[0]
    wgs_lat = wgs[1]
    data.loc[index - 1, '百度搜索的地址'] = best_address[0]
    data.loc[index-1, '纬度'] = wgs_lat
    data.loc[index-1, '经度'] = wgs_lon
# 保存为excel
data.to_excel('689mines_with_wgs84_coordinates.xlsx',index=False)
```



