# 代码审计规则说明文档

## 📋 概述

本项目包含了针对多种编程语言的代码安全审计规则集，用于自动化检测代码中的安全漏洞和风险点。

### 📊 规则统计

| 语言 | 规则数量 | 文件名 |
|------|---------|--------|
| Java | 223 | `java.yaml` |
| .NET/C# | 180 | `net.yaml` |
| Python | 199 | `python.yaml` |
| PHP | 162 | `php.yaml` |
| JavaScript/Node.js | 199 | `js.yaml` |
| SQL | 61 | `sql.yaml` |
| **总计** | **1024** | - |

### 🎨 特性支持

| 特性 | Java | .NET | Python | PHP | JavaScript | SQL |
|------|------|------|--------|-----|------------|-----|
| 正则匹配 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 精确匹配 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 全词匹配 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 大小写控制 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 多模式匹配 | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| AND/OR 逻辑 | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |

**说明**：
- ✅ 所有语言都支持基本的三种匹配类型和大小写控制
- ✅ .NET 规则集广泛使用多模式匹配（13个规则使用 AND 逻辑）
- ⚠️ 其他语言暂未使用多模式匹配，但代码层面已支持

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
| `regex` | Int/Boolean | ✅ | 匹配类型：1/true=正则，2/false=精确，3=全词 |
| `pattern` | String | ✅ | 单个匹配模式（正则或字符串） |
| `caseSensitive` | Boolean | ❌ | 是否区分大小写（默认true） |
| `patterns` | List<String> | ❌ | 多个匹配模式（高级功能） |
| `matchLogic` | String | ❌ | 多模式匹配逻辑："AND"或"OR"（默认OR） |

## 🔍 匹配类型说明

本工具支持三种匹配模式，通过 `regex` 字段的不同值来区分：

### 代码中的定义

```kotlin
// ScanRule.kt
companion object {
    const val MATCH_REGEX = 1      // 正则匹配
    const val MATCH_EXACT = 2      // 精确匹配（默认）
    const val MATCH_WHOLE_WORD = 3 // 全词匹配
}
```

| regex 值 | 匹配类型 | UI 显示 | YAML 配置（旧格式） | YAML 配置（新格式-推荐） |
|---------|---------|---------|-------------------|---------------------|
| `2` | 精确匹配 | "不启用匹配（默认精确）" | `regex: false` 或不写 | `regex: 2` |
| `1` | 正则匹配 | "启用正则匹配" | `regex: true` | `regex: 1` |
| `3` | 全词匹配 | "启用全词匹配" | ❌ 不支持 | `regex: 3` |

---

### 1. 不启用匹配（默认精确匹配）- regex = 2

**YAML配置（两种格式都支持）**：
```yaml
# 旧格式（Boolean）
- name: "eval函数"
  function: "eval("
  description: "执行代码"
  regex: false  # 或者不写这一行，默认就是 false
  pattern: "eval("

# 新格式（Int）- 推荐
- name: "eval函数"
  function: "eval("
  description: "执行代码"
  regex: 2  # 明确指定为精确匹配
  pattern: "eval("
```

**代码实现**：
```kotlin
code.contains(pattern, ignoreCase = !caseSensitive)
```

**特点**：
- ✅ 精确字符串匹配（使用 `String.contains()`）
- ✅ 性能最快
- ✅ 适合匹配固定的函数名或关键字
- ❌ 无法处理空格、换行等变化
- ❌ 不支持正则表达式语法

**匹配示例**：
```java
// ✅ 会匹配
eval("code");

// ❌ 不会匹配
eval ("code");  // 有空格
eval  ("code"); // 多个空格
eval(           // 有换行
  "code"
);
```

**适用场景**：
- SQL 关键字：`DROP TABLE`, `DELETE FROM`
- 固定格式的函数调用：`eval(`, `exec(`
- 简单的字符串检测

---

### 2. 启用正则匹配 - regex = 1

**YAML配置（两种格式都支持）**：
```yaml
# 旧格式（Boolean）
- name: "eval函数"
  function: "eval("
  description: "执行代码"
  regex: true  # 会被转换为 1
  pattern: "eval\\s*\\("  # 使用正则表达式

# 新格式（Int）- 推荐
- name: "eval函数"
  function: "eval("
  description: "执行代码"
  regex: 1  # 明确指定为正则匹配
  pattern: "eval\\s*\\("  # 使用正则表达式
```

**代码实现**：
```kotlin
val regexPattern = pattern.toRegex()
regexPattern.containsMatchIn(code)
```

**特点**：
- ✅ 支持完整的正则表达式语法
- ✅ 可以处理空格、换行、可选字符等
- ✅ 最灵活、最强大
- ✅ 你写什么正则就匹配什么
- ⚠️ 性能相对较慢
- ⚠️ 需要正确转义特殊字符（`.`, `(`, `)`, `[`, `]` 等）

**匹配示例**：
```java
// pattern: "eval\\s*\\("
// ✅ 都会匹配
eval("code");
eval ("code");
eval  ("code");
eval(
  "code"
);

// pattern: "Runtime\\.getRuntime\\(\\)\\.exec\\s*\\("
// ✅ 匹配
Runtime.getRuntime().exec("cmd");
Runtime.getRuntime().exec ("cmd");
```

**常用正则模式**：
- `\s*` - 匹配0个或多个空白字符
- `\s+` - 匹配1个或多个空白字符
- `\w+` - 匹配单词字符（字母、数字、下划线）
- `\.` - 匹配点号（需要转义）
- `\(` - 匹配左括号（需要转义）
- `\)` - 匹配右括号（需要转义）
- `(<[^>]+>)?` - 匹配可选的泛型参数
- `\b` - 单词边界

**适用场景**：
- 复杂的方法调用：`Runtime\.getRuntime\(\)\.exec\s*\(`
- 变量声明：`ScriptEngine\s+(\w+)`
- 带空格的模式：`eval\s*\(`
- 可选参数：`new\s+ProcessBuilder\s*\(`

---

### 3. 启用全词匹配 - regex = 3

**YAML配置（只支持新格式）**：
```yaml
# ⚠️ 注意：全词匹配只能使用新格式（Int）
- name: "eval函数"
  function: "eval"
  description: "执行代码"
  regex: 3  # 必须明确写 3，表示全词匹配
  pattern: "eval"  # 只写关键字，不需要手动加 \b

# ❌ 旧格式不支持全词匹配
# regex: true  # 这会被转换为 1（正则匹配），不是 3（全词匹配）
```

**代码实现**：
```kotlin
// 自动在 pattern 两边加上 \b，并转义特殊字符
val wordBoundaryPattern = "\\b${Regex.escape(pattern)}\\b".toRegex()
wordBoundaryPattern.containsMatchIn(code)
```

**特点**：
- ✅ 只匹配完整的单词（自动添加 `\b` 边界）
- ✅ 避免部分匹配导致的误报
- ✅ 自动转义特殊字符（使用 `Regex.escape()`）
- ✅ 适合匹配关键字
- ⚠️ 只能匹配单个完整单词
- ⚠️ 不适合复杂的模式（如 `Runtime.getRuntime()`）

**匹配示例**：
```java
// pattern: "eval"
// ✅ 会匹配
eval("code");
var result = eval(input);
if (eval) { }

// ❌ 不会匹配
evaluation();      // eval 是单词的一部分
medieval();        // eval 在单词中间
myeval();          // eval 是标识符的一部分
```

**适用场景**：
- 关键字匹配：`eval`, `exec`, `system`
- 避免误报：匹配 `eval` 但不匹配 `evaluation`
- 简单的单词检测

**⚠️ 不适用场景**：
```yaml
# ❌ 错误用法
pattern: "Runtime.getRuntime().exec("  # 这不是一个"单词"，会被当作整体加 \b
```

---

### 匹配类型对比表

| 特性 | 精确匹配 (regex=2) | 正则匹配 (regex=1) | 全词匹配 (regex=3) |
|------|-------------------|-------------------|-------------------|
| **性能** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **灵活性** | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **准确性** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **处理空格** | ❌ | ✅ | ❌ |
| **正则语法** | ❌ | ✅ 完全支持 | ✅ 自动添加 `\b` |
| **特殊字符** | 不需要转义 | 需要手动转义 | 自动转义 |
| **适用场景** | 固定字符串 | 复杂模式 | 单个关键字 |

---

### ⚠️ 重要说明：YAML 格式兼容性

代码支持**两种 YAML 格式**：

#### 旧格式（Boolean）- 向后兼容
```yaml
regex: true   # 会被转换为 1 (正则匹配)
regex: false  # 会被转换为 2 (精确匹配)
```

**限制**：
- ❌ 无法表示全词匹配（regex=3）
- ⚠️ 只能区分正则匹配和精确匹配

#### 新格式（Int）- 推荐使用
```yaml
regex: 1  # 正则匹配
regex: 2  # 精确匹配
regex: 3  # 全词匹配
```

**优势**：
- ✅ 支持所有三种匹配类型
- ✅ 更明确、更清晰
- ✅ 避免歧义

#### 转换规则

代码中的转换逻辑（`RuleManager.kt`）：
```kotlin
val regex = when (val regexValue = ruleMap["regex"]) {
    is Int -> regexValue  // 新格式：直接使用数字
    is Boolean -> if (regexValue) 1 else 2  // 旧格式：true=1, false=2
    else -> 2  // 默认精确匹配
}
```

| YAML 写法 | 解析结果 | 匹配类型 |
|----------|---------|---------|
| `regex: 1` | 1 | 正则匹配 |
| `regex: 2` | 2 | 精确匹配 |
| `regex: 3` | 3 | 全词匹配 |
| `regex: true` | 1 | 正则匹配 |
| `regex: false` | 2 | 精确匹配 |
| 不写 `regex` | 2 | 精确匹配（默认） |

**建议**：
- 🎯 新规则使用新格式（`regex: 1/2/3`）
- 🔄 旧规则可以继续使用旧格式（向后兼容）
- ⚡ 如果需要全词匹配，必须使用新格式（`regex: 3`）

---

### 推荐使用场景

#### 场景1：使用精确匹配 (regex=2)

**适合**：固定格式的字符串、SQL关键字

```yaml
# SQL 关键字检测（旧格式）
- name: "DROP TABLE"
  function: "DROP TABLE"
  description: "删除表"
  regex: false  # 或不写
  pattern: "DROP TABLE"

# SQL 关键字检测（新格式-推荐）
- name: "DROP TABLE"
  function: "DROP TABLE"
  description: "删除表"
  regex: 2  # 明确指定精确匹配
  pattern: "DROP TABLE"

# 固定函数调用（无空格变化）
- name: "eval函数"
  function: "eval("
  description: "执行代码"
  regex: 2  # 推荐使用新格式
  pattern: "eval("
```

#### 场景2：使用正则匹配 (regex=1)

**适合**：复杂的代码模式、需要处理空格/换行

```yaml
# 方法调用检测（处理空格）- 旧格式
- name: "Runtime.exec命令执行"
  function: "Runtime.getRuntime().exec("
  description: "执行系统命令"
  regex: true  # 旧格式
  pattern: "Runtime\\.getRuntime\\(\\)\\.exec\\s*\\("

# 方法调用检测（处理空格）- 新格式（推荐）
- name: "Runtime.exec命令执行"
  function: "Runtime.getRuntime().exec("
  description: "执行系统命令"
  regex: 1  # 新格式，明确指定正则匹配
  pattern: "Runtime\\.getRuntime\\(\\)\\.exec\\s*\\("

# 变量声明检测
- name: "ScriptEngine代码执行"
  function: "ScriptEngine"
  description: "脚本引擎"
  regex: 1
  pattern: "ScriptEngine\\s+(\\w+)"

# 静态方法调用
- name: "HttpClients.createDefault"
  function: "HttpClients.createDefault("
  description: "创建HTTP客户端"
  regex: 1
  pattern: "HttpClients\\.createDefault\\s*\\("

# 对象实例化
- name: "ProcessBuilder命令执行"
  function: "ProcessBuilder"
  description: "构建系统命令"
  regex: 1
  pattern: "new\\s+ProcessBuilder\\s*\\("
```

#### 场景3：使用全词匹配 (regex=3)

**适合**：单个关键字匹配、避免误报

```yaml
# ⚠️ 注意：全词匹配只能使用新格式（regex: 3）
# 关键字检测（避免误报）
- name: "eval关键字"
  function: "eval"
  description: "eval关键字"
  regex: 3  # 必须明确写 3，不能写 true
  pattern: "eval"  # 不需要手动加 \b，代码会自动添加

# 匹配结果：
# ✅ eval("code")
# ✅ var x = eval(input)
# ❌ evaluation()  （不会误报）
# ❌ medieval()    （不会误报）

# 更多示例
- name: "exec关键字"
  function: "exec"
  description: "exec关键字"
  regex: 3
  pattern: "exec"  # 匹配 exec，但不匹配 execute

- name: "system关键字"
  function: "system"
  description: "system关键字"
  regex: 3
  pattern: "system"  # 匹配 system，但不匹配 filesystem
```

**⚠️ 不要用全词匹配处理复杂模式**：
```yaml
# ❌ 错误用法
- name: "Runtime.exec"
  function: "Runtime.getRuntime().exec("
  description: "执行命令"
  regex: 3  # 全词匹配
  pattern: "Runtime.getRuntime().exec("  # 这不是单个"词"
  # 结果：会被当作整体加 \b，无法正确匹配

# ✅ 正确做法：使用正则匹配
- name: "Runtime.exec"
  function: "Runtime.getRuntime().exec("
  description: "执行命令"
  regex: 1  # 正则匹配
  pattern: "Runtime\\.getRuntime\\(\\)\\.exec\\s*\\("
```

---

### 三种匹配类型的关系图

```
匹配类型层次结构：
├── 精确匹配 (regex=2)
│   └── 使用 String.contains()
│   └── 最快，但最不灵活
│
├── 正则匹配 (regex=1)
│   └── 使用 pattern.toRegex()
│   └── 完全自由的正则表达式
│   └── 最灵活，最强大
│
└── 全词匹配 (regex=3)
    └── 使用 "\\b${Regex.escape(pattern)}\\b".toRegex()
    └── 正则匹配的特殊形式
    └── 自动添加单词边界 + 自动转义
```

**关键理解**：
- **全词匹配是正则匹配的一个特殊子集**
- 全词匹配 = 自动添加 `\b` + 自动转义特殊字符
- 如果你需要更复杂的模式，应该使用正则匹配而不是全词匹配

---

## 🔥 高级特性：多模式匹配

### 1. caseSensitive - 大小写敏感控制

所有三种匹配类型都支持 `caseSensitive` 字段：

```yaml
# 区分大小写（默认）
- name: "eval函数"
  function: "eval("
  description: "执行代码"
  regex: 1
  pattern: "eval\\s*\\("
  caseSensitive: true  # 默认值，可以不写

# 不区分大小写
- name: "eval函数（不区分大小写）"
  function: "eval("
  description: "执行代码"
  regex: 1
  pattern: "eval\\s*\\("
  caseSensitive: false  # 匹配 eval、Eval、EVAL 等
```

**代码实现**：
```kotlin
// 正则匹配
val regexPattern = if (caseSensitive) {
    pattern.toRegex()
} else {
    pattern.toRegex(RegexOption.IGNORE_CASE)
}

// 精确匹配
code.contains(pattern, ignoreCase = !caseSensitive)

// 全词匹配
val wordBoundaryPattern = if (caseSensitive) {
    "\\b${Regex.escape(pattern)}\\b".toRegex()
} else {
    "\\b${Regex.escape(pattern)}\\b".toRegex(RegexOption.IGNORE_CASE)
}
```

**匹配示例**：
```java
// caseSensitive: false
// pattern: "eval\\s*\\("

// ✅ 都会匹配
eval("code");
Eval("code");
EVAL("code");
eVaL("code");
```

---

### 2. 多模式匹配（patterns + matchLogic）

支持在一个规则中定义多个匹配模式，并通过 `matchLogic` 控制逻辑关系。

#### 基本格式

```yaml
- name: "规则名称"
  function: "函数名"
  description: "描述"
  regex: 1  # 匹配类型
  pattern: "主模式"  # 向后兼容，可选
  patterns:  # 多个模式
    - "模式1"
    - "模式2"
    - "模式3"
  matchLogic: "AND"  # AND: 所有模式都必须匹配; OR: 任意模式匹配（默认）
  caseSensitive: true  # 可选，默认true
```

#### matchLogic 说明

| matchLogic 值 | 匹配逻辑 | 说明 |
|--------------|---------|------|
| `"OR"` | 任意模式匹配 | 只要有一个模式匹配就算匹配（默认） |
| `"AND"` | 所有模式都匹配 | 所有模式都必须匹配才算匹配 |

**代码实现**：
```kotlin
when (matchLogic) {
    MatchLogic.AND -> {
        // AND 逻辑：所有模式都必须匹配
        patternsToMatch.all { pattern ->
            matchSinglePattern(code, pattern)
        }
    }
    MatchLogic.OR -> {
        // OR 逻辑：任意一个模式匹配即可
        patternsToMatch.any { pattern ->
            matchSinglePattern(code, pattern)
        }
    }
}
```

---

### 3. 多模式匹配实战示例

#### 示例1：OR 逻辑 - 检测多种危险函数

```yaml
- name: "命令执行函数"
  function: "exec/system/shell_exec"
  description: "检测多种命令执行函数"
  regex: 1
  patterns:
    - "\\bexec\\s*\\("
    - "\\bsystem\\s*\\("
    - "\\bshell_exec\\s*\\("
  matchLogic: "OR"  # 任意一个匹配即可
```

**匹配结果**：
```php
// ✅ 匹配（包含 exec）
exec("ls -la");

// ✅ 匹配（包含 system）
system("whoami");

// ✅ 匹配（包含 shell_exec）
shell_exec("cat /etc/passwd");
```

#### 示例2：AND 逻辑 - 检测复杂的漏洞模式

```yaml
- name: "BinaryFormatter不安全反序列化"
  function: "BinaryFormatter.Deserialize"
  description: "检测BinaryFormatter的Deserialize调用"
  regex: 1
  patterns:
    - "BinaryFormatter"  # 必须包含 BinaryFormatter
    - "\\.Deserialize\\s*\\("  # 必须包含 .Deserialize( 调用
  matchLogic: "AND"  # 两个模式都必须匹配
```

**匹配结果**：
```csharp
// ✅ 匹配（同时包含两个模式）
BinaryFormatter formatter = new BinaryFormatter();
object obj = formatter.Deserialize(stream);

// ❌ 不匹配（只有 BinaryFormatter，没有 Deserialize）
BinaryFormatter formatter = new BinaryFormatter();

// ❌ 不匹配（只有 Deserialize，没有 BinaryFormatter）
formatter.Deserialize(stream);
```

#### 示例3：AND 逻辑 - 检测特定上下文的漏洞

```yaml
- name: "AjaxMethod中的危险类型转换"
  function: "AjaxMethod + Convert"
  description: "检测AjaxMethod中对object参数进行类型转换"
  regex: 1
  patterns:
    - "\\[AjaxMethod\\]"  # 必须有 AjaxMethod 特性
    - "\\bobject\\b"  # 必须有 object 类型参数
    - "Convert\\."  # 必须有类型转换操作
  matchLogic: "AND"  # 三者必须同时出现
```

**匹配结果**：
```csharp
// ✅ 匹配（三个模式都存在）
[AjaxMethod]
public string ProcessData(object data)
{
    string result = Convert.ToString(data);
    return result;
}

// ❌ 不匹配（缺少 AjaxMethod 特性）
public string ProcessData(object data)
{
    string result = Convert.ToString(data);
    return result;
}

// ❌ 不匹配（缺少 Convert 操作）
[AjaxMethod]
public string ProcessData(object data)
{
    return data.ToString();
}
```

#### 示例4：复杂的 OR 模式 - 检测多种序列化器

```yaml
- name: "不安全的序列化器"
  function: "Unsafe Serializers"
  description: "检测多种不安全的序列化器"
  regex: 1
  patterns:
    - "BinaryFormatter"
    - "ObjectStateFormatter"
    - "LosFormatter"
    - "SoapFormatter"
    - "NetDataContractSerializer"
  matchLogic: "OR"  # 任意一个出现即报告
```

---

### 4. 多模式匹配的优势

✅ **减少规则数量**：
- 不需要为每个危险函数创建单独的规则
- 一个规则可以检测多种相似的模式

✅ **提高准确性**：
- AND 逻辑可以减少误报
- 只有满足所有条件才报告

✅ **灵活的组合**：
- 可以组合不同的模式
- 适应复杂的漏洞检测场景

✅ **易于维护**：
- 相关的模式集中在一起
- 修改更方便

---

### 5. 多模式匹配的注意事项

⚠️ **AND 逻辑的限制**：
- 所有模式必须在**同一个文件**中匹配
- 不支持跨文件匹配
- 适合检测同一代码段中的多个特征

⚠️ **性能考虑**：
- AND 逻辑需要检查所有模式
- 模式越多，性能开销越大
- 建议将最具特征性的模式放在前面

⚠️ **正则表达式复杂度**：
- 每个 pattern 都是独立的正则表达式
- 避免过于复杂的正则
- 优先使用简单、高效的模式

---

### 6. 实际应用场景

#### .NET 反序列化检测（使用最多）

```yaml
# 检测 13 种不同的反序列化模式
critical:
  - name: "BinaryFormatter不安全反序列化"
    patterns:
      - "BinaryFormatter"
      - "\\.Deserialize\\s*\\("
    matchLogic: "AND"

  - name: "NetDataContractSerializer反序列化"
    patterns:
      - "NetDataContractSerializer"
      - "\\.Deserialize\\s*\\(|\\.ReadObject\\s*\\("
    matchLogic: "AND"

  # ... 更多反序列化规则
```

#### Java 反射调用检测

```yaml
critical:
  - name: "Method.invoke反射调用"
    patterns:
      - "Method\\s+(\\w+)"  # Method 变量声明
      - "\\.invoke\\s*\\("  # invoke 方法调用
    matchLogic: "AND"
```

#### PHP 文件包含检测

```yaml
high:
  - name: "动态文件包含"
    patterns:
      - "include\\s*\\(|require\\s*\\("  # include/require
      - "\\$_GET|\\$_POST|\\$_REQUEST"  # 用户输入
    matchLogic: "AND"
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

## 🧪 规则测试功能

工具内置了规则测试功能，可以在添加或修改规则时进行测试。

### 测试界面

在规则编辑器中点击"测试正则"按钮，可以：

1. **输入测试代码**：在文本框中输入要测试的代码
2. **查看匹配结果**：显示是否匹配以及匹配的位置
3. **验证匹配类型**：测试精确匹配、正则匹配、全词匹配
4. **验证大小写**：测试 caseSensitive 设置是否正确

### 测试示例

#### 测试正则匹配

```yaml
# 规则配置
regex: 1
pattern: "eval\\s*\\("
caseSensitive: true
```

**测试代码**：
```javascript
eval("code");
eval ("code");
eval  ("code");
```

**测试结果**：
```
✅ [正则模式] 找到 3 处匹配:

匹配 1: eval(
位置: 0..5

匹配 2: eval (
位置: 15..21

匹配 3: eval  (
位置: 32..39
```

#### 测试全词匹配

```yaml
# 规则配置
regex: 3
pattern: "eval"
caseSensitive: false
```

**测试代码**：
```javascript
eval("code");
Eval("code");
evaluation();
```

**测试结果**：
```
✅ [全词模式] 找到 2 处匹配:

匹配 1: eval
位置: 0..4

匹配 2: Eval
位置: 15..19

提示：evaluation() 不会被匹配（不是完整单词）
```

#### 测试多模式匹配

```yaml
# 规则配置
regex: 1
patterns:
  - "BinaryFormatter"
  - "\\.Deserialize\\s*\\("
matchLogic: "AND"
```

**测试代码**：
```csharp
BinaryFormatter formatter = new BinaryFormatter();
object obj = formatter.Deserialize(stream);
```

**测试结果**：
```
✅ [AND 逻辑] 所有模式都匹配:

模式 1: BinaryFormatter ✅
模式 2: \.Deserialize\s*\( ✅

结论：规则匹配成功
```

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
- 使用规则测试功能验证规则的准确性

### 3. 规则定制

可以根据项目需求：

- 调整规则的风险等级
- 添加项目特定的规则
- 禁用不适用的规则
- 修改正则表达式以提高准确性
- 使用多模式匹配减少误报

### 4. 最佳实践

- **定期更新规则**：随着新漏洞的发现，及时更新规则集
- **集成到CI/CD**：将代码审计集成到持续集成流程中
- **培训开发人员**：让开发人员了解常见的安全问题
- **结合其他工具**：代码审计工具应与其他安全测试工具配合使用
- **使用测试功能**：在添加新规则前，务必使用测试功能验证

---

## ❓ 常见问题（FAQ）

### Q1: regex 字段应该写 true/false 还是 1/2/3？

**A**: 推荐使用新格式（数字）：
- `regex: 1` - 正则匹配
- `regex: 2` - 精确匹配
- `regex: 3` - 全词匹配

旧格式（Boolean）仍然支持，但无法表示全词匹配：
- `regex: true` → 转换为 1（正则匹配）
- `regex: false` → 转换为 2（精确匹配）

### Q2: 什么时候使用全词匹配（regex: 3）？

**A**: 全词匹配适合以下场景：
- ✅ 匹配单个关键字：`eval`, `exec`, `system`
- ✅ 避免误报：匹配 `eval` 但不匹配 `evaluation`
- ❌ 不适合复杂模式：`Runtime.getRuntime().exec(`（应使用正则匹配）

### Q3: caseSensitive 默认值是什么？

**A**: 默认值是 `true`（区分大小写）。如果不写 `caseSensitive` 字段，规则会区分大小写。

```yaml
# 这两个配置是等价的
- name: "eval函数"
  pattern: "eval("
  # 默认 caseSensitive: true

- name: "eval函数"
  pattern: "eval("
  caseSensitive: true  # 明确指定
```

### Q4: 多模式匹配的 AND 逻辑是如何工作的？

**A**: AND 逻辑要求所有模式都在**同一个文件**中匹配：

```yaml
patterns:
  - "BinaryFormatter"
  - "\\.Deserialize\\s*\\("
matchLogic: "AND"
```

只有当文件中**同时包含**这两个模式时才会报告。不支持跨文件匹配。

### Q5: 为什么我的正则表达式不工作？

**A**: 常见原因：
1. **忘记转义特殊字符**：`.` 应写成 `\\.`，`(` 应写成 `\\(`
2. **YAML 转义问题**：在 YAML 中，`\` 需要写成 `\\`
3. **regex 字段设置错误**：确保 `regex: 1`（不是 `regex: 2` 或 `regex: false`）

**正确示例**：
```yaml
regex: 1
pattern: "Runtime\\.getRuntime\\(\\)\\.exec\\s*\\("
```

**错误示例**：
```yaml
regex: 2  # ❌ 这是精确匹配，不是正则匹配
pattern: "Runtime.getRuntime().exec("  # ❌ 没有转义特殊字符
```

### Q6: 如何测试我的规则是否正确？

**A**: 使用内置的规则测试功能：
1. 在规则编辑器中点击"测试正则"按钮
2. 输入测试代码
3. 查看匹配结果
4. 根据结果调整规则

### Q7: 多模式匹配会影响性能吗？

**A**:
- **OR 逻辑**：性能影响较小，找到第一个匹配就返回
- **AND 逻辑**：需要检查所有模式，性能开销稍大
- **建议**：将最具特征性的模式放在前面，可以提前终止匹配

### Q8: 可以在一个规则中混用不同的匹配类型吗？

**A**: 不可以。一个规则只能有一个 `regex` 值，所有 patterns 都使用相同的匹配类型。

如果需要不同的匹配类型，应该创建多个规则。

### Q9: pattern 和 patterns 有什么区别？

**A**:
- `pattern`：单个匹配模式（字符串）
- `patterns`：多个匹配模式（字符串数组）

如果同时定义了两者：
- 使用 `patterns` 进行匹配
- `pattern` 仅用于向后兼容和显示

**推荐做法**：
- 单模式匹配：只写 `pattern`
- 多模式匹配：写 `patterns`（可选写 `pattern` 作为主模式）

### Q10: 如何为新语言添加规则？

**A**:
1. 在 `src/main/resources/rules/` 目录创建新的 YAML 文件
2. 按照标准格式编写规则
3. 在 `RuleManager.kt` 中注册新语言
4. 测试规则是否正常工作

**示例**：
```yaml
language: "Go"
version: "1.0"
description: "Go 代码审计规则集"

rules:
  critical:
    - name: "命令执行"
      function: "exec.Command"
      description: "执行系统命令"
      regex: 1
      pattern: "exec\\.Command\\s*\\("
```

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

## 📐 规则编写最佳实践

### 1. 选择合适的匹配类型

| 场景 | 推荐匹配类型 | 示例 |
|------|------------|------|
| 固定字符串 | 精确匹配 (regex: 2) | `DROP TABLE` |
| 需要处理空格 | 正则匹配 (regex: 1) | `eval\s*\(` |
| 单个关键字 | 全词匹配 (regex: 3) | `eval` |
| 复杂模式 | 正则匹配 (regex: 1) | `Runtime\.getRuntime\(\)\.exec\s*\(` |

### 2. 正则表达式编写技巧

#### ✅ 好的实践

```yaml
# 1. 使用 \s* 处理可选空格
pattern: "eval\\s*\\("  # 匹配 eval( 和 eval (

# 2. 使用 \b 单词边界（或使用全词匹配）
pattern: "\\beval\\b"  # 只匹配完整的 eval

# 3. 使用 \w+ 匹配变量名
pattern: "ScriptEngine\\s+(\\w+)"  # 匹配变量声明

# 4. 转义特殊字符
pattern: "Runtime\\.getRuntime\\(\\)"  # 正确转义 . 和 ()

# 5. 使用可选匹配
pattern: "(<[^>]+>)?"  # 可选的泛型参数
```

#### ❌ 避免的错误

```yaml
# 1. 忘记转义特殊字符
pattern: "Runtime.getRuntime().exec("  # ❌ . 和 () 有特殊含义

# 2. 过于宽泛的模式
pattern: ".*eval.*"  # ❌ 会匹配任何包含 eval 的内容

# 3. 过于复杂的正则
pattern: "(?:(?:public|private|protected)\\s+)?(?:static\\s+)?..."  # ❌ 太复杂

# 4. 不必要的捕获组
pattern: "(eval)\\s*\\("  # ❌ 不需要捕获 eval
pattern: "eval\\s*\\("    # ✅ 更简洁
```

### 3. 减少误报的技巧

#### 技巧1：使用更具体的模式

```yaml
# ❌ 容易误报
- name: "ScriptEngine"
  pattern: "ScriptEngine"  # 会匹配 import、注释等

# ✅ 更精确
- name: "ScriptEngine"
  pattern: "ScriptEngine\\s+(\\w+)"  # 只匹配变量声明
```

#### 技巧2：使用多模式 AND 逻辑

```yaml
# ✅ 减少误报
- name: "BinaryFormatter反序列化"
  patterns:
    - "BinaryFormatter"
    - "\\.Deserialize\\s*\\("
  matchLogic: "AND"  # 两者都存在才报告
```

#### 技巧3：使用全词匹配

```yaml
# ❌ 会误报
- name: "eval函数"
  regex: 1
  pattern: "eval"  # 会匹配 evaluation、medieval

# ✅ 不会误报
- name: "eval函数"
  regex: 3
  pattern: "eval"  # 只匹配完整的 eval
```

### 4. 提高性能的技巧

#### 技巧1：避免回溯

```yaml
# ❌ 可能导致回溯
pattern: ".*eval.*"

# ✅ 更高效
pattern: "eval"  # 使用精确匹配
```

#### 技巧2：将最具特征性的模式放在前面

```yaml
# ✅ 好的顺序
patterns:
  - "BinaryFormatter"  # 最具特征性
  - "\\.Deserialize\\s*\\("  # 次要特征
matchLogic: "AND"
```

#### 技巧3：使用精确匹配代替正则

```yaml
# 如果不需要处理空格，使用精确匹配
- name: "DROP TABLE"
  regex: 2  # 精确匹配，性能最好
  pattern: "DROP TABLE"
```

### 5. 规则命名规范

#### ✅ 好的命名

```yaml
- name: "Runtime.exec命令执行"  # 清晰描述功能和风险
- name: "BinaryFormatter不安全反序列化"  # 说明具体问题
- name: "eval函数代码注入"  # 包含漏洞类型
```

#### ❌ 不好的命名

```yaml
- name: "危险函数"  # ❌ 太模糊
- name: "规则1"  # ❌ 没有意义
- name: "exec"  # ❌ 只有函数名，没有说明风险
```

### 6. 描述字段编写规范

#### ✅ 好的描述

```yaml
description: "执行系统命令，可能存在命令注入风险。应验证输入来源并使用参数化方式执行命令。"
# 包含：风险说明 + 修复建议
```

#### ❌ 不好的描述

```yaml
description: "危险"  # ❌ 太简单
description: "Runtime.exec"  # ❌ 只重复函数名
```

### 7. 风险等级划分标准

| 等级 | 标准 | 示例 |
|------|------|------|
| **Critical** | 可直接导致 RCE、任意代码执行 | `eval()`, `Runtime.exec()` |
| **High** | 可能导致数据泄露、SQL注入、XXE | SQL注入、LDAP注入、XXE |
| **Medium** | 可能导致 SSRF、信息泄露、弱加密 | HTTP请求、弱加密算法 |
| **Low** | 代码质量问题、潜在风险 | 异常处理、日志记录 |

### 8. 测试规则的检查清单

在提交规则前，确保：

- [ ] 规则名称清晰描述检测内容
- [ ] 描述字段说明了风险和影响
- [ ] 选择了合适的匹配类型
- [ ] 正则表达式正确转义了特殊字符
- [ ] 使用测试功能验证了规则
- [ ] 测试了正面案例（应该匹配的代码）
- [ ] 测试了负面案例（不应该匹配的代码）
- [ ] 风险等级划分合理
- [ ] 没有明显的误报

---

## 🤝 贡献指南

欢迎贡献新的规则或改进现有规则！

### 贡献步骤
1. Fork 本项目
2. 创建特性分支
3. 添加或修改规则
4. **使用测试功能验证规则**
5. 确保遵循规则编写最佳实践
6. 提交 Pull Request

### 规则编写规范
- 规则名称应清晰描述检测内容
- 描述字段应说明风险类型和影响
- 正则表达式应尽量精确，避免误报
- 为新规则添加测试用例
- 遵循上述"规则编写最佳实践"

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

