title = "2018-08-17"
weight = 100

------

# 20180817-mybatis

## 1. 多表关联查询(queryById 和queryAll) 

特点: 查询多张表，结果集不和现有的bean对应

1. 原始查询语句
   ![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fudlku18wkj30xw03gdfn.jpg)
2. 类定义
  ![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fudmfylvuhj30bo084wei.jpg)
3. mybatis　mapper映射文件配置
   ![](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fudlr3etd2j30m10bdmxv.jpg)
   注: 在｀select｀标签中，　改用`resultMap`而非原来的, 表示`resultType`自定义结果映射．然后在一个单独的｀resultMap｀标签中定义关联查询返回的每一列．其中`asscoiation`标签表示关联的其他类的属性，　也需要把列和类的属性对应；

4. 查询
   ![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fudlzyhqgyj30jv05zq34.jpg)
