# 20180807-java-多线程-注解
## 0. apache操作文件
```
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

## 1. 操作excel(org.apache.poi)
```
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
  <version>3.17</versio>
</dependency>
```

```
fis = new FileInputStream("/home/huh/Desktop/student_list.xls");
HSSFWorkbook workbook = new HSSFWorkbook(fis);  // 获取workbook
```

## 2. 多线程
继承Thread, 重写run方法. <br />
实现Runnable <br />
实现Callable <br />

前二者无法各种子线程的异常, 第三种配合FeatureTask可以实现子线程异常的跟踪.

## 3. 线程池
ThreadPoolExecuting

## 4. 枚举单例
public enum enumSingleton {INSTANCE;}

## 5. 异常处理
throws: 在定义方法时声明异常,但不处理异常,异常由方法的调用者处理.

throw: 抛出异常的实例化对象 `throw new Excetpion`

自定义异常: 继承Excetion.

## 6. 多态
表现:
1. 方法的重载和重写;
2. 对象的多态性.

6.1 对象的多态性
1. 向上转型: 父类 父类对象的引用 = 子类对象;  //程序自动完成 <br />
2. 向下转型: 子类 子类对象的引用 = (子类)父类对象; //强制类型转换 <br />
注: 向下转型之前, 要先进行向上转型.

## 7. 抽象类与接口
尽量使用抽象类, 不要继承已经完成好的类.

## 8. 泛型(Generic)
8.1 在JDK1.5中新增的属性, 解决数据类型的安全性问题. <br />
8.2 原理: 在类声明时通过一个标识来表示类中某个属性的类型或者某个方法的返回值及参数类型. <br />
8.3 声明格式: `访问权限 class 类名 <泛型, 泛型...> {}`  <br />
8.4 对象创建: `类名<具体类型> 对象的引用 = new 类名<具体类型>();`  <br />
8.5 usage:
```
class Point<T> {
  private T x;
  public Point(T x) {
    this.x = x;
  }

  public T getX(){return x;}
  public void setX(T x) {this.x = x;}
}

main() {
  Point<String> p = new Point<String>();

  //通配符
  Point<Integer> p2 = new Point<Integer>();
  p2.setX(100);
  say(p2)
}

// 通配符
say(Point<?> p) {
  sys(p.getX());
}
```
8.6 通配符 ===> `?`  <br />
8.7 接口的泛型的定义与类定义泛型的格式一致  <br />
8.8 泛型方法
格式: `访问权限 <泛型标识> 泛型标识 方法名(泛型标识 参数名){}`
usage: `public <T> T add(T x, T y){}`  <br />
8.9 泛型数组 ==> 结合泛型方法使用, 在泛型方法的参数列表中定义, 在调用时参数任意类型的数组.  <br />
