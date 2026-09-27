# Automated Testing Framework

自动化接口测试框架

## 项目简介

这是一个基于 Python + pytest 的企业级自动化接口测试框架，支持 YAML 格式的测试用例定义，集成 Allure 报告，支持多种数据库连接（MySQL、Redis、ClickHouse、MongoDB、Oracle），并提供完整的测试数据管理和报告生成功能。

## 核心特性

- **YAML 测试用例**：支持 YAML 格式定义测试用例，便于维护和版本管理
- **多数据库支持**：内置 MySQL、Redis、ClickHouse、MongoDB、Oracle、SSH 连接器
- **参数化替换**：支持测试数据的动态参数化和提取
- **断言机制**：提供丰富的断言方法，支持响应内容、状态码、响应时间等验证
- **数据提取**：支持从响应中提取数据并保存供后续用例使用
- **Mock 服务**：内置 Flask 模拟服务器，支持独立测试
- **报告生成**：集成 Allure 测试报告，支持 Jenkins 持续集成
- **通知集成**：支持钉钉机器人自动发送测试结果

## 项目结构

```
├── base/                      # 基础请求类
│   ├── apiutil.py            # 通用请求处理
│   ├── apiutil_business.py   # 业务场景请求处理
│   ├── generateId.py         # ID生成器
│   └── removefile.py         # 文件清理工具
├── common/                    # 公共工具模块
│   ├── assertions.py         # 断言工具
│   ├── connection.py         # 数据库连接（MySQL/Redis/ClickHouse/MongoDB/Oracle/SSH）
│   ├── debugtalk.py          # 动态函数库（时间戳、加密、随机数等）
│   ├── dingRobot.py          # 钉钉通知
│   ├── handleExcel.py        # Excel操作
│   ├── operationcsv.py       # CSV操作
│   ├── operxml.py            # XML操作
│   ├── readyaml.py           # YAML读取
│   ├── recordlog.py          # 日志记录
│   ├── semail.py             # 邮件发送
│   └── sendrequest.py        # HTTP请求
├── conf/                      # 配置文件
│   ├── operationConfig.py    # 配置管理
│   └── setting.py            # 设置
├── testcase/                  # 测试用例
│   ├── Business interface/   # 业务场景测试
│   ├── ProductManager/       # 商品管理测试
│   └── Single interface/     # 单接口测试
├── mock_server/              # Mock服务器
│   └── api_server/           # Flask模拟接口
├── data/                      # 测试数据
├── run.py                     # 启动文件
└── pytest.ini                 # Pytest配置
```

## 环境要求

- Python 3.8+
- pytest
- requests
- PyYAML
- MySQL / Redis / ClickHouse / MongoDB (根据需求)

## 安装依赖

```bash
pip install -r requirements.txt
```

或使用 uv 安装：

```bash
uv pip install -r requirements.txt
```

## 配置说明

### 环境配置 (environment.xml)

配置测试环境的 base_url 和全局参数。

### 数据库配置

在 `conf/operationConfig.py` 中配置数据库连接信息：

- MySQL 配置
- Redis 配置
- ClickHouse 配置
- MongoDB 配置
- Oracle 配置
- SSH 配置

### 邮件配置

在配置文件中设置邮件服务器信息：

```python
EMAIL = {
    'host': 'smtp.example.com',
    'user': 'test@example.com',
    'passwd': 'password'
}
```

## 测试用例格式

### YAML 用例示例

```yaml
- config:
    name: "测试用例名称"
    base_url: "http://api.example.com"
    verify: true
    
- test:
    name: "接口测试"
    request:
      url: "/user/login"
      method: "POST"
      json:
        username: "test"
        password: "123456"
    validate:
      - eq: ["status_code", 200]
      - eq: ["content.code", 0]
```

### 提取数据

支持从响应中提取数据供后续用例使用：

```yaml
extract:
  - key: "user_id"
    json: "$.data.userId"
```

### 数据替换

使用 `${key}` 引用已提取的数据：

```yaml
request:
  url: "/user/${user_id}"
```

## 运行测试

### 运行所有测试

```bash
pytest
```

### 运行指定测试

```bash
# 运行单个测试文件
pytest testcase/Single interface/test_debug_api.py

# 运行指定标记的测试
pytest -m "smoke"
```

### 生成报告

```bash
# 生成 Allure 报告
pytest --alluredir=allure-results

# 查看报告
allure serve allure-results
```

### 启动 Mock 服务器

```bash
cd mock_server/api_server
python base/flask_service.py
```

## 内置函数 (DebugTalk)

框架提供丰富的动态函数支持：

- `${timestamp()}` - 获取当前时间戳（10位）
- `${timestamp_thirteen()}` - 获取13位时间戳
- `${md5_encryption(params)}` - MD5加密
- `${sha1_encryption(params)}` - SHA1加密
- `${base64_encryption(params)}` - Base64加密
- `${vehicle_random()}` - 生成随机车牌号
- `${start_time()}` - 获取今日开始时间
- `${end_time()}` - 获取今日结束时间
- `${read_csv_data(file_name, index)}` - 读取CSV数据

## 断言说明

支持多种断言方式：

```yaml
validate:
  - eq: ["status_code", 200]           # 相等断言
  - ne: ["code", 404]                   # 不相等断言
  - contains: ["content", "success"]    # 包含断言
  - gt: ["response_time", 100]          # 大于断言
  - jsonpath: ["$.data.id", 123]        # JSON路径断言
```

## 持续集成

### Jenkins 集成

框架内置 Jenkins 支持，可自动获取构建编号、构建状态，并生成测试报告。

### 钉钉通知

配置钉钉机器人后，测试完成可自动发送结果通知：

```python
from common.dingRobot import send_dd_msg
send_dd_msg("测试完成：成功 10，失败 2")
```

## 许可证

MIT License