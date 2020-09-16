# HaE - Highlighter and Extractor

## 介绍

**HaE**是基于 `BurpSuite` 插件 `JavaAPI` 开发的请求高亮标记与信息提取的辅助型插件。

![-w1070](images/16000706401522.jpg)

该插件可以通过自定义正则的方式匹配**响应报文**，可以自行决定符合该自定义正则匹配的相应请求是否需要高亮标记、信息提取。

**注**：`HaE`的使用，对测试人员来说需要基本的正则表达式基础，由于`Java`正则表达式的库并没有`Python`的优雅或方便，在使用正则的，HaE要求使用者必须使用`()`将所需提取的表达式内容包含；例如你要匹配一个**Shiro应用**的响应报文，正常匹配规则为`rememberMe=delete`，如果你要提取这段内容的话就需要变成`(rememberMe=delete)`。

## 使用方法

插件装载：`Extender - Extensions - Add - Select File - Next`

初次装载`HaE`会初始化配置文件，默认配置文件内置一个正则：`Email`，初始化的配置文件会放在与`BurpSuite Jar`包同级目录下。

![-w330](images/16000708493657.jpg)

除了初始化的配置文件外，还有`init.hae`，该文件用于存储配置文件路径；`HaE`支持自定义配置文件路径，你可以通过点击`Select File`按钮进行选择自定义配置文件。

![-w477](images/16000710069404.jpg)

HaE支持三个动作：

1. 重载规则（Reload）：当你不使用HaE UI界面去修改配置文件内的规则时，而是直接基于配置文件进行修改规则时可使用；
2. 新建规则（New）：新建规则会自动添加一行表格数据，单击或双击进行修改数据即可自动保存；
3. 删除规则（Delete）：单击选中某条规则时，按下该按钮即可删除规则。

**注**：HaE的操作都是基于表单UI的方式，操作即会自动保存。

## 插件优点

1. 多选项自定义控制适配需求；
2. 多颜色高亮分类，将BurpSuite的所有高亮颜色集成：`red, orange, yellow, green, cyan, blue, pink, magenta, gray`；
3. 颜色升级算法：利用下标的方式进行优先级排序，当满足2个同颜色条件则以优先级顺序上升颜色。（例如：**两个正则，颜色为橘黄色，该请求两个正则都匹配到了，那么将升级为红色**）
4. 简单的配置文件格式选用JSON格式，格式为
    ```
    {name: {"loaded": isLoaded:,"regex": regexText, "highlight": isHighlight, "extract": isExtract, "color": colorText}}
    ```
5. 内置简单缓存，在“多正则、大数据”的场景下减少卡顿现象。

## 实际使用

使用 RGPerson 生成测试数据，放入网站根目录文件中：

![-w467](images/16000719723284.jpg)

访问该地址，在`Proxy - HTTP History`中可以看见高亮请求，响应标签页中含有`MarkINFO`标签，其中将匹配到的信息提取了出来。

![-w1047](images/16000720732854.jpg)


## 正则优化

有些正则在实战应用场景中并不理想

在正则匹配手机号、身份证号码的时候（纯数字类）会存在一些误报（这里匹配身份证号码无法进行校验，误报率很高），但手机号处理这一块可以解决：

原正则：

```
1[3-9]\d{9}
```

误报场景：`12315188888888123`，这时候会匹配到`15188888888`，而实际上这一段并不是手机号，所以修改正则为：

```
[^0-9]+(1[3-9]\d{9})[^0-9]+
```

也就是要求匹配的手机号前后不能为0-9的数字。

## 实战用法

1. CMS指纹识别，Discuz正则：`(Powered by Discuz!)`
2. OSS对象存储信息泄露，正则：`([A|a]ccess[K|k]ey[I|i]d|[A|a]ccess[K|k]ey[S|s]ecret)`
3. 内网地址信息提取，正则：`(?:10\.\d{1,3}\.\d{1,3}\.\d{1,3})|(?:172\.(?:(?:1[6-9])|(?:2\d)|(?:3[01]))\.\d{1,3}\.\d{1,3})|(?:192\.168\.\d{1,3}\.\d{1,3})`
4. 实战插件关联搭配，漏洞挖掘案例：https://mp.weixin.qq.com/s/5vNn7dMRZBtv0ojPBAHV7Q

...还有诸多使用方法等待大家去发掘。

## 文末

随笔：正义感是一个不可丢失的东西。

Github项目地址（BUG、需求、正则欢迎提交）：https://github.com/gh0stkey/HaE

### 收录正则列表

身份证号码（来自：https://github.com/gh0stkey/HaE/issues/3）：

```
[^0-9]([1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx])|([1-9]\d{5}\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{2}[0-9Xx])[^0-9]
```

邮箱地址：

```
([\w-]+(?:\.[\w-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?)
```

