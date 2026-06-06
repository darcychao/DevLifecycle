# Python 编码规范

## 文档元数据
- **语言:** Python
- **版本:** 3.11+
- **包管理器:** Poetry / pip / uv
- **测试框架:** pytest
- **参考标准:** PEP 8, PEP 20 (Zen of Python), PEP 484 (Type Hints), Google Python Style Guide

---

## 1. 命名规范

### 1.1 文件命名
- **模式:** snake_case（全小写+下划线）
- **示例:** `user_service.py`, `date_utils.py`, `api_client.py`
- **入口文件:** `__init__.py`, `main.py`
- **测试文件:** `test_<模块名>.py` — `test_user_service.py`
- **配置文件:** `settings.py`, `config.py`

### 1.2 变量命名
- **局部/实例变量:** snake_case — `user_name = 'Alice'`
- **常量:** UPPER_SNAKE_CASE — `MAX_RETRY_COUNT = 3`
- **私有变量:** 单下划线前缀 — `_internal_cache`, `_private_method()`
- **模块级私有:** 单下划线前缀 — `_module_private_variable`
- **名称修饰（避免子类冲突）:** 双下划线前缀 — `__very_private`
- **布尔值:** `is`/`has`/`can`/`should` 前缀 — `is_active`, `has_permission`

```python
# 正确
MAX_TIMEOUT_SECONDS = 5
user_name = 'Alice'
is_visible = True
_internal_cache: dict[str, User] = {}

# 错误
max_timeout = 5        # 常量应大写
userName = 'Alice'     # 应使用 snake_case
visible = True         # 布尔值应有前缀
```

### 1.3 函数/方法命名
- **模式:** snake_case，动词开头
- **动词选择:** get / set / create / delete / update / fetch / validate / format / parse / has / is
- **工厂方法:** `create_` / `from_` — `create_user()`, `from_dict()`
- **魔术方法:** 双下划线 — `__str__`, `__repr__`, `__len__`

```python
# 正确
def get_user_by_id(user_id: int) -> User | None: ...
def validate_email(email: str) -> bool: ...
def create_user(name: str, email: str) -> User: ...

# 错误
def user(id): ...           # 缺少动词
def process(data): ...      # 动词含义模糊
```

### 1.4 类命名
- **模式:** PascalCase
- **异常类:** 以 `Error` 或 `Exception` 结尾 — `UserNotFoundError`, `ValidationError`
- **抽象基类:** 无固定前缀要求，使用 `ABC` 或 `abc.ABCMeta`

```python
class UserService:
    """用户业务服务。"""
    ...

class UserNotFoundError(Exception):
    """用户未找到异常。"""
    ...

class AbstractRepository(ABC):
    """仓库抽象基类。"""
    ...
```

---

## 2. 代码结构

### 2.1 文件组织 (PEP 8)
```
1. Shebang 行 (#!/usr/bin/env python3)（如需要）
2. 编码声明（Python 3 默认 UTF-8，无需）
3. 模块级 docstring
4. __all__ 声明（如需要）
5. import 语句（分组：标准库 → 第三方 → 本地模块）
6. 模块级常量和变量
7. 类和函数定义
```

```python
"""用户服务模块。

提供用户注册、查询、更新等核心业务逻辑。
"""

from __future__ import annotations

import logging
from datetime import datetime, timezone
from typing import Protocol

from pydantic import BaseModel, EmailStr

from .models import User
from .repository import UserRepository
from .exceptions import UserNotFoundError

logger = logging.getLogger(__name__)

DEFAULT_PAGE_SIZE = 20


class UserCreateRequest(BaseModel):
    name: str
    email: EmailStr


async def create_user(request: UserCreateRequest) -> User:
    """创建新用户并发送欢迎邮件。"""
    ...
```

### 2.2 函数规范
- **最大行数:** 20 行（软限制）
- **最大参数:** 4 个（超出使用 Keyword-Only 参数或数据类）
- **单一职责:** 一个函数只做一件事
- **尽早返回:** 错误/边界条件尽早 return
- **纯函数优先:** 优先使用纯函数，将副作用放在外层

```python
# 正确：keyword-only 参数减少位置参数
def search_users(
    query: str,
    *,
    page: int = 1,
    page_size: int = DEFAULT_PAGE_SIZE,
    include_inactive: bool = False,
    sort_by: str = "name",
) -> list[User]: ...

# 错误：过多位置参数
def search_users(query, page, page_size, include_inactive, sort_by): ...
```

### 2.3 类规范
- **最大方法数:** 10 个公共方法（考虑拆分）
- **组合优于继承**
- **使用 `@dataclass`** 定义数据容器
- **使用 `@property`** 提供计算属性

---

## 3. 格式化

### 3.1 缩进
- **风格:** 空格
- **宽度:** 4 个空格
- **续行:** 与开始分隔符对齐或悬挂缩进 4 个空格
- **不使用 Tab**

```python
# 正确：悬挂缩进
result = some_long_function_name(
    first_argument,
    second_argument,
    third_argument,
)

# 正确：与分隔符对齐
result = some_long_function_name(first_argument,
                                  second_argument,
                                  third_argument)
```

### 3.2 行长度
- **最大:** 88 字符（Black 默认）~ 100 字符（PEP 8 软限制 79）
- **推荐:** 88 字符（与 Black 一致）
- **长字符串、注释:** 72 字符

### 3.3 空行
- **顶层类/函数之间:** 2 行
- **类内方法之间:** 1 行
- **逻辑分组之间:** 1 行

### 3.4 引号
- **字符串:** 双引号 `"hello"`（Black 默认）
- **文档字符串:** 三双引号 `"""..."""`
- **包含引号的字符串:** 使用另一种引号避免转义

```python
# 正确
name = "Alice"
message = f"Hello, {name}!"
quote = 'He said, "Hello!"'

# 文档字符串
def get_user(id: int) -> User:
    """获取用户信息。"""
    ...
```

### 3.5 空格
- 赋值符两侧加空格: `a = 1`
- 函数参数逗号后加空格: `fn(a, b, c)`
- 默认参数 `=` 两侧不加空格: `def fn(a=1):`
- 切片冒号两侧不加空格: `items[1:3]`
- 列表尾部逗号（多行时）: `items = [1, 2, 3,]`

---

## 4. 类型提示 (Type Hints)

### 4.1 基本原则
- **公共 API 必须标注类型**
- **私有函数建议标注**（非强制）
- **局部变量能推断时省略**

### 4.2 Python 3.11+ 新语法
```python
# 联合类型：使用 | 代替 Optional 和 Union
def find_user(user_id: int) -> User | None:
    """None 表示未找到。"""
    ...

# 内置泛型：直接使用 list/dict 代替 typing.List/Dict
def get_users(*, page: int = 1) -> list[User]:
    ...

def process_scores(scores: dict[str, float]) -> dict[str, str]:
    ...

# 可调用对象
from collections.abc import Callable

Handler = Callable[[User], None]
```

### 4.3 Protocol（结构化子类型）
```python
from typing import Protocol

class UserRepository(Protocol):
    """用户仓库协议：任何实现此接口的对象均可使用。"""

    async def find_by_id(self, user_id: int) -> User | None: ...
    async def save(self, user: User) -> User: ...
    async def delete(self, user_id: int) -> None: ...
```

### 4.4 TypedDict 与 dataclass
```python
from dataclasses import dataclass, field
from datetime import datetime, timezone


@dataclass(frozen=True)  # 不可变
class User:
    id: int
    name: str
    email: str
    created_at: datetime = field(default_factory=lambda: datetime.now(timezone.utc))
```

### 4.5 类型别名
```python
type UserId = int
type JsonDict = dict[str, object]
type Result[T] = T | Exception
```

---

## 5. 异步编程

### 5.1 async/await
```python
import asyncio
from typing import Any


async def fetch_user_data(user_id: int) -> dict[str, Any]:
    user, posts, settings = await asyncio.gather(
        fetch_user(user_id),
        fetch_posts(user_id),
        fetch_settings(user_id),
    )
    return {"user": user, "posts": posts, "settings": settings}
```

### 5.2 异步上下文管理器
```python
async def read_file(path: str) -> str:
    async with aiofiles.open(path, "r") as f:
        return await f.read()
```

### 5.3 异步生成器
```python
from collections.abc import AsyncIterator

async def paginated_fetch(url: str, *, page_size: int = 100) -> AsyncIterator[dict]:
    page = 1
    while True:
        data = await fetch_page(url, page=page, page_size=page_size)
        if not data:
            break
        for item in data:
            yield item
        page += 1
```

---

## 6. 错误处理

### 6.1 自定义异常层次
```python
class AppError(Exception):
    """应用基础异常。"""
    def __init__(self, message: str, *, code: str = "INTERNAL_ERROR") -> None:
        super().__init__(message)
        self.code = code


class NotFoundError(AppError):
    """资源未找到。"""
    def __init__(self, resource: str, identifier: object) -> None:
        super().__init__(
            f"{resource} with id '{identifier}' not found",
            code="NOT_FOUND",
        )


class ValidationError(AppError):
    """输入验证失败。"""
    def __init__(self, message: str, errors: list[dict[str, object]]) -> None:
        super().__init__(message, code="VALIDATION_ERROR")
        self.errors = errors
```

### 6.2 异常处理规则
- **捕获具体异常**，不捕获 `Exception`（除非在最外层）
- **空 except 块禁止**
- **始终记录或传播异常**
- **使用 `raise ... from ...` 保留异常链**

```python
# 正确
try:
    user = await user_repository.find_by_id(user_id)
except DatabaseError as e:
    logger.exception("查询用户失败: user_id=%s", user_id)
    raise UserServiceError("Unable to fetch user") from e

# 错误
try:
    user = await user_repository.find_by_id(user_id)
except Exception:
    pass  # 禁止空 except
```

### 6.3 EAFP 模式（Python 惯用法）
```python
# EAFP: Easier to Ask for Forgiveness than Permission
# 正确：先尝试，失败再处理
try:
    value = mapping[key]
except KeyError:
    value = default

# 不推荐：LBYL (Look Before You Leap)
if key in mapping:
    value = mapping[key]
else:
    value = default
```

---

## 7. 数据结构

### 7.1 选择合适的数据结构
- **list:** 有序、可变序列，`items = ["a", "b"]`
- **tuple:** 不可变序列，`point = (1, 2)`
- **dict:** 键值映射，`{"name": "Alice"}`
- **set:** 无序、不重复，`tags = {"python", "web"}`
- **frozenset:** 不可变集合（可用作 dict 键）
- **dataclass:** 结构化数据容器
- **NamedTuple:** 轻量级不可变数据容器

### 7.2 推导式
```python
# 列表推导 — 简单场景
active_names = [u.name for u in users if u.is_active]

# 字典推导
user_map = {u.id: u for u in users}

# 集合推导
unique_tags = {tag for u in users for tag in u.tags}

# 嵌套推导不宜超过一层
# 错误：两层嵌套难以阅读
result = [x for sublist in nested for x in sublist if x > 0]

# 正确：展开为循环
result = []
for sublist in nested:
    for x in sublist:
        if x > 0:
            result.append(x)
```

### 7.3 生成器表达式
- 大数据集使用生成器节省内存

```python
# 正确：生成器（惰性求值）
total = sum(item.price for item in cart)

# 大文件逐行处理
with open("large_file.txt") as f:
    long_lines = (line for line in f if len(line) > 80)
    for line in long_lines:
        process(line)
```

---

## 8. 上下文管理器

### 8.1 资源管理
```python
# 文件操作
with open("data.json", "r") as f:
    data = json.load(f)

# 数据库连接
async with db.transaction():
    await db.execute(query)

# 自定义上下文管理器
from contextlib import contextmanager


@contextmanager
def timer(name: str):
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    logger.info("%s took %.3fs", name, elapsed)
```

---

## 9. 日志

### 9.1 日志配置
```python
import logging

logger = logging.getLogger(__name__)  # 每个模块使用自己的 logger
```

### 9.2 日志级别
- **DEBUG:** 开发调试信息
- **INFO:** 业务流程节点
- **WARNING:** 可恢复异常
- **ERROR:** 需要关注的错误
- **CRITICAL:** 影响系统运行的严重错误

```python
# 正确：使用占位符（惰性求值）
logger.info("用户 %s 注册成功", user.id)

# 错误：f-string（提前求值）
logger.info(f"用户 {user.id} 注册成功")

# 异常日志
try:
    process(data)
except ProcessingError:
    logger.exception("处理数据失败")  # 自动记录 traceback
```

---

## 10. 测试

### 10.1 pytest 测试结构
```python
import pytest
from pytest_mock import MockerFixture


class TestUserService:
    """用户服务测试集合。"""

    @pytest.fixture
    def user_repository(self, mocker: MockerFixture):
        return mocker.Mock()

    @pytest.fixture
    def service(self, user_repository):
        return UserService(user_repository)

    async def test_create_user_returns_saved_user(
        self,
        service: UserService,
        user_repository,
    ):
        """创建用户成功时返回已保存的用户对象。"""
        # Arrange
        request = CreateUserRequest(name="Alice", email="alice@example.com")
        expected_user = User(id=1, name="Alice", email="alice@example.com")
        user_repository.save.return_value = expected_user

        # Act
        result = await service.create_user(request)

        # Assert
        assert result == expected_user
        user_repository.save.assert_called_once()

    async def test_create_user_raises_validation_error_for_duplicate_email(
        self, service: UserService, user_repository
    ):
        """重复邮箱注册时抛出 ValidationError。"""
        ...
```

### 10.2 测试命名
- **函数:** `test_<被测函数>_<场景>_<预期结果>`
- **类:** `Test<被测模块>` 或 `Test<被测类>`
- **docstring:** 中文描述测试意图

### 10.3 Fixtures
```python
# conftest.py — 共享 fixtures
@pytest.fixture
async def db_session():
    """创建测试数据库会话，测试后回滚。"""
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    async with AsyncSession(engine) as session:
        yield session
        await session.rollback()
```

---

## 11. 文档字符串

### 11.1 模块级
```python
"""用户管理模块。

提供用户注册、查询、更新等核心功能。

Classes:
    UserService: 用户业务逻辑
    CreateUserRequest: 创建用户请求模型

Exceptions:
    UserNotFoundError: 用户未找到
    DuplicateEmailError: 邮箱重复
"""
```

### 11.2 函数/方法级（Google Style）
```python
def create_user(
    name: str,
    email: str,
    *,
    role: str = "user",
    send_welcome_email: bool = True,
) -> User:
    """创建新用户。

    Args:
        name: 用户名，1-100 字符。
        email: 邮箱地址，需为有效格式。
        role: 用户角色，默认为 "user"。
        send_welcome_email: 是否发送欢迎邮件。

    Returns:
        创建成功的用户对象。

    Raises:
        ValidationError: 输入参数无效时。
        DuplicateEmailError: 邮箱已被注册时。
    """
    ...
```

### 11.3 类级
```python
class UserService:
    """用户业务服务。

    处理用户相关的所有业务逻辑，包括注册、查询、更新和删除。

    Attributes:
        repository: 用户数据仓库。
        email_service: 邮件发送服务。

    Example:
        >>> service = UserService(repository, email_service)
        >>> user = await service.create_user("Alice", "alice@example.com")
    """
    ...
```

---

## 12. 导入规范

### 12.1 导入顺序
```python
# 1. __future__ 导入
from __future__ import annotations

# 2. 标准库
import asyncio
import json
from datetime import datetime, timezone
from pathlib import Path
from typing import Protocol

# 3. 第三方库
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from sqlalchemy import select

# 4. 本地模块
from .models import User
from .repository import UserRepository
from .utils import format_date
```

### 12.2 导入规则
- 每行一个导入（`from X import a, b, c` 除外）
- 移除未使用的导入
- 避免通配符导入 `from module import *`（`__all__` 定义的除外）
- 避免循环导入

---

## 13. 性能与最佳实践

### 13.1 内存
- 使用生成器处理大数据集
- 类中使用 `__slots__` 减少内存占用（大量实例场景）
- 避免在循环中创建临时列表

### 13.2 集合操作
```python
# 高效查找
valid_ids: set[int] = {1, 2, 3}
if user_id in valid_ids:  # O(1)
    ...

# 字典合并（Python 3.9+）
merged = dict1 | dict2
```

### 13.3 字符串
```python
# 使用 join 而非 + 拼接大量字符串
result = "".join(parts)

# 使用 f-string 格式化
message = f"User {user.name} has {len(user.posts)} posts"
```

---

## 14. 安全规范

### 14.1 输入验证
- 使用 Pydantic 进行数据验证
- 所有外部输入在边界处验证
- 白名单校验优于黑名单

```python
from pydantic import BaseModel, field_validator


class CreateUserRequest(BaseModel):
    name: str
    email: str
    role: str = "user"

    @field_validator("name")
    @classmethod
    def name_must_be_valid(cls, v: str) -> str:
        v = v.strip()
        if not v or len(v) > 100:
            raise ValueError("Name must be between 1 and 100 characters")
        return v
```

### 14.2 SQL 注入防护
- 始终使用参数化查询或 ORM

```python
# 正确：参数化查询
await session.execute(
    select(User).where(User.email == email)
)

# 错误：字符串拼接
await session.execute(
    text(f"SELECT * FROM users WHERE email = '{email}'")
)
```

### 14.3 密钥管理
- 禁止硬编码密钥
- 使用 `python-dotenv` 或 secret manager
- 禁止记录敏感数据

---

## 15. 工具链配置

### 15.1 pyproject.toml
```toml
[tool.ruff]
target-version = "py311"
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "SIM", "TCH"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.mypy]
strict = true
python_version = "3.11"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

### 15.2 推荐工具链
| 工具 | 用途 |
|------|------|
| **ruff** | Linting + 格式化（替代 flake8 + isort + Black） |
| **mypy** | 静态类型检查 |
| **pytest** | 测试框架 |
| **coverage** | 测试覆盖率 |

---

## 附录A: 命名速查表

| 类型 | 规则 | 示例 |
|------|------|------|
| 文件 | snake_case | `user_service.py` |
| 包 | 全小写无下划线 | `userservice` |
| 变量 | snake_case | `user_name` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 函数 | snake_case（动词开头） | `get_user_by_id()` |
| 类 | PascalCase | `UserService` |
| 异常 | PascalCase + Error/Exception | `UserNotFoundError` |
| 模块私有 | 单下划线前缀 | `_internal_cache` |
| 名称修饰 | 双下划线前缀 | `__very_private` |
| 布尔值 | is/has/can/should | `is_active` |
| 测试函数 | test_<描述> | `test_create_user` |
| 测试类 | Test<PascalCase> | `TestUserService` |

## 附录B: 禁止事项

- [ ] 使用裸 `except:` 或 `except Exception: pass`
- [ ] 通配符导入 `from module import *`
- [ ] 可变默认参数: `def fn(items=[]):`
- [ ] 硬编码密钥或密码
- [ ] 使用 `assert` 进行业务逻辑校验（可用 `-O` 禁用）
- [ ] 字符串拼接构建 SQL 查询
- [ ] f-string 用于日志格式化
- [ ] 超过 4 个位置参数的函数
- [ ] 循环依赖
- [ ] 使用 `type()` 进行类型比较（使用 `isinstance()`）
