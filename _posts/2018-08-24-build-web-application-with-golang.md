---
layout:     post
title:      "build-web-application-with-golang"
subtitle:   "通过golang构建web应用"
date:       2018-08-24 17:00:00
author:     "Mickey"
header-img: "img/post-bg-2015.jpg"
tags:
    - golang
---

最近逛github的时候，看到一个golang的学习资料，star 2w+，粗略翻了一下，感觉写的不错，在此做下笔记，用以温故知新，[博客地址](https://github.com/astaxie/build-web-application-with-golang/)

* chapter one

    * gopath, goroot, gobin的区别

        在终端输入 `go env` 命令，可以看到go的一些环境变量，goroot指代go安装的路径，gopath可以理解为项目的工作路径，gopath下一般由三个目录组成，`src`、`bin`、`pkg`，`src`下面放置我们项目的源代码，`bin`目录下放置可执行文件(main包 go install)，`pkg`下存放.a文件，是go编译生成的
    
* chapter two

    * golang中当数组作为函数的参数时，传递的是数组的副本，在函数中修改不会影响到原数组，如果需要修改数组，可以传递指针
    * golang的`if`有一个强大的地方就是条件判断语句里面允许声明一个变量，这个变量的作用域只能在该条件逻辑块内，在其他地方就不起作用了，如下所示

        ```
        // 计算获取值x,然后根据x返回的大小，判断是否大于10。
        if x := computedValue(); x > 10 {
            fmt.Println("x is greater than 10")
        } else {
            fmt.Println("x is less than 10")
        }

        //这个地方如果这样调用就编译出错了，因为x是条件里面的变量
        fmt.Println(x)
        ```

    * golang的函数也支持变参，即接受变参的函数是有着不定数量的参数的

        ```
        func myfunc(arg ...int) {}
        ```

        `arg ...int`告诉Go这个函数接受不定数量的参数。注意，这些参数的类型全部是`int`。在函数体中，变量`arg`是一个`int`的`slice`：

        ```
        for _, n := range arg {
            fmt.Printf("And the number is: %d\n", n)
        }
        ```
    * golang有一个非常不错的设计 `defer`，你可以在函数中添加多个`defer`语句。当函数执行到最后时，这些`defer`语句会按照`逆序执行(先进后出模式)`，最后该函数返回。特别是当你在进行一些打开资源的操作或异步完成标示时，非常有用

        ```
        func ReadWrite() bool {
            file.Open("file")
            defer file.Close()
            if failureX {
                return false
            }
            if failureY {
                return false
            }
            return true
        }
        ```
    
    * Panic和Recover

        Go没有像Java那样的异常机制，它不能抛出异常，而是使用了`panic`和`recover`机制。一定要记住，你应当把它作为最后的手段来使用，也就是说，你的代码中应当没有，或者很少有`panic`的东西。

        Panic
        > 是一个内建函数，可以中断原有的控制流程，进入一个panic状态中。当函数F调用panic，函数F的执行被中断，但是F中的延迟函数会正常执行，然后F返回到调用它的地方。在调用的地方，F的行为就像调用了panic。这一过程继续向上，直到发生panic的goroutine中所有调用的函数返回，此时程序退出。panic可以直接调用panic产生。也可以由运行时错误产生，例如访问越界的数组。

        Recover
        > 是一个内建的函数，可以让进入panic状态的goroutine恢复过来。recover仅在延迟函数中有效。在正常的执行过程中，调用recover会返回nil，并且没有其它任何效果。如果当前的goroutine陷入panic状态，调用recover可以捕获到panic的输入值，并且恢复正常的执行。

        下面这个函数演示了如何在过程中使用`panic`

        ```
        var user = os.Getenv("USER")

        func init() {
            if user == "" {
                panic("no value for $USER")
            }
        }
        ```

        下面这个函数检查作为其参数的函数在执行时是否会产生`panic`:

        ```
        func throwsPanic(f func()) (b bool) {
            defer func() {
                if x := recover(); x != nil {
                    b = true
                }
            }()
            f() //执行函数f，如果f中出现了panic，那么就可以恢复回来
            return
        }
        ```

    * struct的method将指针作为receiver

        ```
        func (b *Box) SetColor(c Color) {
            b.color = c
        }
        ```

        我们经常会看到上面类似的`method`，为啥使用`*Box`作为接收器而不使用`Box`呢，使用数值作为receiver的时候，其实`func`中的b是一个copy，如果需要修改的话，需要传递指针，和数值和数组相似

    * golang的空interface(interface {})不含任何的method，因此任何类型都实现了空interface，空的interface对描述起不到任何作用，但是空interface在我们需要存储任意类型的数值的时候相当有用，因为它可以存储任意类型的数值

        ```
        // 定义a为空接口
        var a interface{}
        var i int = 5
        s := "Hello world"
        // a可以存储任意类型的数值
        a = i
        a = s
        ```

    * 我们知道golang的interface可以接收实现方法的任何类型，那么是否有python中的type，js中的typeof类似的方法来让我们知道实现接口的是哪种具体类型呢，Go语言里面有一个语法，可以直接判断是否是该类型的变量： value, ok = element.(T)，这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型

        ```
        package main

        import (
            "fmt"
            "strconv"
        )

        type Element interface{}
        type List [] Element

        type Person struct {
            name string
            age int
        }

        //定义了String方法，实现了fmt.Stringer
        func (p Person) String() string {
            return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
        }

        func main() {
            list := make(List, 3)
            list[0] = 1 // an int
            list[1] = "Hello" // a string
            list[2] = Person{"Dennis", 70}

            for index, element := range list {
                if value, ok := element.(int); ok {
                    fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
                } else if value, ok := element.(string); ok {
                    fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
                } else if value, ok := element.(Person); ok {
                    fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
                } else {
                    fmt.Printf("list[%d] is of a different type\n", index)
                }
            }
        }
        ```

        此外，在switch中还有更简单的做法

        ```
        package main

        import (
            "fmt"
            "strconv"
        )

        type Element interface{}
        type List [] Element

        type Person struct {
            name string
            age int
        }

        //打印
        func (p Person) String() string {
            return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
        }

        func main() {
            list := make(List, 3)
            list[0] = 1 //an int
            list[1] = "Hello" //a string
            list[2] = Person{"Dennis", 70}

            for index, element := range list{
                switch value := element.(type) {
                    case int:
                        fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
                    case string:
                        fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
                    case Person:
                        fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
                    default:
                        fmt.Println("list[%d] is of a different type", index)
                }
            }
        }
        ```

        需要注意的是，`element.(type)`语法不能在switch外的任何逻辑里面使用，如果你要在switch外面判断一个类型就使用`value, ok := element.(type)`方法