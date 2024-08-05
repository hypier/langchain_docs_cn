---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/e2b_data_analysis.ipynb
---

# E2B 数据分析

[E2B 的云环境](https://e2b.dev) 是 LLM 的优秀运行时沙箱。

E2B 的数据分析沙箱允许在沙箱环境中安全执行代码。这非常适合构建工具，如代码解释器或类似 ChatGPT 的高级数据分析。

E2B 数据分析沙箱允许您：
- 运行 Python 代码
- 通过 matplotlib 生成图表
- 在运行时动态安装 Python 包
- 在运行时动态安装系统包
- 运行 shell 命令
- 上传和下载文件

我们将创建一个简单的 OpenAI 代理，使用 E2B 的数据分析沙箱对上传的文件进行分析，使用 Python。

获取您的 OpenAI API 密钥和 [E2B API 密钥](https://e2b.dev/docs/getting-started/api-key)，并将其设置为环境变量。

您可以在 [这里](https://e2b.dev/docs) 找到完整的 API 文档。

您需要安装 `e2b` 以开始使用：

```python
%pip install --upgrade --quiet  langchain e2b langchain-community
```

```python
from langchain_community.tools import E2BDataAnalysisTool
```

```python
import os

from langchain.agents import AgentType, initialize_agent
from langchain_openai import ChatOpenAI

os.environ["E2B_API_KEY"] = "<E2B_API_KEY>"
os.environ["OPENAI_API_KEY"] = "<OPENAI_API_KEY>"
```

在创建 `E2BDataAnalysisTool` 实例时，您可以传递回调函数以监听沙箱的输出。这在创建更具响应性的用户界面时非常有用，尤其是结合 LLM 的流式输出。

```python
# Artifacts 是在调用 `plt.show()` 时由 matplotlib 创建的图表
def save_artifact(artifact):
    print("生成了新的 matplotlib 图表:", artifact.name)
    # 下载 artifact 作为 `bytes`，并让用户自行显示（例如在前端）
    file = artifact.download()
    basename = os.path.basename(artifact.name)

    # 将图表保存到 `charts` 目录
    with open(f"./charts/{basename}", "wb") as f:
        f.write(file)

e2b_data_analysis_tool = E2BDataAnalysisTool(
    # 将环境变量传递给沙箱
    env_vars={"MY_SECRET": "secret_value"},
    on_stdout=lambda stdout: print("stdout:", stdout),
    on_stderr=lambda stderr: print("stderr:", stderr),
    on_artifact=save_artifact,
)
```

上传一个示例 CSV 数据文件到沙箱，以便我们可以用代理进行分析。您可以使用例如 [这个文件](https://storage.googleapis.com/e2b-examples/netflix.csv)，它包含 Netflix 电视节目的信息。

```python
with open("./netflix.csv") as f:
    remote_path = e2b_data_analysis_tool.upload_file(
        file=f,
        description="关于 Netflix 电视节目的数据，包括标题、类别、导演、发行日期、演员、年龄评级等。",
    )
    print(remote_path)
```
```output
name='netflix.csv' remote_path='/home/user/netflix.csv' description='关于 Netflix 电视节目的数据，包括标题、类别、导演、发行日期、演员、年龄评级等。'
```
创建一个 `Tool` 对象并初始化 Langchain 代理。

```python
tools = [e2b_data_analysis_tool.as_tool()]

llm = ChatOpenAI(model="gpt-4", temperature=0)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True,
    handle_parsing_errors=True,
)
```

现在我们可以向代理询问有关我们之前上传的 CSV 文件的问题。

```python
agent.run(
    "在 2000 年到 2010 年之间，Netflix 上最长的 5 部电影是什么？创建一张它们时长的图表。"
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
调用: `e2b_data_analysis`，参数为 `{'python_code': "import pandas as pd\n\n# 加载数据\nnetflix_data = pd.read_csv('/home/user/netflix.csv')\n\n# 将 'release_year' 列转换为整数\nnetflix_data['release_year'] = netflix_data['release_year'].astype(int)\n\n# 筛选 2000 年到 2010 年间发行的电影\nfiltered_data = netflix_data[(netflix_data['release_year'] >= 2000) & (netflix_data['release_year'] <= 2010) & (netflix_data['type'] == 'Movie')]\n\n# 删除 'duration' 不可用的行\nfiltered_data = filtered_data[filtered_data['duration'].notna()]\n\n# 将 'duration' 列转换为整数\nfiltered_data['duration'] = filtered_data['duration'].str.replace(' min','').astype(int)\n\n# 获取最长的 5 部电影\nlongest_movies = filtered_data.nlargest(5, 'duration')\n\n# 创建条形图\nimport matplotlib.pyplot as plt\n\nplt.figure(figsize=(10,5))\nplt.barh(longest_movies['title'], longest_movies['duration'], color='skyblue')\nplt.xlabel('时长（分钟）')\nplt.title('2000-2010 年 Netflix 上最长的 5 部电影')\nplt.gca().invert_yaxis()\nplt.savefig('/home/user/longest_movies.png')\n\nlongest_movies[['title', 'duration']]"}`


[0mstdout:                              title  duration
stdout: 1019                        Lagaan       224
stdout: 4573                  Jodhaa Akbar       214
stdout: 2731      Kabhi Khushi Kabhie Gham       209
stdout: 2632  No Direction Home: Bob Dylan       208
stdout: 2126          What's Your Raashee?       203
[36;1m[1;3m{'stdout': "                             title  duration\n1019                        Lagaan       224\n4573                  Jodhaa Akbar       214\n2731      Kabhi Khushi Kabhie Gham       209\n2632  No Direction Home: Bob Dylan       208\n2126          What's Your Raashee?       203", 'stderr': ''}[0m[32;1m[1;3m在 2000 年到 2010 年之间，Netflix 上最长的 5 部电影是：

1. Lagaan - 224 分钟
2. Jodhaa Akbar - 214 分钟
3. Kabhi Khushi Kabhie Gham - 209 分钟
4. No Direction Home: Bob Dylan - 208 分钟
5. What's Your Raashee? - 203 分钟

以下是显示它们时长的图表：

![最长电影](sandbox:/home/user/longest_movies.png)[0m

[1m> 完成链。[0m
```

```output
"在 2000 年到 2010 年之间，Netflix 上最长的 5 部电影是：\n\n1. Lagaan - 224 分钟\n2. Jodhaa Akbar - 214 分钟\n3. Kabhi Khushi Kabhie Gham - 209 分钟\n4. No Direction Home: Bob Dylan - 208 分钟\n5. What's Your Raashee? - 203 分钟\n\n以下是显示它们时长的图表：\n\n![最长电影](sandbox:/home/user/longest_movies.png)"
```

E2B 还允许您在运行时动态安装 Python 和系统（通过 `apt`）包，如下所示：

```python
# 安装 Python 包
e2b_data_analysis_tool.install_python_packages("pandas")
```
```output
stdout: Requirement already satisfied: pandas in /usr/local/lib/python3.10/dist-packages (2.1.1)
stdout: Requirement already satisfied: python-dateutil>=2.8.2 in /usr/local/lib/python3.10/dist-packages (from pandas) (2.8.2)
stdout: Requirement already satisfied: pytz>=2020.1 in /usr/local/lib/python3.10/dist-packages (from pandas) (2023.3.post1)
stdout: Requirement already satisfied: numpy>=1.22.4 in /usr/local/lib/python3.10/dist-packages (from pandas) (1.26.1)
stdout: Requirement already satisfied: tzdata>=2022.1 in /usr/local/lib/python3.10/dist-packages (from pandas) (2023.3)
stdout: Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.10/dist-packages (from python-dateutil>=2.8.2->pandas) (1.16.0)
```
此外，您还可以像这样从沙箱中下载任何文件：

```python
# 路径是沙箱中的远程路径
files_in_bytes = e2b_data_analysis_tool.download_file("/home/user/netflix.csv")
```

最后，您可以通过 `run_command` 在沙箱内运行任何 shell 命令。

```python
# 安装 SQLite
e2b_data_analysis_tool.run_command("sudo apt update")
e2b_data_analysis_tool.install_system_packages("sqlite3")

# 检查 SQLite 版本
output = e2b_data_analysis_tool.run_command("sqlite3 --version")
print("版本: ", output["stdout"])
print("错误: ", output["stderr"])
print("退出代码: ", output["exit_code"])
```
```output
stderr: 
stderr: WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
stderr: 
stdout: Hit:1 http://security.ubuntu.com/ubuntu jammy-security InRelease
stdout: Hit:2 http://archive.ubuntu.com/ubuntu jammy InRelease
stdout: Hit:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
stdout: Hit:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
stdout: 正在读取软件包列表...
stdout: 正在建立依赖关系树...
stdout: 正在读取状态信息...
stdout: 所有软件包都是最新的。
stdout: 正在读取软件包列表...
stdout: 正在建立依赖关系树...
stdout: 正在读取状态信息...
stdout: 建议的软件包：
stdout:   sqlite3-doc
stdout: 以下新软件包将被安装：
stdout:   sqlite3
stdout: 0 升级，1 新安装，0 删除，0 未升级。
stdout: 需要获取 768 kB 的档案。
stdout: 安装后将使用 1873 kB 的额外磁盘空间。
stdout: 获取:1 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 sqlite3 amd64 3.37.2-2ubuntu0.1 [768 kB]
stderr: debconf: delaying package configuration, since apt-utils is not installed
stdout: 在 0s 内获取 768 kB (2258 kB/s)
stdout: 正在选择之前未选中的软件包 sqlite3。
(正在读取数据库 ... 23999 个文件和目录当前已安装。)
stdout: 正在准备解压 .../sqlite3_3.37.2-2ubuntu0.1_amd64.deb ...
stdout: 正在解压 sqlite3 (3.37.2-2ubuntu0.1) ...
stdout: 设置 sqlite3 (3.37.2-2ubuntu0.1) ...
stdout: 3.37.2 2022-01-06 13:25:41 872ba256cbf61d9290b571c0e6d82a20c224ca3ad82971edc46b29818d5dalt1
版本:  3.37.2 2022-01-06 13:25:41 872ba256cbf61d9290b571c0e6d82a20c224ca3ad82971edc46b29818d5dalt1
错误:  
退出代码:  0
```
当您的代理完成后，请不要忘记关闭沙箱。

```python
e2b_data_analysis_tool.close()
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)