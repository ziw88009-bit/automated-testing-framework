# Automated Testing Framework

Automated API Testing Framework

## Project Introduction

This is an enterprise-level automated API testing framework based on Python + pytest. It supports test case definitions in YAML format, integrates Allure reports, supports multiple database connections (MySQL, Redis, ClickHouse, MongoDB, Oracle), and provides comprehensive test data management and report generation capabilities.

## Core Features

- **YAML Test Cases**: Supports YAML format for defining test cases, facilitating maintenance and version control
- **Multi-Database Support**: Built-in connectors for MySQL, Redis, ClickHouse, MongoDB, Oracle, and SSH
- **Parameterized Substitution**: Supports dynamic parameterization and extraction of test data
- **Assertion Mechanism**: Provides rich assertion methods supporting validation of response content, status codes, response times, etc.
- **Data Extraction**: Supports extracting data from responses and saving it for subsequent test cases
- **Mock Service**: Built-in Flask mock server supporting independent testing
- **Report Generation**: Integrates Allure test reports, supporting Jenkins continuous integration
- **Notification Integration**: Supports automatic sending of test results via DingTalk bots

## Project Structure

```
├── base/                      # Base Request Classes
│   ├── apiutil.py            # General request processing
│   ├── apiutil_business.py   # Business scenario request processing
│   ├── generateId.py         # ID Generator
│   └── removefile.py         # File cleanup tool
├── common/                    # Common Utility Modules
│   ├── assertions.py         # Assertion tools
│   ├── connection.py         # Database connections (MySQL/Redis/ClickHouse/MongoDB/Oracle/SSH)
│   ├── debugtalk.py          # Dynamic function library (timestamps, encryption, random numbers, etc.)
│   ├── dingRobot.py          # DingTalk notification
│   ├── handleExcel.py        # Excel operations
│   ├── operationcsv.py       # CSV operations
│   ├── operxml.py            # XML operations
│   ├── readyaml.py           # YAML reading
│   ├── recordlog.py          # Log recording
│   ├── semail.py             # Email sending
│   └── sendrequest.py        # HTTP requests
├── conf/                      # Configuration Files
│   ├── operationConfig.py    # Config management
│   └── setting.py            # Settings
├── testcase/                  # Test Cases
│   ├── Business interface/   # Business scenario tests
│   ├── ProductManager/       # Product Management tests
│   └── Single interface/     # Single interface tests
├── mock_server/              # Mock Server
│   └── api_server/           # Flask simulated interfaces
├── data/                      # Test Data
├── run.py                     # Startup file
└── pytest.ini                 # Pytest configuration
```

## Environment Requirements

- Python 3.8+
- pytest
- requests
- PyYAML
- MySQL / Redis / ClickHouse / MongoDB (as required)

## Install Dependencies

```bash
pip install -r requirements.txt
```

Or install using uv:

```bash
uv pip install -r requirements.txt
```

## Configuration Instructions

### Environment Configuration (environment.xml)

Configure the test environment's base_url and global parameters.

### Database Configuration

Configure database connection information in `conf/operationConfig.py`:

- MySQL Configuration
- Redis Configuration
- ClickHouse Configuration
- MongoDB Configuration
- Oracle Configuration
- SSH Configuration

### Email Configuration

Set email server information in the configuration file:

```python
EMAIL = {
    'host': 'smtp.example.com',
    'user': 'test@example.com',
    'passwd': 'password'
}
```

## Test Case Format

### YAML Case Example

```yaml
- config:
    name: "Test Case Name"
    base_url: "http://api.example.com"
    verify: true
    
- test:
    name: "Interface Test"
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

### Extracting Data

Supports extracting data from responses for use in subsequent test cases:

```yaml
extract:
  - key: "user_id"
    json: "$.data.userId"
```

### Data Replacement

Reference extracted data using `${key}`:

```yaml
request:
  url: "/user/${user_id}"
```

## Running Tests

### Run All Tests

```bash
pytest
```

### Run Specific Tests

```bash
# Run a single test file
pytest testcase/Single interface/test_debug_api.py

# Run tests with specific markers
pytest -m "smoke"
```

### Generate Reports

```bash
# Generate Allure report
pytest --alluredir=allure-results

# View report
allure serve allure-results
```

### Start Mock Server

```bash
cd mock_server/api_server
python base/flask_service.py
```

## Built-in Functions (DebugTalk)

The framework provides rich dynamic function support:

- `${timestamp()}` - Get current timestamp (10 digits)
- `${timestamp_thirteen()}` - Get 13-digit timestamp
- `${md5_encryption(params)}` - MD5 encryption
- `${sha1_encryption(params)}` - SHA1 encryption
- `${base64_encryption(params)}` - Base64 encryption
- `${vehicle_random()}` - Generate random license plate number
- `${start_time()}` - Get today's start time
- `${end_time()}` - Get today's end time
- `${read_csv_data(file_name, index)}` - Read CSV data

## Assertion Instructions

Supports multiple assertion methods:

```yaml
validate:
  - eq: ["status_code", 200]           # Equality assertion
  - ne: ["code", 404]                   # Non-equality assertion
  - contains: ["content", "success"]    # Contains assertion
  - gt: ["response_time", 100]          # Greater than assertion
  - jsonpath: ["$.data.id", 123]        # JSONPath assertion
```

## Continuous Integration

### Jenkins Integration

The framework has built-in Jenkins support, capable of automatically retrieving build numbers, build status, and generating test reports.

### DingTalk Notification

After configuring the DingTalk bot, test completion results can be automatically sent upon test completion:

```python
from common.dingRobot import send_dd_msg
send_dd_msg("Test Completed: Success 10, Failure 2")
```

## License

MIT License