## AI开发


### 常用工具

- Homebrew : macOS上装各种软件，类似Android的应用市场
- pyenv: 管理多个python版本，切换用哪个，类似多个JDK版本切换
- venv: 给每个项目隔离依赖库，类似Android中每个Gradle项目独立的dependencies
- pip: 在venv里面装具体的库，类似Gradle中的implementation
- Ollama: 在本地跑大模型，提供API，相当于一个跑在你电脑上的模型服务器


关键:  

- pyenv选Python版本
- 用该版本建venv隔离依赖
- pip在venv里装库
- 代码调用Ollama提供的本地模型

#### pyenv

安装: `brew install pyenv`
配置:  这里用的是zsh，把pyenv初始化写进~/.zshrc: 
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```
生效: `source ~/.zshrc`

那句 eval "$(pyenv init -)" 是灵魂——它让 pyenv 接管 python 命令。漏了它，你切了版本却发现 python --version 没变化，就是这个原因。


### 安装Python

```
# 查看已安装python版本，初次为空
pyenv versions

# 安装一个版本
pyenv install 3.12.4

# 设为全局默认
pyenv global 3.12.4
```

安装完后可以通过`python --version` 验证版本。 

### 版本切换的三个层级

```
pyenv global 3.12.4  # 全局默认
pyenv local 3.12.4  # 当前目录(项目级)，生成.python-version文件，最常用
pyenv shell 3.12.4 # 仅当前中端口临时用
```

优先级: local(项目) > shell(终端) > global(全局)  

pyenv local生成的.python-version文件会跟着项目走，cd进目录自动切换到对应版本，就像gradle-wrapper.properties锁定Gradle版本，建议提交Git。 

### 创建项目 + 虚拟环境(venv)

- 创建项目目录
- 进入目录，锁定这个项目的python版本`pyenv local 3.12.4 # 生成.python-version文件`
- 创建并激活虚拟环境

```
python -m venv venv # 创建虚拟环境(生成venv文件夹)
source venv/bin/activate # 激活
```
激活成功后，命令行提示符前面会出现(venv): 
`(venv) c@Cs-MacBook-Pro aitest1 %`
判断有没有激活，就看有没有(venv)，每次开始工作都要先激活，用完再deactivate退出。 

venv 一旦创建，就绑定了创建它时 pyenv 选定的那个 Python 版本。想换 Python 版本，要先 deactivate、pyenv local 改版本、删掉旧 venv、重新 python -m venv venv。

### Ollama部署本地模型

Ollama是一个让你在本地电脑上一个命令行就能跑开源大模型的工具。 
它的关键特征是: 提供一个OpenAI兼容的本地API（地址http://localhost:11434/v1）

这意味着: 你用标准的openai Python库写代码，只要把base_url指向本地Ollama，就能调用本地模型。  

将来要切到真正的云端OpenAI/其他厂商，改一个URL即可，代码几乎不用动。 

安装:  `brew install ollama`
启动:  `ollama serve`

看到它监听127.0.0.1:11434就说明服务起来了，保持这个窗口开着。

#### 拉取模型

新开一个终端窗口，拉两个模型: 

```
# 对话模型：qwen2.5（7B，中文能力好，约 4-5GB）
ollama pull qwen2.5

# 向量模型：nomic-embed-text（做第 03 章 RAG 用，很小）
ollama pull nomic-embed-text
```

命令行先聊两句，确认模型能用: 
```
ollama run qwen2.5
```

查看已装模型:  
```
ollama list
```


回到刚才激活了venv那个终端的项目目录窗口，安装需要用到的库:  
```
pip install openai python-dotenv chromadb fastapi uvicorn
```

记录当前项目的依赖清单:  

```
pip freeze > requirements.txt
```
requirements.txt要提交Git，别人执行pip install -r requirements.txt就能一键复现同样的依赖。 

#### 写配置文件(把连哪个模型抽出来)

即使使用本地Ollama，也要用.env管理配置 -- 这样将来切云端只改这一个文件，代码不动。   

新建.env(你的真实配置，不提交Git):  

```
# 指向本地 Ollama 的 OpenAI 兼容接口
LLM_BASE_URL=http://localhost:11434/v1
# Ollama 不校验 key，随便填一个非空值即可
LLM_API_KEY=ollama
# 对话模型
LLM_MODEL=qwen2.5
# 向量模型（后面用）
EMBED_MODEL=nomic-embed-text
```
新建 .env.example（模板，只有变量名，提交 Git）：
```
LLM_BASE_URL=
LLM_API_KEY=
LLM_MODEL=
EMBED_MODEL=
```

新建 .gitignore：
```
venv/
.env
__pycache__/
*.pyc
.chroma/
```

注意 .env 和 venv/ 都被 ignore 掉了：密钥不进 Git、笨重的虚拟环境不进 Git。.python-version、.env.example、requirements.txt 则要提交，它们让环境可复现。


```python
import os

from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

client = OpenAI(
    base_url = os.environ["LLM_BASE_URL"],
    api_key = os.environ["LLM_API_KEY"],
)

resp = client.chat.completions.create(
    model=os.environ["LLM_MODEL"],
    messages=[
        {"role": "user", "content": "用一句话确认你在正常工作。"},
    ],
)


print("模型恢复: ", resp.choices[0].message.content)
print("环境成功")
``` 





