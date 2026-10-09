# Test-Automation-Framework

基于 Pytest 的接口自动化测试框架。测试用例以 **YAML 数据驱动** 方式编写,配合 **DebugTalk 动态函数**、**接口关联(extract)**、**多源断言**与 **Allure 报告**,并内置一套 **Flask Mock 服务**用于本地联调,开箱即可在无真实环境的情况下跑通完整链路。

## 环境要求

| 组件 | 要求 |
|---|---|
| Python | 主框架 **>= 3.12**;运行 mock_server 需 **>= 3.13** |
| 包管理器 | [uv](https://docs.astral.sh/uv/)(推荐,仓库已带 `uv.lock`)或 pip |
| Allure CLI | 可选,生成 Allure 报告时需要(见下文「报告」) |

锁定版本:Python 3.12/3.13 均适配(pandas 2.2.3、numpy 2.2.1)。

## 快速开始

```bash
# 1. 安装依赖(根目录)
uv sync
# 或: pip install .

# 2. 创建配置文件(模板见下文「配置说明」)
#    conf/config.ini —— 该文件已被 .gitignore 排除,需要手动创建

# 3. (可选)启动配套 Mock 服务
cd mock_server/api_server
uv sync --python 3.13
uv run python base/flask_service.py   # 监听 127.0.0.1:8787

# 4. 运行测试(库内用例默认直连本地 Mock http://127.0.0.1:8787)
cd ../..
uv run pytest ./testcase
# 或一键运行并生成 Allure 报告:
uv run python run.py
```

## 配置说明

### conf/config.ini

被 `.gitignore` 排除,不会随仓库分发,需按如下结构手动创建(所有值替换成你自己的环境):

```ini
[api_envi]
host = http://127.0.0.1:8787

[MYSQL]
host = *****
port = 3306
username = root
password = *****
database = zqsjfx

[REDIS]
host = 127.0.0.1
port = 6379
username = ""
password = ""
db = 0

[CLICKHOUSE]
host = *******
port = 8123
username = default
password =
timeout = 10
db = default

[MongoDB]
host = 127.0.0.1
port = 27017
username = admin
password = *****
database = admin

[EMAIL]
host = smtp.163.com
port = 25
user = 发件人邮箱地址
passwd = 邮箱授权码(非登录密码)
addressee = 收件邮箱,多个用;隔开
subject = 接口测试

[SSH]
host = ******
port = 22
username = root
password = *******
timeout = 10
command = cat /logs/web_app/zjjtsjzf/zjjtsjzf-api/common-default.log

[REPORT_TYPE]
; 默认为 allure,生成 Allure 报告;设为 tm,生成 tmreport 报告
type = allure
```

### conf/setting.py 常用开关

| 配置项 | 说明 | 默认 |
|---|---|---|
| `REPORT_TYPE` | 报告风格:`allure` / `tm` | `allure` |
| `dd_msg` | 是否在测试结束后推送钉钉消息 | `False` |
| `API_TIMEOUT` | 接口超时时间(秒) | `60` |
| `SHEET_ID` | Excel 数据读取的 sheet 序号 | `0` |
| `FILE_PATH` | 日志、报告、Excel、SQL 等各类文件路径 | - |

## 编写测试用例

用例只需维护 YAML 文件,由 `pytest.mark.parametrize` + `get_testcase_yaml()` 参数化执行,基本不用写 Python 代码:

```yaml
- baseInfo:
    api_name: 电网系统登录
    url: /coupApply/cms/login_dw
    method: post
    header:
      Content-Type: application/json;charset=UTF-8
  testCase:
    - case_name: 检查登录正常
      json:
        username: test123
        password: qwe666
      validation:
        - contains: { 'msg': '调用成功' }
      extract:
        user_token: $.data.user_token   # 提取响应字段,写入 extract.yaml
```

```python
@pytest.mark.parametrize('base_info,testcase', get_testcase_yaml('./testcase/ProductManager/login_dw.yaml'))
def test_login_dw(self, base_info, testcase):
    RequestBase().specification_yaml(base_info, testcase)
```

## 技术亮点

- **YAML 数据驱动**:一个 YAML 文件承载接口信息(baseInfo)+ 多条用例(testCase),`get_testcase_yaml` 自动拆分参数化,测试点与代码分离,新增用例零代码。
- **DebugTalk 动态函数**:在 YAML 中以 `${func(args)}` 形式调用 `common/debugtalk.py` 中定义的任意函数(取当前时间、随机数、读写数据等),`apiutil_business.replace_load` 自动解析替换。
- **接口关联**:用例 `extract` 段用 JSONPath 从响应中提取变量(如登录 token)写入 `extract.yaml`,后续用例通过 `${get_extract_yaml(key)}` 引用,实现跨接口参数传递;`conftest.py` 的 session 级 fixture 会在每次运行前清空该文件。
- **多源断言**:`validation` 段支持 `contains`、`eq` 等多种断言方式,同时封装了 MySQL / Redis / ClickHouse / MongoDB / SSH 连接工具,可直接对数据库结果做二次校验。
- **报告与通知**:Allure(`allure.feature/story/dynamic.title` 组织用例层级)+ tmreport 双风格报告;测试结束自动打印摘要,可扩展推送到钉钉(`common/dingRobot.py`)、邮件(`common/semail.py`),并支持 Jenkins 集成(`common/Pjenkins.py`)。
- **配套 Mock 服务**:`mock_server/` 是基于 Flask 的接口 Mock(带 JWT 鉴权、faker 造数),覆盖库内用例所用接口,默认 `api_envi.host` 即指向它,实现环境隔离、一键本地联调。
- **工程化**:uv 管理依赖与锁定版本、pytest.ini 统一收集规则、Excel/CSV/YAML/XML 多种数据源读取(`common/handleExcel.py`、`operationcsv.py`、`operxml.py`)。

## 目录结构

```
├── base/              请求执行引擎、接口调用封装
├── common/            通用模块:断言/日志/请求/钉钉/邮件/Jenkins/各类文件读写
├── conf/              config.ini(本地配置,不入库)、setting.py、operationConfig.py
├── data/              测试数据:Excel、CSV、YAML、SQL(xml)
├── testcase/          测试用例:YAML 用例 + pytest 脚本
├── mock_server/       Flask Mock 服务(独立子项目,Python >= 3.13)
├── report/            生成的测试报告(不入库)
├── run.py             一键运行入口(Allure / tm 报告)
├── conftest.py        session 级 fixture:清空 extract、清理报告、结果摘要
├── environment.xml    Allure 环境信息(运行写入 report/temp)
├── extract.yaml       运行时提取的接口关联变量(不入库,自动生成)
├── pyproject.toml / uv.lock   依赖声明与锁定
└── pytest.ini         pytest 收集规则
```

## 报告

- **Allure 模式**(默认):`python run.py` 或 `REPORT_TYPE=allure` 时,执行 `pytest --alluredir ./report/temp` 后调用 `allure serve` 自动打开报告。需先安装 Allure CLI(如 `scoop install allure` / `brew install allure`)。
- **tm 模式**:`conf/config.ini` 的 `[REPORT_TYPE] type = tm` 后运行 `python run.py`,生成 `report/tmreport/testReport.html`(依赖 `pytest-tmreport` 插件)。
- 直接执行 `uv run pytest ./testcase` 则只在控制台输出结果,不生成报告。

## 常见问题

- **找不到接口地址/URL 为空**:`conf/config.ini` 未创建或 `[api_envi] host` 未配置,请先按模板创建。
- **`allure` 命令找不到**:Allure 报告模式需要本机安装 Allure CLI;只想要 console 结果可直接 `uv run pytest ./testcase`。
- **mock_server 无法启动**:需要 Python >= 3.13,且在其目录下单独 `uv sync`。
- **用例运行报中文乱码**:YAML 文件请确保为 UTF-8 编码(见 `common/readyaml.py` 的编码检查)。