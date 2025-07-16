# Jq 命令使用

### kubernetes

tab为分割符打印字段输出

```shell
jq -r '.status.es.nodes | sort_by(.endTime) | .[] | [.name, .startTime, .endTime] | @tsv' 
```

### **1. 基础语法**

#### **(1) 理解 `jq` 的执行流程**

```
bash


# 示例输入 JSON
{
  "name": "Alice",
  "age": 25,
  "hobbies": ["reading", "coding"]
}
```

#### **(2) 基本命令结构**

```
bash


# 提取字段
jq '.name' input.json  # 输出 "Alice"

# 多字段组合
jq '{name, length}' input.json  # 输出 { "name": "Alice", "length": 5 }

# 管道操作（类似 shell 管道）
jq '.hobbies | .[0]' input.json  # 输出 "reading"
```

------

### **2. 核心操作符与函数**

#### **(1) 条件与过滤**

```
bash


# select：按条件筛选
jq 'select(.age > 20)' input.json  # 输出整个对象

# test：正则匹配
jq 'select(.name | test("A.*"))' input.json  # 匹配以 A 开头的名字
```

#### **(2) 数组与对象操作**

```
bash


# map：对数组元素逐个处理
jq '.hobbies | map(ascii_upcase)' input.json  # 输出 ["READING", "CODING"]

# add：合并数组
jq '[1,2,3] | add'  # 输出 6

# group_by：分组统计
jq '[{a:1}, {a:2}, {a:1}] | group_by(.a)'  # 按 a 字段分组
```

#### **(3) 时间与数学运算**

```
bash


# 时间戳转换
jq 'now | strftime("%Y-%m-%d")'  # 输出当前日期

# 耗时计算
jq '[.status.es.nodes[].endTime, .status.es.nodes[].startTime] | map(fromdate) | max - min' input.json
```

------

### **3. 高阶技巧**

#### **(1) 变量与函数**

```
bash


# 定义变量 $var
jq 'def max_age: .age + 10; {name, max_age}' input.json

# 自定义函数（计算耗时）
jq '
  def duration: (fromdate) - (fromdate);
  .status.es.nodes[] 
  | {name, duration: (duration)}
' input.json
```

#### **(2) 流式处理**

```
bash


# 处理大型 JSON 文件（逐行读取）
jq -c '.. | objects | select(has("name"))' huge.json
```

#### **(3) 错误处理**

```
bash


# 默认值（字段不存在时返回空）
jq '.missing_field // "default"' input.json

# 错误抑制（字段为空时不报错）
jq '.missing_field?' input.json
```

------

### **4. 实战学习资源**

#### **(1) 官方文档**

- **必读**：[jq 官方手册](https://stedolan.github.io/jq/manual/)

- 重点章节

  ：

  - Built-in functions（内置函数）
  - Advanced features（高级特性）

#### **(2) 交互式教程**

- [jqplay.org](https://jqplay.org/)：在线练习平台。

- 示例：

  ```
  bash
  
  
  # 在 jqplay.org 输入以下内容
  {
    "items": [
      {"name": "A", "value": 10},
      {"name": "B", "value": 20}
    ]
  }
  ```

  ```
  bash
  
  
  # 查询语句
  .items[] | select(.value > 15) | .name
  ```

#### **(3) 书籍推荐**

- 《jq Pocket Reference》：快速查阅语法。
- 《Mastering jq》：深入解析高级用法。

------

### **5. 典型场景练习**

#### **(1) 日志分析**

```
bash


# 提取错误日志并统计频率
cat logs.json | jq -r 'select(.level == "error") | .message' | sort | uniq -c
```

#### **(2) Kubernetes 数据清洗**

```
bash


# 提取所有 Pod 名称和状态
kubectl get pods -o json | jq -r '.items[] | [.metadata.name, .status.phase] | @tsv'
```

#### **(3) AWS CLI 输出格式化**

```
bash


# 提取 EC2 实例 ID 和标签
aws ec2 describe-instances | jq -r '.Reservations[].Instances[] | [.InstanceId, .Tags[].Value] | @tsv'
```

------

### **6. 进阶调试技巧**

#### **(1) 逐步调试**

```
bash


# 查看中间结果
jq 'debug | .status.es.nodes' input.json
```

#### **(2) 多命令组合**

```
bash


# 统计每个节点的耗时并排序
jq '
  .status.es.nodes
  | map(
    .duration = ((.endTime | fromdate) - (.startTime | fromdate))
  )
  | sort_by(.duration)
' input.json
```

#### **(3) 复杂嵌套结构**

```
bash


# 提取嵌套字段（如 .a.b.c）
jq '.. | objects | select(has("c"))' input.json
```

