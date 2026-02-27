# API参考

# 消息发送API

消息发送API模块专门负责发送各种类型的消息，支持文本、表情包、图片等多种消息类型。

## 导入方式



```
from src.plugin_system.apis import send_api
# 或者
from src.plugin_system import send_api
```

## 主要功能

### 1. 发送文本消息



```
async def text_to_stream(
    text: str,
    stream_id: str,
    typing: bool = False,
    set_reply: bool = False,
    reply_message: Optional["DatabaseMessages"] = None,
    storage_message: bool = True,
    selected_expressions: Optional[List[int]] = None,
) -> bool:
```

发送文本消息到指定的流

**Args:**

- `text` (str): 要发送的文本内容
- `stream_id` (str): 聊天流ID
- `typing` (bool): 是否显示正在输入
- `set_reply` (bool): 是否设置为引用回复
- `reply_message` (Optional["DatabaseMessages"]): 要回复的消息对象（当 `set_reply=True` 时必需）
- `storage_message` (bool): 是否存储消息到数据库
- `selected_expressions` (Optional[List[int]]): 选中的表情包ID列表

**Returns:**

- `bool` - 是否发送成功

### 2. 发送表情包



```
async def emoji_to_stream(
    emoji_base64: str,
    stream_id: str,
    storage_message: bool = True,
    set_reply: bool = False,
    reply_message: Optional["DatabaseMessages"] = None,
) -> bool:
```

向指定流发送表情包。

**Args:**

- `emoji_base64` (str): 表情包的base64编码
- `stream_id` (str): 聊天流ID
- `storage_message` (bool): 是否存储消息到数据库
- `set_reply` (bool): 是否设置为引用回复
- `reply_message` (Optional["DatabaseMessages"]): 要回复的消息对象（当 `set_reply=True` 时必需）

**Returns:**

- `bool` - 是否发送成功

### 3. 发送图片



```
async def image_to_stream(
    image_base64: str,
    stream_id: str,
    storage_message: bool = True,
    set_reply: bool = False,
    reply_message: Optional["DatabaseMessages"] = None,
) -> bool:
```

向指定流发送图片。

**Args:**

- `image_base64` (str): 图片的base64编码
- `stream_id` (str): 聊天流ID
- `storage_message` (bool): 是否存储消息到数据库
- `set_reply` (bool): 是否设置为引用回复
- `reply_message` (Optional["DatabaseMessages"]): 要回复的消息对象（当 `set_reply=True` 时必需）

**Returns:**

- `bool` - 是否发送成功

### 4. 发送命令



```
async def command_to_stream(command: Union[str, dict], stream_id: str, storage_message: bool = True, display_message: str = "") -> bool:
```

向指定流发送命令。

**Args:**

- `command` (Union[str, dict]): 命令内容
- `stream_id` (str): 聊天流ID
- `storage_message` (bool): 是否存储消息到数据库
- `display_message` (str): 显示消息

**Returns:**

- `bool` - 是否发送成功

### 5. 发送自定义类型消息



```
async def custom_to_stream(
    message_type: str,
    content: str | Dict,
    stream_id: str,
    display_message: str = "",
    typing: bool = False,
    reply_message: Optional["DatabaseMessages"] = None,
    set_reply: bool = False,
    storage_message: bool = True,
    show_log: bool = True,
) -> bool:
```

向指定流发送自定义类型消息。

**Args:**

- `message_type` (str): 消息类型，如"text"、"image"、"emoji"、"video"、"file"等
- `content` (str | Dict): 消息内容（通常是base64编码或文本，也可以是字典）
- `stream_id` (str): 聊天流ID
- `display_message` (str): 显示消息
- `typing` (bool): 是否显示正在输入
- `reply_message` (Optional["DatabaseMessages"]): 要回复的消息对象
- `set_reply` (bool): 是否设置为引用回复
- `storage_message` (bool): 是否存储消息到数据库
- `show_log` (bool): 是否显示日志

**Returns:**

- `bool` - 是否发送成功

## 使用示例

### 1. 基础文本发送，并回复消息



```
from src.plugin_system.apis import send_api

async def send_hello(chat_stream, reply_to_message):
    """发送问候消息并回复"""
    
    success = await send_api.text_to_stream(
        text="Hello, world!",
        stream_id=chat_stream.stream_id,
        typing=True,
        set_reply=True,
        reply_message=reply_to_message,
        storage_message=True
    )
    
    return success
```

### 2. 发送表情包



```
from src.plugin_system.apis import emoji_api
async def send_emoji_reaction(chat_stream, emotion):
    """根据情感发送表情包"""
    # 获取表情包
    emoji_result = await emoji_api.get_by_emotion(emotion)
    if not emoji_result:
        return False
    
    emoji_base64, description, matched_emotion = emoji_result
    
    # 发送表情包
    success = await send_api.emoji_to_stream(
        emoji_base64=emoji_base64,
        stream_id=chat_stream.stream_id,
        storage_message=False # 不存储到数据库
    )
    
    return success
```

## 消息类型说明

### 支持的消息类型

- `"text"`：纯文本消息
- `"emoji"`：表情包消息
- `"image"`：图片消息
- `"command"`：命令消息
- `"video"`：视频消息（如果支持）
- `"audio"`：音频消息（如果支持）

### 回复消息说明

回复消息需要使用 `reply_message` 参数传入 `DatabaseMessages` 对象，并设置 `set_reply=True`。

`DatabaseMessages` 对象可以通过 `message_api` 获取，例如：



```
from src.plugin_system.apis import message_api, send_api

# 获取最近的消息
messages = message_api.get_recent_messages(chat_stream.stream_id, hours=1, limit=1)
if messages:
    latest_message = messages[0]
    # 回复这条消息
    await send_api.text_to_stream(
        text="这是回复",
        stream_id=chat_stream.stream_id,
        set_reply=True,
        reply_message=latest_message
    )
```

## 注意事项

1. **异步操作**：所有发送函数都是异步的，必须使用`await`
2. **错误处理**：发送失败时返回False，成功时返回True
3. **发送频率**：注意控制发送频率，避免被平台限制
4. **内容限制**：注意平台对消息内容和长度的限制
5. **权限检查**：确保机器人有发送消息的权限
6. **编码格式**：图片和表情包需要使用base64编码
7. **存储选项**：可以选择是否将发送的消息存储到数据库





# 消息API

消息API提供了强大的消息查询、计数和格式化功能，让你轻松处理聊天消息数据。

## 导入方式



```
from src.plugin_system.apis import message_api
# 或者
from src.plugin_system import message_api
```

## 功能概述

消息API主要提供三大类功能：

- **消息查询** - 按时间、聊天、用户等条件查询消息
- **消息计数** - 统计新消息数量
- **消息格式化** - 将消息转换为可读格式

## 主要功能

### 1. 按照事件查询消息



```
def get_messages_by_time(
    start_time: float, end_time: float, limit: int = 0, limit_mode: str = "latest", filter_mai: bool = False
) -> List[DatabaseMessages]:
```

获取指定时间范围内的消息。

**Args:**

- `start_time` (float): 开始时间戳
- `end_time` (float): 结束时间戳
- `limit` (int): 限制返回消息数量，0为不限制
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

**注意：** `DatabaseMessages` 对象包含以下主要属性：

- `message_id`: 消息ID
- `time`: 时间戳
- `processed_plain_text`: 处理后的纯文本内容
- `user_info`: 用户信息对象
- `chat_info`: 聊天信息对象
- `additional_config`: 额外配置字典

更多信息请参考 `src.common.data_models.database_data_model.DatabaseMessages` 类。

### 2. 获取指定聊天中指定时间范围内的信息



```
def get_messages_by_time_in_chat(
    chat_id: str,
    start_time: float,
    end_time: float,
    limit: int = 0,
    limit_mode: str = "latest",
    filter_mai: bool = False,
    filter_command: bool = False,
    filter_intercept_message_level: Optional[int] = None,
) -> List[DatabaseMessages]:
```

获取指定聊天中指定时间范围内的消息。

**Args:**

- `chat_id` (str): 聊天ID
- `start_time` (float): 开始时间戳
- `end_time` (float): 结束时间戳
- `limit` (int): 限制返回消息数量，0为不限制
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False
- `filter_command` (bool): 是否过滤命令消息，默认False
- `filter_intercept_message_level` (Optional[int]): 过滤拦截消息级别，None为不过滤

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表，不是字典）

### 3. 获取指定聊天中指定时间范围内的信息（包含边界）



```
def get_messages_by_time_in_chat_inclusive(
    chat_id: str,
    start_time: float,
    end_time: float,
    limit: int = 0,
    limit_mode: str = "latest",
    filter_mai: bool = False,
    filter_command: bool = False,
    filter_intercept_message_level: Optional[int] = None,
) -> List[DatabaseMessages]:
```

获取指定聊天中指定时间范围内的消息（包含边界）。

**Args:**

- `chat_id` (str): 聊天ID
- `start_time` (float): 开始时间戳（包含）
- `end_time` (float): 结束时间戳（包含）
- `limit` (int): 限制返回消息数量，0为不限制
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False
- `filter_command` (bool): 是否过滤命令消息，默认False
- `filter_intercept_message_level` (Optional[int]): 过滤拦截消息级别，None为不过滤

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 4. 获取指定聊天中指定用户在指定时间范围内的消息



```
def get_messages_by_time_in_chat_for_users(
    chat_id: str,
    start_time: float,
    end_time: float,
    person_ids: List[str],
    limit: int = 0,
    limit_mode: str = "latest",
) -> List[Dict[str, Any]]:
```

获取指定聊天中指定用户在指定时间范围内的消息。

**Args:**

- `chat_id` (str): 聊天ID
- `start_time` (float): 开始时间戳
- `end_time` (float): 结束时间戳
- `person_ids` (List[str]): 用户ID列表
- `limit` (int): 限制返回消息数量，0为不限制
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 5. 随机选择一个聊天，返回该聊天在指定时间范围内的消息



```
def get_random_chat_messages(
    start_time: float,
    end_time: float,
    limit: int = 0,
    limit_mode: str = "latest",
    filter_mai: bool = False,
) -> List[Dict[str, Any]]:
```

随机选择一个聊天，返回该聊天在指定时间范围内的消息。

**Args:**

- `start_time` (float): 开始时间戳
- `end_time` (float): 结束时间戳
- `limit` (int): 限制返回消息数量，0为不限制
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 6. 获取指定用户在所有聊天中指定时间范围内的消息



```
def get_messages_by_time_for_users(
    start_time: float,
    end_time: float,
    person_ids: List[str],
    limit: int = 0,
    limit_mode: str = "latest",
) -> List[Dict[str, Any]]:
```

获取指定用户在所有聊天中指定时间范围内的消息。

**Args:**

- `start_time` (float): 开始时间戳
- `end_time` (float): 结束时间戳
- `person_ids` (List[str]): 用户ID列表
- `limit` (int): 限制返回消息数量，0为不限制
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 7. 获取指定时间戳之前的消息



```
def get_messages_before_time(
    timestamp: float,
    limit: int = 0,
    filter_mai: bool = False,
) -> List[Dict[str, Any]]:
```

获取指定时间戳之前的消息。

**Args:**

- `timestamp` (float): 时间戳
- `limit` (int): 限制返回消息数量，0为不限制
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False

**Returns:**

- `List[Dict[str, Any]]` - 消息列表

### 8. 获取指定聊天中指定时间戳之前的消息



```
def get_messages_before_time_in_chat(
    chat_id: str,
    timestamp: float,
    limit: int = 0,
    filter_mai: bool = False,
    filter_intercept_message_level: Optional[int] = None,
) -> List[DatabaseMessages]:
```

获取指定聊天中指定时间戳之前的消息。

**Args:**

- `chat_id` (str): 聊天ID
- `timestamp` (float): 时间戳
- `limit` (int): 限制返回消息数量，0为不限制
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False
- `filter_intercept_message_level` (Optional[int]): 过滤拦截消息级别，None为不过滤

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 9. 获取指定用户在指定时间戳之前的消息



```
def get_messages_before_time_for_users(
    timestamp: float,
    person_ids: List[str],
    limit: int = 0,
) -> List[Dict[str, Any]]:
```

获取指定用户在指定时间戳之前的消息。

**Args:**

- `timestamp` (float): 时间戳
- `person_ids` (List[str]): 用户ID列表
- `limit` (int): 限制返回消息数量，0为不限制

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 10. 获取指定聊天中最近一段时间的消息



```
def get_recent_messages(
    chat_id: str,
    hours: float = 24.0,
    limit: int = 100,
    limit_mode: str = "latest",
    filter_mai: bool = False,
) -> List[Dict[str, Any]]:
```

获取指定聊天中最近一段时间的消息。

**Args:**

- `chat_id` (str): 聊天ID
- `hours` (float): 最近多少小时，默认24小时
- `limit` (int): 限制返回消息数量，默认100条
- `limit_mode` (str): 限制模式，`"earliest"`获取最早记录，`"latest"`获取最新记录
- `filter_mai` (bool): 是否过滤掉机器人的消息，默认False

**Returns:**

- `List[DatabaseMessages]` - 消息列表（返回的是 `DatabaseMessages` 对象列表）

### 11. 计算指定聊天中从开始时间到结束时间的新消息数量



```
def count_new_messages(
    chat_id: str,
    start_time: float = 0.0,
    end_time: Optional[float] = None,
) -> int:
```

计算指定聊天中从开始时间到结束时间的新消息数量。

**Args:**

- `chat_id` (str): 聊天ID
- `start_time` (float): 开始时间戳
- `end_time` (Optional[float]): 结束时间戳，如果为None则使用当前时间

**Returns:**

- `int` - 新消息数量

### 12. 计算指定聊天中指定用户从开始时间到结束时间的新消息数量



```
def count_new_messages_for_users(
    chat_id: str,
    start_time: float,
    end_time: float,
    person_ids: List[str],
) -> int:
```

计算指定聊天中指定用户从开始时间到结束时间的新消息数量。

**Args:**

- `chat_id` (str): 聊天ID
- `start_time` (float): 开始时间戳
- `end_time` (float): 结束时间戳
- `person_ids` (List[str]): 用户ID列表

**Returns:**

- `int` - 新消息数量

### 13. 将消息列表构建成可读的字符串



```
def build_readable_messages_to_str(
    messages: List[DatabaseMessages],
    replace_bot_name: bool = True,
    timestamp_mode: str = "relative",
    read_mark: float = 0.0,
    truncate: bool = False,
    show_actions: bool = False,
) -> str:
```

将消息列表构建成可读的字符串。

**Args:**

- `messages` (List[DatabaseMessages]): 消息列表（`DatabaseMessages` 对象列表）
- `replace_bot_name` (bool): 是否将机器人的名称替换为"你"
- `timestamp_mode` (str): 时间戳显示模式，`"relative"`或`"absolute"`
- `read_mark` (float): 已读标记时间戳，用于分割已读和未读消息
- `truncate` (bool): 是否截断长消息
- `show_actions` (bool): 是否显示动作记录

**Returns:**

- `str` - 格式化后的可读字符串

### 14. 将消息列表构建成可读的字符串，并返回详细信息



```
async def build_readable_messages_with_details(
    messages: List[DatabaseMessages],
    replace_bot_name: bool = True,
    timestamp_mode: str = "relative",
    truncate: bool = False,
) -> Tuple[str, List[Tuple[float, str, str]]]:
```

将消息列表构建成可读的字符串，并返回详细信息。

**Args:**

- `messages` (List[DatabaseMessages]): 消息列表（`DatabaseMessages` 对象列表）
- `replace_bot_name` (bool): 是否将机器人的名称替换为"你"
- `timestamp_mode` (str): 时间戳显示模式，`"relative"`或`"absolute"`
- `truncate` (bool): 是否截断长消息

**Returns:**

- `Tuple[str, List[Tuple[float, str, str]]]` - 格式化后的可读字符串和详细信息元组列表(时间戳, 昵称, 内容)

### 15. 从消息列表中提取不重复的用户ID列表



```
async def get_person_ids_from_messages(
    messages: List[Dict[str, Any]],
) -> List[str]:
```

从消息列表中提取不重复的用户ID列表。

**Args:**

- `messages` (List[Dict[str, Any]]): 消息列表

**Returns:**

- `List[str]` - 用户ID列表

### 16. 从消息列表中移除机器人的消息



```
def filter_mai_messages(
    messages: List[DatabaseMessages],
) -> List[DatabaseMessages]:
```

从消息列表中移除机器人的消息。

**Args:**

- `messages` (List[DatabaseMessages]): 消息列表，每个元素是 `DatabaseMessages` 对象

**Returns:**

- `List[DatabaseMessages]` - 过滤后的消息列表

## 注意事项

1. **时间戳格式**：所有时间参数都使用Unix时间戳（float类型）
2. **异步函数**：部分函数是异步函数，需要使用 `await`
3. **性能考虑**：查询大量消息时建议设置合理的 `limit` 参数
4. **消息格式**：返回的消息是字典格式，包含时间戳、发送者、内容等信息
5. **用户ID**：`person_ids` 参数接受字符串列表，用于筛选特定用户的消息

# 聊天API

聊天API模块专门负责聊天信息的查询和管理，帮助插件获取和管理不同的聊天流。

## 导入方式



```
from src.plugin_system import chat_api
# 或者
from src.plugin_system.apis import chat_api
```

## 主要功能

### 1. 获取所有的聊天流



```
def get_all_streams(platform: Optional[str] | SpecialTypes = "qq") -> List[ChatStream]:
```

**Args**:

- `platform`：平台筛选，默认为"qq"，可以使用`SpecialTypes`枚举类中的`SpecialTypes.ALL_PLATFORMS`来获取所有平台的聊天流。

**Returns**:

- `List[ChatStream]`：聊天流列表

### 2. 获取群聊聊天流



```
def get_group_streams(platform: Optional[str] | SpecialTypes = "qq") -> List[ChatStream]:
```

**Args**:

- `platform`：平台筛选，默认为"qq"，可以使用`SpecialTypes`枚举类中的`SpecialTypes.ALL_PLATFORMS`来获取所有平台的群聊流。

**Returns**:

- `List[ChatStream]`：群聊聊天流列表

### 3. 获取私聊聊天流



```
def get_private_streams(platform: Optional[str] | SpecialTypes = "qq") -> List[ChatStream]:
```

**Args**:

- `platform`：平台筛选，默认为"qq"，可以使用`SpecialTypes`枚举类中的`SpecialTypes.ALL_PLATFORMS`来获取所有平台的私聊流。

**Returns**:

- `List[ChatStream]`：私聊聊天流列表

### 4. 根据群ID获取聊天流



```
def get_stream_by_group_id(group_id: str, platform: Optional[str] | SpecialTypes = "qq") -> Optional[ChatStream]:
```

**Args**:

- `group_id`：群聊ID
- `platform`：平台筛选，默认为"qq"，可以使用`SpecialTypes`枚举类中的`SpecialTypes.ALL_PLATFORMS`来获取所有平台的群聊流。

**Returns**:

- `Optional[ChatStream]`：聊天流对象，如果未找到返回None

### 5. 根据用户ID获取私聊流



```
def get_stream_by_user_id(user_id: str, platform: Optional[str] | SpecialTypes = "qq") -> Optional[ChatStream]:
```

**Args**:

- `user_id`：用户ID
- `platform`：平台筛选，默认为"qq"，可以使用`SpecialTypes`枚举类中的`SpecialTypes.ALL_PLATFORMS`来获取所有平台的私聊流。

**Returns**:

- `Optional[ChatStream]`：聊天流对象，如果未找到返回None

### 6. 获取聊天流类型



```
def get_stream_type(chat_stream: ChatStream) -> str:
```

**Args**:

- `chat_stream`：聊天流对象

**Returns**:

- `str`：聊天流类型，可能的值包括`private`（私聊流），`group`（群聊流）以及`unknown`（未知类型）。

### 7. 获取聊天流信息



```
def get_stream_info(chat_stream: ChatStream) -> Dict[str, Any]:
```

**Args**:

- `chat_stream`：聊天流对象

**Returns**:

- ```
  Dict[str, Any]
  ```

  ：聊天流的详细信息，包括但不限于：

  - `stream_id`：聊天流ID
  - `platform`：平台名称
  - `type`：聊天流类型
  - `group_id`：群聊ID
  - `group_name`：群聊名称
  - `user_id`：用户ID
  - `user_name`：用户名称

### 8. 获取聊天流统计摘要



```
def get_streams_summary() -> Dict[str, int]:
```

**Returns**:

- ```
  Dict[str, int]
  ```

  ：聊天流统计信息摘要，包含以下键：

  - `total_streams`：总聊天流数量
  - `group_streams`：群聊流数量
  - `private_streams`：私聊流数量
  - `qq_streams`：QQ平台流数量

## 注意事项

1. 大部分函数在参数不合法时候会抛出异常，请确保你的程序进行了捕获。
2. `ChatStream`对象包含了聊天的完整信息，包括用户信息、群信息等。

# LLM API

LLM API模块提供与大语言模型交互的功能，让插件能够使用系统配置的LLM模型进行内容生成。

## 导入方式



```
from src.plugin_system.apis import llm_api
# 或者
from src.plugin_system import llm_api
```

## 主要功能

### 1. 查询可用模型



```
def get_available_models() -> Dict[str, TaskConfig]:
```

获取所有可用的模型配置。

**Return：**

- `Dict[str, TaskConfig]`：模型配置字典，key为模型名称，value为模型配置对象。

### 2. 使用模型生成内容



```
async def generate_with_model(
    prompt: str, model_config: TaskConfig, request_type: str = "plugin.generate", **kwargs
) -> Tuple[bool, str, str, str]:
```

使用指定模型生成内容。

**Args:**

- `prompt`：提示词。
- `model_config`：模型配置对象（从 `get_available_models` 获取）。
- `request_type`：请求类型标识，默认为 `"plugin.generate"`。
- `**kwargs`：其他模型特定参数，如 `temperature`、`max_tokens` 等。

**Return：**

- `Tuple[bool, str, str, str]`：返回一个元组，包含（是否成功, 生成的内容, 推理过程, 模型

# 回复生成器API

回复生成器API模块提供智能回复生成功能，让插件能够使用系统的回复生成器来产生自然的聊天回复。

## 导入方式



```
from src.plugin_system.apis import generator_api
# 或者
from src.plugin_system import generator_api
```

## 主要功能

### 1. 回复器获取



```
def get_replyer(
    chat_stream: Optional[ChatStream] = None,
    chat_id: Optional[str] = None,
    model_set_with_weight: Optional[List[Tuple[TaskConfig, float]]] = None,
    request_type: str = "replyer",
) -> Optional[DefaultReplyer]:
```

获取回复器对象

优先使用chat_stream，如果没有则使用chat_id直接查找。

使用 ReplyerManager 来管理实例，避免重复创建。

**Args:**

- `chat_stream`: 聊天流对象
- `chat_id`: 聊天ID（实际上就是`stream_id`）
- `model_set_with_weight`: 模型配置列表，每个元素为 `(TaskConfig, weight)` 元组
- `request_type`: 请求类型，用于记录LLM使用情况，可以不写

**Returns:**

- `DefaultReplyer`: 回复器对象，如果获取失败则返回None

#### 示例



```
# 使用聊天流获取回复器
replyer = generator_api.get_replyer(chat_stream=chat_stream)

# 使用平台和ID获取回复器
replyer = generator_api.get_replyer(chat_id="123456789")
```

### 2. 回复生成



```
async def generate_reply(
    chat_stream: Optional[ChatStream] = None,
    chat_id: Optional[str] = None,
    action_data: Optional[Dict[str, Any]] = None,
    reply_message: Optional["DatabaseMessages"] = None,
    think_level: int = 1,
    extra_info: str = "",
    reply_reason: str = "",
    available_actions: Optional[Dict[str, ActionInfo]] = None,
    chosen_actions: Optional[List["ActionPlannerInfo"]] = None,
    unknown_words: Optional[List[str]] = None,
    enable_tool: bool = False,
    enable_splitter: bool = True,
    enable_chinese_typo: bool = True,
    request_type: str = "generator_api",
    from_plugin: bool = True,
    reply_time_point: Optional[float] = None,
) -> Tuple[bool, Optional["LLMGenerationDataModel"]]:
```

生成回复

优先使用chat_stream，如果没有则使用chat_id直接查找。

**Args:**

- `chat_stream`: 聊天流对象（优先）
- `chat_id`: 聊天ID（备用）
- `action_data`: 动作数据（向下兼容，包含`reply_to`和`extra_info`）
- `reply_message`: 回复的消息对象
- `think_level`: 思考级别，0为轻量回复，1为中等回复
- `extra_info`: 额外信息，用于补充上下文
- `reply_reason`: 回复原因
- `available_actions`: 可用动作字典，格式为 `{"action_name": ActionInfo}`
- `chosen_actions`: 已选动作列表
- `unknown_words`: Planner 在 reply 动作中给出的未知词语列表，用于黑话检索
- `enable_tool`: 是否启用工具调用
- `enable_splitter`: 是否启用消息分割器
- `enable_chinese_typo`: 是否启用错字生成器
- `request_type`: 请求类型（可选，记录LLM使用）
- `from_plugin`: 是否来自插件
- `reply_time_point`: 回复时间点

**Returns:**

- ```
  Tuple[bool, Optional["LLMGenerationDataModel"]]
  ```

  : (是否成功, LLM生成数据模型)

  - 成功时返回 `LLMGenerationDataModel` 对象，包含 `reply_set`、`content`、`prompt`、`model` 等属性
  - 失败时返回 `None`

#### 示例



```
success, llm_response = await generator_api.generate_reply(
    chat_stream=chat_stream,
    reply_message=message,
    extra_info="这是额外的上下文信息",
    reply_reason="用户询问了问题",
    available_actions=action_info,
    enable_tool=True,
    think_level=1
)
if success and llm_response:
    # 访问回复集合
    if llm_response.reply_set:
        for reply_content in llm_response.reply_set.reply_data:
            print(f"回复类型: {reply_content.content_type}, 内容: {reply_content.content}")
    # 访问原始内容
    print(f"原始回复: {llm_response.content}")
    # 访问提示词
    print(f"使用的提示词: {llm_response.prompt}")
    # 访问模型信息
    print(f"使用的模型: {llm_response.model}")
```

### 3. 回复重写



```
async def rewrite_reply(
    chat_stream: Optional[ChatStream] = None,
    reply_data: Optional[Dict[str, Any]] = None,
    chat_id: Optional[str] = None,
    enable_splitter: bool = True,
    enable_chinese_typo: bool = True,
    raw_reply: str = "",
    reason: str = "",
    reply_to: str = "",
    request_type: str = "generator_api",
) -> Tuple[bool, Optional["LLMGenerationDataModel"]]:
```

重写回复，使用新的内容替换旧的回复内容。

优先使用chat_stream，如果没有则使用chat_id直接查找。

**Args:**

- `chat_stream`: 聊天流对象（优先）
- `reply_data`: 回复数据字典（向下兼容备用，当其他参数缺失时从此获取）
- `chat_id`: 聊天ID（备用）
- `enable_splitter`: 是否启用消息分割器
- `enable_chinese_typo`: 是否启用错字生成器
- `raw_reply`: 原始回复内容
- `reason`: 重写原因
- `reply_to`: 回复对象
- `request_type`: 请求类型（可选，记录LLM使用）

**Returns:**

- ```
  Tuple[bool, Optional["LLMGenerationDataModel"]]
  ```

  : (是否成功, LLM生成数据模型)

  - 成功时返回 `LLMGenerationDataModel` 对象，包含 `reply_set`、`content`、`prompt` 等属性
  - 失败时返回 `None`

#### 示例



```
success, llm_response = await generator_api.rewrite_reply(
    chat_stream=chat_stream,
    raw_reply="原始回复内容",
    reason="重写原因",
    reply_to="麦麦:你好"
)
if success and llm_response:
    # 访问回复集合
    if llm_response.reply_set:
        for reply_content in llm_response.reply_set.reply_data:
            print(f"回复类型: {reply_content.content_type}, 内容: {reply_content.content}")
    # 访问原始内容
    print(f"重写后的回复: {llm_response.content}")
    # 访问提示词
    print(f"使用的提示词: {llm_response.prompt}")
```

## LLMGenerationDataModel 对象说明

`generate_reply` 和 `rewrite_reply` 函数返回的 `LLMGenerationDataModel` 对象包含以下主要属性：

- `reply_set`: `ReplySetModel` 对象，包含处理后的回复内容列表
- `content`: 原始LLM生成的文本内容
- `processed_output`: 处理后的输出列表（分割后的文本）
- `prompt`: 使用的提示词
- `model`: 使用的模型名称
- `timing`: 生成耗时信息
- `reasoning`: 推理过程（如果有）

### ReplySetModel 结构

`reply_set` 是一个 `ReplySetModel` 对象，包含 `reply_data` 属性，这是一个 `ReplyContent` 对象列表。

每个 `ReplyContent` 对象包含：

- `content_type`: 内容类型（`ReplyContentType.TEXT`、`ReplyContentType.EMOJI`、`ReplyContentType.IMAGE` 等）
- `content`: 内容数据（文本字符串、base64编码的图片等）

### 使用示例



```
success, llm_response = await generator_api.generate_reply(...)
if success and llm_response and llm_response.reply_set:
    for reply_content in llm_response.reply_set.reply_data:
        if reply_content.content_type == ReplyContentType.TEXT:
            print(f"文本: {reply_content.content}")
        elif reply_content.content_type == ReplyContentType.EMOJI:
            print(f"表情包: {reply_content.content[:50]}...")  # base64数据很长
```

### 4. 自定义提示词回复



```
async def generate_response_custom(
    chat_stream: Optional[ChatStream] = None,
    chat_id: Optional[str] = None,
    request_type: str = "generator_api",
    prompt: str = "",
) -> Optional[str]:
```

生成自定义提示词回复

优先使用chat_stream，如果没有则使用chat_id直接查找。

**Args:**

- `chat_stream`: 聊天流对象
- `chat_id`: 聊天ID（备用）
- `request_type`: 请求类型（可选，记录LLM使用）
- `prompt`: 自定义提示词

**Returns:**

- `Optional[str]`: 生成的自定义回复内容，如果生成失败则返回None

## 注意事项

1. **异步操作**：部分函数是异步的，须使用`await`
2. **聊天流依赖**：需要有效的聊天流对象才能正常工作
3. **性能考虑**：回复生成可能需要一些时间，特别是使用LLM时
4. **回复格式**：返回的回复集合是元组列表，包含类型和内容
5. **上下文感知**：生成器会考虑聊天上下文和历史消息，除非你用的是自定义提示词。

```
int:
```

获取当前可用表情包的数量

### 5. 获取表情包系统信息



```
def get_info() -> Dict[str, Any]:
```

获取表情包系统的基本信息

**Returns：**

- ```
  Dict[str, Any]
  ```

  ：包含表情包数量、描述等信息的字典，包含以下键：

  - `current_count`：当前表情包数量
  - `max_count`：最大表情包数量
  - `available_emojis`：当前可用的表情包数量

### 6. 获取所有可用的情感标签



```
def get_emotions() -> List[str]:
```

获取所有可用的情感标签 **（已经去重）**

### 7. 获取所有表情包描述



```
def get_descriptions() -> List[str]:
```

获取所有表情包的描述列表

## 场景描述说明

### 常用场景描述

表情包系统支持多种具体的场景描述，举例如下：

- **开心类场景**：开心的大笑、满意的微笑、兴奋的手舞足蹈
- **无奈类场景**：表示无奈和沮丧、轻微的讽刺、无语的摇头
- **愤怒类场景**：愤怒和不满、生气的瞪视、暴躁的抓狂
- **惊讶类场景**：震惊的表情、意外的发现、困惑的思考
- **可爱类场景**：卖萌的表情、撒娇的动作、害羞的样子

### 情感关键词示例

系统支持的情感关键词举例如下：

- 大笑、微笑、兴奋、手舞足蹈
- 无奈、沮丧、讽刺、无语、摇头
- 愤怒、不满、生气、瞪视、抓狂
- 震惊、意外、困惑、思考
- 卖萌、撒娇、害羞、可爱

### 匹配机制

- **精确匹配**：优先匹配完整的场景描述，如"开心的大笑"
- **关键词匹配**：如果没有精确匹配，则根据关键词进行模糊匹配
- **语义匹配**：系统会理解场景的语义含义进行智能匹配

## 注意事项

1. **异步函数**：部分函数是异步的，需要使用 `await`
2. **返回格式**：表情包以base64编码返回，可直接用于发送
3. **错误处理**：所有函数都有错误处理，失败时返回None，空列表或默认值
4. **使用统计**：系统会记录表情包的使用次数
5. **文件依赖**：表情包依赖于本地文件，确保表情包文件存在
6. **编码格式**：返回的是base64编码的图片数据，可直接用于网络传输
7. **场景理解**：系统能理解具体的场景描述，比简单的情感分类更准确

###  4. 判断用户是否已知



```
async def is_person_known(platform: str, user_id: int) -> bool:
```

判断是否认识某个用户

**Args:**

- `platform`：平台名称
- `user_id`：用户ID

**Returns:**

- `bool`：是否认识该用户

### 5. 根据用户名获取Person ID



```
def get_person_id_by_name(person_name: str) -> str:
```

根据用户名获取person_id

**Args:**

- `person_name`：用户名

**Returns:**

- `str`：person_id，如果未找到返回空字符串

## 常用字段说明

### 基础信息字段

- `nickname`：用户昵称
- `platform`：平台信息
- `user_id`：用户ID

### 关系信息字段

- `impression`：对用户的印象
- `points`: 用户特征点

其他字段可以参考`PersonInfo`类的属性（位于`src.common.database.database_model`）

## 注意事项

1. **异步操作**：部分查询函数都是异步的，需要使用`await`
2. **性能考虑**：批量查询优于单个查询
3. **隐私保护**：确保用户信息的使用符合隐私政策
4. **数据一致性**：person_id是用户的唯一标识，应妥善保存和使用

# 数据库API

数据库API模块提供通用的数据库操作功能，支持查询、创建、更新和删除记录，采用Peewee ORM模型。

## 导入方式



```
from src.plugin_system.apis import database_api
# 或者
from src.plugin_system import database_api
```

## 主要功能

### 1. 通用数据库操作



```
async def db_query(
    model_class: Type[Model],
    data: Optional[Dict[str, Any]] = None,
    query_type: Optional[str] = "get",
    filters: Optional[Dict[str, Any]] = None,
    limit: Optional[int] = None,
    order_by: Optional[List[str]] = None,
    single_result: Optional[bool] = False,
) -> Union[List[Dict[str, Any]], Dict[str, Any], None]:
```

执行数据库查询操作的通用接口。

**Args:**

- ```
  model_class
  ```

  : Peewee模型类。

  - Peewee模型类可以在`src.common.database.database_model`模块中找到，如`ActionRecords`、`Messages`等。

- `data`: 用于创建或更新的数据

- ```
  query_type
  ```

  : 查询类型

  - 可选值: `get`, `create`, `update`, `delete`, `count`。

- `filters`: 过滤条件字典，键为字段名，值为要匹配的值。

- `limit`: 限制结果数量。

- ```
  order_by
  ```

  : 排序字段列表，使用字段名，前缀'-'表示降序。

  - 排序字段，前缀`-`表示降序，例如`-time`表示按时间字段（即`time`字段）降序

- `single_result`: 是否只返回单个结果。

**Returns:**

- 根据查询类型返回不同的结果：
  - `get`: 返回查询结果列表或单个结果。（如果 `single_result=True`）
  - `create`: 返回创建的记录。
  - `update`: 返回受影响的行数。
  - `delete`: 返回受影响的行数。
  - `count`: 返回记录数量。

#### 示例

1. 查询最近10条消息



```
messages = await database_api.db_query(
    Messages,
    query_type="get",
    filters={"chat_id": chat_stream.stream_id},
    limit=10,
    order_by=["-time"]
)
```

1. 创建一条记录



```
new_record = await database_api.db_query(
    ActionRecords,
    data={"action_id": "123", "time": time.time(), "action_name": "TestAction"},
    query_type="create",
)
```

1. 更新记录



```
updated_count = await database_api.db_query(
    ActionRecords,
    data={"action_done": True},
    query_type="update",
    filters={"action_id": "123"},
)
```

1. 删除记录



```
deleted_count = await database_api.db_query(
    ActionRecords,
    query_type="delete",
    filters={"action_id": "123"}
)
```

1. 计数



```
count = await database_api.db_query(
    Messages,
    query_type="count",
    filters={"chat_id": chat_stream.stream_id}
)
```

### 2. 数据库保存



```
async def db_save(
    model_class: Type[Model], data: Dict[str, Any], key_field: Optional[str] = None, key_value: Optional[Any] = None
) -> Optional[Dict[str, Any]]:
```

保存数据到数据库（创建或更新）

如果提供了key_field和key_value，会先尝试查找匹配的记录进行更新；

如果没有找到匹配记录，或未提供key_field和key_value，则创建新记录。

**Args:**

- `model_class`: Peewee模型类。
- `data`: 要保存的数据字典。
- `key_field`: 用于查找现有记录的字段名，例如"action_id"。
- `key_value`: 用于查找现有记录的字段值。

**Returns:**

- `Optional[Dict[str, Any]]`: 保存后的记录数据，失败时返回None。

#### 示例

创建或更新一条记录



```
record = await database_api.db_save(
    ActionRecords,
    {
        "action_id": "123",
        "time": time.time(),
        "action_name": "TestAction",
        "action_done": True
    },
    key_field="action_id",
    key_value="123"
)
```

### 3. 数据库获取



```
async def db_get(
    model_class: Type[Model],
    filters: Optional[Dict[str, Any]] = None,
    limit: Optional[int] = None,
    order_by: Optional[str] = None,
    single_result: Optional[bool] = False,
) -> Union[List[Dict[str, Any]], Dict[str, Any], None]:
```

从数据库获取记录

这是db_query方法的简化版本，专注于数据检索操作。

**Args:**

- `model_class`: Peewee模型类。
- `filters`: 过滤条件字典，键为字段名，值为要匹配的值。
- `limit`: 限制结果数量。
- `order_by`: 排序字段，使用字段名，前缀'-'表示降序。
- `single_result`: 是否只返回单个结果，如果为True，则返回单个记录字典或None；否则返回记录字典列表或空列表

**Returns:**

- `Union[List[Dict], Dict, None]`: 查询结果列表或单个结果（如果`single_result=True`），失败时返回None。

#### 示例

1. 获取单个记录



```
record = await database_api.db_get(
    ActionRecords,
    filters={"action_id": "123"},
    limit=1
)
```

1. 获取最近10条记录



```
records = await database_api.db_get(
    Messages,
    filters={"chat_id": chat_stream.stream_id},
    limit=10,
    order_by="-time",
)
```

### 4. 动作信息存储



```
async def store_action_info(
    chat_stream=None,
    action_build_into_prompt: bool = False,
    action_prompt_display: str = "",
    action_done: bool = True,
    thinking_id: str = "",
    action_data: Optional[dict] = None,
    action_name: str = "",
    action_reasoning: str = "",
) -> Optional[Dict[str, Any]]:
```

存储动作信息到数据库，是一种针对 Action 的 `db_save()` 的封装函数。

将Action执行的相关信息保存到ActionRecords表中，用于后续的记忆和上下文构建。

**Args:**

- `chat_stream`: 聊天流对象，包含聊天ID等信息。
- `action_build_into_prompt`: 是否将动作信息构建到提示中。
- `action_prompt_display`: 动作提示的显示文本。
- `action_done`: 动作是否完成。
- `thinking_id`: 思考过程的ID。
- `action_data`: 动作的数据字典。
- `action_name`: 动作的名称。
- `action_reasoning`: 动作执行理由。

**Returns:**

- `Optional[Dict[str, Any]]`: 存储后的记录数据，失败时返回None。

#### 示例



```
record = await database_api.store_action_info(
    chat_stream=chat_stream,
    action_build_into_prompt=True,
    action_prompt_display="执行了回复动作",
    action_done=True,
    thinking_id="thinking_123",
    action_data={"content": "Hello"},
    action_name="reply_action",
    action_reasoning="用户询问了问题，需要回复"
)
```

# 配置API

配置API模块提供了配置读取功能，让插件能够安全地访问全局配置和插件配置。

## 导入方式



```
from src.plugin_system.apis import config_api
# 或者
from src.plugin_system import config_api
```

## 主要功能

### 1. 访问全局配置



```
def get_global_config(key: str, default: Any = None) -> Any:
```

**Args**:

- `key`: 命名空间式配置键名，使用嵌套访问，如 "section.subsection.key"，大小写敏感
- `default`: 如果配置不存在时返回的默认值

**Returns**:

- `Any`: 配置值或默认值

#### 示例：

获取机器人昵称



```
bot_name = config_api.get_global_config("bot.nickname", "MaiBot")
```

### 2. 获取插件配置



```
def get_plugin_config(plugin_config: dict, key: str, default: Any = None) -> Any:
```

**Args**:

- `plugin_config`: 插件配置字典
- `key`: 配置键名，支持嵌套访问如 "section.subsection.key"，大小写敏感
- `default`: 如果配置不存在时返回的默认值

**Returns**:

- `Any`: 配置值或默认值

## 注意事项

1. **只读访问**：配置API只提供读取功能，插件不能修改全局配置
2. **错误处理**：所有函数都有错误处理，失败时会记录日志并返回默认值
3. **安全性**：插件通过此API访问配置是安全和隔离的
4. **性能**：频繁访问的配置建议在插件初始化时获取并缓存

# 插件管理API

插件管理API模块提供了对插件的加载、卸载、重新加载以及目录管理功能。

## 导入方式



```
from src.plugin_system.apis import plugin_manage_api
# 或者
from src.plugin_system import plugin_manage_api
```

## 功能概述

插件管理API主要提供以下功能：

- **插件查询** - 列出当前加载的插件或已注册的插件。
- **插件管理** - 加载、卸载、重新加载插件。
- **插件目录管理** - 添加插件目录并重新扫描。

## 主要功能

### 1. 列出当前加载的插件



```
def list_loaded_plugins() -> List[str]:
```

列出所有当前加载的插件。

**Returns:**

- `List[str]` - 当前加载的插件名称列表。

### 2. 列出所有已注册的插件



```
def list_registered_plugins() -> List[str]:
```

列出所有已注册的插件。

**Returns:**

- `List[str]` - 已注册的插件名称列表。

### 3. 获取插件路径



```
def get_plugin_path(plugin_name: str) -> str:
```

获取指定插件的路径。

**Args:**

- `plugin_name` (str): 要查询的插件名称。 **Returns:**
- `str` - 插件的路径，如果插件不存在则 raise ValueError。

### 4. 卸载指定的插件



```
async def remove_plugin(plugin_name: str) -> bool:
```

卸载指定的插件。

**Args:**

- `plugin_name` (str): 要卸载的插件名称。

**Returns:**

- `bool` - 卸载是否成功。

### 5. 重新加载指定的插件



```
async def reload_plugin(plugin_name: str) -> bool:
```

重新加载指定的插件。

**Args:**

- `plugin_name` (str): 要重新加载的插件名称。

**Returns:**

- `bool` - 重新加载是否成功。

### 6. 加载指定的插件



```
def load_plugin(plugin_name: str) -> Tuple[bool, int]:
```

加载指定的插件。

**Args:**

- `plugin_name` (str): 要加载的插件名称。

**Returns:**

- `Tuple[bool, int]` - 加载是否成功，成功或失败的个数。

### 7. 添加插件目录



```
def add_plugin_directory(plugin_directory: str) -> bool:
```

添加插件目录。

**Args:**

- `plugin_directory` (str): 要添加的插件目录路径。

**Returns:**

- `bool` - 添加是否成功。

### 8. 重新扫描插件目录



```
def rescan_plugin_directory() -> Tuple[int, int]:
```

重新扫描插件目录，加载新插件。

**Returns:**

- `Tuple[int, int]` - 成功加载的插件数量和失败的插件数量。

# 组件管理API

组件管理API模块提供了对插件组件的查询和管理功能，使得插件能够获取和使用组件相关的信息。

## 导入方式



```
from src.plugin_system.apis import component_manage_api
# 或者
from src.plugin_system import component_manage_api
```

## 功能概述

组件管理API主要提供以下功能：

- **插件信息查询** - 获取所有插件或指定插件的信息。
- **组件查询** - 按名称或类型查询组件信息。
- **组件管理** - 启用或禁用组件，支持全局和局部操作。

## 主要功能

### 1. 获取所有插件信息



```
def get_all_plugin_info() -> Dict[str, PluginInfo]:
```

获取所有插件的信息。

**Returns:**

- `Dict[str, PluginInfo]` - 包含所有插件信息的字典，键为插件名称，值为 `PluginInfo` 对象。

### 2. 获取指定插件信息



```
def get_plugin_info(plugin_name: str) -> Optional[PluginInfo]:
```

获取指定插件的信息。

**Args:**

- `plugin_name` (str): 插件名称。

**Returns:**

- `Optional[PluginInfo]`: 插件信息对象，如果插件不存在则返回 `None`。

### 3. 获取指定组件信息



```
def get_component_info(component_name: str, component_type: ComponentType) -> Optional[Union[CommandInfo, ActionInfo, EventHandlerInfo]]:
```

获取指定组件的信息。

**Args:**

- `component_name` (str): 组件名称。
- `component_type` (ComponentType): 组件类型。

**Returns:**

- `Optional[Union[CommandInfo, ActionInfo, EventHandlerInfo]]`: 组件信息对象，如果组件不存在则返回 `None`。

### 4. 获取指定类型的所有组件信息



```
def get_components_info_by_type(component_type: ComponentType) -> Dict[str, Union[CommandInfo, ActionInfo, EventHandlerInfo]]:
```

获取指定类型的所有组件信息。

**Args:**

- `component_type` (ComponentType): 组件类型。

**Returns:**

- `Dict[str, Union[CommandInfo, ActionInfo, EventHandlerInfo]]`: 包含指定类型组件信息的字典，键为组件名称，值为对应的组件信息对象。

### 5. 获取指定类型的所有启用的组件信息



```
def get_enabled_components_info_by_type(component_type: ComponentType) -> Dict[str, Union[CommandInfo, ActionInfo, EventHandlerInfo]]:
```

获取指定类型的所有启用的组件信息。

**Args:**

- `component_type` (ComponentType): 组件类型。

**Returns:**

- `Dict[str, Union[CommandInfo, ActionInfo, EventHandlerInfo]]`: 包含指定类型启用组件信息的字典，键为组件名称，值为对应的组件信息对象。

### 6. 获取指定 Action 的注册信息



```
def get_registered_action_info(action_name: str) -> Optional[ActionInfo]:
```

获取指定 Action 的注册信息。

**Args:**

- `action_name` (str): Action 名称。

**Returns:**

- `Optional[ActionInfo]` - Action 信息对象，如果 Action 不存在则返回 `None`。

### 7. 获取指定 Command 的注册信息



```
def get_registered_command_info(command_name: str) -> Optional[CommandInfo]:
```

获取指定 Command 的注册信息。

**Args:**

- `command_name` (str): Command 名称。

**Returns:**

- `Optional[CommandInfo]` - Command 信息对象，如果 Command 不存在则返回 `None`。

### 8. 获取指定 Tool 的注册信息



```
def get_registered_tool_info(tool_name: str) -> Optional[ToolInfo]:
```

获取指定 Tool 的注册信息。

**Args:**

- `tool_name` (str): Tool 名称。

**Returns:**

- `Optional[ToolInfo]` - Tool 信息对象，如果 Tool 不存在则返回 `None`。

### 9. 获取指定 EventHandler 的注册信息



```
def get_registered_event_handler_info(event_handler_name: str) -> Optional[EventHandlerInfo]:
```

获取指定 EventHandler 的注册信息。

**Args:**

- `event_handler_name` (str): EventHandler 名称。

**Returns:**

- `Optional[EventHandlerInfo]` - EventHandler 信息对象，如果 EventHandler 不存在则返回 `None`。

### 10. 全局启用指定组件



```
def globally_enable_component(component_name: str, component_type: ComponentType) -> bool:
```

全局启用指定组件。

**Args:**

- `component_name` (str): 组件名称。
- `component_type` (ComponentType): 组件类型。

**Returns:**

- `bool` - 启用成功返回 `True`，否则返回 `False`。

### 11. 全局禁用指定组件



```
async def globally_disable_component(component_name: str, component_type: ComponentType) -> bool:
```

全局禁用指定组件。

**此函数是异步的，确保在异步环境中调用。**

**Args:**

- `component_name` (str): 组件名称。
- `component_type` (ComponentType): 组件类型。

**Returns:**

- `bool` - 禁用成功返回 `True`，否则返回 `False`。

### 12. 局部启用指定组件



```
def locally_enable_component(component_name: str, component_type: ComponentType, stream_id: str) -> bool:
```

局部启用指定组件。

**Args:**

- `component_name` (str): 组件名称。
- `component_type` (ComponentType): 组件类型。
- `stream_id` (str): 消息流 ID。

**Returns:**

- `bool` - 启用成功返回 `True`，否则返回 `False`。

### 13. 局部禁用指定组件



```
def locally_disable_component(component_name: str, component_type: ComponentType, stream_id: str) -> bool:
```

局部禁用指定组件。

**Args:**

- `component_name` (str): 组件名称。
- `component_type` (ComponentType): 组件类型。
- `stream_id` (str): 消息流 ID。

**Returns:**

- `bool` - 禁用成功返回 `True`，否则返回 `False`。

### 14. 获取指定消息流中禁用的组件列表



```
def get_locally_disabled_components(stream_id: str, component_type: ComponentType) -> list[str]:
```

获取指定消息流中禁用的组件列表。

**Args:**

- `stream_id` (str): 消息流 ID。
- `component_type` (ComponentType): 组件类型。

**Returns:**

- `list[str]` - 禁用的组件名称列表。

# Logging API

Logging API模块提供了获取本体logger的功能，允许插件记录日志信息。

## 导入方式



```
from src.plugin_system.apis import get_logger
# 或者
from src.plugin_system import get_logger
```

## 主要功能

### 1. 获取本体logger



```
def get_logger(name: str) -> structlog.stdlib.BoundLogger:
```

获取本体logger实例。

**Args:**

- `name` (str): 日志记录器的名称。

**Returns:**

- 一个logger实例，有以下方法:
  - `debug`
  - `info`
  - `warning`
  - `error`
  - `critical`



# 工具API

工具API模块提供了获取和管理工具实例的功能，让插件能够访问系统中注册的工具。

## 导入方式



```
from src.plugin_system.apis import tool_api
# 或者
from src.plugin_system import tool_api
```

## 主要功能

### 1. 获取工具实例



```
def get_tool_instance(tool_name: str) -> Optional[BaseTool]:
```

获取指定名称的工具实例。

**Args**:

- `tool_name`: 工具名称字符串

**Returns**:

- `Optional[BaseTool]`: 工具实例，如果工具不存在则返回 None

### 2. 获取LLM可用的工具定义



```
def get_llm_available_tool_definitions():
```

获取所有LLM可用的工具定义列表。

**Returns**:

- ```
  List[Tuple[str, Dict[str, Any]]]
  ```

  : 工具定义列表，每个元素为

   

  ```
  (工具名称, 工具定义字典)
  ```

   

  的元组

  - 其具体定义请参照[tool-components.md](https://docs.mai-mai.org/develop/plugin_develop/tool-components.html)中的工具定义格式。

#### 示例：



```
# 获取所有LLM可用的工具定义
tools = tool_api.get_llm_available_tool_definitions()
for tool_name, tool_definition in tools:
    print(f"工具: {tool_name}")
    print(f"定义: {tool_definition}")
```

## 注意事项

1. **工具存在性检查**：使用前请检查工具实例是否为 None
2. **权限控制**：某些工具可能有使用权限限制
3. **异步调用**：大多数工具方法是异步的，需要使用 await
4. **错误处理**：调用工具时请做好异常处理