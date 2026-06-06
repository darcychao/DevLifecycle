# JavaScript 编码规范

## 文档元数据
- **语言:** JavaScript (ECMAScript)
- **版本:** ES2022+
- **运行环境:** Node.js / Browser
- **参考标准:** Airbnb JavaScript Style Guide, StandardJS

---

## 1. 命名规范

### 1.1 文件命名
- **模式:** kebab-case（小写+连字符）
- **示例:** `user-service.js`, `date-utils.js`, `api-client.js`
- **入口文件:** `index.js`
- **配置文件:** `*.config.js`
- **测试文件:** `*.spec.js` 或 `*.test.js`

### 1.2 变量命名
- **局部变量:** camelCase — `let userName = 'Alice';`
- **常量:** UPPER_SNAKE_CASE — `const MAX_RETRY_COUNT = 3;`
- **布尔值:** 使用 `is`/`has`/`should`/`can` 前缀 — `isActive`, `hasPermission`, `shouldUpdate`
- **集合:** 使用复数或集合后缀 — `users`, `userList`, `userMap`

```javascript
// 正确
const MAX_TIMEOUT_MS = 5000;
let isVisible = true;
const userNames = ['Alice', 'Bob'];
const userMap = new Map();

// 错误
const maxtimeout = 5000;
let visible = true;
const userName = ['Alice', 'Bob'];  // 数组应用复数
```

### 1.3 函数命名
- **模式:** camelCase
- **动词开头:** get / set / create / delete / update / handle / validate / format / parse / fetch / has / is
- **事件处理:** `handle` + 事件名 — `handleClick`, `handleSubmit`
- **异步函数:** 无特殊前缀要求，但内部应明确返回 Promise

```javascript
// 正确
function getUserById(id) { ... }
function validateEmail(email) { ... }
async function fetchUserData(userId) { ... }
function handleFormSubmit(event) { ... }

// 错误
function user(id) { ... }         // 缺少动词
function process(data) { ... }     // 动词含义模糊
```

### 1.4 类命名
- **模式:** PascalCase
- **示例:** `UserService`, `EventEmitter`, `HttpClient`
- **私有属性:** 无前缀，使用 `#` 私有字段语法
- **静态方法:** 无特殊命名要求

```javascript
class UserRepository {
  #cache = new Map();         // 私有字段
  static DEFAULT_LIMIT = 20;   // 静态常量

  async findById(id) { ... }
}
```

---

## 2. 代码结构

### 2.1 文件组织顺序
```
1. 'use strict'; 声明（如使用）
2. import / require 语句（分组：Node内置 → 第三方 → 内部模块）
3. 常量定义
4. 类型定义（JSDoc 类型注释）
5. 模块级变量
6. 函数定义（导出函数优先）
7. export 语句（如使用ES模块）
```

```javascript
'use strict';

// Node 内置模块
const path = require('path');
const fs = require('fs/promises');

// 第三方模块
const express = require('express');
const lodash = require('lodash');

// 内部模块
const { UserService } = require('./services/user-service');
const { formatDate } = require('./utils/date-utils');

// 常量
const DEFAULT_PAGE_SIZE = 20;

// 导出函数
async function createUser(userData) { ... }
function validateUserInput(input) { ... }

module.exports = { createUser, validateUserInput };
```

### 2.2 函数规范
- **最大行数:** 30 行（软限制，超长函数应拆分）
- **最大参数:** 3 个（超出使用对象参数）
- **单一职责:** 一个函数只做一件事
- **尽早返回:** 错误/边界条件尽早 return，减少嵌套
- **纯函数优先:** 优先使用纯函数，避免副作用

```javascript
// 正确：早返回减少嵌套
function validateUser(user) {
  if (!user) return { valid: false, error: 'User is required' };
  if (!user.name) return { valid: false, error: 'Name is required' };
  if (!user.email) return { valid: false, error: 'Email is required' };
  return { valid: true };
}

// 错误：深层嵌套
function validateUser(user) {
  if (user) {
    if (user.name) {
      if (user.email) {
        return { valid: true };
      } else {
        return { valid: false, error: 'Email is required' };
      }
    } else {
      return { valid: false, error: 'Name is required' };
    }
  } else {
    return { valid: false, error: 'User is required' };
  }
}
```

---

## 3. 格式化

### 3.1 缩进
- **风格:** 空格
- **宽度:** 2 个空格
- **不使用 Tab**

### 3.2 行长度
- **最大:** 100 字符
- **例外:** 长 URL、正则表达式、模板字符串

### 3.3 大括号
- **风格:** 1TBS（One True Brace Style）
- 所有控制语句必须使用大括号，即使只有一行

```javascript
// 正确
if (condition) {
  doSomething();
}

// 错误
if (condition) doSomething();     // 缺少大括号
if (condition)
  doSomething();                  // 缺少大括号
```

### 3.4 空格
- 二元运算符两侧加空格: `a + b`, `x === y`
- 逗号后加空格: `[1, 2, 3]`, `{ a: 1, b: 2 }`
- 函数名与括号之间无空格: `fn()`
- 匿名函数: `function () {}`, `async () => {}`
- 行尾不允许空格

### 3.5 引号
- **字符串:** 单引号 `'hello'`
- **模板字符串:** 反引号（需要变量插值时使用）
- **JSX 属性:** 双引号

```javascript
// 正确
const name = 'Alice';
const greeting = `Hello, ${name}!`;
const path = require('path');

// 错误
const name = "Alice";              // 不一致
const greeting = 'Hello, ' + name; // 应使用模板字符串
```

### 3.6 分号
- **必须使用分号**（无ASI依赖）
- 每条语句以分号结尾

---

## 4. 变量声明

### 4.1 声明关键字
- **const** — 优先使用，不会被重新赋值的变量
- **let** — 需要重新赋值时使用
- **var** — 禁止使用

```javascript
// 正确
const name = 'Alice';
let count = 0;
count++;

// 错误
var name = 'Alice';  // 禁止使用 var
```

### 4.2 解构
- 优先使用解构赋值

```javascript
// 正确
const { name, email } = user;
const [first, second] = items;

// 错误
const name = user.name;
const email = user.email;
```

### 4.3 展开运算符
- 优先使用展开运算符代替 `Object.assign` 和 `Array.concat`

```javascript
// 正确
const copy = { ...original, newProp: 'value' };
const merged = [...arr1, ...arr2];

// 错误
const copy = Object.assign({}, original, { newProp: 'value' });
```

---

## 5. 函数

### 5.1 箭头函数
- 匿名回调使用箭头函数
- 简单表达式省略 return 和大括号
- 单参数省略括号

```javascript
// 正确
const doubled = numbers.map(n => n * 2);
const filtered = users.filter(u => u.isActive);
[1, 2, 3].forEach((n) => { console.log(n); });

// 顶层/导出函数使用普通函数声明
function calculateTotal(items) { ... }
```

### 5.2 默认参数
- 使用默认参数语法，避免在函数体内判断

```javascript
// 正确
function createUser(name, options = {}) {
  const { role = 'user', isActive = true } = options;
  ...
}

// 错误
function createUser(name, options) {
  options = options || {};
  ...
}
```

### 5.3 Rest 参数
- 使用 rest 参数代替 `arguments` 对象

```javascript
// 正确
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// 错误
function sum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}
```

---

## 6. 对象与数组

### 6.1 对象字面量简写
```javascript
// 正确
const user = { name, email, getProfile() { ... } };

// 错误
const user = { name: name, email: email, getProfile: function() { ... } };
```

### 6.2 属性访问
- 已知属性使用点号: `user.name`
- 动态属性使用方括号: `user[propName]`

### 6.3 数组方法
- 优先使用高阶方法: `map`, `filter`, `reduce`, `find`, `some`, `every`
- 避免 `for` 循环进行数组遍历

---

## 7. 字符串

### 7.1 模板字符串
- 字符串拼接时使用模板字符串
- 多行字符串使用模板字符串

```javascript
// 正确
const url = `/api/users/${userId}/posts?page=${page}`;
const message = `
  Hello ${name},
  Your order #${orderId} has been shipped.
`;

// 错误
const url = '/api/users/' + userId + '/posts?page=' + page;
```

---

## 8. 模块

### 8.1 ES 模块（推荐）
```javascript
// 导入
import { UserService } from './services/user-service.js';
import express from 'express';
import * as utils from './utils/index.js';

// 导出
export { createUser, validateUser };
export default UserService;
```

### 8.2 CommonJS（Node.js 兼容场景）
```javascript
// 导入
const { UserService } = require('./services/user-service');
const express = require('express');

// 导出
module.exports = { createUser, validateUser };
```

### 8.3 导入规则
- 禁止通配符导入（ES模块中 `import *` 除外）
- 禁止未使用的导入
- 禁止循环依赖

---

## 9. 异步编程

### 9.1 async/await（首选）
```javascript
// 正确
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);
    return { ...user, posts };
  } catch (error) {
    console.error('Failed to get user data:', error);
    throw new Error(`Unable to fetch data for user ${userId}`);
  }
}
```

### 9.2 Promise
- 需要并行执行时使用 `Promise.all` / `Promise.allSettled`
- 避免 Promise 嵌套

```javascript
// 正确：并行请求
const [user, settings] = await Promise.all([
  fetchUser(userId),
  fetchSettings(userId),
]);

// 错误：串行等待（无依赖关系）
const user = await fetchUser(userId);
const settings = await fetchSettings(userId);
```

### 9.3 回调
- 新代码禁止使用回调风格
- 使用 `util.promisify` 包装遗留回调 API

---

## 10. 错误处理

### 10.1 错误类型
```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class NotFoundError extends Error {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`);
    this.name = 'NotFoundError';
  }
}
```

### 10.2 try/catch
- 仅在需要处理错误的层面使用
- 空 catch 块禁止
- 始终记录或有意义地处理错误

### 10.3 异步错误传播
- async 函数中的错误自动转为 rejected Promise
- 顶层调用必须处理 rejection: `await fn()` 或用 `.catch()`

---

## 11. 注释

### 11.1 何时注释
- **不注释代码做了什么** — 清晰的命名已经说明
- **注释为什么这么做** — 非显而易见的约束、Bug 变通方案、算法选择
- **不保留注释掉的代码** — 删除它，git 历史中有存档

### 11.2 注释格式
```javascript
// 单行：简短说明为什么

/*
 * 多行：仅用于复杂算法逻辑说明。
 * 说明算法的步骤和原理。
 */
```

### 11.3 JSDoc
- 公共 API 函数使用 JSDoc
- 不需要为显而易见的 getter/setter 撰写文档

```javascript
/**
 * 根据ID获取用户信息
 * @param {string} userId - 用户唯一标识
 * @param {Object} [options] - 可选参数
 * @param {boolean} [options.includePosts=false] - 是否包含用户帖子
 * @returns {Promise<User>} 用户对象
 * @throws {NotFoundError} 用户不存在时抛出
 */
async function getUserById(userId, options = {}) { ... }
```

---

## 12. 相等比较

- 始终使用 `===` 和 `!==`（严格相等）
- 禁止使用 `==` 和 `!=`
- 对象比较使用深度比较库或手动比较

```javascript
// 正确
if (value === null) { ... }
if (items.length !== 0) { ... }

// 错误
if (value == null) { ... }
```

---

## 13. 性能规范

- 避免在循环中创建函数或对象
- 高频回调应节流（throttle）或防抖（debounce）
- 大数据集使用 `Map`/`Set` 代替 `Object`/`Array` 查找
- 注意 N+1 查询问题
- 预计算和缓存昂贵操作的结果

---

## 14. 安全规范

### 14.1 输入验证
- 所有外部输入必须在边界处验证
- 使用白名单校验，非黑名单
- 验证：类型、长度、格式、范围

### 14.2 注入防护
- 避免 `eval()`, `new Function()`, `setTimeout(string)`
- SQL: 使用参数化查询
- HTML: 使用 `.textContent` 而非 `.innerHTML`
- URL: 使用 `encodeURIComponent()`

### 14.3 密钥管理
- 禁止硬编码密钥
- 使用环境变量或密钥管理服务
- 禁止打印或记录敏感数据

---

## 15. 工具链配置

### 15.1 ESLint
```json
{
  "extends": ["airbnb-base", "prettier"],
  "env": { "node": true, "es2022": true },
  "rules": {
    "no-var": "error",
    "prefer-const": "error",
    "eqeqeq": ["error", "always"]
  }
}
```

### 15.2 Prettier
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100
}
```

---

## 附录A: 命名速查表

| 类型 | 命名规则 | 示例 |
|------|---------|------|
| 文件 | kebab-case | `user-service.js` |
| 变量 | camelCase | `userName` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 函数 | camelCase（动词开头） | `getUserById()` |
| 类 | PascalCase | `UserService` |
| 布尔值 | is/has + camelCase | `isActive` |
| 私有字段 | #camelCase | `#cache` |
| 事件处理 | handle + 事件名 | `handleClick` |

## 附录B: 禁止事项

- [ ] 使用 `var`
- [ ] 使用 `==` / `!=`
- [ ] 未处理的 Promise rejection
- [ ] 循环依赖
- [ ] 硬编码密钥或敏感信息
- [ ] 使用 `eval()` 或 `new Function()`
- [ ] 空的 catch 块
- [ ] 注释掉的代码
- [ ] 超过 3 个参数的函数
- [ ] 深层嵌套（>3 层）
