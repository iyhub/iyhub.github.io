---
title: Json解析神器
date: 2021-12-17 23:43:17.037
updated: 2021-12-17 23:45:55.314
url: /archives/JsonPath
tags: ["json"]
---

### 更花式高效解析Json

有如下json字符串:

```json
{
    "data":[
        {
            "name":"张三",
            "age":18
        },
        {
            "name":"李四",
            "age":20
        },
        {
            "name":"王五",
            "age":108,
            "boss":true
        }
    ]
}
```

现在有如下需求 

##### 1.如何解析出data元素中的所有`name`字段?

常规做法Gson,Json转对象列表再遍历,有没有更骚的操作?

来看看这个:

```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.2.0</version>
</dependency>
```



```java
import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;


String str  = "{\"data\":[{\"name\":\"张三\",\"age\":18},{\"name\":\"李四\",\"age\":20},{\"name\":\"王五\",\"age\":108,\"boss\":true}]}"
DocumentContext doc = JsonPath.parse();
List<String> names = doc.read("$.data.[*].name");
// names :["张三","李四","王五"]

```

##### 2. 提取最后一个对象的name

```java
//获取在负一位置的元素即最后的元素
String names = doc.read("$.data.[-1].name");
// names :"王五"
```

##### 3. 筛选age>18的元素

```java
// ["李四","王五"]
String names = doc.read("$.data[?(@.age > 18)].name");
```

##### 4.获取含有字段`boss`的元素

```java
// ["李四","王五"]
String names = doc.read("$.data[?(@.boss)].name");
```



### Jsonpath 操作符规则

|           操作            |                   说明                    |
| :-----------------------: | :---------------------------------------: |
|            `$`            |   查询根元素。这将启动所有路径表达式。    |
|            `@`            |         当前节点由过滤谓词处理。          |
|            `*`            | 通配符，必要时可用任何地方的名称或数字。  |
|           `..`            | 深层扫描。 必要时在任何地方可以使用名称。 |
|         `.<name>`         |              点，表示子节点               |
| `['<name>' (, '<name>')]` |               括号表示子项                |
| `[<number> (, <number>)]` |              数组索引或索引               |
|       `[start:end]`       |               数组切片操作                |
|    `[?(<expression>)]`    | 过滤表达式。 表达式必须求值为一个布尔值。 |

```json
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
```

示例:

| `$.store.book[*\].author`                |            获取json中store下book下的所有author值             |
| :--------------------------------------- | :----------------------------------------------------------: |
| `$..author`                              |                 获取所有json中所有author的值                 |
| `$.store.*`                              |                   所有的东西，书籍和自行车                   |
| `$.store..price`                         |                获取json中store下所有price的值                |
| `$..book[2\]`                            |                 获取json中book数组的第3个值                  |
| `$..book[-2\]`                           |                        倒数的第二本书                        |
| `$..book[0,1\]`                          |                           前两本书                           |
| `$..book[:2\]`                           |           从索引0（包括）到索引2（排除）的所有图书           |
| `$..book[1:2\]`                          |           从索引1（包括）到索引2（排除）的所有图书           |
| `$..book[-2:\]`                          |                获取json中book数组的最后两个值                |
| `$..book[2:\]`                           |         获取json中book数组的第3个到最后一个的区间值          |
| `$..book[?(@.isbn)\]`                    |           获取json中book数组中包含isbn字段的所有值           |
| `$.store.book[?(@.price < 10)\]`         |             获取json中book数组中price<10的所有值             |
| `$..book[?(@.price <= $['expensive'\])]` |         获取json中book数组中price<=expensive的所有值         |
| `$..book[?(@.author =~ /.*REES/i)\]`     | 获取json中book数组中的作者以REES结尾的所有值（REES不区分大小写） |
| `$..*`                                   |             逐层列出json中的所有值，层级由外到内             |
| `$..book.length()`                       |                   获取json中book数组的长度                   |

附上个人基于`GoLang语言 Gin 框架`开发的一个工具站

http://json.bettery.top/index

