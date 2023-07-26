---
title: GoLang Web开发小示例
date: 2021-12-17 23:50:52.266
updated: 2021-12-17 23:55:36.668
url: /archives/go-web-example
categories: GoLang
tags: ["golang"]
---

### Go-Gin服务主程序
[原站访问](http://json.bettery.top/index)


[我的Json数据提取站原理](工具使用/Json解析神器-JsonPath/)

```go
package main

import (
	"encoding/json"
	"github.com/gin-gonic/gin"
	"github.com/oliveagle/jsonpath"
	"log"
	"net/http"
)

func main() {
	address := "localhost:1024"
	router := gin.Default()
	router.LoadHTMLGlob("templates/*")
	
	router.POST("/parse",parse)
	router.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", gin.H{
			"title": "Main website",
		})
	})
	router.Run(address)
}


type User struct {
	Content string `json:"content"`
	Expression string `json:"expression"`
}
func parse(c *gin.Context) {
	content := c.PostForm("content")
	log.Println("content:", content)
	jsonObj := User{}
	c.BindJSON(&jsonObj)

	var json_data interface{}
	json.Unmarshal([]byte(jsonObj.Content), &json_data)
	log.Println("传入数据:", jsonObj.Content)
	log.Println("表达式:", jsonObj.Expression)

	res, err := jsonpath.JsonPathLookup(json_data, jsonObj.Expression)
	if (jsonObj.Content == "") {
		c.JSON(http.StatusOK, gin.H{
			"message":    "请输入json",
			"parsed":     "{}",
			"expression": jsonObj.Expression,
		})
	} else if (err != nil) {
		log.Println(err)
		c.JSON(http.StatusOK, gin.H{
			"message":    res,
			"parsed":     "解析异常",
			"expression": jsonObj.Expression,
		})
	} else {
		log.Printf("%v", &jsonObj)
		c.JSON(http.StatusOK, gin.H{
			"message":    "",
			"parsed":     res,
			"expression": jsonObj.Expression,
		})
	}
}
```

### 模板文件

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <!-- import CSS -->
  <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
</head>
<body>
  <div id="app">
  <el-row type="flex" class="row-bg" justify="space-around" :gutter="4">
    <el-col :span="12">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>待提取内容</span>
            <el-button style="float: right; padding: 3px 0" type="text" @click="parse">GO</el-button>
          </div>
          <el-row type="flex" class="row-bg" justify="space-around">
              <el-col :span="4"><el-tag type="primary" @click="test('$')">原始数据</el-tag></el-col>
              <el-col :span="4"><el-tag type="success" @click="test('$.data[*].name')">提取所有的name</el-tag></el-col>
              <el-col :span="4"><el-tag type="success" @click="test('$.data[*].age')">提取所有的age</el-tag></el-col>
              <el-col :span="4"><el-tag type="success" @click="test('$.data[-1].name')">最后一个对象的name</el-tag></el-col>
              <el-col :span="4"><el-tag type="success" @click="test('$.data[?(@.age > 18)]')">筛选age>18</el-tag></el-col>
              <el-col :span="4"><el-tag type="success" @click="test('$.data[?(@.boss)]')">筛选boss</el-tag></el-col>
          </el-row>

          <el-row type="flex" justify="center">
           <el-col :span="24">
           <span>提取表达式</span>
             <el-input
                placeholder="提取表达式"
                v-model="req.expression">
             </el-input>
           </el-col>
          </el-row>


          <el-input
            type="textarea"
            :rows="30"
            placeholder="请输入内容"
            v-model="req.content">
          </el-input>
        </el-card>
    </el-col>


    <el-col :span="12">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>提取到的内容</span>
          </div>
          <el-input
            type="textarea"
            :rows="32"
            placeholder="请输入内容"
            v-model="parsed">
          </el-input>
        </el-card>


    </el-col>
  </el-row>

  </div>
</body>
  <!-- import Vue before Element -->
  <script src="https://unpkg.com/vue/dist/vue.js"></script>
  <!-- import JavaScript -->
  <script src="https://unpkg.com/element-ui/lib/index.js"></script>
  <!-- import axios -->
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <script>
    new Vue({
      el: '#app',
      data: function() {
        return {
        visible: false,
        content: "{\"data\":[{\"name\":\"张三\",\"age\":18},{\"name\":\"李四\",\"age\":20}]}",
        parsed: "转换提取到的内容",
        expression: "$",
        req : {
            "content":"{\"data\":[{\"name\":\"张三\",\"age\":18},{\"name\":\"李四\",\"age\":20},{\"name\":\"王五\",\"age\":108,\"boss\":true}]}",
            "expression":"$"
        }
        }
      },
      methods:{
        parse(){
            this.parsed = "hello";
            axios.post('/parse',this.req).then(res =>
                this.parsed = JSON.stringify(res.data.parsed)
            );
        },
        test(content){
            this.req.expression = content
            this.parse()
        }
      }
    })
  </script>

  <style>
    .el-row {
      margin-bottom: 20px;
      &:last-child {
        margin-bottom: 0;
      }
    }
    .el-col {
      border-radius: 4px;
    }
    .bg-purple-dark {
      background: #99a9bf;
    }
    .bg-purple {
      background: #d3dce6;
    }
    .bg-purple-light {
      background: #e5e9f2;
    }
    .grid-content {
      border-radius: 4px;
      min-height: 36px;
    }
    .row-bg {
      padding: 10px 0;
      background-color: #f9fafc;
    }
  </style>
</html>
```

```

```

