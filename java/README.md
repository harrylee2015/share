# java

## java1.8中文文档
  [java8中文文档](https://www.matools.com/api/java8)
  
  
## MongoDB在Spring框架中的使用
  [关于MongoTemplate的使用](https://juejin.cn/post/7022532575121899533)

## 开发环境

  开发工具： IntelliJ IDEA 
  依赖包管理： maven


## JAVA命令生成jar包
 
 - 写Test.java文件
 ```
 package cn.chain33.jvm.dapp.guess;

import com.google.gson.Gson;

import java.util.LinkedHashMap;

public class Test {
    private LinkedHashMap<Integer, Integer> data=new LinkedHashMap<Integer,Integer>();

    public boolean saveData() {
        Gson gson = new Gson();
        String jsonStr = gson.toJson(this);
        System.out.println(jsonStr);
        return true;
    }

    public static void main(String[] args){
         Test test = new Test();
         test.data.put(1,1);
         test.data.put(2,2);
         test.data.put(2000,2000);
         test.saveData();
    }
}
 ```
- 编译成字节码文件

```
   mkdir classes

   javac  -cp ./gson-2.8.2.jar -d ./classes   Test.java
```
 
- 创建并且编写MANIFEST.MF文件

 ```
 cd classes

 cat> MANIFEST.MF<<EOF

 Mainfest-version: 1.0
 Main-Class: cn.chain33.jvm.dapp.guess

 EOF

 ```

- 生成jar包

```
jar -cvfm Test.jar MANIFEST.MF cn 
```

- 执行jar包

```
 java -cp ./gson-2.8.2.jar -jar Test.jar
```
