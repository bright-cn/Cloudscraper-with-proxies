# 使用 CloudScraper 与代理

[![推广图](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/) 

本指南介绍如何在 CloudScraper 中进行代理集成、IP 轮换以及使用需要身份验证的代理，以实现无缝抓取。

## 目录
- [什么是 CloudScraper？](#什么是-cloudscraper)
- [为什么在 CloudScraper 中使用代理？](#为什么在-cloudscraper-中使用代理)
- [在 CloudScraper 中配置代理](#在-cloudscraper-中配置代理)
- [实现代理轮换](#实现代理轮换)
- [在 CloudScraper 中使用需要身份验证的代理](#在-cloudscraper-中使用需要身份验证的代理)
- [在 CloudScraper 中集成高级代理](#在-cloudscraper-中集成高级代理)
- [结论](#结论)

---

## 什么是 CloudScraper？

[CloudScraper](https://github.com/VeNoMouS/cloudscraper) 是一个 Python 模块，旨在绕过 Cloudflare 的反机器人页面（通常称为 “I’m Under Attack Mode” 或 IUAM）。它在内部基于 Requests（Python 最常用的 HTTP 客户端之一）实现。

## 为什么在 CloudScraper 中使用代理？

当你请求次数过多或触发更复杂的防御机制时，Cloudflare 可能会阻止你的 IP。将代理与 CloudScraper 结合，用于抓取由 Cloudflare 托管的网站，有以下两大好处：

- **提升安全性和匿名性**：通过使用代理来发送请求，可以隐藏真实身份并降低被检测的风险。
- **避免被封锁和中断**：代理可动态更换 IP 地址，从而帮助绕过封锁和速率限制。

## 在 CloudScraper 中配置代理

### 第 1 步：安装 CloudScraper

使用 pip 安装 `cloudscraper`：

```bash
pip install -U cloudscraper
```

其中 `-U` 选项可确保获取最新版本，对应 Cloudflare 反机器人机制的最新解决方案。

### 第 2 步：初始化 CloudScraper

导入 CloudScraper：

```python
import cloudscraper
```

使用 `create_scraper()` 方法创建一个 CloudScraper 实例：

```python
scraper = cloudscraper.create_scraper()
```

这里得到的 `scraper` 对象与 `requests` 库中的 `Session` 类似，可以在绕过 Cloudflare 反机器人措施的同时灵活发起 HTTP 请求。

### 第 3 步：集成代理

定义一个 `proxies` 字典，并将其传递给 `get()` 方法：

```python
proxies = {
    "http": "<YOUR_HTTP_PROXY_URL>",
    "https": "<YOUR_HTTPS_PROXY_URL>"
}

# 通过指定的代理发送请求
response = scraper.get("<YOUR_TARGET_URL>", proxies=proxies)
```

`get()` 方法中的 `proxies` 参数会传递给 Requests，使得你的请求可根据目标 URL 的协议，通过相应的 HTTP 或 HTTPS 代理服务器进行转发。

### 第 4 步：测试 CloudScraper 代理集成设置

为演示方便，可以使用 HTTPBin 项目的 `/ip` 接口来检查调用方的 IP 地址。如果一切正常，会返回代理服务器的 IP。

假设代理服务器的 URL 为 `http://202.159.35.121:443`，可以编写以下脚本：

```python
import cloudscraper

# 创建 CloudScraper 实例
scraper = cloudscraper.create_scraper()

# 指定代理
proxies = {
    "http": "http://202.159.35.121:443",
    "https": "http://202.159.35.121:443"
}

# 通过代理发送请求
response = scraper.get("https://httpbin.org/ip", proxies=proxies)

# 打印 "/ip" 接口的返回结果
print(response.text)
```

你会看到类似如下的输出：

```json
{
    "origin": "202.159.35.121"
}
```

其中显示的 IP 与代理服务器的 IP 相匹配，表明代理功能已正常工作。

> **注意**:  
> 免费代理往往不够稳定或可用时间较短。测试脚本时可以定期更换新的代理 IP。

## 实现代理轮换

从可靠的代理提供商处获取代理列表，并存储在一个数组中：

```python
proxy_list = [
    {"http": "<YOUR_PROXY_URL_1>", "https": "<YOUR_PROXY_URL_1>"},
    # ...
    {"http": "<YOUR_PROXY_URL_n>", "https": "<YOUR_PROXY_URL_n>"},
]
```

然后，使用 `random.choice()` 从列表中随机选择一个代理：

```python
import random

random_proxy = random.choice(proxy_list)
```

在 `get()` 请求中使用随机选择的代理：

```python
response = scraper.get("<YOUR_TARGET_URL>", proxies=random_proxy)
```

如果配置正确，每次运行时都会使用代理列表中的不同代理。示例如下：

```python
import cloudscraper
import random

# 创建 CloudScraper 实例
scraper = cloudscraper.create_scraper()

# 代理列表（将其替换为实际的代理 URL）
proxy_list = [
    {"http": "<YOUR_PROXY_URL_1>", "https": "<YOUR_PROXY_URL_1>"},
    # ...
    {"http": "<YOUR_PROXY_URL_n>", "https": "<YOUR_PROXY_URL_n>"},
]

# 随机从列表中选择一个代理
random_proxy = random.choice(proxy_list)

# 通过随机选择的代理发起请求
# (请用实际的目标 URL 替换 <YOUR_TARGET_URL>)
response = scraper.get("<YOUR_TARGET_URL>", proxies=random_proxy)
```

## 在 CloudScraper 中使用需要身份验证的代理

如果你的代理需要用户名密码认证，你可以在代理 URL 中直接包含认证信息。具体格式如下：

<PROXY_PROTOCOL>://<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_IP_ADDRESS>:<PROXY_PORT>

配置方法示例：

```python
import cloudscraper

# 创建 CloudScraper 实例
scraper = cloudscraper.create_scraper()

# 定义需要身份验证的代理
proxies = {
   "http": "<PROXY_PROTOCOL>://<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_IP_ADDRESS>:<PROXY_PORT>",
   "https": "<PROXY_PROTOCOL>://<YOUR_USERNAME>:<YOUR_PASSWORD>@<PROXY_IP_ADDRESS>:<PROXY_PORT>"
}

# 通过指定的身份验证代理发送请求
response = scraper.get("<YOUR_TARGET_URL>", proxies=proxies)
```

## 在 CloudScraper 中集成高级代理

在生产环境，如果你想要稳定可靠的抓取结果，可以使用类似 [Bright Data](https://www.bright.cn/) 这样的一线代理服务。以下是将 Bright Data 代理集成到 CloudScraper 的示例流程：

1. 注册或登录 Bright Data 账户。

2. 在仪表盘中，点击 “Residential” 区域：

![Bright Data 的代理和爬虫基础设施控制面板](https://github.com/luminati-io/Cloudscraper-with-proxies/blob/main/image-7.png)

3. 通过点击开关来激活代理：

![激活住宅代理区域](https://github.com/luminati-io/Cloudscraper-with-proxies/blob/main/image-8.png)

激活后你会看到类似这样：

![住宅代理区域已开启](https://github.com/luminati-io/Cloudscraper-with-proxies/blob/main/image-9.png)
> **注意**:  
> Bright Data 的住宅代理会自动进行 IP 轮换。

4. 在 “Access Details（访问详情）” 中，复制你的代理主机名、用户名以及密码：

![住宅代理区域的访问详情](https://github.com/luminati-io/Cloudscraper-with-proxies/blob/main/image-10.png)

你的 Bright Data 代理地址一般类似如下格式：
```
http://<PROXY_USERNAME>:<PROXY_PASSWORD>@brd.superproxy.io:33335
```

5. 在 CloudScraper 中配置该代理：

```python
import cloudscraper

# 创建 CloudScraper 实例
scraper = cloudscraper.create_scraper()

# 定义高级代理
proxies = {
    "http": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST>:<PROXY_PORT>",
    "https": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST>:<PROXY_PORT>"
}

# 使用高级代理发送请求
response = scraper.get("https://httpbin.org/ip", proxies=proxies)

# 打印响应结果，验证代理是否正常
print(response.text)
```

完成对 CloudScraper 的代理集成后，即可进行测试与验证。可使用 [https://httpbin.org/ip](https://httpbin.org/ip) 等服务来确认返回的 IP 是否为代理服务器 IP，而不是本地 IP。

## 把全部内容组合在一起

以下是一个综合示例：

```python
import cloudscraper
import random
import time

# 第 1 步：定义一个包含不同类型代理（含认证及无认证）的列表
# 请将 <PROXY_USERNAME>、<PROXY_PASSWORD>、<PROXY_HOST> 和 <PROXY_PORT> 替换为实际的值
proxy_list = [
    {"http": "http://<PROXY_HOST_1>:<PROXY_PORT_1>", "https": "http://<PROXY_HOST_1>:<PROXY_PORT_1>"},
    {"http": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>", 
     "https": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>"},
    {"http": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>", 
     "https": "http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>"}
]

# 第 2 步：创建 CloudScraper 实例
scraper = cloudscraper.create_scraper()

# 第 3 步：定义目标 URL
target_url = "https://httpbin.org/ip"  # 该接口会返回请求方的 IP 地址

# 第 4 步：实现代理轮换并发送请求
def fetch_with_proxy_rotation(proxy_list, target_url, num_requests=5):
    """
    使用代理轮换去获取目标 URL。
    
    参数:
        proxy_list (list): 代理配置列表。
        target_url (str): 要抓取的 URL。
        num_requests (int): 要发送的请求次数。
    """
    for i in range(num_requests):
        # 随机从列表中选取一个代理
        proxy = random.choice(proxy_list)
        
        try:
            # 使用选定的代理发送请求
            print(f"使用代理: {proxy}")
            response = scraper.get(target_url, proxies=proxy, timeout=10)
            
            # 打印返回结果（即代理的 IP）
            print(f"第 {i + 1} 次响应: {response.text}")
        
        except Exception as e:
            # 处理各种异常（如连接超时、代理不可用等）
            print(f"代理 {proxy} 出现错误: {e}")
        
        # 等待一段时间再进行下一次请求，模拟类似于人类的访问间隔
        time.sleep(random.uniform(1, 3))

# 第 5 步：运行函数
fetch_with_proxy_rotation(proxy_list, target_url, num_requests=5)
```

### 输出示例

```python
使用代理: {'http': 'http://<PROXY_HOST_1>:<PROXY_PORT_1>', 'https': 'http://<PROXY_HOST_1>:<PROXY_PORT_1>'}
第 1 次响应: {
    "origin": "203.0.113.1"
}
使用代理: {'http': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>', 'https': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_2>:<PROXY_PORT_2>'}
第 2 次响应: {
    "origin": "198.51.100.2"
}
使用代理: {'http': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>', 'https': 'http://<PROXY_USERNAME>:<PROXY_PASSWORD>@<PROXY_HOST_3>:<PROXY_PORT_3>'}
第 3 次响应: {
    "origin": "192.0.2.3"
}
...
```

## 结论

Bright Data 拥有全球顶尖的代理服务器，为世界 500 强企业和超过 20,000 名客户提供服务。其全球代理网络包含：

- [数据中心代理](https://www.bright.cn/proxy-types/datacenter-proxies) – 超过 770,000 个数据中心 IP。  
- [住宅代理](https://www.bright.cn/proxy-types/residential-proxies) – 在 195 个以上国家拥有超过 7,200 万条住宅 IP。  
- [ISP 代理](https://www.bright.cn/proxy-types/isp-proxies) – 超过 700,000 个 ISP IP。  
- [移动代理](https://www.bright.cn/proxy-types/mobile-proxies) – 超过 700 万条移动 IP。

[注册一个免费的 Bright Data 账户](https://www.bright.cn) 试用我们的代理服务器吧。
