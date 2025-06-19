# 快手账户购买与自动化管理 SDK

![GitHub last commit](https://img.shields.io/github/last-commit/ks-account/sdk?style=flat-square)
![GitHub issues](https://img.shields.io/github/issues/ks-account/sdk?style=flat-square)
![Python Version](https://img.shields.io/badge/python-3.8%2B-blue?style=flat-square)

本项目提供了一个完整的快手(Kwai)账户购买、验证与管理自动化SDK，支持批量操作、安全验证绕过和数据分析功能。使用Python 3.8+开发，包含完整的API文档和示例代码。

## 目录

- [功能特性](#功能特性)
- [安装指南](#安装指南)
- [快速开始](#快速开始)
- [API文档](#api文档)
- [账户购买模块](#账户购买模块)
- [自动化管理](#自动化管理)
- [安全验证](#安全验证)
- [数据分析](#数据分析)
- [常见问题](#常见问题)
- [贡献指南](#贡献指南)
- [许可证](#许可证)

## 功能特性

- 快手账户批量购买接口
- 自动化账户验证系统
- 多账户会话管理
- 安全验证码自动识别
- 账户数据分析报表
- 模拟真人操作算法
- 代理IP池集成
- 异常检测与自动恢复

## 安装指南

### 系统要求

- Python 3.8+
- Chrome/Chromium浏览器(用于自动化操作)
- Redis服务器(用于会话缓存)

### 安装步骤

```bash
# 克隆仓库
git clone https://github.com/ks-account/sdk.git
cd sdk

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/MacOS
venv\Scripts\activate    # Windows

# 安装依赖
pip install -r requirements.txt

# 安装Chromium驱动 (Linux示例)
wget https://chromedriver.storage.googleapis.com/114.0.5735.90/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
sudo mv chromedriver /usr/local/bin/
```

### 配置文件

创建`config.yaml`文件：

```yaml
database:
  host: localhost
  port: 6379
  password: "your_redis_password"

api:
  ks_purchase:
    endpoint: "https://api.ks-purchase.com/v3"
    api_key: "your_api_key_here"

proxy:
  enable: true
  pool: "http://your-proxy-service.com/api/v1"
  refresh_interval: 3600

security:
  captcha_key: "your_2captcha_key"
  max_retry: 5
```

## 快速开始

### 购买单个快手账户

```python
from ks_sdk import AccountPurchaser

purchaser = AccountPurchaser.from_config("config.yaml")

# 购买基础账户
account = purchaser.purchase(
    account_type="basic",
    region="CN",
    min_age=30,
    max_price=15.00
)

print(f"Purchased account: {account.username} | Password: {account.password}")
```

### 批量购买账户

```python
from ks_sdk import BatchPurchaser

batch = BatchPurchaser(
    concurrency=5,
    retry_times=3,
    config_file="config.yaml"
)

results = batch.run(
    count=20,
    params={
        "account_type": "verified",
        "region": "CN",
        "min_followers": 1000
    }
)

print(f"Successfully purchased {len(results.success)} accounts")
print(f"Failed purchases: {len(results.failed)}")
```

## API文档

### 核心类

#### `AccountPurchaser`

```python
class AccountPurchaser:
    def __init__(self, api_endpoint: str, api_key: str, proxy_config: dict = None):
        """
        初始化账户购买客户端
        
        参数:
            api_endpoint: 购买API端点
            api_key: API认证密钥
            proxy_config: 代理配置字典
        """
        
    def purchase(self, account_type: str, **kwargs) -> Account:
        """
        购买单个账户
        
        参数:
            account_type: 账户类型(basic/verified/premium)
            kwargs: 其他过滤条件
        
        返回:
            Account对象
        """
    
    def check_balance(self) -> float:
        """检查账户余额"""
```

#### `AccountManager`

```python
class AccountManager:
    def __init__(self, storage_backend: Union[Redis, SQLAlchemy]):
        """
        初始化账户管理器
        
        参数:
            storage_backend: 存储后端(Redis或SQLAlchemy实例)
        """
    
    def add_account(self, account: Account):
        """添加账户到管理池"""
    
    def get_random_account(self, filters: dict = None) -> Account:
        """随机获取符合条件的账户"""
    
    def export_accounts(self, format: str = 'csv') -> str:
        """导出账户数据(csv/json)"""
```

## 账户购买模块

### 账户类型定义

```python
from enum import Enum

class AccountType(Enum):
    BASIC = 1       # 基础账户
    VERIFIED = 2    # 已认证账户
    PREMIUM = 3     # 高级账户(带粉丝)
    CUSTOM = 4      # 自定义参数账户
```

### 价格计算算法

```python
def calculate_price(account_type: AccountType, params: dict) -> float:
    """
    计算账户价格算法
    
    参数:
        account_type: 账户类型枚举
        params: 包含age/followers等参数的字典
    
    返回:
        计算后的价格
    """
    base_price = {
        AccountType.BASIC: 8.0,
        AccountType.VERIFIED: 25.0,
        AccountType.PREMIUM: 50.0,
        AccountType.CUSTOM: 30.0
    }.get(account_type, 0)
    
    # 自定义价格计算
    age_factor = max(0, (params.get('age', 0) - 180) * 0.1
    follower_factor = params.get('followers', 0) / 1000 * 2.5
    engagement_factor = params.get('engagement_rate', 0.03) / 0.03 * 1.2
    
    return round(base_price + age_factor + follower_factor * engagement_factor, 2)
```

### 购买请求示例

```python
import asyncio
from ks_sdk import AsyncAccountPurchaser

async def purchase_multiple():
    purchaser = AsyncAccountPurchaser.from_config("config.yaml")
    tasks = [
        purchaser.purchase(account_type="verified", min_followers=5000),
        purchaser.purchase(account_type="premium", region="US"),
        purchaser.purchase(account_type="custom", tags=["fashion", "beauty"])
    ]
    
    results = await asyncio.gather(*tasks, return_exceptions=True)
    for result in results:
        if isinstance(result, Exception):
            print(f"Purchase failed: {str(result)}")
        else:
            print(f"Purchased account: {result.username}")

asyncio.run(purchase_multiple())
```

## 自动化管理

### 会话保持技术

```python
class SessionManager:
    def __init__(self, browser_path: str = None):
        self.browser_path = browser_path or self.detect_browser()
        self.sessions = {}
    
    def create_session(self, account: Account, stealth: bool = True):
        """创建带伪装特征的浏览器会话"""
        options = webdriver.ChromeOptions()
        
        if stealth:
            options.add_argument("--disable-blink-features=AutomationControlled")
            options.add_experimental_option("excludeSwitches", ["enable-automation"])
            options.add_experimental_option("useAutomationExtension", False)
        
        # 设置用户代理和分辨率
        profile = self.generate_fingerprint(account.region)
        for key, value in profile.items():
            if key.startswith('--'):
                options.add_argument(f"{key}={value}")
        
        driver = webdriver.Chrome(options=options)
        self.sessions[account.username] = driver
        return driver
    
    def generate_fingerprint(self, region: str) -> dict:
        """生成浏览器指纹"""
        # 实现基于地区的指纹生成逻辑
        pass
```

### 自动化发布内容

```python
from ks_sdk import KsAutomation

automation = KsAutomation(
    proxy_enabled=True,
    captcha_service='2captcha'
)

account = Account(username="test123", password="pass123")
post_data = {
    "content": "Hello from SDK!",
    "images": ["/path/to/image.jpg"],
    "tags": ["technology", "programming"],
    "schedule_time": None  # 立即发布
}

result = automation.post_content(account, post_data)
if result.success:
    print(f"Post published! URL: {result.post_url}")
else:
    print(f"Failed to publish: {result.error}")
```

## 安全验证

### 验证码处理系统

```python
class CaptchaSolver:
    def __init__(self, service: str, api_key: str):
        self.service = service
        self.api_key = api_key
    
    def solve_image_captcha(self, image_path: str) -> str:
        """处理图片验证码"""
        if self.service == '2captcha':
            return self._solve_2captcha(image_path)
        elif self.service == 'anticaptcha':
            return self._solve_anticaptcha(image_path)
        else:
            raise ValueError("Unsupported captcha service")
    
    def _solve_2captcha(self, image_path: str) -> str:
        """2Captcha服务实现"""
        import requests
        
        with open(image_path, 'rb') as f:
            response = requests.post(
                'http://2captcha.com/in.php',
                files={'file': f},
                data={'key': self.api_key, 'method': 'post'}
            )
        
        if 'ERROR' in response.text:
            raise CaptchaError(response.text.split('|')[1])
        
        captcha_id = response.text.split('|')[1]
        
        # 轮询获取结果
        for _ in range(30):
            time.sleep(5)
            result = requests.get(
                f'http://2captcha.com/res.php?key={self.api_key}&action=get&id={captcha_id}'
            ).text
            
            if 'CAPCHA_NOT_READY' in result:
                continue
            elif 'ERROR' in result:
                raise CaptchaError(result.split('|')[1])
            else:
                return result.split('|')[1]
        
        raise CaptchaError("Timeout while solving captcha")
```

### 设备指纹绕过

```python
def generate_device_fingerprint() -> dict:
    """生成随机设备指纹"""
    import random
    import uuid
    
    brands = ['Xiaomi', 'Huawei', 'Oppo', 'Vivo', 'Samsung', 'Apple']
    models = {
        'Xiaomi': ['Redmi Note 12', 'Mi 11', 'Redmi K50'],
        'Apple': ['iPhone 13', 'iPhone 12', 'iPhone SE']
    }
    
    brand = random.choice(brands)
    model = random.choice(models.get(brand, ['Unknown']))
    
    return {
        'device_id': str(uuid.uuid4()),
        'brand': brand,
        'model': model,
        'os_version': f"Android {random.randint(8, 12)}.{random.randint(0, 4)}",
        'screen_resolution': f"{random.randint(720, 1440)}x{random.randint(1280, 2560)}",
        'cpu_cores': random.choice([4, 6, 8]),
        'memory': f"{random.choice([4, 6, 8])}GB",
        'ip_address': f"{random.randint(1, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(1, 255)}"
    }
```

## 数据分析

### 账户质量评估

```python
class AccountAnalyzer:
    def __init__(self, ks_api_key: str):
        self.ks_api_key = ks_api_key
    
    def analyze_account(self, username: str) -> dict:
        """分析账户质量"""
        data = self._fetch_account_data(username)
        
        # 计算活跃度分数(0-100)
        activity_score = self._calculate_activity_score(data)
        
        # 计算粉丝质量分数
        follower_score = self._calculate_follower_quality(data['followers'])
        
        return {
            'username': username,
            'activity_score': activity_score,
            'follower_score': follower_score,
            'engagement_rate': data['engagement_rate'],
            'risk_level': self._assess_risk(data)
        }
    
    def _fetch_account_data(self, username: str) -> dict:
        """从快手API获取账户数据"""
        # 实现API请求逻辑
        pass
    
    def _calculate_activity_score(self, data: dict) -> float:
        """基于发布频率、互动等计算活跃度"""
        posts_last_month = data.get('posts_30d', 0)
        avg_comments = data.get('avg_comments', 0)
        avg_likes = data.get('avg_likes', 0)
        
        score = min(posts_last_month, 30) * 1.5  # 最多45分
        score += min(avg_comments / 10, 30)      # 最多30分
        score += min(avg_likes / 20, 25)         # 最多25分
        
        return min(score, 100)
```

### 数据可视化示例

```python
import matplotlib.pyplot as plt
import pandas as pd

def visualize_accounts(accounts: list):
    """绘制账户数据图表"""
    df = pd.DataFrame(accounts)
    
    plt.figure(figsize=(12, 6))
    
    # 活跃度分布
    plt.subplot(1, 2, 1)
    df['activity_score'].hist(bins=20, color='skyblue')
    plt.title('Activity Score Distribution')
    plt.xlabel('Score')
    plt.ylabel('Count')
    
    # 粉丝质量散点图
    plt.subplot(1, 2, 2)
    plt.scatter(df['follower_score'], df['engagement_rate'], 
               c=df['risk_level'], cmap='RdYlGn')
    plt.title('Follower Quality vs Engagement')
    plt.xlabel('Follower Score')
    plt.ylabel('Engagement Rate')
    
    plt.tight_layout()
    plt.savefig('account_analysis.png')
    plt.close()
```

## 常见问题

### 购买相关

**Q: 购买账户时出现"风控拦截"错误怎么办？**

A: 尝试以下解决方案：
1. 更换代理IP
2. 调整购买间隔时间
3. 使用`AccountPurchaser`的`stealth_mode`参数

```python
purchaser = AccountPurchaser(
    api_endpoint="https://api.ks-purchase.com/v3",
    api_key="your_key",
    stealth_mode=True,  # 启用隐身模式
    delay_between_requests=30  # 30秒间隔
)
```

### 验证相关

**Q: 如何提高验证码识别成功率？**

A: 推荐以下优化方法：

1. 使用混合验证码服务
```python
from ks_sdk import HybridCaptchaSolver

solver = HybridCaptchaSolver(
    services=['2captcha', 'anticaptcha'],
    api_keys={
        '2captcha': 'key1',
        'anticaptcha': 'key2'
    }
)
```

2. 预处理验证码图片
```python
from ks_sdk.image_processing import preprocess_captcha

def solve_captcha(image_path):
    processed = preprocess_captcha(
        image_path,
        enhance=True,
        remove_noise=True
    )
    return solver.solve_image_captcha(processed)
```

## 贡献指南

我们欢迎各种形式的贡献！请遵循以下步骤：

1. Fork本仓库
2. 创建特性分支 (`git checkout -b feature/your-feature`)
3. 提交更改 (`git commit -am 'Add some feature'`)
4. 推送到分支 (`git push origin feature/your-feature`)
5. 创建Pull Request

### 代码风格要求

```python
def example_function(param1: int, param2: str = "default") -> dict:
    """
    符合规范的函数示例
    
    参数:
        param1: 整数参数说明
        param2: 字符串参数说明(默认"default")
    
    返回:
        包含结果的字典
    
    异常:
        ValueError: 当参数无效时抛出
    """
    if param1 < 0:
        raise ValueError("param1 must be positive")
    
    return {
        "result": param2 * param1,
        "status": "success"
    }
```

## 许可证

本项目采用 MIT 许可证 - 详情请参阅 [LICENSE](LICENSE) 文件。

```
Copyright 2025 Kwai Account SDK Project

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

**项目维护者**: @ks-sdk-team  
**最后更新时间**: 2025年6月  
**版本**: v2.3.1
