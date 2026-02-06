LangChain 学习

基于 `1.2.8` 版本

## 安装

[LangChain Python 集成包](https://docs.langchain.com/oss/python/integrations/providers/overview)

```shell
pip install -U langchain
pip install -U langchain-google-genai
pip install -U langchain-openai
```

## 常见例子

### 创建一个 Agent

```python
# -*- coding: utf-8 -*-
import os

from langchain.tools import tool  
from langchain.agents import create_agent  
from langchain_google_genai import ChatGoogleGenerativeAI  
  
  
@tool  
def get_weather(city: str) -> str:  
    """  
    获取指定城市的天气  
    Args:        city: 要获取的城市名  
    """    match city:  
        case "北京":  
            return f"{city}今天有暴雨"  
        case "上海":  
            return f"{city}今天天气晴朗"  
        case "广州":  
            return f"{city}今天有雾霾"  
        case _:  
            return f"{city}今天天气不错"  
  
  
tools = [  
    get_weather,  
]
  
api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)
  
agent = create_agent(  
    model=model,  
    tools=tools,  
    system_prompt="你是一个生活小助手"  
)  

messages = {"messages": [{"role": "user", "content": "北京的天气怎么样"}]} 
  
result = agent.invoke(messages)  
  
print(result)
```

### 创建模型

```python
# -*- coding: utf-8 -*-
import os  
  
from langchain.agents import create_agent  
from langchain_core.messages import SystemMessage, HumanMessage  
from langchain_google_genai import ChatGoogleGenerativeAI  
from langchain_openai import ChatOpenAI  
  
api_key = os.environ.get("GEMINI_API_KEY")  

# 1.直接使用特定供应商库  
genai_model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)  

# 2.大多数大模型厂商兼容 OpenAI 接口  
openai_model = ChatOpenAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/"  
)  
  
message = [  
    SystemMessage("你是一个生活小助手"),  
    HumanMessage("北京今天的天气怎么样")  
]  
  
print(genai_model.invoke(message))  
print(openai_model.invoke(message))

  
agent = create_agent(  
    model=genai_model,  
)
messages = {"messages": message} 
print(agent.invoke(messages))
```

### 提示词模版

```python
# -*- coding: utf-8 -*-
import os

from langchain_google_genai import ChatGoogleGenerativeAI  
from langchain_core.prompts import ChatPromptTemplate 


api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)
  
prompt = ChatPromptTemplate(  
    [  
        ("system", "你是一个生活小助手"),  
        ("user", "{city}今天的天气怎么样")  
    ]  
)  
  
prompt = prompt.format(city="北京")  
  
result = model.invoke(prompt)  
print(result)
```

### 消息模版

```python
# -*- coding: utf-8 -*-
import os

from langchain_google_genai import ChatGoogleGenerativeAI  
from langchain_core.messages import SystemMessage, HumanMessage

api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)

message = [  
    SystemMessage("你是一个生活小助手"),  
    HumanMessage("北京今天的天气怎么样")  
]  
  
result = model.invoke(message)  
print(result)
```

### 格式化输出

```python
# -*- coding: utf-8 -*-
import os

from langchain_google_genai import ChatGoogleGenerativeAI  
from langchain_core.output_parsers import StrOutputParser

api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)

prompt = ChatPromptTemplate(  
    [  
        ("system", "你是一个生活小助手"),  
        ("user", "{city}今天的天气怎么样")  
    ]  
)  
  
prompt = prompt.format(city="北京")  
  
result = model.invoke(prompt)  
str_parser = StrOutputParser()  
str_result = str_parser.invoke(result)
print(str_result)
```

### 结构化输出

```python
# -*- coding: utf-8 -*-
import os

from langchain_google_genai import ChatGoogleGenerativeAI  
from langchain.agents.structured_output import ToolStrategy
from pydantic import BaseModel

api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)

class UserInfo(BaseModel):  
    name: str  
    email: str  
    phone: str  
  
agent = create_agent(  
    model=model, 
    response_format=ToolStrategy(UserInfo)  
)  
  
messages = {  
    "messages": [  
        {  
            "role": "user",  
            "content": "从 张三 zhangsan@qq.com  18800000000 提取用户信息"  
        }  
    ]  
}  
  
result = agent.invoke(messages)  
print(result["structured_response"])
```

### 链语法

```python
# -*- coding: utf-8 -*-
import os

from langchain_google_genai import ChatGoogleGenerativeAI  
from langchain_core.prompts import ChatPromptTemplate

api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)

prompt = ChatPromptTemplate(  
    [  
        ("system", "你是一个生活小助手"),  
        ("user", "{city}今天的天气怎么样")  
    ]  
)  
  
str_parser = StrOutputParser()  
_chain = prompt | model | str_parser  
result = _chain.invoke({"city": "北京"})  
print(result)
```

### 工具调用


```python
# -*- coding: utf-8 -*-
import os

from langchain.tools import tool  
from langchain.agents import create_agent  
from langchain_google_genai import ChatGoogleGenerativeAI  
from pydantic import BaseModel, Field
from typing import Literal

class InventoryInput(BaseModel):  
    region: str = Field(description="仓库位置")  
    goods: str = Field(description="要查询的商品")  
    units: Literal["个", "条"] = Field(  
        default="个",  
        description="仓库商品库存单位"  
    )  
  
  
@tool(name_or_callable="get_inventory", description="获取仓库的商品库存", args_schema=InventoryInput)  
def get_inventory(region: str, goods: str, units: str = "个") -> str:  
    inventory = f"{region}{goods}当前库存是100{units}"  
    return inventory  
  
  
print(get_inventory.name, get_inventory.description)  
  
tools = [  
    get_inventory,  
]  
  
api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)
  
agent = create_agent(  
    model=model,  
    tools=tools  
)  

messages = {"messages": [{"role": "user", "content": "北京仓的牛仔裤还有多少个"}]} 
  
result = agent.invoke(messages)
  
print(result)
```

### 中间件

#### 自动总结摘要

```python
# -*- coding: utf-8 -*-
import os  
  
from langchain.agents import create_agent  
from langchain.agents.middleware import SummarizationMiddleware  
from langchain_google_genai import ChatGoogleGenerativeAI  
from langgraph.checkpoint.memory import InMemorySaver  
  
api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)  
  
# 多轮会话自动总结摘要  
summarizationMiddleware = SummarizationMiddleware(  
    model=model,  
    trigger=("tokens", 4000),  
    keep=("messages", 20)  
)  
  
checkpointer = InMemorySaver()  
  
config = {"configurable": {"thread_id": "1"}}  
  
agent = create_agent(  
    model=model,  
    middleware=[summarizationMiddleware],  
    checkpointer=checkpointer  
)  
  
messages = {"messages": [{"role": "user", "content": "最近有什么新闻"}]}  
  
for _ in range(5):  
    result = agent.invoke(messages, config=config)  
  
    print(result)
```

#### 动态提示词

```python
# -*- coding: utf-8 -*-
import os  
from typing import TypedDict  
  
from langchain.agents import create_agent  
from langchain.agents.middleware import ModelRequest, dynamic_prompt  
from langchain_google_genai import ChatGoogleGenerativeAI  
  
api_key = os.environ.get("GEMINI_API_KEY")  
  
model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)  
  
  
class Context(TypedDict, total=False):  
    user_role: str  
   
@dynamic_prompt  
def user_role_prompt(request: ModelRequest) -> str:
    """模型请求中间件，根据角色设置不同的系统提示词"""
    user_role = request.runtime.context.get("user_role", "user")  
    base_prompt = "你是一个助手"  
  
    if user_role == "expert":  
        return f"{base_prompt} 提供专业的回答"  
    if user_role == "beginner":  
        return f"{base_prompt} 提供简单的回答"  
    return base_prompt  
  
  
agent = create_agent(  
    model=model,  
    middleware=[user_role_prompt],  
    context_schema=Context  
)  
  
messages = {  
    "messages": [  
        {"role": "user", "content": "解释一下什么是机器学习"},  
    ]  
}  
  
context = {"user_role": "beginner"}  
  
result = agent.invoke(messages, context=context)  
  
print(result)
```

#### 动态模型

```python
# -*- coding: utf-8 -*-
import os  
  
from langchain.agents import create_agent  
from langchain.agents.middleware import ModelRequest, dynamic_prompt, wrap_model_call, ModelResponse  
from langchain_google_genai import ChatGoogleGenerativeAI  
from langgraph.checkpoint.memory import InMemorySaver  
  
api_key = os.environ.get("GEMINI_API_KEY")  
  
base_model = ChatGoogleGenerativeAI(  
    model="gemini-2.5-flash-lite",  
    api_key=api_key,  
)  
  
  
@wrap_model_call  
def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:  
    """根据会话的复杂度选择模型"""  
    message_count = len(request.state["messages"])  
    model = base_model  
    if message_count > 5:  
        model = ChatGoogleGenerativeAI(  
            model="gemini-2.5-flash",  
            api_key=api_key,  
        )  
    return handler(request.override(model=model))  
  
  
checkpointer = InMemorySaver()  
  
config = {"configurable": {"thread_id": "1"}}  
  
agent = create_agent(  
    model=base_model,  
    middleware=[dynamic_model_selection],  
)  
  
messages = {  
    "messages": [  
        {"role": "user", "content": "解释一下什么是机器学习"},  
    ]  
}  
for _, in range(5):  
    result = agent.invoke(messages, config=config)  
  
    print(result)
```

#### 处理中断

```python
# -*- coding: utf-8 -*-
import os

from langgraph.types import Command
from langchain_core.tools import tool
from langchain_google_genai import ChatGoogleGenerativeAI
from langgraph.checkpoint.memory import MemorySaver
from deepagents import create_deep_agent


@tool
def delete_file(file_path: str) -> str:
    """删除一个文件"""
    return f"{file_path} 已删除"


@tool
def read_file(file_path: str) -> str:
    """读取一个文件"""
    return f"{file_path} 文件内容是"


@tool
def send_email(to: str, subject: str) -> str:
    """
    发送邮件
    Args:
        to: 邮箱地址
        subject: 邮件主题
    """
    return f"已经成功给{to}发送主题为{subject}邮件"


api_key = os.environ.get("GEMINI_API_KEY")

model = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash-lite",
    api_key=api_key,
)

tools = [delete_file, read_file, send_email]

# 设置终中断配置
interrupt_on = {
    "delete_file": True,
    "read_file": False,
    # 自定义批准选项
    "send_email": {"allowed_decisions": ["approve", "reject"]}
}

checkpoint = MemorySaver()

config = {"configurable": {"thread_id": "1"}}

agent = create_deep_agent(
    model=model,
    tools=tools,
    interrupt_on=interrupt_on,
    checkpointer=checkpoint
)

messages = {"messages": [{"role": "user", "content": "删除文件 text.log"}]}
# messages = {"messages": [{"role": "user", "content": "给 zhangsan@qq.com 发送一封主题为请求批准的邮件"}]}

result = agent.invoke(messages, config=config)

print(result)

# 处理中断信息
if result.get("__interrupt__"):
    interrupts = result.get("__interrupt__")[0].value
    print(interrupts)
    action_requests = interrupts["action_requests"]
    review_configs = interrupts["review_configs"]

    config_map = {cfg["action_name"]: cfg for cfg in review_configs}

    for action in action_requests:
        review_config = config_map[action["name"]]
        print(f"review_configs={review_configs}")
        print(f"tool={action['name']}")
        print(f"args={action['args']}")
        print(f"description={action['description']}")

        # 批准
        decisions = [
            {"type": "approve"}
        ]

        result = agent.invoke(Command(resume={"decisions": decisions}), config=config)

        print(result)
```