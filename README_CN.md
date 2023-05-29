# Python Poe API

[![PyPi 版本](https://img.shields.io/pypi/v/poe-api.svg)](https://pypi.org/project/poe-api/)

这是一个反向工程的API包装器,用于Quora的Poe,允许您免费访问OpenAI的ChatGPT和GPT-4,以及Anthropic的Claude。

## 目录:
- [Python Poe API](#python-poe-api)
  - [目录:](#目录)
  - [功能:](#功能)
  - [安装:](#安装)
  - [文档:](#文档)
    - [寻找你的令牌:](#寻找你的令牌)
    - [使用客户端:](#使用客户端)
      - [下载可用的机器人:](#下载可用的机器人)
      - [使用第三方机器人:](#使用第三方机器人)
      - [创建新机器人:](#创建新机器人)
      - [编辑机器人:](#编辑机器人)
      - [发送消息:](#发送消息)
      - [清除对话上下文:](#清除对话上下文)
      - [下载对话历史记录:](#下载对话历史记录)
      - [删除消息:](#删除消息)
      - [清除对话:](#清除对话)
      - [获取剩余消息:](#获取剩余消息)
    - [杂项:](#杂项)
      - [更改日志级别:](#更改日志级别)
      - [设置自定义用户代理:](#设置自定义用户代理)
      - [设置自定义设备ID:](#设置自定义设备id)
  - [版权:](#版权)
    - [版权声明:](#版权声明)

*目录由[markdown-toc](http://ecotrust-canada.github.io/markdown-toc)生成。*

## 功能:
 - 使用令牌登录
 - 代理请求+ websocket
 - 下载机器人列表
 - 发送消息
 - 流机器人响应
 - 清除对话上下文
 - 下载对话历史记录
 - 删除消息
 - 清除整个对话
 - 创建自定义机器人
 - 编辑您的自定义机器人
 - 使用现成的第三方机器人

## 安装:
您可以通过运行以下命令安装此库:
```
pip3 install poe-api
```

## 文档:
示例可以在`/examples`目录中找到。要运行这些示例,请将令牌作为命令行参数传递。
```
python3 examples/temporary_message.py "TOKEN_HERE"
```

### 寻找你的令牌:
在任何Web浏览器中登录[Poe](https://poe.com),然后打开浏览器的开发者工具(也称为“检查”),并查找`p-b` cookie的值在以下菜单中:
 - Chromium: Devtools > Application > Cookies > poe.com
 - Firefox: Devtools > Storage > Cookies
 - Safari: Devtools > Storage > Cookies

### 使用客户端:
要使用这个库,简单地导入`poe`和创建一个`poe.Client`实例。客户端类接受以下参数:
 - `token` - 要使用的令牌。
 - `proxy = None` - 要使用的代理,格式为`protocol://host:port`。推荐使用`socks5h`协议,因为它也代理DNS查询。
 - `device_id = None` - 要使用的设备ID。如果未指定,则将随机生成并存储在磁盘上。
 - `headers = headers` - 要使用的标头。默认为`poe.headers`中指定的标头。
 - `client_identifier = client_identifier` - 将传递到TLS客户端库中的客户端标识符。默认为`poe.client_identifier`中指定的标识符。

常规示例:
```python
import poe
client = poe.Client("TOKEN_HERE")
```

代理示例:
```python
import poe
client = poe.Client("TOKEN_HERE", proxy="socks5h://178.62.100.151:59166")
```

请注意,以下示例假定`client`是`poe.Client`实例的名称。如果令牌无效,将引发RuntimeError。

#### 下载可用的机器人:
客户端在初始化时下载所有可用的机器人,并将它们存储在`client.bots`中。可以在`client.bot_names`中找到将机器人代号映射到其显示名称的字典。如果您想刷新这些值,可以调用`client.get_bots`。此函数接受以下参数:
 - `download_next_data = True` - 是否重新下载`__NEXT_DATA__`,如果机器人列表已更改,这是必需的。

```python
print(json.dumps(client.bot_names, indent=2)) 
"""
{
  "capybara": "Sage",
  "a2": "Claude-instant",
  "nutria": "Dragonfly",
  "a2_100k": "Claude-instant-100k",
  "beaver": "GPT-4",
  "chinchilla": "ChatGPT",
  "a2_2": "Claude+"
}  
"""
```

请注意,对于免费帐户,Claude+(a2_2)每天最多可以发送3条消息,GPT-4(beaver)每天最多可以发送1条消息。 Claude-instant-100k(a2_100k) 对免费帐户完全不可访问。 对于其他所有聊天机器人,似乎每分钟有10条消息的速率限制。

#### 使用第三方机器人:
要获取第三方机器人列表,请使用`client.explore_bots`,它接受以下参数:
 - `end_cursor = None` - 用于获取列表的游标。 
 - `count = 25` - 返回的机器人数。

该函数将返回包含机器人列表和下一页的游标的字典:
```python
print(json.dumps(client.explore_bots(count=1), indent=2))
"""
{
  "bots": [
    {
      "id": "Qm90OjEwMzI2MDI=",
      "displayName": "leocooks",
      "deletionState": "not_deleted",
      "image": {
        "__typename": "UrlBotImage",
        "url": "https://qph.cf2.quoracdn.net/main-thumb-pb-1032602-200-uorvomwowfgmatdvrtwajtwwqlujmmgu.jpeg"
      },
      "botId": 1032602,
      "followerCount": 1922,
      "description": "Above average meals for average cooks, made simple by world-renowned chef, Leonardo",
      "__typename": "Bot"
    }
  ],
  "end_cursor": "1000172"
}
"""
```

由于显示名称与自定义机器人的代号相同,您可以简单地将机器人的显示名称传递给`client.send_message`以向其发送消息。

#### 创建新机器人:
您可以使用`client.create_bot`函数创建一个新机器人,它接受以下参数:
 - `handle` - 新机器人的句柄。
 - `prompt = ""` - 新机器人的提示。
 - `base_model = "chinchilla"` - 新机器人使用的模型。这必须是`"chinchilla"`(ChatGPT)或`"a2"`(Claude)。
 - `description = ""` - 新机器人的描述。
 - `intro_message = ""` - 新机器人的介绍消息。如果这是一个空字符串,那么机器人不会有入门消息。
 - `prompt_public = True` - 提示是否应该公开可见。 
 - `pfp_url = None` - 机器人的个人资料图片网址。目前,使用此库实际上无法上传自定义图像。
 - `linkification = False` - 机器人是否应该将响应中的某些文本变成可点击的链接。
 - `markdown_rendering = True` - 是否为机器人的响应启用Markdown渲染。
 - `suggested_replies = False` - 机器人是否应该在每次响应后建议可能的回复。
 - `private = False` - 机器人是否应该是私有的。

如果您希望新机器人使用自己的API(如[这里](https://github.com/poe-platform/api-bot-tutorial)详细说明),请使用这些参数:
 - `api_key = None` - 新机器人的API密钥。
 - `api_bot = False` - 机器人是否启用了API功能。
 - `api_url = None` - 新机器人的API URL。

如何创建和编辑机器人的完整示例位于`examples/create_bot.py`。
```python
new_bot = client.create_bot(bot_name, "prompt goes here", base_model="a2")
```

#### 编辑机器人:
您可以使用`client.edit_bot`函数编辑自定义机器人,它接受以下参数:
 - `bot_id` - 要编辑的机器人的`botId`。
 - `handle` - 要编辑的机器人的句柄。
 - `prompt` - 新机器人的提示。
 - `base_model = "chinchilla"` - 机器人现在使用的新模型。这必须是`"chinchilla"`(ChatGPT)或`"a2"`(Claude)。以前,可以将此设置为`"beaver"`(GPT-4),这会绕过免费帐户的限制,但现在已修补。
 - `description = ""` - 机器人的新描述。
 - `intro_message = ""` - 机器人的新简介消息。如果这是一个空字符串,那么机器人不会有入门消息。
 - `prompt_public = True` - 提示是否应该公开可见。  
 - `pfp_url = None` - 机器人的个人资料图片网址。目前,使用此库实际上无法上传自定义图像。
 - `linkification = False` - 机器人是否应该将响应中的某些文本变成可点击的链接。
 - `markdown_rendering = True` - 是否为机器人的响应启用Markdown渲染。
 - `suggested_replies = False` - 机器人是否应该在每次响应后建议可能的回复。 
 - `private = False` - 机器人是否应该是私有的。

机器人API相关参数:
 - `api_key = None` - 机器人的新API密钥。 
 - `api_url = None` - 机器人的新API URL。

如何创建和编辑机器人的完整示例位于`examples/create_bot.py`。
```python
edit_result = client.edit_bot(1086981, "bot_handle_here", base_model="beaver")
```

#### 发送消息:
您可以使用`client.send_message`函数向聊天机器人发送消息,它接受以下参数:
 - `chatbot` - 聊天机器人的代号。(例如:`capybara`)
 - `message` - 要发送给聊天机器人的消息。
 - `with_chat_break = False` - 会话上下文是否应该清除。   
 - `timeout = 20` - 在接收的块之间的最大秒数内未收到错误之前引发`RuntimeError`。

该函数是一个生成器,每当更新时就返回最新版本的生成消息。

流式示例:
```python
message = "概括GNU GPL v3"
for chunk in client.send_message("capybara", message):
  print(chunk["text_new"], end="", flush=True)
```

非流示例:
```python 
message = "概括GNU GPL v3"
for chunk in client.send_message("capybara", message):
  pass
print(chunk["text"])
```

您还可以使用`threading`并行发送多条消息,并单独接收它们的响应,如`/examples/parallel_messages.py`中所示。请注意,如果您发送消息太快,服务器会给出错误,但请求最终会成功。

#### 清除对话上下文:
如果您想在不发送消息的情况下清除对话的上下文,可以使用`client.send_chat_break`。唯一的参数是要清除上下文的机器人的代号。

```python
client.send_chat_break("capybara")
```
该函数返回代表聊天休息的消息。

#### 下载对话历史记录: 
要下载对话中的过去消息,请使用`client.get_message_history`函数,它接受以下参数:
 - `chatbot` - 聊天机器人的代号。
 - `count = 25` - 要下载的消息数。 
 - `cursor = None` - 从最新消息开始的消息ID。
 
请注意,如果您不指定游标,客户端将不得不执行额外的请求来确定最新的游标是什么。

返回的消息从最旧到最新排序。

```python
message_history = client.get_message_history("capybara", count=10)
print(json.dumps(message_history, indent=2))
"""
[
  {
    "node": {
      "id": "TWVzc2FnZToxMDEwNzYyODU=",
      "messageId": 101076285,
      "creationTime": 1679298157718888,
      "text": "",
      "author": "chat_break",
      "linkifiedText": "",
      "state": "complete",
      "suggestedReplies": [],
      "vote": null,
      "voteReason": null,
      "__typename": "Message"
    },
    "cursor": "101076285",
    "id": "TWVzc2FnZUVkZ2U6MTAxMDc2Mjg1OjEwMTA3NjI4NQ=="
  },
  ... 
]
"""
```

#### 删除消息:
要删除消息,请使用`client.delete_message`函数,它接受单个参数。您可以将单个消息ID传递给它以删除单个消息,或者可以传递一组消息ID以一次删除多个消息。

```python
#删除单条消息
client.delete_message(96105719)

#一次删除多条消息
client.delete_message([96105719, 96097108, 96097078, 96084421, 96084402])
```

#### 清除对话:
要清除整个对话或仅清除最后几条消息,可以使用`client.purge_conversation`函数。此函数接受以下参数:
 - `chatbot` - 聊天机器人的代号。 
 - `count = -1` - 要删除的消息数,从最新消息开始。默认行为是删除每条消息。

```python
#仅清除最后10条消息
client.purge_conversation("capybara", count=10)  

#清除整个对话
client.purge_conversation("capybara")
```

#### 获取剩余消息:
要获取对话配额中剩余的消息数,请使用`client.get_remaining_messages`函数。此函数接受以下参数:
 - `chatbot` - 聊天机器人的代号。

该函数将返回剩余消息数,或者如果机器人没有配额,则返回`None`。

### 杂项:
#### 更改日志级别: 
默认情况下,`poe-api`使用`logging.INFO`级别。要更改此级别,请使用`poe.logging.setLevel`,如下所示:
```python
import poe

poe.logging.setLevel(logging.DEBUG)
```

#### 设置自定义用户代理:
默认情况下,`poe-api`将使用`openai-client/0.1.0`作为用户代理。如果您想使用自己的用户代理,请在创建客户端之前将`poe.USER_AGENT`设置为所需的值:
```python
import poe

poe.USER_AGENT = "my-custom-user-agent/1.0.0"
client = poe.Client("TOKEN") 
```

#### 设置自定义设备ID:
默认情况下,`poe-api`将随机生成并使用设备ID。如果您希望使用特定的设备ID,可以在创建客户端时将`device_id`参数设置为所需的值:
```python 
client = poe.Client("TOKEN", device_id="my-device-id")
```

## 版权:
### 版权声明: 
该库由Anthropic,PBC开发和维护。 
版权所有© 2020-2021 Anthropic,PBC 。保留所有权利。

该软件由Anthropic根据MIT许可提供。 您可以在许可文件中找到完整的许可证文本。

此外,Anthropic特此声明:此软件和相关材料仅供研究、评估和内部商业目的使用。 在使用或分发此软件之前,您有责任确保遵守所有适用的法律法规。
