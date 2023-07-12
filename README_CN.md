# Python Poe API

[![PyPi Version ↗](https://img.shields.io/pypi/v/poe-api.svg)]([https://pypi.org/project/poe-api/) ↗](https://pypi.org/project/poe-api/))

这是一个为 Quora 的 Poe 反向工程的 API 包装器，它允许您免费访问 OpenAI 的 ChatGPT、GPT-4，以及 Anthropic 的 Claude。

## 目录:
- [功能](#功能)
- [安装](#安装)
- [文档](#文档)
  * [查找您的令牌](#查找您的令牌)
  * [使用客户端](#使用客户端)
    + [下载可用的机器人](#下载可用的机器人)
    + [使用第三方机器人](#使用第三方机器人)
    + [创建新机器人](#创建新机器人)
    + [编辑机器人](#编辑机器人)
    + [发送消息](#发送消息)
    + [清除对话上下文](#清除对话上下文)
    + [下载对话历史](#下载对话历史)
    + [删除消息](#删除消息)
    + [清除对话](#清除对话)
    + [清除所有对话](#清除所有对话)
    + [获取剩余消息数](#获取剩余消息数)
  * [其他](#其他)
    + [更改日志级别](#更改日志级别)
    + [设置自定义用户代理](#设置自定义用户代理)
    + [设置自定义设备ID](#设置自定义设备id)
- [版权](#版权)
  * [版权声明](#版权声明)

*目录由 [markdown-toc ↗](http://ecotrust-canada.github.io/markdown-toc) 生成。*

## 功能:
- 使用令牌登录
- 代理请求 + WebSocket
- 下载机器人列表
- 发送消息
- 流式获取机器人响应
- 清除对话上下文
- 下载对话历史
- 删除消息
- 清除整个对话
- 创建自定义机器人
- 编辑自定义机器人
- 使用现有的第三方机器人

## 安装:
您可以通过以下命令安装此库:
```
pip3 install poe-api
```

## 文档:
示例可以在 `/examples` 目录中找到。要运行这些示例，请将您的令牌作为命令行参数传递。
```
python3 examples/temporary_message.py "TOKEN_HERE"
```

### 查找您的令牌:
在任何桌面 Web 浏览器上登录 [Poe ↗](https://poe.com)，然后打开您的浏览器的开发者工具（也称为 "检查"），在以下菜单中查找 `p-b` 的 cookie 值:
 - Chromium: Devtools > Application > Cookies > poe.com
 - Firefox: Devtools > Storage > Cookies
 - Safari: Devtools > Storage > Cookies

### 使用客户端:
要使用此库，只需导入 `poe` 并创建一个 `poe.Client` 实例。Client 类接受以下参数:
 - `token` - 要使用的令牌。
 - `proxy = None` - 要使用的代理，格式为 `protocol://host:port`。推荐使用 `socks5h` 协议，因为它还代理 DNS 查询。
 - `device_id = None` - 要使用的设备 ID。如果未指定，将随机生成并存储在磁盘上。
 - `headers = headers` - 要使用的标头。默认为 `poe.headers` 中指定的标头。
 - `client_identifier = client_identifier` - 将传递给 TLS 客户端库的客户端标识符。默认为 `poe.client_identifier` 中指定的标识符。

普通示例:
```python
import poe
client = poe.Client("TOKEN_HERE")
```

代理示例:
```python
import poe
client = poe.Client("TOKEN_HERE", proxy="socks5h://178.62.100.151:59166")
```

请注意，以下示例假设 `client` 是您的 `poe.Client` 实例的名称。如果令牌无效，将引发 RuntimeError。

#### 下载可用的机器人:
客户端在初始化时下载所有可用的机器人，并将它们存储在 `client.bots` 中。将机器人代码名称映射到其显示名称的字典可以在 `client.bot_names` 中找到。如果您想刷新这些值，可以调用 `client.get_bots`。此函数接受以下参数:
 - `download_next_data = True` - 是否重新下载 `__NEXT_DATA__`，如果机器人列表已更改，则需要此参数。

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

请注意，在免费账户上，Claude+ (a2_2) 每天限制为 3 条消息，GPT-4 (beaver) 每天限制为 1 条消息。对于其他聊天机器人，似乎每分钟有 10 条消息的限制。

#### 使用第三方机器人:
要获取第三方机器人列表，使用 `client.explore_bots`，它接受以下参数:
 - `end_cursor = None` - 在获取列表时使用的游标。
 - `count = 25` - 返回的机器人数量。

该函数将返回一个包含机器人列表和下一页的游标的字典:
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

要获取特定的第三方机器人，可以使用 `client.get_bot_by_codename`，它接受机器人的代码名称作为唯一参数。
```python
client.get_bot_by_codename("JapaneseTutor")
```

由于自定义机器人的显示名称与代码名称相同，因此可以直接将机器人的显示名称传递给 `client.send_message` 来发送消息。

#### 创建新机器人:
可以使用 `client.create_bot` 函数创建新的机器人，它接受以下参数:
 - `handle` - 新机器人的句柄。
 - `prompt = ""` - 新机器人的提示。
 - `base_model = "chinchilla"` - 新机器人使用的模型。必须是 `"chinchilla"` (ChatGPT) 或 `"a2"` (Claude)。
 - `description = ""` - 新机器人的描述。
 - `intro_message = ""` - 新机器人的介绍消息。如果这是一个空字符串，则机器人将没有介绍消息。
 - `prompt_public = True` - 提示是否应该对公众可见。
 - `pfp_url = None` - 机器人的个人资料图片的 URL。目前，使用此库无法上传自定义图像。
 - `linkification = False` - 机器人是否应将响应中的一些文本转换为可点击的链接。
 - `markdown_rendering = True` - 是否启用机器人响应的 Markdown 渲染。
 - `suggested_replies = False` - 机器人在每个响应后是否应该建议可能的回复。
 - `private = False` - 机器人是否应为私有的。

如果要使用自己的 API 创建新机器人（详见[此处 ↗](https://github.com/poe-platform/api-bot-tutorial)），请使用以下参数:
 - `api_key = None` - 新机器人的 API 密钥。
 - `api_bot = False` - 机器人是否启用 API 功能。
 - `api_url = None` - 新机器人的 API URL。

有关创建和编辑机器人的完整示例位于 `examples/create_bot.py`。

```python
new_bot = client.create_bot(bot_name, "prompt goes here", base_model="a2")
```

#### 编辑机器人:
可以使用 `client.edit_bot` 函数编辑自定义机器人，它接受以下参数:
 - `bot_id` - 要编辑的机器人的 `botId`。
 - `handle` - 要编辑的机器人的句柄。
 - `prompt` - 新机器人的提示。
 - `base_model = "chinchilla"` - 机器人使用的新模型。必须是 `"chinchilla"` (ChatGPT) 或 `"a2"` (Claude)。此前，可以将其设置为 `"beaver"` (GPT-4)，以绕过免费账户的限制，但现在已修复。
 - `description = ""` - 机器人的新描述。
 - `intro_message = ""` - 机器人的新介绍消息。如果这是一个空字符串，则机器人将没有介绍消息。
 - `prompt_public = True` - 提示是否应该对公众可见。
 - `pfp_url = None` - 机器人的个人资料图片的 URL。目前，使用此库无法上传自定义图像。
 - `linkification = False` - 机器人是否应将响应中的一些文本转换为可点击的链接。
 - `markdown_rendering = True` - 是否启用机器人响应的 Markdown 渲染。
 - `suggested_replies = False` - 机器人在每个响应后是否应该建议可能的回复。
 - `private = False` - 机器人是否应为私有的。

与机器人 API 相关的参数:
 - `api_key = None` - 机器人的新 API 密钥。
 - `api_url = None` - 机器人的新 API URL。

有关创建和编辑机器人的完整示例位于 `examples/create_bot.py`。

```python
edit_result = client.edit_bot(1086981, "bot_handle_here", base_model="a2")
```

#### 发送消息:
您可以使用 `client.send_message` 函数向聊天机器人发送消息，它接受以下参数:
 - `chatbot` - 聊天机器人的代码名称（例如：`capybara`）。
 - `message` - 要发送给聊天机器人的消息。
 - `with_chat_break = False` - 是否清除对话上下文。
 - `timeout = 20` - 接收到消息块之间的最大时间间隔（以秒为单位），超过此时间将引发 `RuntimeError`。
 - `async_recv = True` - 是否将 `receive_POST` 请求设为异步。如果禁用此选项，则在消息完成后还需等待约 3 秒。
 - `suggest_callback = None` - 建议回复的回调函数。有关如何使用此选项的示例，请参见 `examples/send_message.py`。

该函数是一个生成器，每当生成的消息更新时，就会返回最新版本的消息。

流式示例:
```python
message = "Summarize the GNU GPL v3"
for chunk in client.send_message("capybara", message):
  print(chunk["text_new"], end="", flush=True)
```

非流式示例:
```python
message = "Summarize the GNU GPL v3"
for chunk in client.send_message("capybara", message):
  pass
print(chunk["text"])
```

您还可以使用 `threading` 并同时发送多条消息并接收它们的响应，如 `/examples/parallel_messages.py` 中所示。请注意，如果发送消息过快，服务器将返回错误，但请求最终会成功。

#### 清除对话上下文:
如果您想清除对话的上下文而不发送消息，可以使用 `client.send_chat_break`。它只接受一个参数，即要清除上下文的机器人的代码名称。

```python
client.send_chat_break("capybara")
```
该函数返回表示聊天中断的消息。

#### 下载对话历史:
要下载对话中的过去消息，使用 `client.get_message_history` 函数，它接受以下参数:
 - `chatbot` - 聊天机器人的代码名称。
 - `count = 25` - 要下载的消息数量。
 - `cursor = None` - 要从其开始的消息ID，而不是最新的消息。

请注意，如果不指定游标，客户端将执行额外的请求以确定最新的游标。

返回的消息按从旧到新的顺序排列。

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
要删除消息，使用 `client.delete_message` 函数，它接受一个参数。您可以将单个消息ID传递给它以删除单个消息，或者可以传递消息ID列表以一次删除多个消息。

```python
# 删除单个消息
client.delete_message(96105719)

# 一次删除多个消息
client.delete_message([96105719, 96097108, 96097078, 96084421, 96084402])
```

#### 清除对话:
要清除整个对话或仅清除最后几条消息，可以使用 `client.purge_conversation` 函数。该函数接受以下参数:
 - `chatbot` - 聊天机器人的代码名称。
 - `count = -1` - 要删除的消息数量，从最新的消息开始计算。默认行为是删除所有消息。

```python
# 只清除最后的 10 条消息
client.purge_conversation("capybara", count=10)

# 清除整个对话
client.purge_conversation("capybara")
```

#### 清除所有对话:
要清除账户中的所有对话，使用 `client.purge_all_conversations` 函数。此函数不需要任何参数。

```python
>>> client.purge_all_conversations()
```

#### 获取剩余消息数:
要获取对话配额中剩余的消息数量，请使用 `client.get_remaining_messages` 函数。此函数接受以下参数:
 - `chatbot` - 聊天机器人的代码名称。

该函数将返回剩余的消息数量，如果机器人没有配额，则返回 `None`。

```python
>>> client.get_remaining_messages("beaver")
1
```

### 其他:
#### 更改日志级别:
如果要显示调试消息，请调用 `poe.logger.setLevel`。

```python
import poe
import logging
poe.logger.setLevel(logging.INFO)
```

#### 设置自定义用户代理:
如果要更改伪装的标头，请在导入库后设置 `poe.headers`。

要使用浏览器自己的标头，请访问[此网站 ↗](https://headers.uniqueostrich18.repl.co/)，并复制粘贴其内容。
```python
import poe
poe.headers = {
  "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0",
  "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
  "Accept-Encoding": "gzip, deflate, br",
  "Accept-Language": "en-US,en;q=0.5",
  "Te": "trailers",
  "Upgrade-Insecure-Requests": "1"
}
```

以下标头将被忽略和覆盖:
```python
{
  "Referrer": "https://poe.com/",
  "Origin": "https://poe.com",
  "Host": "poe.com",
  "Sec-Fetch-Dest": "empty",
  "Sec-Fetch-Mode": "cors",
  "Sec-Fetch-Site": "same-origin"
}
```

以前，这是通过 `poe.user_agent` 设置的，但现在完全忽略该变量。

您还需要将 `poe.client_identifier` 设置为与您设置的用户代理匹配。有关一些示例值，请参见[Python-TLS-Client 文档 ↗](https://github.com/FlorianREGAZ/Python-Tls-Client#examples)。请注意，伪装 Chrome/Firefox 版本 >= 110 可能会被检测到。

```python
poe.client_identifier = "chrome_107"
```

### 设置自定义设备ID:
如果要更改伪装的设备ID，可以使用 `poe.set_device_id`，它接受以下参数:
 - `user_id` - 要更改设备ID的账户的用户ID。用户ID可以在 `client.viewer["poeUser"]["id"]` 中找到。
 - `device_id` - 新的设备ID。这是一个 32 个字符的 UUID 字符串，格式如下: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

```python
poe.set_device_id("UGMlVXqlcLYyMOATMDsKNTMz", "6d659b04-043a-41f8-97c7-fb7d7fe9ad34")
```

设备ID保存在 Unix-like 系统上的 `~/.config/poe-api/device_id.json`，在 Windows 上的 `C:\Users\<user>\AppData\Roaming\poe-api\device_id.json`。

另外，可以使用 `poe.get_device_id` 函数或 `client.device_id` 来获取保存的设备ID。
```python
>>> poe.get_device_id("UGMlVXqlcLYyMOATMDsKNTMz")
#6d659b04-043a-41f8-97c7-fb7d7fe9ad34

>>> client.device_id
#6d659b04-043a-41f8-97c7-fb7d7fe9ad34
```

## 版权:
本程序根据 [GNU GPL v3 ↗](https://www.gnu.org/licenses/gpl-3.0.txt) 授权许可。除了 GraphQL 查询之外，大部分代码都是由 [ading2210 ↗](https://github.com/ading2210) 编写的。

Quora 的 `poe-tag-id` 标头的反向工程由 [xtekky ↗](https://github.com/xtekky) 在 [PR #39 ↗](https://github.com/ading2210/poe-api/pull/39) 中完成。

`client.get_remaining_messages` 函数由 [Snowad14 ↗](https://github.com/Snowad14) 在 [PR #46 ↗](https://github.com/ading2210/poe-api/pull/46) 中编写。

规避检测和获取第三方机器人由 [acheong08 ↗](https://github.com/acheong08/) 在 [PR #79 ↗](https://github.com/ading2210/poe-api/pull/79) 中完成。

大部分 GraphQL 查询取自 [muharamdani/poe ↗](https://github.com/muharamdani/poe)，该项目根据 ISC 许可证授权。
