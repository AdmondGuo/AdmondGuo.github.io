---
layout: post
category: 工程实践
---

# 2020-07-05-Spring工程的三个参数：Program arguments | Enviroment variables | VM Options
![idea中的设置](/public/img/20200830165631161.png)

## 1.Program arguments
- 程序参数，是会被传入主函数中的参数
- 在idea中设置时，每个参数需要以空格隔开，否则将会被识别成一个参数
```java
public static void main(String[] args) {
    arg1 = args[0];
    arg2 = args[1];
    //do something ...
    SpringApplication.run(App.class);
}
```

## 2.Enviroment variables

- 没有前缀，优先级低于 VM options ，即如果VM options 有一个变量和 Environment variable中的变量的key相同，则以VM options 中为准
- 如果用命令行启动，这个参数需要在运行java类以前使用 set JAVA_HOME=D:\jdk1.8.0_05 这种方式进行临时修改，这种方式只在当前cmd窗口有效

## 3.VM Options
- 传入JVM中
- 需要以 -D 或 -X 或 -XX 开头，每个参数最好使用空格隔开

