# 用Python爬取微博数据生成词云图片

很早之前写过一篇怎么利用微博数据制作词云图片出来，之前的写得不完整，而且只能使用自己的数据，现在重新整理了一下，任何的微博数据都可以制作出来，放在今天应该比较应景。

一年一度的虐汪节，是继续蹲在角落默默吃狗粮还是主动出击告别单身汪加入散狗粮的行列就看你啦，七夕送什么才有心意，程序猿可以试试用一种特别的方式来表达你对女神的心意。有一个创意是把她过往发的微博整理后用词云展示出来。本文教你怎么用Python快速创建出有心意词云，即使是Python小白也能分分钟做出来。

### 准备工作

本环境基于Python3，理论上Python2.7也是可行的，先安装必要的第三方依赖包：

```
# requirement.txt
jieba==0.38
matplotlib==2.0.2
numpy==1.13.1
pyparsing==2.2.0
requests==2.18.4
scipy==0.19.1
wordcloud==1.3.1

```

requirement.txt文件中包含上面的几个依赖包，如果用pip方式安装失败，推荐使用Anaconda安装

```bash
pip install -r requirement.txt
```

### 第一步：分析网址

打开微博移动端网址 https://m.weibo.cn/searchs ，找到女神的微博ID，进入她的微博主页，分析浏览器发送请求的过程

![weibo-home2.png](https://foofish.net/images/weibo-home2.png)

打开 Chrome 浏览器的调试功能，选择 Network 菜单，观察到获取微博数据的的接口是 https://m.weibo.cn/api/container/getIndex ，后面附带了一连串的参数，这里面有些参数是根据用户变化的，有些是固定的，先提取出来。

```
uid=1192515960&
luicode=10000011&
lfid=100103type%3D3%26q%3D%E6%9D%8E%E5%86%B0%E5%86%B0&
featurecode=20000320&
type=user&
containerid=1076031192515960

```

再来分析接口的返回结果，返回数据是一个JSON字典结构，total 是微博总条数，每一条具体的微博内容封装在 cards 数组中，具体内容字段是里面的 text 字段。很多干扰信息已隐去。

```python
{
    "cardlistInfo": {
        "containerid": "1076031192515960",
        "total": 4754,
        "page": 2
    },
    "cards": [
        {
            "card_type": 9,
            "mblog": {
                "created_at": "08-26",
                "idstr": "4145069944506080",
                "text": "瑞士一日游圆满结束...",
            }
        }]
}
```

### 第二步：构建请求头和查询参数

分析完网页后，我们开始用 requests 模拟浏览器构造爬虫获取数据，因为这里获取用户的数据无需登录微博，所以我们不需要构造 cookie信息，只需要基本的请求头即可，具体需要哪些头信息也可以从浏览器中获取，首先构造必须要的请求参数，包括请求头和查询参数。

![weibo-home2-header.png](https://foofish.net/images/weibo-home2-header.png)

```python
headers = {
    "Host": "m.weibo.cn",
    "Referer": "https://m.weibo.cn/u/1705822647",
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) "
                  "Version/9.0 Mobile/13B143 Safari/601.1",
}

params = {"uid": "{uid}",
          "luicode": "20000174",
          "featurecode": "20000320",
          "type": "uid",
          "value": "1705822647",
          "containerid": "{containerid}",
          "page": "{page}"}
```

- uid是微博用户的id
- containerid虽然不什么意思，但也是和具体某个用户相关的参数
- page 分页参数

### 第三步：构造简单爬虫

通过返回的数据能查询到总微博条数 total，爬取数据直接利用 requests 提供的方法把 json 数据转换成 Python 字典对象，从中提取出所有的 text 字段的值并放到 blogs 列表中，提取文本之前进行简单过滤，去掉无用信息。顺便把数据写入文件，方便下次转换时不再重复爬取。

```python
def fetch_data(uid=None, container_id=None):
    """
    抓取数据，并保存到CSV文件中
    :return:
    """
    page = 0
    total = 4754
    blogs = []
    for i in range(0, total // 10):
        params['uid'] = uid
        params['page'] = str(page)
        params['containerid'] = container_id
        res = requests.get(url, params=params, headers=HEADERS)
        cards = res.json().get("cards")

        for card in cards:
            # 每条微博的正文内容
            if card.get("card_type") == 9:
                text = card.get("mblog").get("text")
                text = clean_html(text)
                blogs.append(text)
        page += 1
        print("抓取第{page}页，目前总共抓取了 {count} 条微博".format(page=page, count=len(blogs)))
        with codecs.open('weibo1.txt', 'w', encoding='utf-8') as f:
            f.write("\n".join(blogs))
```

![weibo-home3-spider.gif](https://foofish.net/images/weibo-home3-spider.gif)

### 第四步：分词处理并构建词云

爬虫了所有数据之后，先进行分词，这里用的是结巴分词，按照中文语境将句子进行分词处理，分词过程中过滤掉停止词，处理完之后找一张参照图，然后根据参照图通过词语拼装成图。

```python
def generate_image():
    data = []
    jieba.analyse.set_stop_words("./stopwords.txt")

    with codecs.open("weibo1.txt", 'r', encoding="utf-8") as f:
        for text in f.readlines():
            data.extend(jieba.analyse.extract_tags(text, topK=20))
        data = " ".join(data)
        mask_img = imread('./52f90c9a5131c.jpg', flatten=True)
        wordcloud = WordCloud(
            font_path='msyh.ttc',
            background_color='white',
            mask=mask_img
        ).generate(data)
        plt.imshow(wordcloud.recolor(color_func=grey_color_func, random_state=3),
                   interpolation="bilinear")
        plt.axis('off')
        plt.savefig('./heart2.jpg', dpi=1600)
```

最终效果图：

![weibo-home2-header2.png](https://foofish.net/images/weibo-home2-header2.png)