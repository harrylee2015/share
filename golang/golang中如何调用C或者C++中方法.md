#golang中如何调用C或者C++中方法

## 导出接口方法

```c++
#include <iostream>
//导出C 接口
extern "C" {
    void hello_tracy(){
        std::cout << "hello tracy" << std::endl;
    }
    
}

```

## 编译成动态库文件

```
 g++ -std=c++11 -shared -fPIC -o libtracy.so tracy_wrapper.cpp
```
* g++: 这是 GNU 编译器集合中用于编译 C++ 程序的命令。它会调用 C++ 编译器来处理源文件。

* -std=c++11: 这个选项告诉编译器使用 C++11 标准进行编译。这表示源代码中可以使用 C++11 提供的特性和语法。

* -shared: 这个选项告诉编译器生成一个共享库（shared library）而不是可执行程序。

* -fPIC: 这是 Position Independent Code（PIC）的缩写，它告诉编译器生成位置无关的代码。这在共享库中特别重要，因为共享库需要在不同的内存位置加载并运行。

* -o libtracy.so: 这个选项指定生成的共享库的输出文件名为 libtracy.so。

* tracy_wrapper.cpp: 这是待编译的 C++ 源文件的名称，编译器将对其进行编译并生成共享库。


## 编写go引用

```go
package main

/*
#cgo LDFLAGS: -L. -ltracy
extern void hello_tracy();
*/
import "C"

func main() {

	C.hello_tracy()

}

```

*  #cgo LDFLAGS 指令用于告诉 Go 编译器链接时需要使用的参数，其中 -L. 表示在当前目录查找库文件，-ltracy 表示链接 libtracy.so 库。


## 设置 LD_LIBRARY_PATH 环境变量

```bash
export LD_LIBRARY_PATH=/path/to/directory:$LD_LIBRARY_PATH

```

* /path/to/directory 表示生成动态库文件所在目录


## 运行go文件

```
go run tracy.go
```



## go调用C中方法

```c
// example.c
#include <stdio.h>

void sayHelloFromC() {
    printf("Hello from C!\n");
}
```

## 编译成共享库文件

```
gcc -shared -fPIC -o libexample.so example.c
```

* gcc: 这是 GNU Compiler Collection 的缩写，是一个编译器工具，用于编译 C 代码。

* -shared: 这个选项告诉 gcc 编译器生成一个共享库文件，而不是可执行文件。共享库文件可以在程序运行时动态加载。

* -fPIC: 这个选项表示编译为位置无关的代码（Position Independent Code），这是因为共享库文件需要在内存中被动态加载，位置无关的代码可以在内存中的任何地方执行。

* -o libexample.so: 这个选项指定了生成的共享库文件的名称为 libexample.so，-o 后接文件名表示输出文件的名称。

* example.c: 这是要编译成共享库的源文件，即 C 语言源代码文件。



## 设置 LD_LIBRARY_PATH 环境变量

```bash
export LD_LIBRARY_PATH=/path/to/directory:$LD_LIBRARY_PATH

```

* /path/to/directory 表示生成动态库文件所在目录

## 调用共享库中方法

```go
// example.go
package main

/*
#cgo LDFLAGS: -L. -lexample
extern void sayHelloFromC();
*/
import "C"

func main() {
	C.sayHelloFromC()
}

```

*  #cgo LDFLAGS 指令用于告诉 Go 编译器链接时需要使用的参数，其中 -L. 表示在当前目录查找库文件，-lexample 表示链接 libexample.so 库。