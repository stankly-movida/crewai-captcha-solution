# CrewAI CapSolver 集成：CAPTCHA 绕过工具

[![CrewAI](https://img.shields.io/badge/CrewAI-Python%20框架-blue?style=for-the-badge&logo=python)](https://github.com/crewAIInc/crewAI)
[![CapSolver](https://img.shields.io/badge/CapSolver-CAPTCHA%20解决方案-green?style=for-the-badge)](https://www.capsolver.com/?utm_source=github&utm_medium=repo&utm_campaign=crewai-capsolver)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

## 🚀 概览

本仓库提供了一套定制的 **CrewAI 工具**，旨在与 **CapSolver API** 集成，使自主 AI 智能体能够在执行基于网络的任务时，自动解决各种 CAPTCHA 挑战（reCAPTCHA、Turnstile、Cloudflare 等）。

此集成确保您的 CrewAI 工作流即使在处理受保护的网站时，也能保持不间断和可扩展。

## ✨ 特性

*   **`CaptchaSolverTool`**：用于解决基于令牌的 CAPTCHA（reCAPTCHA v2/v3、Turnstile）的通用工具。
*   **`CloudflareChallengeTool`**：用于绕过 Cloudflare 5 秒挑战的专用工具，返回必要的 Cookie 和 User-Agent。
*   **强大的错误处理**：内置轮询和超时逻辑，确保任务可靠完成。

## 🛠️ 安装

1.  **克隆仓库：**
    ```bash
    git clone https://github.com/your-username/crewai-capsolver-tools.git
    cd crewai-capsolver-tools
    ```

2.  **安装依赖：**
    ```bash
    pip install crewai 'crewai[tools]' requests pydantic
    ```

## ⚙️ 配置

1.  **获取您的 CapSolver API 密钥：**
    注册 CapSolver 并获取您的 API 密钥。
    > **提示：** [在此处注册 CapSolver](https://www.capsolver.com/?utm_source=github&utm_medium=repo&utm_campaign=crewai-capsolver)，使用优惠码 **CREWAI** 充值可额外获得 **6%** 奖励。

2.  **设置 API 密钥：**
    出于安全考虑，强烈建议将您的 API 密钥设置为环境变量。
    ```bash
    export CAPSOLVER_API_KEY="YOUR_CAPSOLVER_API_KEY"
    ```
    或者，您可以直接编辑 `capsolver_tools.py` 中的 `CAPSOLVER_API_KEY` 变量。

## 💡 使用示例

以下示例演示了如何使用定制的 CapSolver 工具设置 CrewAI 智能体，并将获得的令牌用于网络操作。

### `example_crew.py`

```python
import os
from crewai import Agent, Task, Crew, Process
from capsolver_tools import CaptchaSolverTool # 导入自定义工具

# 确保 API 密钥已设置
if not os.getenv("CAPSOLVER_API_KEY"):
    print("错误：未设置 CAPSOLVER_API_KEY 环境变量。")
    exit()

# 1. 定义工具
captcha_solver = CaptchaSolverTool()

# 2. 定义智能体
# 智能体被赋予工具，并被告知何时使用它。
captcha_agent = Agent(
    role='CAPTCHA 解决专家',
    goal='解决 reCAPTCHA 和 Turnstile 挑战，为其他智能体提供网络访问权限。',
    backstory="一位使用 AI 服务绕过自动化网络保护的专家。",
    tools=[captcha_solver],
    verbose=True,
    allow_delegation=False
)

# 3. 定义任务
# 智能体将在发现输入模式与其目标匹配时自动调用工具。
target_url = "https://www.example.com/protected-page"
site_key = "YOUR_SITE_KEY" # 目标页面中的 data-sitekey

solve_task = Task(
    description=f"使用 captcha_solver 工具获取以下页面的 reCAPTCHA 令牌：URL: {target_url}, Site Key: {site_key}。CAPTCHA 类型为 ReCaptchaV2TaskProxyLess。",
    agent=captcha_agent,
    expected_output="成功检索到的 reCAPTCHA 令牌（一个长字符串）。"
)

# 4. 创建并运行 Crew
crew = Crew(
    agents=[captcha_agent],
    tasks=[solve_task],
    process=Process.sequential,
    verbose=2
)

result = crew.kickoff()
print("\n\n########################")
print("## Crew 执行结果")
print("########################")
print(result)

# 获得的令牌随后可被另一个智能体（例如，网络抓取智能体）用于提交表单并访问受保护的内容。
```

## 🔗 提交辅助函数

为了完整性，这里提供了用于提交从 CapSolver 获得的令牌/Cookie 的辅助函数。

### `submission_helpers.py`

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import requests

def submit_recaptcha_token(driver: webdriver.Chrome, token: str):
    """将 reCAPTCHA 令牌注入到隐藏的文本区域并提交表单。"""
    # ... (代码逻辑不变)

def access_protected_page_with_cf_solution(url: str, cf_solution: dict) -> str:
    """
    使用 Cloudflare 挑战解决方案（Cookie 和 User-Agent）
    通过 requests 会话访问受保护的页面。
    """
    # ... (代码逻辑不变)
```

## 📚 资源

*   [CrewAI 文档](https://www.crewai.com/)
*   [CapSolver API 文档](https://docs.capsolver.com/)
*   [注册 CapSolver（使用优惠码 **CREWAI** 获得 6% 奖励）](https://www.capsolver.com/?utm_source=github&utm_medium=repo&utm_campaign=crewai-capsolver)
