## Agent

大模型是无法感知或改变外接环境的   

之前阶段的模型只能 输入文字 -> 输出文字，但现实任务往往需要行动:   

- 查实时天气得联网
- 算数得用计算器
- 查你的文档得检索
- 发邮件得调邮件接口

但模型自己做不了这些。  

Agent = 大模型(大脑) + 工具(手脚) + 循环决策(自主性)

你给Agent一个目标，它会自己思考需要哪些步骤、每一步调用什么工具、根据结果决定下一步，直到完成任务。   


一个类比:    

- 纯模型 = 一个被关在房间里、只能动嘴的顾问，你问什么他答什么，但他不能起身做任何事。  

- Agent = 给这个顾问配了电脑、电话、双手，并告诉他你可以自己用这些工具去把事做成。  


#### 基石: Tool / Function Calling (函数调用)

这是Agent的核心机制，它让模型能请求调用你写的函数。   

关键要理解一件反直觉的事: 模型本身不能执行任何代码。  

它能做的是 -- 告诉你 我想调用get_weather这个函数，参数是city = 北京，真正执行函数的是你的程序，执行完把结果再喂回给模型。  

完整流程:   


1. 你告诉模型：我有这些工具可用（附上每个工具的名字、功能、参数说明）
2. 用户提问："北京今天天气怎么样？"
3. 模型判断：这需要调 get_weather 工具 → 返回"请调用 get_weather(city='北京')"
4. 【你的程序】真正执行 get_weather("北京") → 得到"晴，25℃"
5. 你把结果喂回给模型
6. 模型基于结果，生成人话回复："北京今天晴，25 度，很适合出门。"


Android功能类比： 

像极了系统权限回调 -- App(模型)不能直接读取通讯录，它得请求这个能力，由系统(你的程序)实际执行后把结果回调给它。 模型请求调用工具，你的代码实际执行并回传。   

#### 代码实现

第一步: 定义工具函数(就是普通Python函数)

```python
def get_weather(city: str) -> str: 
    return f"{city}今天晴，25℃"

def calculate(expression: str) -> str: 
    return str(eval(expression)) 
```

第二步: 用模型能理解的格式描述这些工具
```json
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "查询指定城市的当前天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "城市名，如 北京"}
                },
                "required": ["city"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "计算一个数学表达式",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string", "description": "如 2 * (3 + 4)"}
                },
                "required": ["expression"],
            },
        },
    },
]
```

这上面的description极其重要 -- 模型就是靠它判断该不该用、怎么用这个工具。  

如果描述写的含糊，模型就会用错或不用。把它当成写给模型看的API文档。  

第三步: 把工具告诉模型，并处理它的调用请求

```python
import json

def run_agent(user_input):
    messages = [{"role": "user", "content": user_input}]

    while True:  # 这个循环是 Agent 的灵魂，见 3.3
        response = client.chat.completions.create(
            model="模型名称",
            messages=messages,
            tools=tools,   # 告诉模型有哪些工具
        )
        msg = response.choices[0].message
        messages.append(msg)

        # 模型没有要求调工具 → 说明它已能直接回答，结束
        if not msg.tool_calls:
            return msg.content

        # 模型要求调用一个或多个工具
        for tool_call in msg.tool_calls:
            name = tool_call.function.name
            args = json.loads(tool_call.function.arguments)

            # 【你的程序】真正执行对应函数
            if name == "get_weather":
                result = get_weather(**args)
            elif name == "calculate":
                result = calculate(**args)

            # 把执行结果喂回给模型
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result,
            })
        # 循环回到开头，模型拿到工具结果后决定下一步
```

上面代码里面while True就是Agent区别于普通一问一答的关键。  




那怎么来解决这个问题呢？    

就是通过让大模型调用对应的工具Tool，就可以，例如:    

- 读写文件内容
- 查看文件列表
- 运行终端命令


### Agent运行模式  



##### ReAct模式

思考与行动:  Reasoning and Acting

ReAct = Reasoning(推理) + Acting(行动)的循环

想(Reason) : 我需要做什么？  -> 做(Act): 调用工具 -> 看(Observe): 结果如何？ 
然后一直一轮轮的循环上面的过程，直到它认为任务完成。举个多步的例子:   

用户: 帮我查下明天北京适不适合跑步   

- 想: 需要知道明天天气 -> 做: 调get_weather -> 看: 下雨
- 想: 小于不适合户外跑步 -> 已有足够信息 -> 结束，回复用户建议改室内跑步

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/react_1.png?raw=true)               



##### Plan And Execute 

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/plan_and_execute_1.png?raw=true)               



