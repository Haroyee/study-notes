# MaiBot插件开发文档

> 欢迎来到MaiBot插件系统开发文档！这里是你开始插件开发旅程的最佳起点。

## 新手入门

- [📖 快速开始指南](https://docs.mai-mai.org/develop/plugin_develop/quick-start.html) - 快速创建你的第一个插件

## 组件功能详解

- [🧱 Action组件详解](https://docs.mai-mai.org/develop/plugin_develop/action-components.html) - 掌握最核心的Action组件
- [💻 Command组件详解](https://docs.mai-mai.org/develop/plugin_develop/command-components.html) - 学习直接响应命令的组件
- [🔧 Tool组件详解](https://docs.mai-mai.org/develop/plugin_develop/tool-components.html) - 了解如何扩展信息获取能力
- [⚙️ 配置文件系统指南](https://docs.mai-mai.org/develop/plugin_develop/configuration-guide.html) - 学会使用自动生成的插件配置文件
- [📄 Manifest系统指南](https://docs.mai-mai.org/develop/plugin_develop/manifest-guide.html) - 了解插件元数据管理和配置架构

Command vs Action vs Tool选择指南

1. 使用Command的场景

- ✅ 用户需要明确调用特定功能
- ✅ 需要精确的参数控制
- ✅ 管理和配置操作
- ✅ 查询和信息显示

1. 使用Action的场景

- ✅ 增强麦麦的智能行为
- ✅ 根据上下文自动触发
- ✅ 情绪和表情表达
- ✅ 随机化的互动

1. 使用Tool的场景

- 直接为麦麦提供信息
- 在回复时提供更多可用信息
- 简单易用，功能专一

## API浏览

### 消息发送与处理API

- [📤 发送API](https://docs.mai-mai.org/develop/plugin_develop/api/send-api.html) - 各种类型消息发送接口
- [消息API](https://docs.mai-mai.org/develop/plugin_develop/api/message-api.html) - 消息获取，消息构建，消息查询接口
- [聊天流API](https://docs.mai-mai.org/develop/plugin_develop/api/chat-api.html) - 聊天流管理和查询接口

### AI与生成API

- [LLM API](https://docs.mai-mai.org/develop/plugin_develop/api/llm-api.html) - 大语言模型交互接口，可以使用内置LLM生成内容
- [✨ 回复生成器API](https://docs.mai-mai.org/develop/plugin_develop/api/generator-api.html) - 智能回复生成接口，可以使用内置风格化生成器

### 表情包API

- [😊 表情包API](https://docs.mai-mai.org/develop/plugin_develop/api/emoji-api.html) - 表情包选择和管理接口

### 关系系统API

- [人物信息API](https://docs.mai-mai.org/develop/plugin_develop/api/person-api.html) - 用户信息，处理麦麦认识的人和关系的接口

### 数据与配置API

- [🗄️ 数据库API](https://docs.mai-mai.org/develop/plugin_develop/api/database-api.html) - 数据库操作接口
- [⚙️ 配置API](https://docs.mai-mai.org/develop/plugin_develop/api/config-api.html) - 配置读取和用户信息接口

### 插件和组件管理API

- [🔌 插件API](https://docs.mai-mai.org/develop/plugin_develop/api/plugin-manage-api.html) - 插件加载和管理接口
- [🧩 组件API](https://docs.mai-mai.org/develop/plugin_develop/api/component-manage-api.html) - 组件注册和管理接口

### 日志API

- [📜 日志API](https://docs.mai-mai.org/develop/plugin_develop/api/logging-api.html) - logger实例获取接口

### 工具API

- [🔧 工具API](https://docs.mai-mai.org/develop/plugin_develop/api/tool-api.html) - tool获取接口

## 一个方便的小设计

我们在`__init__.py`中定义了一个`__all__`变量，包含了所有需要导出的类和函数。 这样在其他地方导入时，可以直接使用 `from src.plugin_system import *` 来导入所有插件相关的类和函数。 或者你可以直接使用 `from src.plugin_system import BasePlugin, register_plugin, ComponentInfo` 之类的方式来导入你需要的部分。

# 🚀 快速开始指南

本指南将带你从零开始创建一个功能完整的MaiCore插件。

## 📖 概述

这个指南将带你快速创建你的第一个MaiCore插件。我们将创建一个简单的问候插件，展示插件系统的基本概念。

以下代码都在我们的`plugins/hello_world_plugin/`目录下。

### 一个方便的小设计

在开发中，我们在`__init__.py`中定义了一个`__all__`变量，包含了所有需要导出的类和函数。 这样在其他地方导入时，可以直接使用 `from src.plugin_system import *` 来导入所有插件相关的类和函数。 或者你可以直接使用 `from src.plugin_system import BasePlugin, register_plugin, ComponentInfo` 之类的方式来导入你需要的部分。

### 📂 准备工作

确保你已经：

1. 克隆了MaiCore项目
2. 安装了Python依赖
3. 了解基本的Python语法

## 🏗️ 创建插件

### 1. 创建插件目录

在项目根目录的 `plugins/` 文件夹下创建你的插件目录

这里我们创建一个名为 `hello_world_plugin` 的目录

### 2. 创建`_manifest.json`文件

在插件目录下面创建一个 `_manifest.json` 文件，内容如下：



```
{
  "manifest_version": 1,
  "name": "Hello World 插件",
  "version": "1.0.0",
  "description": "一个简单的 Hello World 插件",
  "author": {
    "name": "你的名字"
  }
}
```

有关 `_manifest.json` 的详细说明，请参考 [Manifest文件指南](https://docs.mai-mai.org/develop/plugin_develop/manifest-guide.html)。

### 3. 创建最简单的插件

让我们从最基础的开始！创建 `plugin.py` 文件：



```
from typing import List, Tuple, Type
from src.plugin_system import BasePlugin, register_plugin, ComponentInfo

@register_plugin # 注册插件
class HelloWorldPlugin(BasePlugin):
    """Hello World插件 - 你的第一个MaiCore插件"""

    # 以下是插件基本信息和方法（必须填写）
    plugin_name = "hello_world_plugin"
    enable_plugin = True  # 启用插件
    dependencies = []  # 插件依赖列表（目前为空）
    python_dependencies = []  # Python依赖列表（目前为空）
    config_file_name = "config.toml"  # 配置文件名
    config_schema = {}  # 配置文件模式（目前为空）

    def get_plugin_components(self) -> List[Tuple[ComponentInfo, Type]]: # 获取插件组件
        """返回插件包含的组件列表（目前是空的）"""
        return []
```

🎉 恭喜！你刚刚创建了一个最简单但完整的MaiCore插件！

**解释一下这些代码：**

- 首先，我们在`plugin.py`中定义了一个HelloWorldPlugin插件类，继承自 `BasePlugin` ，提供基本功能。
- 通过给类加上，`@register_plugin` 装饰器，我们告诉系统"这是一个插件"
- `plugin_name` 等是插件的基本信息，必须填写
- `get_plugin_components()` 返回插件的功能组件，现在我们没有定义任何 Action, Command 或者 EventHandler，所以返回空列表。

### 4. 测试基础插件

现在就可以测试这个插件了！启动MaiCore：

直接通过启动器运行MaiCore或者 `python bot.py`

在日志中你应该能看到插件被加载的信息。虽然插件还没有任何功能，但它已经成功运行了！

![1750326700269](https://docs.mai-mai.org/assets/1750326700269.O4MUaJar.png)

### 5. 添加第一个功能：问候Action

现在我们要给插件加入一个有用的功能，我们从最好玩的Action做起

Action是一类可以让MaiCore根据自身意愿选择使用的“动作”，在MaiCore中，不论是“回复”还是“不回复”，或者“发送表情”以及“禁言”等等，都是通过Action实现的。

你可以通过编写动作，来拓展MaiCore的能力，包括发送语音，截图，甚至操作文件，编写代码......

现在让我们给插件添加第一个简单的功能。这个Action可以对用户发送一句问候语。

在 `plugin.py` 文件中添加Action组件，完整代码如下：



```
from typing import List, Tuple, Type
from src.plugin_system import (
    BasePlugin, register_plugin, BaseAction, 
    ComponentInfo, ActionActivationType, ChatMode
)

# ===== Action组件 =====

class HelloAction(BaseAction):
    """问候Action - 简单的问候动作"""

    # === 基本信息（必须填写）===
    action_name = "hello_greeting"
    action_description = "向用户发送问候消息"
    activation_type = ActionActivationType.ALWAYS  # 始终激活

    # === 功能描述（必须填写）===
    action_parameters = {"greeting_message": "要发送的问候消息"}
    action_require = ["需要发送友好问候时使用", "当有人向你问好时使用", "当你遇见没有见过的人时使用"]
    associated_types = ["text"]

    async def execute(self) -> Tuple[bool, str]:
        """执行问候动作 - 这是核心功能"""
        # 发送问候消息
        greeting_message = self.action_data.get("greeting_message", "")
        base_message = self.get_config("greeting.message", "嗨！很开心见到你！😊")
        message = base_message + greeting_message
        await self.send_text(message)

        return True, "发送了问候消息"

@register_plugin
class HelloWorldPlugin(BasePlugin):
    """Hello World插件 - 你的第一个MaiCore插件"""

    # 插件基本信息
    plugin_name = "hello_world_plugin"
    enable_plugin = True
    dependencies = []
    python_dependencies = []
    config_file_name = "config.toml"
    config_schema = {}

    def get_plugin_components(self) -> List[Tuple[ComponentInfo, Type]]:
        """返回插件包含的组件列表"""
        return [
            # 添加我们的问候Action
            (HelloAction.get_action_info(), HelloAction),
        ]
```

**解释一下这些代码：**

- `HelloAction` 是我们定义的问候动作类，继承自 `BaseAction`，并实现了核心功能。
- 在 `HelloWorldPlugin` 中，我们通过 `get_plugin_components()` 方法，通过调用`get_action_info()`这个内置方法将 `HelloAction` 注册为插件的一个组件。
- 这样一来，当插件被加载时，问候动作也会被一并加载，并可以在MaiCore中使用。
- `execute()` 函数是Action的核心，定义了当Action被MaiCore选择后，具体要做什么
- `self.send_text()` 是发送文本消息的便捷方法

Action 组件中有关`activation_type`、`action_parameters`、`action_require`、`associated_types` 等的详细说明请参考 [Action组件指南](https://docs.mai-mai.org/develop/plugin_develop/action-components.html)。

### 6. 测试问候Action

重启MaiCore，然后在聊天中发送任意消息，比如：



```
你好
```

MaiCore可能会选择使用你的问候Action，发送回复：



```
嗨！很开心见到你！😊
```

![1750332508760](https://docs.mai-mai.org/assets/1750332508760.C4j-Rdtf.png)

> **💡 小提示**：MaiCore会智能地决定什么时候使用它。如果没有立即看到效果，多试几次不同的消息。

🎉 太棒了！你的插件已经有实际功能了！

### 7. 添加第二个功能：时间查询Command

现在让我们添加一个Command组件。Command和Action不同，它是直接响应用户命令的：

Command是最简单，最直接的响应，不由LLM判断选择使用



```
# 在现有代码基础上，添加Command组件
import datetime
from src.plugin_system import BaseCommand
#导入Command基类

class TimeCommand(BaseCommand):
    """时间查询Command - 响应/time命令"""

    command_name = "time"
    command_description = "查询当前时间"

    # === 命令设置（必须填写）===
    command_pattern = r"^/time$"  # 精确匹配 "/time" 命令

    async def execute(self) -> Tuple[bool, Optional[str], bool]:
        """执行时间查询"""
        # 获取当前时间
        time_format: str = "%Y-%m-%d %H:%M:%S"
        now = datetime.datetime.now()
        time_str = now.strftime(time_format)

        # 发送时间信息
        message = f"⏰ 当前时间：{time_str}"
        await self.send_text(message)

        return True, f"显示了当前时间: {time_str}", True

@register_plugin
class HelloWorldPlugin(BasePlugin):
    """Hello World插件 - 你的第一个MaiCore插件"""

    # 插件基本信息
    plugin_name = "hello_world_plugin"
    enable_plugin = True
    dependencies = []
    python_dependencies = []
    config_file_name = "config.toml"
    config_schema = {}

    def get_plugin_components(self) -> List[Tuple[ComponentInfo, Type]]:
        return [
            (HelloAction.get_action_info(), HelloAction),
            (TimeCommand.get_command_info(), TimeCommand),
        ]
```

同样的，我们通过 `get_plugin_components()` 方法，通过调用`get_action_info()`这个内置方法将 `TimeCommand` 注册为插件的一个组件。

**Command组件解释：**

- `command_pattern` 使用正则表达式匹配用户输入
- `^/time$` 表示精确匹配 "/time"

有关 Command 组件的更多信息，请参考 [Command组件指南](https://docs.mai-mai.org/develop/plugin_develop/command-components.html)。

### 8. 测试时间查询Command

重启MaiCore，发送命令：



```
/time
```

你应该会收到回复：



```
⏰ 当前时间：2024-01-01 12:00:00
```

🎉 太棒了！现在你已经了解了基本的 Action 和 Command 组件的使用方法。你可以根据自己的需求，继续扩展插件的功能，添加更多的 Action 和 Command 组件，让你的插件更加丰富和强大！

------

## 进阶教程

如果你想让插件更加灵活和强大，可以参考接下来的进阶教程。

### 1. 添加配置文件

想要为插件添加配置文件吗？让我们一起来配置`config_schema`属性！

> **🚨 重要：不要手动创建config.toml文件！**
>
> 我们需要在插件代码中定义配置Schema，让系统自动生成配置文件。

首先，在插件类中定义配置Schema：



```
from src.plugin_system import ConfigField

@register_plugin
class HelloWorldPlugin(BasePlugin):
    """Hello World插件 - 你的第一个MaiCore插件"""

    # 插件基本信息
    plugin_name: str = "hello_world_plugin"  # 内部标识符
    enable_plugin: bool = True
    dependencies: List[str] = []  # 插件依赖列表
    python_dependencies: List[str] = []  # Python包依赖列表
    config_file_name: str = "config.toml"  # 配置文件名

    # 配置Schema定义
    config_schema: dict = {
        "plugin": {
            "name": ConfigField(type=str, default="hello_world_plugin", description="插件名称"),
            "version": ConfigField(type=str, default="1.0.0", description="插件版本"),
            "enabled": ConfigField(type=bool, default=False, description="是否启用插件"),
        },
        "greeting": {
            "message": ConfigField(type=str, default="嗨！很开心见到你！😊", description="默认问候消息"),
            "enable_emoji": ConfigField(type=bool, default=True, description="是否启用表情符号"),
        },
        "time": {"format": ConfigField(type=str, default="%Y-%m-%d %H:%M:%S", description="时间显示格式")},
    }

    def get_plugin_components(self) -> List[Tuple[ComponentInfo, Type]]:
        return [
            (HelloAction.get_action_info(), HelloAction),
            (TimeCommand.get_command_info(), TimeCommand),
        ]
```

这会生成一个如下的 `config.toml` 文件：



```
# hello_world_plugin - 自动生成的配置文件
# 我的第一个MaiCore插件，包含问候功能和时间查询等基础示例

# 插件基本信息
[plugin]

# 插件名称
name = "hello_world_plugin"

# 插件版本
version = "1.0.0"

# 是否启用插件
enabled = false


# 问候功能配置
[greeting]

# 默认问候消息
message = "嗨！很开心见到你！😊"

# 是否启用表情符号
enable_emoji = true


# 时间查询配置
[time]

# 时间显示格式
format = "%Y-%m-%d %H:%M:%S"
```

然后修改Action和Command代码，通过 `get_config()` 方法让它们读取配置（配置的键是命名空间式的）：



```
class HelloAction(BaseAction):
    """问候Action - 简单的问候动作"""

    # === 基本信息（必须填写）===
    action_name = "hello_greeting"
    action_description = "向用户发送问候消息"
    activation_type = ActionActivationType.ALWAYS  # 始终激活

    # === 功能描述（必须填写）===
    action_parameters = {"greeting_message": "要发送的问候消息"}
    action_require = ["需要发送友好问候时使用", "当有人向你问好时使用", "当你遇见没有见过的人时使用"]
    associated_types = ["text"]

    async def execute(self) -> Tuple[bool, str]:
        """执行问候动作 - 这是核心功能"""
        # 发送问候消息
        greeting_message = self.action_data.get("greeting_message", "")
        base_message = self.get_config("greeting.message", "嗨！很开心见到你！😊")
        message = base_message + greeting_message
        await self.send_text(message)

        return True, "发送了问候消息"

class TimeCommand(BaseCommand):
    """时间查询Command - 响应/time命令"""

    command_name = "time"
    command_description = "查询当前时间"

    # === 命令设置（必须填写）===
    command_pattern = r"^/time$"  # 精确匹配 "/time" 命令

    async def execute(self) -> Tuple[bool, str, bool]:
        """执行时间查询"""
        import datetime

        # 获取当前时间
        time_format: str = self.get_config("time.format", "%Y-%m-%d %H:%M:%S")  # type: ignore
        now = datetime.datetime.now()
        time_str = now.strftime(time_format)

        # 发送时间信息
        message = f"⏰ 当前时间：{time_str}"
        await self.send_text(message)

        return True, f"显示了当前时间: {time_str}", True
```

**配置系统工作流程：**

1. **定义Schema**: 在插件代码中定义配置结构
2. **自动生成**: 启动插件时，系统会自动生成 `config.toml` 文件
3. **用户修改**: 用户可以修改生成的配置文件
4. **代码读取**: 使用 `self.get_config()` 读取配置值

**绝对不要手动创建 `config.toml` 文件！**

更详细的配置系统介绍请参考 [配置指南](https://docs.mai-mai.org/develop/plugin_develop/configuration-guide.html)。

### 2. 创建说明文档

你可以创建一个 `README.md` 文件，描述插件的功能和使用方法。

### 3. 发布到插件市场

如果你想让更多人使用你的插件，可以将它发布到MaiCore的插件市场。

这部分请参考 [plugin-repo](https://github.com/Maim-with-u/plugin-repo) 的文档。

------

🎉 恭喜你！你已经成功的创建了自己的插件了！

# 📄 插件Manifest系统指南

## 概述

MaiBot插件系统现在强制要求每个插件都必须包含一个 `_manifest.json` 文件。这个文件描述了插件的基本信息、依赖关系、组件等重要元数据。

### 🔄 配置架构：Manifest与Config的职责分离

为了避免信息重复和提高维护性，我们采用了**双文件架构**：

- **`_manifest.json`** - 插件的**静态元数据**
  - 插件身份信息（名称、版本、描述）
  - 开发者信息（作者、许可证、仓库）
  - 系统信息（兼容性、组件列表、分类）
- **`config.toml`** - 插件的**运行时配置**
  - 启用状态 (`enabled`)
  - 功能参数配置
  - 用户可调整的行为设置

这种分离确保了：

- ✅ 元数据信息统一管理
- ✅ 运行时配置灵活调整
- ✅ 避免重复维护
- ✅ 更清晰的职责划分

## 🔧 Manifest文件结构

### 必需字段

以下字段是必需的，不能为空：



```
{
  "manifest_version": 1,
  "name": "插件显示名称",
  "version": "1.0.0",
  "description": "插件功能描述",
  "author": {
    "name": "作者名称"
  }
}
```

### 可选字段

以下字段都是可选的，可以根据需要添加：



```
{
  "license": "MIT",
  "host_application": {
    "min_version": "1.0.0",
    "max_version": "4.0.0"
  },
  "homepage_url": "https://github.com/your-repo",
  "repository_url": "https://github.com/your-repo",
  "keywords": ["关键词1", "关键词2"],
  "categories": ["分类1", "分类2"],
  "default_locale": "zh-CN",
  "locales_path": "_locales",
  "plugin_info": {
    "is_built_in": false,
    "plugin_type": "general",
    "components": [
      {
        "type": "action",
        "name": "组件名称",
        "description": "组件描述"
      }
    ]
  }
}
```

## 🛠️ 管理工具

### 使用manifest_tool.py

我们提供了一个命令行工具来帮助管理manifest文件：



```
# 扫描缺少manifest的插件
python scripts/manifest_tool.py scan src/plugins

# 为插件创建最小化manifest文件
python scripts/manifest_tool.py create-minimal src/plugins/my_plugin --name "我的插件" --author "作者"

# 为插件创建完整manifest模板
python scripts/manifest_tool.py create-complete src/plugins/my_plugin --name "我的插件"

# 验证manifest文件
python scripts/manifest_tool.py validate src/plugins/my_plugin
```

### 验证示例

验证通过的示例：



```
✅ Manifest文件验证通过
```

验证失败的示例：



```
❌ 验证错误:
  - 缺少必需字段: name
  - 作者信息缺少name字段或为空
⚠️ 验证警告:
  - 建议填写字段: license
  - 建议填写字段: keywords
```

## 🔄 迁移指南

### 对于现有插件

1. **检查缺少manifest的插件**：

   

   ```
   python scripts/manifest_tool.py scan src/plugins
   ```

2. **为每个插件创建manifest**：

   

   ```
   python scripts/manifest_tool.py create-minimal src/plugins/your_plugin
   ```

3. **编辑manifest文件**，填写正确的信息。

4. **验证manifest**：

   

   ```
   python scripts/manifest_tool.py validate src/plugins/your_plugin
   ```

### 对于新插件

创建新插件时，建议的步骤：

1. **创建插件目录和基本文件**

2. 创建完整manifest模板

   ：

   

   ```
   python scripts/manifest_tool.py create-complete src/plugins/new_plugin
   ```

3. **根据实际情况修改manifest文件**

4. **编写插件代码**

5. **验证manifest文件**

## 📋 字段说明

### 基本信息

- `manifest_version`: manifest格式版本，当前为1

- `name`: 插件显示名称（必需）

- `version`: 插件版本号（必需）

- `description`: 插件功能描述（必需）

- ```
  author
  ```

  : 作者信息（必需）

  - `name`: 作者名称（必需）
  - `url`: 作者主页（可选）

### 许可和URL

- `license`: 插件许可证（可选，建议填写）
- `homepage_url`: 插件主页（可选）
- `repository_url`: 源码仓库地址（可选）

### 分类和标签

- `keywords`: 关键词数组（可选，建议填写）
- `categories`: 分类数组（可选，建议填写）

#### 🏷️ Categories分类系统说明

插件系统使用预定义的分类系统来组织和管理插件。`categories` 字段必须使用以下**英文分类标识符**中的一个或多个：

| 英文分类标识符                | 中文显示名称   | 说明                   |
| :---------------------------- | :------------- | :--------------------- |
| `Group Management`            | 群组管理       | 群组相关的管理功能插件 |
| `Entertainment & Interaction` | 娱乐互动       | 娱乐和互动类插件       |
| `Utility Tools`               | 实用工具       | 实用工具类插件         |
| `Content Generation`          | 内容生成       | 内容生成相关插件       |
| `Multimedia`                  | 多媒体         | 多媒体处理插件         |
| `External Integration`        | 外部集成       | 外部服务集成插件       |
| `Data Analysis & Insights`    | 数据分析与洞察 | 数据分析相关插件       |
| `Other`                       | 其他           | 其他未分类插件         |

**示例用法**：



```
{
  "categories": ["Entertainment & Interaction"],
  // 或者多个分类
  "categories": ["Utility Tools", "External Integration"]
}
```

⚠️ **重要提醒**：

- 必须使用表格中的**英文分类标识符**，否则插件无法被正确分类
- 如果不确定分类，建议使用 `Other`
- 系统会自动将英文分类映射为中文显示名称

### 兼容性

- ```
  host_application
  ```

  : 主机应用兼容性（必填）

  - `min_version`: 最低兼容版本（必填）
  - `max_version`: 最高兼容版本（可选，但建议填写，不填写默认到最新版本）

⚠️ 在不填写最高兼容版本的情况下，插件将默认支持从最低版本以上的所有版本。**（由于我们在不同版本对插件系统进行了大量的重构，这种情况几乎不可能。）**

### 国际化

- `default_locale`: 默认语言（可选）
- `locales_path`: 语言文件目录（可选）

### 插件特定信息

- ```
  plugin_info
  ```

  : 插件详细信息（可选）

  - `is_built_in`: 是否为内置插件
  - `plugin_type`: 插件类型
  - `components`: 组件列表

## 示例文件

以下是一个遵循了上述良好实践的 `manifest.json` 文件范例。



```
{
  "manifest_version": 1, # 说明文件版本
  "name": "示例插件", # 插件名称
  "version": "1.0.0", # 插件版本
  "description": "这是一个示例插件", # 插件介绍
  "author": { # 插件作者信息
    "name": "MaiBot 团队", # 插件作者名称
    "url": "https://github.com/MaiM-with-u", # 插件作者主页
  },
  "license": "MIT", # 插件许可版本

  "host_application": {
    "min_version": "0.7.0", # 插件适配麦麦最低版本
    "max_version": "0.8.0" # （可选）插件适配麦麦最高版本
  },
  "homepage_url": "https://github.com/MaiM-with-u", # （可选）插件主页
  "repository_url": "https://github.com/MaiM-with-u/plugin-repo", # （可选）插件仓库地址
  "keywords": ["Example"], # 插件关键词
  "categories": ["Other"], # 插件分类
  
  "default_locale": "CN", # 插件默认语言
  "locales_path": "_locales" # （可选）插件语言文件夹
}
```

## ⚠️ 注意事项

1. **强制要求**：所有插件必须包含`_manifest.json`文件，否则无法加载
2. **编码格式**：manifest文件必须使用UTF-8编码
3. **JSON格式**：文件必须是有效的JSON格式
4. **必需字段**：`manifest_version`、`name`、`version`、`description`、`author.name`是必需的
5. **版本兼容**：当前只支持`manifest_version = 1`

## 🔍 常见问题

### Q: 可以不填写可选字段吗？

A: 可以。所有标记为"可选"的字段都可以不填写，但建议至少填写`license`和`keywords`。

### Q: manifest验证失败怎么办？

A: 根据验证器的错误提示修复相应问题。错误会导致插件加载失败，警告不会。

## 📚 参考示例

查看内置插件的manifest文件作为参考：

- `src/plugins/built_in/core_actions/_manifest.json`
- `src/plugins/built_in/tts_plugin/_manifest.json`
- `src/plugins/hello_world_plugin/_manifest.json`



# Action组件详解

## 📖 什么是Action

Action是给麦麦在回复之外提供额外功能的智能组件，**由麦麦的决策系统自主选择是否使用**，具有随机性和拟人化的调用特点。Action不是直接响应用户命令，而是让麦麦根据聊天情境智能地选择合适的动作，使其行为更加自然和真实。

### Action的特点

- 🧠 **智能激活**：麦麦根据多种条件智能判断是否使用
- 🎲 **可随机性**：可以使用随机数激活，增加行为的不可预测性，更接近真人交流
- 🤖 **拟人化**：让麦麦的回应更自然、更有个性
- 🔄 **情境感知**：基于聊天上下文做出合适的反应

------

## 🎯 Action组件的基本结构

首先，所有的Action都应该继承`BaseAction`类。

其次，每个Action组件都应该实现以下基本信息：



```
class ExampleAction(BaseAction):
    action_name = "example_action" # 动作的唯一标识符
    action_description = "这是一个示例动作，用于测试和展示" # 动作描述
    activation_type = ActionActivationType.ALWAYS # 默认使用 ALWAYS，这个Action一直可用 
    associated_types = ["text", "emoji", ...] # 关联类型,该插件会用到什么类型的数据
    parallel_action = False # 更大概率不与其他动作一起使用
    action_parameters = {"param1": "参数1的说明", "param2": "参数2的说明", ...}
    # Action使用场景描述 - 帮助LLM判断何时"选择"使用
    action_require = ["使用场景描述1", "使用场景描述2", ...]

    async def execute(self) -> Tuple[bool, str]:
        """
        执行Action的主要逻辑
        
        Returns:
            Tuple[bool, str]: (是否成功, 执行结果描述)
        """
        # ---- 执行动作的逻辑 ----
        return True, "执行成功"
```

#### associated_types: 该Action会发送的消息类型，例如文本、表情等。

这部分由Adapter传递给处理器。

以 MaiBot-Napcat-Adapter 为例，可选项目如下：

| 类型     | 说明        | 格式                            |
| :------- | :---------- | :------------------------------ |
| text     | 文本消息    | str                             |
| emoji    | 表情消息    | str: 表情包的无头base64         |
| image    | 图片消息    | str: 图片的无头base64           |
| reply    | 回复消息    | str: 回复的消息ID               |
| voice    | 语音消息    | str: wav格式语音的无头base64    |
| command  | 命令消息    | 参见Adapter文档                 |
| voiceurl | 语音URL消息 | str: wav格式语音的URL           |
| music    | 音乐消息    | str: 这首歌在网易云音乐的音乐id |
| videourl | 视频URL消息 | str: 视频的URL                  |
| file     | 文件消息    | str: 文件的路径                 |

**请知悉，对于不同的处理器，其支持的消息类型可能会有所不同。在开发时请注意。**

#### action_parameters: 该Action的参数说明。

这是一个字典，键为参数名，值为参数说明。这个字段可以帮助LLM理解如何使用这个Action，并由LLM返回对应的参数，最后传递到 Action 的 **`action_data`** 属性中。其格式与你定义的格式完全相同 **（除非LLM哈气了，返回了错误的内容）**。

------

## Action 调用的决策机制

Action采用**两层决策机制**来优化性能和决策质量：

**第一层：激活控制（Activation Control）**

激活决定这个Action是否进入决策候选池。不被激活的Action麦麦永远不会选择。

**第二层：使用决策（Usage Decision）**

在Action被激活后，使用条件决定麦麦什么时候会 **“选择”** 使用这个Action。

### 决策参数详解

#### 第一层：ActivationType 激活类型说明

| 激活类型                                                     | 说明                                 | 使用场景                 |
| :----------------------------------------------------------- | :----------------------------------- | :----------------------- |
| [`NEVER`](https://docs.mai-mai.org/develop/plugin_develop/action-components.html#never-激活) | 从不激活，Action对麦麦不可见         | 临时禁用某个Action       |
| [`ALWAYS`](https://docs.mai-mai.org/develop/plugin_develop/action-components.html#always-激活) | 永远激活，Action总是在麦麦的候选池中 | 核心功能，如回复、不回复 |
| `RANDOM`                                                     | 基于随机概率决定是否激活             | 增加行为随机性的功能     |
| `KEYWORD`                                                    | 当检测到特定关键词时激活             | 明确触发条件的功能       |

#### 激活类型简述

- `NEVER`: 永不激活（动作始终禁用）

  

  ```
  class DisabledAction(BaseAction):
      activation_type = ActionActivationType.NEVER
  ```

- `ALWAYS`: 永远激活（始终可用）

  

  ```
  class AlwaysActivatedAction(BaseAction):
      activation_type = ActionActivationType.ALWAYS
  ```

- `RANDOM`: 随机概率激活，需指定概率

  

  ```
  class SurpriseAction(BaseAction):
      activation_type = ActionActivationType.RANDOM
      random_activation_probability = 0.1  # 10%概率
  ```

- `KEYWORD`: 匹配关键词时激活，需设定关键词列表和是否区分大小写

  

  ```
  class GreetingAction(BaseAction):
      activation_type = ActionActivationType.KEYWORD
      activation_keywords = ["你好", "hello", "hi", "嗨"]
      keyword_case_sensitive = False
  ```

更多KEYWORD类型实例可参考 `plugins/hello_world_plugin` 下的 `ByeAction`。

#### 第二层：使用决策

**在Action被激活后，使用条件决定麦麦什么时候会"选择"使用这个Action**。

这一层由以下因素综合决定：

- `action_require`：使用场景描述，帮助LLM判断何时选择
- `action_parameters`：所需参数，影响Action的可执行性
- 当前聊天上下文和麦麦的决策逻辑

------

### 决策流程示例



```
class EmojiAction(BaseAction):
    # 第一层：激活控制
    activation_type = ActionActivationType.RANDOM  # 随机激活
    random_activation_probability = 0.1  # 10%概率激活

    # 第二层：使用决策
    action_require = [
        "表达情绪时可以选择使用",
        "增加聊天趣味性",
        "不要连续发送多个表情"
    ]
```

**决策流程**：

1. **第一层激活判断**：
   - 使用随机数进行决策，有10%的概率，麦麦"知道"可以使用这个Action
2. **第二层使用决策**：
   - 即使Action被激活，麦麦还会根据 `action_require` 中的条件判断是否真正选择使用
   - 例如：如果刚刚已经发过表情，根据"不要连续发送多个表情"的要求，麦麦可能不会选择这个Action

------

## Action 内置属性说明



```
class BaseAction:
    def __init__(self):
        # 消息相关属性
        self.log_prefix: str          # 日志前缀
        self.group_id: str            # 群组ID
        self.group_name: str          # 群组名称
        self.user_id: str             # 用户ID
        self.user_nickname: str       # 用户昵称
        self.platform: str            # 平台类型 (qq, telegram等)
        self.chat_id: str             # 聊天ID
        self.chat_stream: ChatStream  # 聊天流对象
        self.is_group: bool           # 是否群聊

        # 消息体
        self.action_message: dict     # 消息数据

        # Action相关属性
        self.action_data: dict        # Action执行时的数据
        self.thinking_id: str         # 思考ID
```

action_message为一个字典，包含的键值对如下（省略了不必要的键值对）



```
{
    "message_id": "1234567890",  # 消息id，str
    "time": 1627545600.0,  # 时间戳，float
    "chat_id": "abcdef123456",  # 聊天ID，str
    "reply_to": None,  # 回复消息id，str或None
    "interest_value": 0.85,  # 兴趣值，float
    "is_mentioned": True,  # 是否被提及，bool
    "chat_info_last_active_time": 1627548600.0,  # 最后活跃时间，float
    "processed_plain_text": None,  # 处理后的文本，str或None
    "additional_config": None,  # Adapter传来的additional_config，dict或None
    "is_emoji": False,  # 是否为表情，bool
    "is_picid": False,  # 是否为图片ID，bool
    "is_command": False  # 是否为命令，bool
}
```

部分值的格式请自行查询数据库。

------

## Action 内置方法说明



```
class BaseAction:
    def get_config(self, key: str, default=None):
        """获取插件配置值，使用嵌套键访问"""
    
    async def wait_for_new_message(self, timeout: int = 1200) -> Tuple[bool, str]:
        """等待新消息或超时"""

    async def send_text(self, content: str, reply_to: str = "", reply_to_platform_id: str = "", typing: bool = False) -> bool:
        """发送文本消息"""

    async def send_emoji(self, emoji_base64: str) -> bool:
        """发送表情包"""

    async def send_image(self, image_base64: str) -> bool:
        """发送图片"""

    async def send_custom(self, message_type: str, content: str, typing: bool = False, reply_to: str = "") -> bool:
        """发送自定义类型消息"""

    async def store_action_info(self, action_build_into_prompt: bool = False, action_prompt_display: str = "", action_done: bool = True) -> None:
        """存储动作信息到数据库"""

    async def send_command(self, command_name: str, args: Optional[dict] = None, display_message: str = "", storage_message: bool = True) -> bool:
        """发送命令消息"""
```

具体参数与用法参见`BaseAction`基类的定义。

# 💻 Command组件详解

## 📖 什么是Command

Command是直接响应用户明确指令的组件，与Action不同，Command是**被动触发**的，当用户输入特定格式的命令时立即执行。

Command通过正则表达式匹配用户输入，提供确定性的功能服务。

### 🎯 Command的特点

- 🎯 **确定性执行**：匹配到命令立即执行，无随机性
- ⚡ **即时响应**：用户主动触发，快速响应
- 🔍 **正则匹配**：通过正则表达式精确匹配用户输入
- 🛑 **拦截控制**：可以控制是否阻止消息继续处理
- 📝 **参数解析**：支持从用户输入中提取参数

------

## 🛠️ Command组件的基本结构

首先，Command组件需要继承自`BaseCommand`类，并实现必要的方法。



```
class ExampleCommand(BaseCommand):
    command_name = "example" # 命令名称，作为唯一标识符
    command_description = "这是一个示例命令" # 命令描述
    command_pattern = r"" # 命令匹配的正则表达式

    async def execute(self) -> Tuple[bool, Optional[str], bool]:
        """
        执行Command的主要逻辑

        Returns:
            Tuple[bool, str, bool]: 
                - 第一个bool表示是否成功执行
                - 第二个str是执行结果消息
                - 第三个bool表示是否需要阻止消息继续处理
        """
        # ---- 执行命令的逻辑 ----
        return True, "执行成功", False
```

**`command_pattern`**: 该Command匹配的正则表达式，用于精确匹配用户输入。

请注意：如果希望能获取到命令中的参数，请在正则表达式中使用有命名的捕获组，例如`(?P<param_name>pattern)`。

这样在匹配时，内部实现可以使用`re.match.groupdict()`方法获取到所有捕获组的参数，并以字典的形式存储在`self.matched_groups`中。

### 匹配样例

假设我们有一个命令`/example param1=value1 param2=value2`，对应的正则表达式可以是：



```
class ExampleCommand(BaseCommand):
    command_name = "example"
    command_description = "这是一个示例命令"
    command_pattern = r"/example (?P<param1>\w+) (?P<param2>\w+)"

    async def execute(self) -> Tuple[bool, Optional[str], bool]:
        # 获取匹配的参数
        param1 = self.matched_groups.get("param1")
        param2 = self.matched_groups.get("param2")
        
        # 执行逻辑
        return True, f"参数1: {param1}, 参数2: {param2}", False
```

------

## Command 内置方法说明



```
class BaseCommand:
    def get_config(self, key: str, default=None):
        """获取插件配置值，使用嵌套键访问"""

    async def send_text(self, content: str, reply_to: str = "") -> bool:
        """发送回复消息"""

    async def send_type(self, message_type: str, content: str, display_message: str = "", typing: bool = False, reply_to: str = "") -> bool:
        """发送指定类型的回复消息到当前聊天环境"""

    async def send_command(self, command_name: str, args: Optional[dict] = None, display_message: str = "", storage_message: bool = True) -> bool:
        """发送命令消息"""

    async def send_emoji(self, emoji_base64: str) -> bool:
        """发送表情包"""

    async def send_image(self, image_base64: str) -> bool:
        """发送图片"""
```

具体参数与用法参见`BaseCommand`基类的定义。

# 工具组件详解

## 📖 什么是工具

工具是MaiBot的信息获取能力扩展组件。如果说Action组件功能五花八门，可以拓展麦麦能做的事情，那么Tool就是在某个过程中拓宽了麦麦能够获得的信息量。

### 🎯 工具的特点

- 🔍 **信息获取增强**：扩展麦麦获取外部信息的能力
- 📊 **数据丰富**：帮助麦麦获得更多背景信息和实时数据
- 🔌 **插件式架构**：支持独立开发和注册新工具
- ⚡ **自动发现**：工具会被系统自动识别和注册

### 🆚 Tool vs Action vs Command 区别

| 特征         | Action           | Command      | Tool               |
| :----------- | :--------------- | :----------- | :----------------- |
| **主要用途** | 扩展麦麦行为能力 | 响应用户指令 | 扩展麦麦信息获取   |
| **触发方式** | 麦麦智能决策     | 用户主动触发 | LLM根据需要调用    |
| **目标**     | 让麦麦做更多事情 | 提供具体功能 | 让麦麦知道更多信息 |
| **使用场景** | 增强交互体验     | 功能服务     | 信息查询和分析     |

## 🏗️ Tool组件的基本结构

每个工具必须继承 `BaseTool` 基类并实现以下属性和方法：



```
from src.plugin_system import BaseTool

class MyTool(BaseTool):
    # 工具名称，必须唯一
    name = "my_tool"
    
    # 工具描述，告诉LLM这个工具的用途
    description = "这个工具用于获取特定类型的信息"
    
    # 参数定义，仅定义参数
    # 比如想要定义一个类似下面的openai格式的参数表，则可以这么定义:
    # {
    #     "type": "object",
    #     "properties": {
    #         "query": {
    #             "type": "string",
    #             "description": "查询参数"
    #         },
    #         "limit": {
    #             "type": "integer", 
    #             "description": "结果数量限制"
    #         }
    #     },
    #     "required": ["query"]
    # }
    parameters = [
        ("query", "string", "查询参数", True),  # 必填参数
        ("limit", "integer", "结果数量限制", False)  # 可选参数
    ]

    available_for_llm = True  # 是否对LLM可用
    
    async def execute(self, function_args: Dict[str, Any]):
        """执行工具逻辑"""
        # 实现工具功能
        result = f"查询结果: {function_args.get('query')}"
        
        return {
            "name": self.name,
            "content": result
        }
```

### 属性说明

| 属性          | 类型        | 说明                          |
| :------------ | :---------- | :---------------------------- |
| `name`        | str         | 工具的唯一标识名称            |
| `description` | str         | 工具功能描述，帮助LLM理解用途 |
| `parameters`  | list[tuple] | 参数定义                      |

其构造而成的工具定义为:



```
{"name": cls.name, "description": cls.description, "parameters": cls.parameters}
```

### 方法说明

| 方法      | 参数            | 返回值 | 说明             |
| :-------- | :-------------- | :----- | :--------------- |
| `execute` | `function_args` | `dict` | 执行工具核心逻辑 |

------

## 🎨 完整工具示例

完成一个天气查询工具



```
from src.plugin_system import BaseTool
import aiohttp
import json

class WeatherTool(BaseTool):
    """天气查询工具 - 获取指定城市的实时天气信息"""
    
    name = "weather_query"
    description = "查询指定城市的实时天气信息，包括温度、湿度、天气状况等"
    available_for_llm = True  # 允许LLM调用此工具
    parameters = [
        ("city", "string", "要查询天气的城市名称，如：北京、上海、纽约", True),
        ("country", "string", "国家代码，如：CN、US，可选参数", False)
    ]
    
    async def execute(self, function_args: dict):
        """执行天气查询"""
        try:
            city = function_args.get("city")
            country = function_args.get("country", "")
            
            # 构建查询参数
            location = f"{city},{country}" if country else city
            
            # 调用天气API（示例）
            weather_data = await self._fetch_weather(location)
            
            # 格式化结果
            result = self._format_weather_data(weather_data)
            
            return {
                "name": self.name,
                "content": result
            }
            
        except Exception as e:
            return {
                "name": self.name,
                "content": f"天气查询失败: {str(e)}"
            }
    
    async def _fetch_weather(self, location: str) -> dict:
        """获取天气数据"""
        # 这里是示例，实际需要接入真实的天气API
        api_url = f"http://api.weather.com/v1/current?q={location}"
        
        async with aiohttp.ClientSession() as session:
            async with session.get(api_url) as response:
                return await response.json()
    
    def _format_weather_data(self, data: dict) -> str:
        """格式化天气数据"""
        if not data:
            return "暂无天气数据"
        
        # 提取关键信息
        city = data.get("location", {}).get("name", "未知城市")
        temp = data.get("current", {}).get("temp_c", "未知")
        condition = data.get("current", {}).get("condition", {}).get("text", "未知")
        humidity = data.get("current", {}).get("humidity", "未知")
        
        # 格式化输出
        return f"""
🌤️ {city} 实时天气
━━━━━━━━━━━━━━━━━━
🌡️ 温度: {temp}°C
☁️ 天气: {condition}
💧 湿度: {humidity}%
━━━━━━━━━━━━━━━━━━
        """.strip()
```

------

## 🚨 注意事项和限制

### 当前限制

1. **适用范围**：主要适用于信息获取场景
2. **配置要求**：必须开启工具处理器

### 开发建议

1. **功能专一**：每个工具专注单一功能
2. **参数明确**：清晰定义工具参数和用途
3. **错误处理**：完善的异常处理和错误反馈
4. **性能考虑**：避免长时间阻塞操作
5. **信息准确**：确保获取信息的准确性和时效性

## 🎯 最佳实践

### 1. 工具命名规范

#### ✅ 好的命名

```
name = "weather_query"        # 清晰表达功能
name = "knowledge_search"     # 描述性强
name = "stock_price_check"    # 功能明确
```

#### ❌ 避免的命名

```
name = "tool1"               # 无意义
name = "wq"                  # 过于简短
name = "weather_and_news"    # 功能过于复杂
```

### 2. 描述规范

#### ✅ 良好的描述

```
description = "查询指定城市的实时天气信息，包括温度、湿度、天气状况"
```

#### ❌ 避免的描述

```
description = "天气"         # 过于简单
description = "获取信息"      # 不够具体
```

### 3. 参数设计

#### ✅ 合理的参数设计

```
parameters = [
    ("city", "string", "城市名称，如：北京、上海", True),
    ("unit", "string", "温度单位：celsius 或 fahrenheit", False)
]
```

#### ❌ 避免的参数设计

```
parameters = [
    ("data", "string", "数据", True)  # 参数过于模糊
]
```

### 4. 结果格式化

#### ✅ 良好的结果格式

```
def _format_result(self, data):
    return f"""
🔍 查询结果
━━━━━━━━━━━━
📊 数据: {data['value']}
📅 时间: {data['timestamp']}
📝 说明: {data['description']}
━━━━━━━━━━━━
    """.strip()
```

#### ❌ 避免的结果格式

```
def _format_result(self, data):
    return str(data)  # 直接返回原始数据
```

# ⚙️ 插件配置完整指南

本文档将全面指导你如何为你的插件**定义配置**和在组件中**访问配置**，帮助你构建一个健壮、规范且自带文档的配置系统。

> **🚨 重要原则：任何时候都不要手动创建 config.toml 文件！**
>
> 系统会根据你在代码中定义的 `config_schema` 自动生成配置文件。手动创建配置文件会破坏自动化流程，导致配置不一致、缺失注释和文档等问题。

## 配置版本管理

### 🎯 版本管理概述

插件系统提供了强大的**配置版本管理机制**，可以在插件升级时自动处理配置文件的迁移和更新，确保配置结构始终与代码保持同步。

### 🔄 配置版本管理工作流程

不存在存在无版本有版本匹配不匹配插件加载检查配置文件配置文件存在?生成默认配置读取当前版本有版本信息?跳过版本检查
直接加载配置版本匹配?直接加载配置配置迁移生成新配置结构迁移旧配置值保存迁移后配置配置加载完成

### 📊 版本管理策略

#### 1. 配置版本定义

在 `config_schema` 的 `plugin` 节中定义 `config_version`：



```
config_schema = {
    "plugin": {
        "enabled": ConfigField(type=bool, default=False, description="是否启用插件"),
        "config_version": ConfigField(type=str, default="1.2.0", description="配置文件版本"),
    },
    # 其他配置...
}
```

#### 2. 版本检查行为

- **无版本信息** (`config_version` 不存在)
  - 系统会**跳过版本检查**，直接加载现有配置
  - 适用于旧版本插件的兼容性处理
  - 日志显示：`配置文件无版本信息，跳过版本检查`
- **有版本信息** (存在 `config_version` 字段)
  - 比较当前版本与期望版本
  - 版本不匹配时自动执行配置迁移
  - 版本匹配时直接加载配置

#### 3. 配置迁移过程

当检测到版本不匹配时，系统会：

1. **生成新配置结构** - 根据最新的 `config_schema` 生成新的配置结构
2. **迁移配置值** - 将旧配置文件中的值迁移到新结构中
3. **处理新增字段** - 新增的配置项使用默认值
4. **更新版本号** - `config_version` 字段自动更新为最新版本
5. **保存配置文件** - 迁移后的配置直接覆盖原文件**（不保留备份）**

### 🔧 实际使用示例

#### 版本升级场景

假设你的插件从 v1.0 升级到 v1.1，新增了权限管理功能：

**旧版本配置 (v1.0.0):**



```
[plugin]
enabled = true
config_version = "1.0.0"

[mute]
min_duration = 60
max_duration = 3600
```

**新版本Schema (v1.1.0):**



```
config_schema = {
    "plugin": {
        "enabled": ConfigField(type=bool, default=False, description="是否启用插件"),
        "config_version": ConfigField(type=str, default="1.1.0", description="配置文件版本"),
    },
    "mute": {
        "min_duration": ConfigField(type=int, default=60, description="最短禁言时长（秒）"),
        "max_duration": ConfigField(type=int, default=2592000, description="最长禁言时长（秒）"),
    },
    "permissions": {  # 新增的配置节
        "allowed_users": ConfigField(type=list, default=[], description="允许的用户列表"),
        "allowed_groups": ConfigField(type=list, default=[], description="允许的群组列表"),
    }
}
```

**迁移后配置 (v1.1.0):**



```
[plugin]
enabled = true  # 保留原值
config_version = "1.1.0"  # 自动更新

[mute]
min_duration = 60  # 保留原值
max_duration = 3600  # 保留原值

[permissions]  # 新增节，使用默认值
allowed_users = []
allowed_groups = []
```

### ⚠️ 重要注意事项

#### 版本号管理

- 当你修改 `config_schema` 时，**必须同步更新** `config_version`
- 请使用语义化版本号 (例如：`1.0.0`, `1.1.0`, `2.0.0`)

## 配置定义

配置的定义在你的插件主类（继承自 `BasePlugin`）中完成，主要通过两个类属性：

1. `config_section_descriptions`: 一个字典，用于描述配置文件的各个区段（`[section]`）。
2. `config_schema`: 核心部分，一个嵌套字典，用于定义每个区段下的具体配置项。

### `ConfigField`：配置项的基石

每个配置项都通过一个 `ConfigField` 对象来定义。



```
from dataclasses import dataclass
from src.plugin_system.base.config_types import ConfigField

@dataclass
class ConfigField:
    """配置字段定义"""
    type: type          # 字段类型 (例如 str, int, float, bool, list)
    default: Any        # 默认值
    description: str    # 字段描述 (将作为注释生成到配置文件中)
    example: Optional[str] = None       # 示例值 (可选)
    required: bool = False              # 是否必需 (可选, 主要用于文档提示)
    choices: Optional[List[Any]] = None # 可选值列表 (可选)
```

### 配置示例

让我们以一个功能丰富的 `MutePlugin` 为例，看看如何定义它的配置。



```
# src/plugins/built_in/mute_plugin/plugin.py

from src.plugin_system import BasePlugin, register_plugin, ConfigField
from typing import List, Tuple, Type

@register_plugin
class MutePlugin(BasePlugin):
    """禁言插件"""

    # 这里是插件基本信息，略去

    # 步骤1: 定义配置节的描述
    config_section_descriptions = {
        "plugin": "插件启用配置",
        "components": "组件启用控制",
        "mute": "核心禁言功能配置",
        "smart_mute": "智能禁言Action的专属配置",
        "logging": "日志记录相关配置"
    }

    # 步骤2: 使用ConfigField定义详细的配置Schema
    config_schema = {
        "plugin": {
            "enabled": ConfigField(type=bool, default=False, description="是否启用插件")
        },
        "components": {
            "enable_smart_mute": ConfigField(type=bool, default=True, description="是否启用智能禁言Action"),
            "enable_mute_command": ConfigField(type=bool, default=False, description="是否启用禁言命令Command")
        },
        "mute": {
            "min_duration": ConfigField(type=int, default=60, description="最短禁言时长（秒）"),
            "max_duration": ConfigField(type=int, default=2592000, description="最长禁言时长（秒），默认30天"),
            "templates": ConfigField(
                type=list,
                default=["好的，禁言 {target} {duration}，理由：{reason}", "收到，对 {target} 执行禁言 {duration}"],
                description="成功禁言后发送的随机消息模板"
            )
        },
        "smart_mute": {
            "keyword_sensitivity": ConfigField(
                type=str,
                default="normal",
                description="关键词激活的敏感度",
                choices=["low", "normal", "high"] # 定义可选值
            ),
        },
        "logging": {
            "level": ConfigField(
                type=str,
                default="INFO",
                description="日志记录级别",
                choices=["DEBUG", "INFO", "WARNING", "ERROR"]
            ),
            "prefix": ConfigField(type=str, default="[MutePlugin]", description="日志记录前缀", example="[MyMutePlugin]")
        }
    }

    # 这里是插件方法，略去
```

当 `mute_plugin` 首次加载且其目录中不存在 `config.toml` 时，系统会自动创建以下文件：



```
# mute_plugin - 自动生成的配置文件
# 群聊禁言管理插件，提供智能禁言功能

# 插件启用配置
[plugin]

# 是否启用插件
enabled = false


# 组件启用控制
[components]

# 是否启用智能禁言Action
enable_smart_mute = true

# 是否启用禁言命令Command
enable_mute_command = false


# 核心禁言功能配置
[mute]

# 最短禁言时长（秒）
min_duration = 60

# 最长禁言时长（秒），默认30天
max_duration = 2592000

# 成功禁言后发送的随机消息模板
templates = ["好的，禁言 {target} {duration}，理由：{reason}", "收到，对 {target} 执行禁言 {duration}"]


# 智能禁言Action的专属配置
[smart_mute]

# 关键词激活的敏感度
# 可选值: low, normal, high
keyword_sensitivity = "normal"


# 日志记录相关配置
[logging]

# 日志记录级别
# 可选值: DEBUG, INFO, WARNING, ERROR
level = "INFO"

# 日志记录前缀
# 示例: [MyMutePlugin]
prefix = "[MutePlugin]"
```

------

## 配置访问

如果你想要在你的组件中访问配置，可以通过组件内置的 `get_config()` 方法访问配置。

其参数为一个命名空间化的字符串。以上面的 `MutePlugin` 为例，你可以这样访问配置：



```
enable_smart_mute = self.get_config("components.enable_smart_mute", True)
```

如果尝试访问了一个不存在的配置项，系统会自动返回默认值（你传递的）或者 `None`。

------

## 最佳实践与注意事项

**🚨 核心原则：永远不要手动创建 config.toml 文件！**

1. **🔥 绝不手动创建配置文件**: **任何时候都不要手动创建 `config.toml` 文件**！必须通过在 `plugin.py` 中定义 `config_schema` 让系统自动生成。
   - ❌ **禁止**：`touch config.toml`、手动编写配置文件
   - ✅ **正确**：定义 `config_schema`，启动插件，让系统自动生成
2. **Schema优先**: 所有配置项都必须在 `config_schema` 中声明，包括类型、默认值和描述。
3. **描述清晰**: 为每个 `ConfigField` 和 `config_section_descriptions` 编写清晰、准确的描述。这会直接成为你的插件文档的一部分。
4. **配置文件只供修改**: 自动生成的 `config.toml` 文件只应该被用户**修改**，而不是从零创建。

# 插件依赖管理系统

现在的Python依赖包管理依然存在问题，请保留你的`python_dependencies`属性，等待后续重构。

## 📚 详细教程

### PythonDependency 类详解

`PythonDependency`是依赖声明的核心类：



```
PythonDependency(
    package_name="PIL",          # 导入时的包名
    version=">=11.2.0",          # 版本要求
    optional=False,              # 是否为可选依赖
    description="图像处理库",     # 依赖描述
    install_name="pillow"        # pip安装时的包名（可选）
)
```

#### 参数说明

| 参数           | 类型 | 必需 | 说明                                                         |
| :------------- | :--- | :--- | :----------------------------------------------------------- |
| `package_name` | str  | ✅    | Python导入时使用的包名（如`requests`）                       |
| `version`      | str  | ❌    | 版本要求，使用pip格式（如`>=1.0.0`, `==2.1.3`）              |
| `optional`     | bool | ❌    | 是否为可选依赖，默认`False`                                  |
| `description`  | str  | ❌    | 依赖的用途描述                                               |
| `install_name` | str  | ❌    | pip安装时的包名，默认与`package_name`相同，用于处理安装名称和导入名称不一致的情况 |

#### 版本格式示例



```
# 常用版本格式
PythonDependency("requests", ">=2.25.0")           # 最小版本
PythonDependency("numpy", ">=1.20.0,<2.0.0")       # 版本范围
PythonDependency("pillow", "==8.3.2")              # 精确版本
PythonDependency("scipy", ">=1.7.0,!=1.8.0")       # 排除特定版本
```

# 插件配置 Schema 开发文档

本文档介绍如何使用 MaiBot 的插件配置系统，让你的插件支持 WebUI 可视化配置。

## 概述

MaiBot 的插件配置系统允许插件开发者通过声明式的方式定义配置项，WebUI 会自动根据配置 Schema 生成配置表单。

### 核心组件

| 组件            | 说明                                                  |
| :-------------- | :---------------------------------------------------- |
| `ConfigField`   | 配置字段定义，描述单个配置项的类型、默认值、UI 控件等 |
| `ConfigSection` | 配置节元数据，描述一组相关配置的标题、图标等          |
| `ConfigTab`     | 标签页定义，用于将多个 section 组织到一个标签页       |
| `ConfigLayout`  | 页面布局定义，支持自动布局、标签页布局等              |

## 快速开始

### 基础用法



```
from src.plugin_system.base.base_plugin import BasePlugin
from src.plugin_system.base.config_types import ConfigField

class MyPlugin(BasePlugin):
    plugin_name = "my_plugin"
    enable_plugin = True
    dependencies = []
    python_dependencies = []
    config_file_name = "config.toml"
    
    # 定义配置 Schema
    config_schema = {
        "plugin": {
            "name": ConfigField(
                type=str,
                default="my_plugin",
                description="插件名称",
                required=True
            ),
            "enabled": ConfigField(
                type=bool,
                default=True,
                description="是否启用插件"
            ),
        },
        "api": {
            "key": ConfigField(
                type=str,
                default="",
                description="API 密钥",
                input_type="password",
                placeholder="请输入您的 API 密钥"
            ),
            "timeout": ConfigField(
                type=int,
                default=30,
                description="请求超时时间（秒）",
                min=1,
                max=120
            ),
        }
    }
    
    # 可选：Section 元数据
    config_section_descriptions = {
        "plugin": "插件基本信息",
        "api": "API 配置",
    }
```

## ConfigField 详解

### 构造参数

#### 基础字段（必需）

| 参数          | 类型   | 说明                                                    |
| :------------ | :----- | :------------------------------------------------------ |
| `type`        | `type` | 字段类型：`str`, `int`, `float`, `bool`, `list`, `dict` |
| `default`     | `Any`  | 默认值                                                  |
| `description` | `str`  | 字段描述，也用作默认标签                                |

#### 验证相关

| 参数         | 类型        | 说明                         |
| :----------- | :---------- | :--------------------------- |
| `required`   | `bool`      | 是否必需，默认 `False`       |
| `choices`    | `List[Any]` | 可选值列表，用于下拉选择     |
| `min`        | `float`     | 最小值（数字类型）           |
| `max`        | `float`     | 最大值（数字类型）           |
| `step`       | `float`     | 步进值（数字类型）           |
| `pattern`    | `str`       | 正则验证（字符串类型）       |
| `max_length` | `int`       | 最大长度（字符串类型）       |
| `example`    | `str`       | 示例值，用于生成配置文件注释 |

#### UI 显示控制

| 参数          | 类型   | 说明                               |
| :------------ | :----- | :--------------------------------- |
| `label`       | `str`  | 显示标签，默认使用 `description`   |
| `placeholder` | `str`  | 输入框占位符                       |
| `hint`        | `str`  | 字段下方的提示文字                 |
| `icon`        | `str`  | 字段图标名称                       |
| `hidden`      | `bool` | 是否在 UI 中隐藏，默认 `False`     |
| `disabled`    | `bool` | 是否禁用编辑，默认 `False`         |
| `order`       | `int`  | 排序权重，数字越小越靠前，默认 `0` |

#### 输入控件类型

| 参数         | 类型  | 说明                               |
| :----------- | :---- | :--------------------------------- |
| `input_type` | `str` | 强制指定控件类型，不指定则自动推断 |
| `rows`       | `int` | textarea 行数，默认 `3`            |

支持的 `input_type` 值：

| 值         | 说明                           |
| :--------- | :----------------------------- |
| `text`     | 单行文本输入框                 |
| `password` | 密码输入框（带显示/隐藏切换）  |
| `textarea` | 多行文本域                     |
| `number`   | 数字输入框                     |
| `switch`   | 开关切换（布尔值）             |
| `slider`   | 滑块（需配合 min/max）         |
| `select`   | 下拉选择（需配合 choices）     |
| `list`     | 动态列表编辑器（支持拖拽排序） |
| `color`    | 颜色选择器（计划中）           |
| `code`     | 代码编辑器（计划中）           |
| `file`     | 文件上传（计划中）             |
| `json`     | JSON 编辑器（计划中）          |

#### 分组与条件显示

| 参数            | 类型  | 说明                                 |
| :-------------- | :---- | :----------------------------------- |
| `group`         | `str` | 字段分组，在 section 内再细分        |
| `depends_on`    | `str` | 依赖的字段路径，如 `"section.field"` |
| `depends_value` | `Any` | 依赖字段需要的值                     |

#### 列表类型专用

| 参数          | 类型             | 说明                                             |
| :------------ | :--------------- | :----------------------------------------------- |
| `item_type`   | `str`            | 数组元素类型：`"string"`, `"number"`, `"object"` |
| `item_fields` | `Dict[str, Any]` | 当 `item_type="object"` 时，定义对象的字段结构   |
| `min_items`   | `int`            | 数组最小元素数量                                 |
| `max_items`   | `int`            | 数组最大元素数量                                 |

**列表类型示例：**



```
# 简单字符串列表
"keywords": ConfigField(
    type=list,
    default=["关键词1", "关键词2"],
    description="触发关键词",
    item_type="string",
    placeholder="输入关键词",
    max_items=20,
    hint="最多支持 20 个关键词"
),

# 数字列表
"retry_delays": ConfigField(
    type=list,
    default=[1, 2, 5, 10],
    description="重试延迟（秒）",
    item_type="number",
    min_items=1,
    max_items=5,
),

# 对象列表（复杂结构）
"api_endpoints": ConfigField(
    type=list,
    default=[{"name": "主服务器", "url": "https://api.example.com"}],
    description="API 端点列表",
    item_type="object",
    item_fields={
        "name": {"type": "string", "label": "名称", "placeholder": "端点名称"},
        "url": {"type": "string", "label": "URL", "placeholder": "https://..."},
        "priority": {"type": "number", "label": "优先级", "default": 0},
    },
    min_items=1,
    max_items=5,
),
```

### 控件自动推断规则

如果不指定 `input_type`，系统会根据以下规则自动选择控件：

| 条件                                 | 控件类型               |
| :----------------------------------- | :--------------------- |
| `type=bool`                          | Switch 开关            |
| `type=int/float` + 有 `min` 和 `max` | Slider 滑块            |
| `type=int/float`                     | NumberInput 数字输入框 |
| `type=str` + 有 `choices`            | Select 下拉选择        |
| `type=str` + `input_type="password"` | Password 密码框        |
| `type=str` + `input_type="textarea"` | Textarea 文本域        |
| `type=list`                          | DynamicList 动态列表   |
| `type=dict`                          | JSON 编辑器            |
| `type=str`（默认）                   | Input 文本框           |

### 示例



```
config_schema = {
    "api": {
        # 密码输入框
        "api_key": ConfigField(
            type=str,
            default="",
            description="API 密钥",
            input_type="password",
            placeholder="sk-xxxxxxxx",
            required=True,
            hint="从服务商控制台获取"
        ),
        
        # 带范围限制的滑块
        "temperature": ConfigField(
            type=float,
            default=0.7,
            description="生成温度",
            min=0.0,
            max=2.0,
            step=0.1,
            hint="较高的值会使输出更随机"
        ),
        
        # 下拉选择
        "model": ConfigField(
            type=str,
            default="gpt-4",
            description="使用的模型",
            choices=["gpt-3.5-turbo", "gpt-4", "gpt-4-turbo"],
            icon="cpu"
        ),
        
        # 多行文本
        "system_prompt": ConfigField(
            type=str,
            default="",
            description="系统提示词",
            input_type="textarea",
            rows=5,
            placeholder="输入系统提示词..."
        ),
        
        # 条件显示：只在 debug 为 True 时显示
        "debug_log_path": ConfigField(
            type=str,
            default="./debug.log",
            description="调试日志路径",
            depends_on="debug.enabled",
            depends_value=True
        ),
    }
}
```

## ConfigSection 详解

用于为配置节添加元数据，如标题、图标、是否默认折叠等。

### 方式一：简单字符串



```
config_section_descriptions = {
    "plugin": "插件基本信息",
    "api": "API 配置",
}
```

### 方式二：使用 ConfigSection



```
from src.plugin_system.base.config_types import ConfigSection

config_section_descriptions = {
    "plugin": ConfigSection(
        title="插件基本信息",
        description="配置插件的基本属性",
        icon="settings",
        order=1
    ),
    "api": ConfigSection(
        title="API 配置",
        description="外部 API 的连接参数",
        icon="cloud",
        order=2
    ),
    "advanced": ConfigSection(
        title="高级设置",
        icon="code",
        collapsed=True,  # 默认折叠
        order=99
    ),
}
```

### 方式三：使用便捷函数



```
from src.plugin_system.base.config_types import section_meta

config_section_descriptions = {
    "plugin": section_meta("插件基本信息", icon="settings", order=1),
    "api": section_meta("API 配置", icon="cloud", order=2),
    "advanced": section_meta("高级设置", collapsed=True, order=99),
}
```

### ConfigSection 参数

| 参数          | 类型   | 默认值  | 说明         |
| :------------ | :----- | :------ | :----------- |
| `title`       | `str`  | 必需    | 显示标题     |
| `description` | `str`  | `None`  | 详细描述     |
| `icon`        | `str`  | `None`  | 图标名称     |
| `collapsed`   | `bool` | `False` | 默认是否折叠 |
| `order`       | `int`  | `0`     | 排序权重     |

## 自定义布局

默认情况下，所有 section 会以可折叠面板的形式展示。如果需要更复杂的布局，可以使用 `config_layout`。

### 标签页布局



```
from src.plugin_system.base.config_types import ConfigLayout, ConfigTab

class MyPlugin(BasePlugin):
    # ... 其他属性 ...
    
    config_schema = {
        "plugin": { ... },
        "api": { ... },
        "model": { ... },
        "debug": { ... },
        "logging": { ... },
    }
    
    config_layout = ConfigLayout(
        type="tabs",
        tabs=[
            ConfigTab(
                id="basic",
                title="基础设置",
                sections=["plugin", "api"],
                icon="settings"
            ),
            ConfigTab(
                id="model",
                title="模型配置",
                sections=["model"],
                icon="cpu"
            ),
            ConfigTab(
                id="dev",
                title="开发者",
                sections=["debug", "logging"],
                icon="terminal",
                badge="Dev"  # 显示角标
            ),
        ]
    )
```

### ConfigTab 参数

| 参数       | 类型        | 默认值 | 说明                    |
| :--------- | :---------- | :----- | :---------------------- |
| `id`       | `str`       | 必需   | 标签页 ID               |
| `title`    | `str`       | 必需   | 显示标题                |
| `sections` | `List[str]` | `[]`   | 包含的 section 名称列表 |
| `icon`     | `str`       | `None` | 图标名称                |
| `order`    | `int`       | `0`    | 排序权重                |
| `badge`    | `str`       | `None` | 角标文字                |

### ConfigLayout 参数

| 参数   | 类型              | 默认值   | 说明                              |
| :----- | :---------------- | :------- | :-------------------------------- |
| `type` | `str`             | `"auto"` | 布局类型：`auto`、`tabs`、`pages` |
| `tabs` | `List[ConfigTab]` | `[]`     | 标签页列表                        |

布局类型说明：

| 类型    | 说明                                    |
| :------ | :-------------------------------------- |
| `auto`  | 自动布局，section 以可折叠面板显示      |
| `tabs`  | 标签页布局，顶部标签切换                |
| `pages` | 分页布局，左侧导航 + 右侧内容（计划中） |

## 完整示例



```
from src.plugin_system.base.base_plugin import BasePlugin
from src.plugin_system.base.config_types import (
    ConfigField,
    ConfigSection,
    ConfigLayout,
    ConfigTab,
)
from src.plugin_system.apis.plugin_register_api import register_plugin


@register_plugin
class TTSPlugin(BasePlugin):
    """TTS 语音合成插件"""
    
    plugin_name = "tts_plugin"
    enable_plugin = True
    dependencies = []
    python_dependencies = []
    config_file_name = "config.toml"
    
    # 配置 Schema
    config_schema = {
        "plugin": {
            "name": ConfigField(
                type=str,
                default="tts_plugin",
                description="插件名称",
                required=True
            ),
            "version": ConfigField(
                type=str,
                default="1.0.0",
                description="插件版本"
            ),
            "enabled": ConfigField(
                type=bool,
                default=True,
                description="是否启用插件"
            ),
        },
        "tts": {
            "provider": ConfigField(
                type=str,
                default="azure",
                description="TTS 服务提供商",
                choices=["azure", "google", "local"],
                order=1
            ),
            "api_key": ConfigField(
                type=str,
                default="",
                description="API 密钥",
                input_type="password",
                required=True,
                order=2
            ),
            "voice": ConfigField(
                type=str,
                default="zh-CN-XiaoxiaoNeural",
                description="语音角色",
                placeholder="例如: zh-CN-XiaoxiaoNeural",
                order=3
            ),
            "speed": ConfigField(
                type=float,
                default=1.0,
                description="语速",
                min=0.5,
                max=2.0,
                step=0.1,
                order=4
            ),
        },
        "cache": {
            "enabled": ConfigField(
                type=bool,
                default=True,
                description="启用缓存"
            ),
            "max_size": ConfigField(
                type=int,
                default=100,
                description="最大缓存数量",
                min=10,
                max=1000,
                depends_on="cache.enabled",
                depends_value=True
            ),
            "ttl": ConfigField(
                type=int,
                default=3600,
                description="缓存过期时间（秒）",
                min=60,
                depends_on="cache.enabled",
                depends_value=True
            ),
        },
        "debug": {
            "enabled": ConfigField(
                type=bool,
                default=False,
                description="调试模式"
            ),
            "log_level": ConfigField(
                type=str,
                default="INFO",
                description="日志级别",
                choices=["DEBUG", "INFO", "WARNING", "ERROR"]
            ),
        },
    }
    
    # Section 元数据
    config_section_descriptions = {
        "plugin": ConfigSection(
            title="插件信息",
            icon="info",
            order=1
        ),
        "tts": ConfigSection(
            title="TTS 配置",
            description="语音合成相关设置",
            icon="volume-2",
            order=2
        ),
        "cache": ConfigSection(
            title="缓存设置",
            icon="database",
            order=3
        ),
        "debug": ConfigSection(
            title="调试选项",
            icon="bug",
            collapsed=True,
            order=99
        ),
    }
    
    # 自定义布局（可选）
    config_layout = ConfigLayout(
        type="tabs",
        tabs=[
            ConfigTab(id="main", title="主要设置", sections=["plugin", "tts"]),
            ConfigTab(id="advanced", title="高级", sections=["cache", "debug"]),
        ]
    )
    
    # ... 插件其他实现 ...
```

## 在插件中读取配置



```
class MyPlugin(BasePlugin):
    def some_method(self):
        # 读取单个配置项
        api_key = self.get_config("api.key")
        timeout = self.get_config("api.timeout", default=30)
        
        # 读取整个 section
        api_config = self.config.get("api", {})
        
        # 检查是否启用
        if self.get_config("plugin.enabled", True):
            # 执行逻辑
            pass
```

## WebUI API

WebUI 提供以下 API 用于配置管理：

| 端点                                           | 方法 | 说明                |
| :--------------------------------------------- | :--- | :------------------ |
| `/api/webui/plugins/config/{plugin_id}/schema` | GET  | 获取插件配置 Schema |
| `/api/webui/plugins/config/{plugin_id}`        | GET  | 获取当前配置值      |
| `/api/webui/plugins/config/{plugin_id}`        | PUT  | 更新配置            |
| `/api/webui/plugins/config/{plugin_id}/reset`  | POST | 重置为默认配置      |
| `/api/webui/plugins/config/{plugin_id}/toggle` | POST | 切换启用/禁用状态   |

## 最佳实践

### 1. 合理组织 Section

将相关的配置项放在同一个 section 中，使用有意义的 section 名称。



```
# 好的做法
config_schema = {
    "api": { ... },      # API 相关
    "model": { ... },    # 模型相关
    "cache": { ... },    # 缓存相关
}

# 避免
config_schema = {
    "settings": { ... },  # 太笼统
    "misc": { ... },      # 不清晰
}
```

### 2. 提供有意义的描述和提示



```
ConfigField(
    type=str,
    default="",
    description="OpenAI API 密钥",  # 清晰的描述
    hint="从 platform.openai.com 获取",  # 有帮助的提示
    placeholder="sk-..."  # 格式示例
)
```

### 3. 使用合适的控件类型



```
# 敏感信息用密码框
"api_key": ConfigField(type=str, input_type="password", ...)

# 有限选项用下拉
"model": ConfigField(type=str, choices=["gpt-3.5", "gpt-4"], ...)

# 范围值用滑块
"temperature": ConfigField(type=float, min=0, max=2, ...)

# 长文本用 textarea
"prompt": ConfigField(type=str, input_type="textarea", rows=5, ...)
```

### 4. 设置合理的默认值



```
ConfigField(
    type=int,
    default=30,  # 合理的默认值
    min=1,
    max=120,
    description="超时时间（秒）"
)
```

### 5. 使用条件显示减少界面复杂度



```
"debug_enabled": ConfigField(type=bool, default=False, ...),
"debug_log_path": ConfigField(
    type=str,
    depends_on="debug.debug_enabled",
    depends_value=True,
    ...
)
```

### 6. 使用 order 控制显示顺序



```
"important_field": ConfigField(..., order=1),
"less_important": ConfigField(..., order=10),
"rarely_used": ConfigField(..., order=99),
```

## 迁移指南

如果你的插件使用旧版 `config_schema`，升级非常简单：

### 旧版



```
config_schema = {
    "api": {
        "key": ConfigField(type=str, default="", description="API密钥"),
    }
}
```

### 新版（完全兼容）

旧版代码无需修改即可继续工作。如需使用新功能，只需添加新参数：



```
config_schema = {
    "api": {
        "key": ConfigField(
            type=str,
            default="",
            description="API密钥",
            # 新增的参数
            input_type="password",
            placeholder="请输入 API 密钥",
            hint="从控制台获取",
            required=True,
            order=1
        ),
    }
}
```

## 常见问题

### Q: 配置修改后何时生效？

A: 配置保存到 TOML 文件后，需要重新加载插件才能生效。WebUI 会显示提示信息。

### Q: 如何验证配置值？

A: 目前支持基本验证（必填、范围、正则），复杂验证逻辑需要在插件代码中实现。

### Q: 支持哪些图标？

A: 图标使用 Lucide Icons，参考 https://lucide.dev/icons

### Q: 如何处理敏感配置？

A: 使用 `input_type="password"` 会在 UI 中隐藏输入内容，但配置文件中仍是明文存储。如需加密存储，需要自行实现。

