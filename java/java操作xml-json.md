+++
title = "2018-08-08"
weight = 100
+++

# 20180807-java-xml-json-socket

## 处理xml
```
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
```
```
读:
File xmlFile = new File(xmlPath);
SAXReader xmlReader = new SAXReader();
Document doc = xmlReader.read(xmlFile);  // 获取xmldoc对象
Element rootNode = doc.getRootElement();

写:
doc.add(node)
rootNode.setNode()
rootNode.setAttribute()
```

## 处理json
依赖: gson
```
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```
```
写:
JsonObject json = new JsonObject();
json.add();

读:
JsonParser parser = new JosnParser()
JsonObject json = parser.parser(new FileReader('xxx.json'))
json.get("key_name")
```

## java-http-GET-POST
依赖: apache-HttpComponents

``` get
HttpClient client = HttpClients.createDefault();
HttpGet get = new HttpGet("url");
HttpResponse response = client.execute(get);
HttpEntity entity = response.getEntiy();
String result = EntityUtils.toString(entity);
```

``` post
HttpClient client = HttpClients.createDefault()
HttpPost post = new HttpPost("url")
List<BasicNameValuePair> param = new ArrayList()
param.add(new BasicNameValuePair("key", "value"))
post.setEntity(new UrlEncodingFormEntity(param, 'utf-8'))
HttpResponse response = client.execute(post)
HttpEntity entity = response.getEntiy();
String result = EntityUtils.toString(entity);

```
