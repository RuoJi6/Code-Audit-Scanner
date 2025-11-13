# 代码审计规则说明文档

## 📋 概述

本项目包含了针对多种编程语言的代码安全审计规则集，用于自动化检测代码中的安全漏洞和风险点。

## 🎯 风险等级分类

所有规则按照风险等级分为四个级别：

### 🔴 Critical（严重）
- **描述**：可直接导致远程代码执行、系统命令执行、任意文件读写等严重安全问题
- **典型漏洞**：
  - 命令注入（Command Injection）
  - 代码注入（Code Injection）
  - 反序列化漏洞（Deserialization）
  - 任意类加载（Arbitrary Class Loading）

### 🟠 High（高危）
- **描述**：可能导致数据泄露、权限绕过、XXE、SQL注入等高危安全问题
- **典型漏洞**：
  - SQL注入（SQL Injection）
  - XXE（XML External Entity）
  - LDAP注入（LDAP Injection）
  - 不安全的反序列化
  - 路径遍历（Path Traversal）

### 🟡 Medium（中危）
- **描述**：可能导致信息泄露、SSRF、弱加密等中等风险问题
- **典型漏洞**：
  - SSRF（Server-Side Request Forgery）
  - 弱加密算法使用
  - 不安全的随机数生成
  - 敏感信息泄露
  - 文件操作风险

### 🟢 Low（低危）
- **描述**：代码质量问题、潜在风险点、需要人工审查的代码
- **典型问题**：
  - 异常处理不当
  - 日志记录问题
  - 线程安全问题
  - 资源泄露风险

## 📝 规则格式说明

### 基本格式

```yaml
language: 语言名称
version: "版本号"
description: "规则集描述"

rules:
  critical:
    - name: "规则名称"
      function: "函数/方法名"
      description: "详细描述"
      regex: true/false
      pattern: "匹配模式"
```

### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | String | ✅ | 规则的显示名称 |
| `function` | String | ✅ | 要检测的函数/方法名称 |
| `description` | String | ✅ | 规则的详细描述，说明风险点 |
| `regex` | Boolean | ✅ | 是否使用正则表达式匹配 |
| `pattern` | String | ✅ | 匹配模式（正则或字符串） |
| `caseSensitive` | Boolean | ❌ | 是否区分大小写（默认true） |

### 高级格式（.NET规则）

.NET规则支持多模式匹配：

```yaml
- name: "规则名称"
  function: "函数名"
  description: "描述"
  regex: true
  pattern: "主模式"  # 向后兼容
  patterns:
    - "模式1"
    - "模式2"
    - "模式3"
  matchLogic: "AND"  # AND: 所有模式都必须匹配; OR: 任意模式匹配（默认）
```

## 🔍 正则表达式模式说明

### 1. 变量声明匹配

**模式**：`ClassName\s+(\w+)`

**匹配示例**：
```java
ScriptEngine engine = manager.getEngineByName("js");  // ✅ 匹配
ScriptEngine engine;                                   // ✅ 匹配
public void test(ScriptEngine engine) { }              // ✅ 匹配
```

**不匹配**：
```java
import javax.script.ScriptEngine;  // ❌ 不匹配
// ScriptEngine 在注释中            // ❌ 不匹配
```

### 2. 对象实例化匹配

**模式**：`new\s+ClassName\s*\(`

**匹配示例**：
```java
ProcessBuilder pb = new ProcessBuilder("cmd");  // ✅ 匹配
new ProcessBuilder(args);                       // ✅ 匹配
```

### 3. 方法调用匹配

**模式**：`ClassName\.methodName\s*\(`

**匹配示例**：
```java
Runtime.getRuntime().exec(cmd);           // ✅ 匹配
HttpClients.createDefault();              // ✅ 匹配
```

### 4. 实例方法调用匹配

**模式**：`\.methodName\s*\(`

**匹配示例**：
```java
engine.eval(code);        // ✅ 匹配
connection.execute(sql);  // ✅ 匹配
```

### 5. 泛型支持

**模式**：`ClassName(<[^>]+>)?\s+(\w+)`

**匹配示例**：
```java
Constructor<String> ctor = clazz.getConstructor();     // ✅ 匹配
Constructor ctor = clazz.getConstructor();             // ✅ 匹配
MongoCollection<Document> collection = db.getCollection();  // ✅ 匹配
```

## 📚 各语言规则详解

### Java 规则 (223条)

#### Critical 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| Runtime.exec命令执行 | `Runtime.getRuntime().exec(` | 执行系统命令，可能存在命令注入风险 |
| ProcessBuilder命令执行 | `new ProcessBuilder(` | 构建并执行系统命令，可能存在命令注入风险 |
| ScriptEngine代码执行 | `ScriptEngine` | 执行脚本代码，可能存在代码注入风险 |
| Method反射调用 | `Method` | 反射调用方法，可能被利用执行任意代码 |
| ClassLoader类加载器 | `ClassLoader` | 类加载器加载类，可能加载恶意类 |
| TemplatesImpl模板注入 | `new TemplatesImpl(` | 常用于Java反序列化漏洞利用 |
| InvokerTransformer命令执行 | `new InvokerTransformer(` | 常用于反序列化攻击链 |

#### High 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| XXE漏洞 | `DocumentBuilderFactory` | XML解析，可能存在XXE漏洞 |
| XPathExpression XPath | `XPathExpression` | XPath表达式求值，可能存在XPath注入 |
| LDAP注入 | `DirContext`, `LdapContext` | LDAP目录搜索，可能存在LDAP注入 |
| SQL注入 | `Statement.executeQuery(` | 执行SQL查询，可能存在SQL注入 |
| HQL注入 | `Session.createQuery(` | Hibernate查询，可能存在HQL注入 |
| NoSQL注入 | `MongoDatabase`, `MongoCollection` | MongoDB查询，可能存在NoSQL注入 |

#### Medium 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SSRF风险 | `HttpURLConnection`, `OkHttpClient` | HTTP请求，注意SSRF风险 |
| 弱加密算法 | `DES.getInstance(`, `MD5.getInstance(` | 使用弱加密算法，不安全 |
| 不安全随机数 | `new Random(` | 使用不安全的随机数生成器 |
| 文件操作 | `new FileInputStream(`, `new FileOutputStream(` | 文件读写操作，注意路径遍历 |

#### Low 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| 异常处理 | `Exception`, `Throwable` | 异常类，可能泄露敏感信息 |
| 反射操作 | `Field`, `AccessibleObject` | 反射访问，可能绕过访问控制 |
| 线程操作 | `Thread` | 线程类，注意线程安全 |

---

### .NET/C# 规则 (180条)

#### Critical 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| Process.Start命令执行 | `Process.Start(` | 执行系统命令，可能存在命令注入 |
| BinaryFormatter反序列化 | `BinaryFormatter.Deserialize(` | 不安全的反序列化器，可导致RCE |
| NetDataContractSerializer | `NetDataContractSerializer` | 可能导致反序列化漏洞 |
| ObjectStateFormatter | `ObjectStateFormatter.Deserialize(` | ViewState反序列化攻击 |
| AjaxMethod反序列化 | `[AjaxMethod]` + `object` | Ajax方法中使用object参数的反序列化风险 |
| WCF反序列化 | `[ServiceContract]` + `object` | WCF服务反序列化漏洞 |

#### High 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SQL注入 | `SqlCommand.ExecuteReader(` | SQL查询执行，注意SQL注入 |
| XPath注入 | `XPathNavigator.Evaluate(` | XPath表达式求值，可能存在注入 |
| LDAP注入 | `DirectorySearcher.FindAll(` | LDAP搜索，可能存在LDAP注入 |
| XXE漏洞 | `XmlDocument.Load(` | XML解析，可能存在XXE漏洞 |

#### Medium 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SSRF风险 | `HttpWebRequest`, `WebClient` | HTTP请求，注意SSRF风险 |
| 弱加密算法 | `DES.Create(`, `MD5.Create(` | 使用弱加密算法 |
| 路径遍历 | `File.ReadAllText(`, `File.WriteAllText(` | 文件操作，注意路径遍历 |

---

### Python 规则 (199条)

#### Critical 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| eval函数 | `eval(` | 执行Python表达式，可能存在代码注入 |
| exec函数 | `exec(` | 执行Python代码，可能存在代码注入 |
| os.system命令执行 | `os.system(` | 执行系统命令，可能存在命令注入 |
| subprocess命令执行 | `subprocess.call(`, `subprocess.run(` | 执行系统命令，注意shell=True |
| pickle反序列化 | `pickle.loads(` | 不安全的反序列化，可导致RCE |
| __import__动态导入 | `__import__(` | 动态导入模块，可能导入恶意模块 |

#### High 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SQL注入 | `.execute(`, `.executemany(` | SQL执行，可能存在SQL注入 |
| YAML反序列化 | `yaml.load(` | 不安全的YAML加载，应使用safe_load |
| XML解析 | `xml.etree.ElementTree.parse(` | XML解析，可能存在XXE |

#### Medium 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SSRF风险 | `requests.get(`, `urllib.request.urlopen(` | HTTP请求，注意SSRF |
| 弱加密 | `hashlib.md5(`, `hashlib.sha1(` | 使用弱哈希算法 |
| 文件操作 | `open(` | 文件操作，注意路径遍历 |

---

### PHP 规则 (162条)

#### Critical 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| eval函数 | `eval(` | 执行任意PHP代码，严重的代码注入风险 |
| system函数 | `system(` | 执行外部程序，存在命令注入风险 |
| exec函数 | `exec(` | 执行外部程序，存在命令注入风险 |
| shell_exec函数 | `shell_exec(` | 通过shell执行命令 |
| 反引号命令执行 | `` `command` `` | 反引号执行系统命令 |
| assert函数 | `assert(` | PHP 7之前版本可执行任意代码 |
| create_function | `create_function(` | 动态创建函数，可能导致代码注入 |

#### High 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| unserialize函数 | `unserialize(` | 反序列化，可能存在对象注入漏洞 |
| 文件包含 | `include(`, `require(` | 包含文件，可能存在文件包含漏洞 |
| SQL注入 | `mysql_query(`, `mysqli_query(` | SQL查询，可能存在SQL注入 |
| XML解析 | `simplexml_load_string(` | XML解析，可能存在XXE |

#### Medium 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SSRF风险 | `file_get_contents(`, `curl_exec(` | HTTP请求，注意SSRF |
| 弱加密 | `md5(`, `sha1(` | 使用弱哈希算法 |
| 文件操作 | `fopen(`, `file_put_contents(` | 文件操作，注意路径遍历 |

---

### JavaScript/Node.js 规则 (199条)

#### Critical 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| eval函数 | `eval(` | 执行JavaScript代码，可能存在代码注入 |
| Function构造函数 | `new Function(` | 动态创建函数，可能存在代码注入 |
| child_process.exec | `child_process.exec(` | 执行系统命令，可能存在命令注入 |
| vm.runInNewContext | `vm.runInNewContext(` | 运行代码，可能存在沙箱逃逸 |
| require动态加载 | `require($` | 动态加载模块，注意路径来源 |

#### High 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SQL注入 | `.query(`, `.execute(` | SQL执行，可能存在SQL注入 |
| NoSQL注入 | `.find(`, `.findOne(` | MongoDB查询，可能存在NoSQL注入 |
| 原型污染 | `Object.assign(`, `_.merge(` | 对象合并，可能导致原型污染 |

#### Medium 级别示例

| 规则名称 | 函数/方法 | 风险描述 |
|---------|----------|---------|
| SSRF风险 | `http.request(`, `axios.get(` | HTTP请求，注意SSRF |
| 路径遍历 | `fs.readFile(`, `fs.writeFile(` | 文件操作，注意路径遍历 |
| 不安全的随机数 | `Math.random()` | 不应用于安全相关场景 |

---

### SQL 规则 (61条)

#### Critical 级别

| 规则名称 | SQL语句 | 风险描述 |
|---------|---------|---------|
| DROP DATABASE | `DROP DATABASE` | 删除数据库 |
| DROP TABLE | `DROP TABLE` | 删除表 |
| TRUNCATE TABLE | `TRUNCATE TABLE` | 清空表数据 |
| DELETE FROM | `DELETE FROM` | 删除数据 |

#### High 级别

| 规则名称 | SQL语句 | 风险描述 |
|---------|---------|---------|
| ALTER TABLE | `ALTER TABLE` | 修改表结构 |
| DROP COLUMN | `DROP COLUMN` | 删除列 |
| xp_cmdshell | `xp_cmdshell` | SQL Server执行系统命令 |

#### Medium 级别

| 规则名称 | SQL语句 | 风险描述 |
|---------|---------|---------|
| SELECT | `SELECT` | 查询数据 |
| UPDATE | `UPDATE` | 更新数据 |
| INSERT INTO | `INSERT INTO` | 插入数据 |

---

## 🛠️ 使用建议

### 1. 规则优先级

建议按照以下优先级处理检测结果：

1. **Critical** - 立即修复，这些问题可能导致系统被完全攻陷
2. **High** - 高优先级修复，可能导致数据泄露或权限绕过
3. **Medium** - 中等优先级，需要评估实际风险后修复
4. **Low** - 低优先级，可以在代码重构时一并处理

### 2. 误报处理

某些规则可能产生误报，建议：

- 人工审查所有 Critical 和 High 级别的检测结果
- 对于确认的误报，可以在代码中添加注释说明
- 考虑上下文，判断是否真的存在安全风险

### 3. 规则定制

可以根据项目需求：

- 调整规则的风险等级
- 添加项目特定的规则
- 禁用不适用的规则
- 修改正则表达式以提高准确性

### 4. 最佳实践

- **定期更新规则**：随着新漏洞的发现，及时更新规则集
- **集成到CI/CD**：将代码审计集成到持续集成流程中
- **培训开发人员**：让开发人员了解常见的安全问题
- **结合其他工具**：代码审计工具应与其他安全测试工具配合使用

---

## 📖 参考资料

### 安全标准
- OWASP Top 10
- CWE (Common Weakness Enumeration)
- SANS Top 25

### 相关文档
- [OWASP Code Review Guide](https://owasp.org/www-project-code-review-guide/)
- [CWE/SANS TOP 25](https://www.sans.org/top25-software-errors/)
- [NIST Secure Software Development Framework](https://csrc.nist.gov/projects/ssdf)

---

## 📝 更新日志

### Version 1.0 (当前版本)
- ✅ 支持 Java、.NET、Python、PHP、JavaScript、SQL 六种语言
- ✅ 包含 1024 条安全审计规则
- ✅ 四级风险分类系统
- ✅ 正则表达式优化，减少误报
- ✅ 支持变量声明、对象实例化、方法调用等多种匹配模式
- ✅ .NET规则支持多模式匹配（AND/OR逻辑）

---

## 🤝 贡献指南

欢迎贡献新的规则或改进现有规则！

### 贡献步骤
1. Fork 本项目
2. 创建特性分支
3. 添加或修改规则
4. 测试规则准确性
5. 提交 Pull Request

### 规则编写规范
- 规则名称应清晰描述检测内容
- 描述字段应说明风险类型和影响
- 正则表达式应尽量精确，避免误报
- 为新规则添加测试用例

---

## 📄 许可证

本规则集遵循项目主许可证。

---

## 📧 联系方式

如有问题或建议，请通过以下方式联系：

- 提交 Issue
- 发起 Discussion
- 提交 Pull Request

---

**最后更新时间**: 2025-11-13

