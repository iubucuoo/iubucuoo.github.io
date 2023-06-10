# hugo添加Algolia搜索系统

<!--more-->

其他教程：

[Hugo添加Algolia搜索支持](https://edward852.github.io/post/hugo%E6%B7%BB%E5%8A%A0algolia%E6%90%9C%E7%B4%A2%E6%94%AF%E6%8C%81/#%E7%94%9F%E6%88%90%E7%B4%A2%E5%BC%95%E6%96%87%E4%BB%B6)

[折腾Hugo的loveit主题](https://www.dreamsafari.info/2020/04/hugo-loveit-mod/#25-%E4%B8%BA%E4%B8%BB%E9%A2%98%E5%A2%9E%E5%8A%A0%E6%90%9C%E7%B4%A2%E9%A1%B5)
## 注册Algolia
注册[Algolia](https://www.algolia.com/)
## 生成索引文件
首先修改 config.toml文件
```
[outputs]
    home = ["HTML", "RSS", "Algolia"]

[outputFormats.Algolia]
  baseName = "algolia"
  isPlainText = true
  mediaType = "application/json"
  notAlternative = true
  
[params.algolia]
  appId = "Application ID"
  indexName = "索引名字"
  searchOnlyKey = "Search-Only API Key"
 ```
 **注:**[outputs]下home内必需有"Algolia"，id与key等使用注册Algolia分配的。

执行`hugo`命令algolia.json会生成在public目录下
## 上传索引文件
#### 手动
在Algolia网站内，点测栏Indices，再点Upload record按钮上传生成的algolia.json文件。
#### 自动 (命令行执行)
* 每次更新都需要手动比较麻烦，可以采用自动的配置，在本地安装相关工具实现
1. 使用[atomic-algolia](https://github.com/chrisdmacrae/atomic-algolia)工具来处理
2. [安装npm](https://www.npmjs.com/get-npm)
3. 安装atomic-algolia：
```
npm init #用来生成package.json的
npm install atomic-algolia --save-dev
```
执行后打开package.json，在scripts部分后面添加一句`"algolia": "atomic-algolia"`,添加之后像这样:
```
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "algolia": "atomic-algolia"
},
```
4. 之后还需要新建一个.env文件，添加一下信息：
```
ALGOLIA_APP_ID={{ YOUR_APP_ID }}
ALGOLIA_ADMIN_KEY={{ YOUR_ADMIN_KEY }}
ALGOLIA_INDEX_NAME={{ YOUR_INDEX_NAME }}
ALGOLIA_INDEX_FILE=public/algolia.json
```
填写并保存之后，尝试把索引提交给Algolia:
```
npm run algolia
```
之后再每次执行hugo生成public文件命令后跟一句该语句执行上传就可以了
