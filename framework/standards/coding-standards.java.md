# Java 编码规范

## 文档元数据
- **语言:** Java
- **版本:** 17 LTS+（推荐 21 LTS）
- **构建工具:** Maven / Gradle
- **测试框架:** JUnit 5, Mockito
- **参考标准:** Google Java Style Guide, Oracle Java Code Conventions, Effective Java (Bloch)

---

## 1. 命名规范

### 1.1 文件命名
- **类文件:** 类名 + `.java` — `UserService.java`
- **每个文件一个顶层类**（内部类除外）
- **包名:** 全小写，点号分隔 — `com.example.user.service`
- **模块目录:** 与包名对应 — `src/main/java/com/example/user/service/`

### 1.2 变量命名
- **实例/局部变量:** camelCase — `private String userName;`
- **常量（static final）:** UPPER_SNAKE_CASE — `private static final int MAX_RETRY_COUNT = 3;`
- **枚举常量:** UPPER_SNAKE_CASE — `OrderStatus.PENDING_PAYMENT`
- **布尔值:** `is`/`has`/`can`/`should` 前缀 — `isActive`, `hasPermission`

### 1.3 方法命名
- **模式:** camelCase，动词开头
- **访问器:** `get` / `is` — `getName()`, `isActive()`
- **修改器:** `set` — `setName(String name)`
- **工厂方法:** `create` / `of` / `from` / `valueOf`
- **构建器:** `build` — `build()`

```java
// 正确
public User findById(Long id) { ... }
public boolean isValidEmail(String email) { ... }
public static User of(String name, String email) { ... }

// 错误
public User user(Long id) { ... }         // 缺少动词
public boolean validate(String s) { ... } // 参数命名不清晰
```

### 1.4 类/接口命名
- **类:** PascalCase，名词或名词短语 — `UserService`, `OrderRepository`
- **接口:** PascalCase，无 I 前缀 — `Repository`, `Serializable`
- **抽象类:** PascalCase — `AbstractRepository`, `BaseController`
- **异常类:** 以 `Exception` 结尾 — `UserNotFoundException`
- **测试类:** 被测试类名 + `Test` — `UserServiceTest`

```java
// 正确
public interface UserRepository { ... }
public abstract class AbstractRepository<T> { ... }
public class UserNotFoundException extends RuntimeException { ... }

// 错误
public interface IUserRepository { ... }    // 不使用 I 前缀
public class UserNotFound extends Exception { ... }  // 命名不完整
```

---

## 2. 代码结构

### 2.1 文件组织
```
1. package 声明
2. import 语句（分组：JDK → 第三方 → 项目内部 → static import）
3. 类/接口 Javadoc
4. 类声明
5. 静态常量
6. 静态变量
7. 实例变量
8. 构造函数
9. 静态方法
10. 公共方法
11. 受保护方法
12. 私有方法
13. 内部类/接口
```

```java
package com.example.user.service;

import java.util.List;
import java.util.Optional;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.user.model.User;
import com.example.user.repository.UserRepository;

import static java.util.stream.Collectors.toList;

/**
 * 用户业务服务。
 * 处理用户注册、查询、更新等核心业务逻辑。
 */
@Service
public class UserService {

    private static final int MAX_PAGE_SIZE = 100;

    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    @Transactional
    public User createUser(CreateUserRequest request) {
        validateRequest(request);
        User user = User.of(request);
        User saved = userRepository.save(user);
        emailService.sendWelcomeEmail(saved);
        return saved;
    }

    private void validateRequest(CreateUserRequest request) {
        ...
    }
}
```

### 2.2 方法规范
- **最大行数:** 30 行（软限制）
- **最大参数:** 3 个（超出使用 DTO/参数对象）
- **单一职责:** 一个方法只做一件事
- **尽早返回:** 减少深层嵌套
- **避免 boolean 参数:** 使用枚举或拆分为两个方法

```java
// 正确：使用枚举代替 boolean
public List<User> findUsers(UserStatus status) { ... }

// 错误：boolean 参数语义不清
public List<User> findUsers(boolean includeDisabled) { ... }
```

---

## 3. 格式化

### 3.1 缩进
- **风格:** 空格
- **宽度:** 4 个空格（续行缩进 8 个空格）
- **不使用 Tab**

### 3.2 行长度
- **最大:** 120 字符
- **例外:** 长 URL、Javadoc 中的 `<pre>` 块

### 3.3 大括号
- **风格:** K&R（Egyptian braces）— 左大括号不换行
- **所有控制语句必须使用大括号**，即使只有一行
- **空块:** `{}` 或 `{ /* empty */ }`

```java
// 正确
if (condition) {
    doSomething();
} else {
    doOtherwise();
}

// 错误
if (condition) doSomething();        // 缺少大括号
if (condition)
    doSomething();                   // 缺少大括号
```

### 3.4 空格
- 关键字后加空格: `if (`, `for (`, `while (`, `catch (`
- 二元运算符两侧加空格: `a + b`, `x == y`
- 逗号后加空格: `method(a, b, c)`
- 强制类型转换不加空格（括号内）: `(String)obj`
- 方法名与括号之间无空格: `method()`

### 3.5 每行一条语句
- 声明一个变量一行
- 语句结束后不添加多余空格

---

## 4. 类设计

### 4.1 封装
- 字段必须 `private`（除非是 static final 常量）
- 通过 getter/setter 访问
- 不可变类优先（所有字段 `final`，无 setter）

```java
// 不可变类
public final class User {
    private final Long id;
    private final String name;
    private final String email;
    private final LocalDateTime createdAt;

    private User(Builder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.email = builder.email;
        this.createdAt = builder.createdAt;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public LocalDateTime getCreatedAt() { return createdAt; }

    public static class Builder {
        private Long id;
        private String name;
        private String email;
        private LocalDateTime createdAt = LocalDateTime.now();

        public Builder id(Long id) { this.id = id; return this; }
        public Builder name(String name) { this.name = name; return this; }
        public Builder email(String email) { this.email = email; return this; }

        public User build() {
            Objects.requireNonNull(name, "name must not be null");
            Objects.requireNonNull(email, "email must not be null");
            return new User(this);
        }
    }
}
```

### 4.2 继承
- 使用 `@Override` 注解所有重写方法
- 优先使用组合代替继承
- 继承深度不超过 3 层

### 4.3 接口设计
- 接口应小而专注（接口隔离原则）
- 默认方法谨慎使用
- 函数式接口标注 `@FunctionalInterface`

---

## 5. 注解

### 5.1 常用注解使用规范
```java
// 必须使用
@Override           // 所有重写方法
@Deprecated         // 废弃方法，Javadoc 中注明替代方案
@FunctionalInterface // 函数式接口

// 谨慎使用
@SuppressWarnings   // 最小作用域，注明原因
```

### 5.2 Lombok（如使用）
```java
// 推荐使用的注解
@Data               // 仅用于纯数据类
@Value              // 不可变数据类（推荐优先使用）
@Builder            // 构建器模式
@RequiredArgsConstructor  // 构造器注入（Spring）
@Slf4j              // 日志

// 避免使用
@SneakyThrows       // 隐藏受检异常
@Cleanup            // 不如 try-with-resources 清晰
```

---

## 6. 异常处理

### 6.1 异常类型
```java
// 业务异常（非受检）
public class UserNotFoundException extends RuntimeException {
    private final Long userId;

    public UserNotFoundException(Long userId) {
        super("User not found with id: " + userId);
        this.userId = userId;
    }

    public Long getUserId() { return userId; }
}

// 验证异常
public class ValidationException extends RuntimeException {
    private final List<FieldError> errors;

    public ValidationException(String message, List<FieldError> errors) {
        super(message);
        this.errors = List.copyOf(errors);
    }

    public List<FieldError> getErrors() { return errors; }
}
```

### 6.2 异常处理规则
- 使用非受检异常（RuntimeException）处理可恢复错误
- 使用受检异常（Exception）处理调用者必须处理的场景
- 空 catch 块禁止
- 捕获异常时不要吞噬（至少记录日志）

```java
// 正确
try {
    userRepository.save(user);
} catch (DataIntegrityViolationException e) {
    log.error("Failed to save user: {}", user.getId(), e);
    throw new UserServiceException("Unable to save user", e);
}

// 错误
try {
    userRepository.save(user);
} catch (Exception e) {
    // 空 catch — 禁止
}
```

### 6.3 try-with-resources
```java
// 正确：自动关闭资源
try (var reader = new BufferedReader(new FileReader(path))) {
    return reader.lines().collect(toList());
}
```

---

## 7. 集合与流

### 7.1 集合选择
- **List:** `ArrayList`（随机访问）, `LinkedList`（频繁插入删除）
- **Set:** `HashSet`（通用）, `TreeSet`（有序）, `LinkedHashSet`（插入顺序）
- **Map:** `HashMap`（通用）, `TreeMap`（有序键）, `LinkedHashMap`（插入顺序）
- **声明类型使用接口:** `List<User>` 而非 `ArrayList<User>`

### 7.2 不可变集合（推荐）
```java
// Java 9+
List<String> items = List.of("a", "b", "c");
Map<String, Integer> scores = Map.of("Alice", 95, "Bob", 87);
Set<String> tags = Set.of("java", "spring");

// 复制防御
public List<User> getUsers() {
    return List.copyOf(users);  // 返回不可变副本
}
```

### 7.3 Stream API
```java
// 正确：声明式管道
List<String> activeUserEmails = users.stream()
    .filter(User::isActive)
    .map(User::getEmail)
    .sorted()
    .collect(toList());

// 避免副作用的流操作
// 错误
users.stream().forEach(u -> u.setProcessed(true));  // 副作用

// 正确：使用传统 for-each
for (User user : users) {
    user.setProcessed(true);
}
```

---

## 8. 空值处理

### 8.1 Optional 使用
```java
// 正确：作为返回值
public Optional<User> findById(Long id) {
    return Optional.ofNullable(userRepository.findById(id));
}

// 正确：使用 Optional 方法链
public String getUserDisplayName(Long id) {
    return findById(id)
        .map(User::getName)
        .orElse("Unknown User");
}

// 错误：Optional 作为字段
public class User {
    private Optional<String> nickname;  // 禁止
}

// 错误：Optional 作为参数
public void process(Optional<User> user) { ... }  // 禁止
```

### 8.2 空值检查
```java
// 构造器或关键方法中尽早检查
public UserService(UserRepository repository) {
    this.repository = Objects.requireNonNull(repository, "repository must not be null");
}

// 注解辅助
public void processUser(@NonNull User user) { ... }
```

---

## 9. 线程安全

### 9.1 不可变类（首选）
```java
public final class Configuration {
    private final Map<String, String> properties;

    public Configuration(Map<String, String> properties) {
        this.properties = Map.copyOf(properties);  // 防御性拷贝
    }

    public Map<String, String> getProperties() {
        return properties;  // 已不可变，安全返回
    }
}
```

### 9.2 并发集合
```java
// 选择正确的并发集合
ConcurrentHashMap<String, User> cache = new ConcurrentHashMap<>();
CopyOnWriteArrayList<EventListener> listeners = new CopyOnWriteArrayList<>();
```

### 9.3 同步
- 优先使用 `java.util.concurrent` 包中的高级工具
- 避免使用 `synchronized(this)`
- 使用 `volatile` 确保可见性

---

## 10. 测试规范

### 10.1 测试类结构
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    private User testUser;

    @BeforeEach
    void setUp() {
        testUser = new User.Builder()
            .id(1L)
            .name("Alice")
            .email("alice@example.com")
            .build();
    }

    @Test
    @DisplayName("创建用户成功时返回保存的用户并发送欢迎邮件")
    void shouldReturnSavedUserAndSendEmailWhenCreateUserSucceeds() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("Alice", "alice@example.com");
        when(userRepository.save(any(User.class))).thenReturn(testUser);

        // Act
        User result = userService.createUser(request);

        // Assert
        assertThat(result.getName()).isEqualTo("Alice");
        verify(emailService).sendWelcomeEmail(testUser);
    }

    @Test
    @DisplayName("创建用户时邮箱已存在则抛出 DuplicateEmailException")
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        ...
    }
}
```

### 10.2 测试命名
- **模式:** `should[预期行为]When[条件]`
- 使用 `@DisplayName` 提供中文描述

### 10.3 测试原则
- **AAA 模式:** Arrange — Act — Assert
- **一个测试一个概念**
- **独立测试:** 测试之间不依赖执行顺序
- **快速:** 单元测试应在毫秒级完成

---

## 11. 注释与文档

### 11.1 Javadoc
- 所有公共类和公共方法必须有 Javadoc
- 描述"为什么"而非"是什么"

```java
/**
 * 根据唯一标识查找用户。
 *
 * <p>查询顺序：缓存 → 主库 → 只读副本。
 * 未找到时返回 {@code Optional.empty()}。
 *
 * @param id 用户唯一标识，不能为 {@code null}
 * @return 包含用户的 Optional，未找到时为空
 * @throws IllegalArgumentException 当 id 为 null 时
 */
public Optional<User> findById(Long id) {
    Objects.requireNonNull(id, "id must not be null");
    ...
}
```

### 11.2 行内注释
- 使用 `//` 解释非显而易见的逻辑
- 不注释代码做了什么，注释为什么这么做
- 不保留注释掉的代码

---

## 12. 性能规范

- 字符串拼接循环中使用 `StringBuilder`（非循环中使用 `+`）
- 大对象创建使用 Builder 模式
- 集合初始化时预估容量：`new ArrayList<>(expectedSize)`
- 日志使用占位符而非字符串拼接
- 数据库查询注意 N+1 问题

```java
// 正确：日志使用占位符
log.info("User {} created successfully at {}", user.getId(), user.getCreatedAt());

// 错误：字符串拼接（即使日志级别关闭也会执行）
log.info("User " + user.getId() + " created successfully");
```

---

## 13. 安全规范

### 13.1 输入验证
- 所有外部输入在边界处验证
- 使用 Bean Validation（`@NotNull`, `@Size`, `@Email` 等）
- 白名单校验优于黑名单

### 13.2 SQL 注入防护
- 使用参数化查询或 JPA/Hibernate
- 禁止拼接 SQL 字符串

```java
// 正确：参数化查询
jdbcTemplate.query("SELECT * FROM users WHERE id = ?", userMapper, userId);

// 错误：字符串拼接
jdbcTemplate.query("SELECT * FROM users WHERE id = " + userId, userMapper);
```

### 13.3 密钥管理
- 禁止硬编码密码、密钥、Token
- 使用环境变量、Vault、或配置服务
- 日志禁止输出敏感信息

---

## 14. 工具链配置

### 14.1 Checkstyle（Google Style）
```xml
<!-- checkstyle.xml -->
<module name="Checker">
  <module name="TreeWalker">
    <module name="AvoidStarImport"/>
    <module name="IllegalImport"/>
    <module name="UnusedImports"/>
    <module name="MethodLength">
      <property name="max" value="30"/>
    </module>
    <module name="ParameterNumber">
      <property name="max" value="3"/>
    </module>
  </module>
</module>
```

### 14.2 SpotBugs
```xml
<plugin>
  <groupId>com.github.spotbugs</groupId>
  <artifactId>spotbugs-maven-plugin</artifactId>
  <version>4.8.0</version>
</plugin>
```

### 14.3 编译器设置
```xml
<properties>
  <java.version>21</java.version>
  <maven.compiler.release>21</maven.compiler.release>
  <maven.compiler.parameters>true</maven.compiler.parameters>
</properties>
```

---

## 附录A: 命名速查表

| 类型 | 规则 | 示例 |
|------|------|------|
| 包 | 全小写 | `com.example.user.service` |
| 类 | PascalCase | `UserService` |
| 接口 | PascalCase（无 I 前缀） | `UserRepository` |
| 抽象类 | PascalCase，Abstract 前缀 | `AbstractRepository` |
| 方法 | camelCase（动词开头） | `findById()` |
| 变量 | camelCase | `userName` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| 枚举 | PascalCase | `OrderStatus` |
| 枚举常量 | UPPER_SNAKE_CASE | `PENDING_PAYMENT` |
| 异常类 | PascalCase + Exception 后缀 | `UserNotFoundException` |
| 测试类 | 被测类名 + Test | `UserServiceTest` |
| 布尔值 | is/has/can/should | `isActive` |

## 附录B: 禁止事项

- [ ] 使用通配符导入 `import java.util.*`
- [ ] 空 catch 块
- [ ] 捕获 `Exception` 而不处理或重新抛出
- [ ] Optional 作为字段或方法参数
- [ ] 在 Stream 的 forEach 中制造副作用
- [ ] 字符串拼接用于 SQL/HQL 查询
- [ ] 硬编码密钥、密码
- [ ] 使用已废弃的 API
- [ ] 暴露可变内部状态
- [ ] 超过 3 层继承深度
