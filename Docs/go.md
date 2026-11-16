# GO

目录：

- [工程化](#工程化)
    - [GO项目的包管理模式Go Modules](#go项目的包管理模式go-modules)
    - [常见的环境变量](#常见的环境变量)
- [特性](#特性)
    - [import](#import)
    - [**iota** 变量](#iota-变量)
    - [函数](#函数)
    - [数组](#数组)
    - [defer](#defer)
    - [Map](#map)
    - [结构体、对象](#结构体对象)
    - [多态](#多态)
    - [反射机制(reflect)及其应用](#反射机制reflect及其应用)
    - [goroutine](#goroutine)
    - [channel](#channel)

## 工程化

### GO项目的包管理模式Go Modules

Go语言1.11版本之后默认的管理模式，该管理模式下，一个基础的Go项目通常具备以下项目配置文件

- `go.mod`: 项目的模块身份，依赖版本声明以及解析配置
- `go.sum`: 记录所有依赖模块的加密哈希值，不参与版本选择，负责依赖库的内容安全

对于Go项目的管理，常用到以下的命令:

| 命令                    | 作用                               |
| ----------------------- | ---------------------------------- |
| go mod init module_name | 生成 `go.mod` 文件                 |
| go mod download         | 下载 `go.mod` 文件中指明的所有依赖 |
| go mod tidy             | 整理现有依赖                       |
| go mod graph            | 查看现有依赖结构                   |
| go mod edit             | 编辑 `go.mod` 文件                 |
| go mod vendor           | 导出项目所有的依赖到vendor目录     |
| go mod verify           | 校验一个模块是否被篡改过           |
| go mod why              | 查看为什么需要依赖某模块           |

### 常见的环境变量

GOPROXY：用于设置go模块代理，建议设置国内的镜像站点

> 不同于前端项目在每个项目根目录下都有一个**node_modules**文件夹管理依赖，go的不同项目之间依赖文件是集中式管理的，这些依赖缓存在 **GOPATH/pkg/mod/** 下，不同的项目之间可能共享同一份依赖文件

## 特性

只列举一些相对特殊的特性

### import

- init的执行时机

    设有如下文件结构：

    ```
      main
        --lib1
          --lib1.go
        --lib2
          --lib2.go
        main.go
    ```

    当main分别导入了lib1和lib2，且lib1和lib2不存在导入关系时，上述几个包的初始化流程为：

    ```
      lib1.init()->lib2.init()->main.init()
    ```

- 匿名/具名导入

    ```go
    // 当导入了一个包但并不使用时，go编译器会报错
    // 使用匿名导入的方式会执行对应包的init方法
    package main
    import (
      // 匿名导入包，会执行其init方法，但当前文件中不能再调用被导入的包中的其他方法，
      // 用于触发部分库的注册机制或用在纯副作用场景
      _ ".../lib1"
      // 具名导入，导入后使用myLib2 调用其中的方法
      myLib2 ".../lib2"
    )
    ```

### **iota** 变量

常用于与const配合定义枚举值

```go
const (
  ColorRed 　　　　　= 0
  ColorOrange       = 1
  ColorYellow       = 2
  ColorGrassland    = 3
  ColorCyan         = 4
  ColorBlue         = 5
  ColorPurple       = 6
)
// 使用iota
const (
  ColorRed      	= iota
  ColorOrange
  ColorYellow
  ColorGrassland
  ColorCyan
  ColorBlue
  ColorPurple
)
```

### 函数

返回值类型声明并支持多值返回

```go
func foo1 (a string ,b int) int{
    // ……
    c:=100
    return c
}
// 多个返回值
func foo2(a string,b int ) (int ,int){
    // ……
    return 6666,777
}
// 多个具名返回值
func foo3 (a string,b int)(r1 int ,r2 int){
    // ……
    r1= 100
    r2= 200
    return
}
// 多个同类型具名返回值
func foo4(a string,b int)(r1,r2 int){
    // ……
    r1= 100
    r2= 200
    return
    // 也可以直接
    // return 100,200
}
```

### 数组

1.  定长数组

    定长数组在作为参数被传递时是拷贝传递,且不同长度的数组不被视为同一种类型

    ```go
    package main
    import "fmt"

    func main(){
        var myArray1 [10]int

        for i:=0;i<len(myArray1);i++{
            fmt.Println(myArray1[i])
        }
        // 带初始化的定义
        var myArray2 [10]int{1,2,3,4}
        // 另一种遍历方式
        for index,value := range myArray2{
            fmt.Println("index=",index,",value=",value)
        }

        // 查看数组的数据类型
        fmt.Println("myArray1 types = %T \n",myArray1)
        fmt.Println("myArray2 types = %T \n",myArray2)
    }
    ```

    >

2.  动态数组

    动态数组在作为参数传递时是引用传递

    ```go
    package main

    import "fmt"

    func printArray(myArray []int){
        for _,value := range myArray{ // _ 符号表示匿名，在此处用于省略索引值
            fmt.Println("value = ",value)
        }
    }

    func main(){
        myArray := []int{1,2,3,4}

        fmt.Println("myArray type is %T\n",myArray)
    }
    ```

3.  切片

    ```go
    func main (){
        // 声明一个切片并初始化
        // slice1 := []int {1,2,3}
        // 声明但不初始化(不分配空间)
        var slice1 := []int
        slice1 = make([]int ,3) // 开辟三个int空间
        slice1[0]=100
        // 推荐写法
        // slice1 := make([]int,3) // := 会自动推导出slice1是一个切片
        fmt.Printf("len = %d,slice = %v \n",len(slice1),slice1)

        // 判断一个slice是否为0
        if slice1 == nil {
            fmt.Println("slice1 是一个空切片")
        }else{
            fmt.Println("slice1 是有空间的")
        }
    }
    ```

    - 切片的追加与截取

        ```go
        func main(){
            // make()方法的第三个参数用于定义切片容量（非长度）
            var numbers := make([]int ,3,5) // numbers切片长度为5
            fmt.Println("len = %d,cap=%d,slice = %v\n",len(numbers),cap(numbers),numbers)

            // 向numbers切片追加一个元素1，现在其长度为4，容量为5
            numbers = append(numbers,1)

            // 当不断向该容器中追加元素直到大于cap时，将会继续为其开辟cap大小的空间，
            // 即当为numbers追加第6个元素时，cap将增加5
        }
        ```

        > 切片的默认容量与默认长度相同，当使用 make([]int,2)创建切片时，其cap为2，与len相同

        ```go
        func main(){
            s := []int{1,2,3} // len =3 , cap = 3
            // [1,2)
            s1 := s[0:2] // 截取索引从0开始，到2结束(不包括2)的切片元素
            s1[0] = 100 //s和s1指向同一个地址，当修改s1的元素值时，s中的元素也会被修改
            fmt.Println(s)
            fmt.Println(s1)
            // copy 可以将底层数组的slice一起进行拷贝
            s2 := make([]int ,3) // s2 为 [0,0,0]
            // 将s中的值依次拷贝到s2中
            copy(s2,s)
            fmt.Println(s2)
        }
        ```

### defer

类似析构函数，在函数return之后执行，有defer修饰的代码依次入栈，从栈顶依次执行

```go
package main
improt "fmt"

func func1(){
    fmt.Println("A")
}

func func2(){
    fmt.Println("B")
}

func func3(){
    fmt.Println("C")
}

func main (){
    defer func1()
    defer func2()
    defer func3()
}
// 执行结果为 C, B, A
```

### Map

- 定义

    ```go
    func main(){
        // 声明myMap1是一种map类型，key是string，value是string
        var myMap1 map[string]string
        if myMap1 == nil {
            fmt.Println("myMap1 是一个空map")
        }
        // 在使用map前，需要使用make方法开辟空间
        myMap1 = make(map[string]string,10) // len = 10 ,cap = 10 ，同slice

        myMap1["one"] = "java"
        myMap1["two"] = "c++"
        myMap1["three"] = "python"
        fmt.Println(myMap1)

        // 第二种声明方式
        myMap2 := make(map[int]string)
        myMap2[1] = "java"
        myMap2[2] = "c++"
        myMap2[3] = "python"
        fmt.Println(myMap2)

        // 第三种声明方式
        myMap3 := map[string]string{
            "one":"php",
            "two":"c++",
            "three":"python",
        }
        fmt.Println(myMap3)

    }
    ```

- 使用

    ```go
    func printMap(cityMap map[string]string){ // map作为参数，其传递方式为引用传递
        for key,value := range cityMap{
            fmt.Println("key = ",key)
            fmt.Println("value = ",value)
        }
    }

    func main(){
        cityMap := make(map[string]string)

        // 添加
        cityMap["China"] = "Beijing"
        cityMap["Japan"] = "Tokyo"
        cityMap["USA"] = "NewYork"

        // 遍历
        for key,value := range cityMap{
            fmt.Println("key = ",key)
            fmt.Println("value = ",value)
        }

        // 删除
        delete(cityMap,"China")

        // 修改
        cityMap["USA"] = "DC"

        // 遍历
        printMap(cityMap)
    }
    ```

### 结构体、对象

- 定义

    ```go
    // 声明一种新的数据类型 myint ,是int的一个别名
    type myint int

    // 定义一个结构体
    type Book struct {
        title string
        auth string
    }

    func main(){
        var book1 Book
        book1.title = "Golang"
        book1.auth = "zhang3"

        fmt.Printf("%v \n",book1)
    }
    ```

    > 结构体直接作为参数传递时是拷贝传递

- 将结构体封装成类

    ```go
    // 如果类名首字母大写，表示其他包也能访问
    type Hero struct {
        // 如果类的属性首字母大写，表示该属性对外能被访问，否则只能内部访问
        Name string
        Ad int
        Level int
    }
    func (this Hero)Show(){ 	// 此处使用的Hero是副本，属于拷贝传递
        fmt.Println("Name = ",this.Name)
        fmt.Println("Ad = ",this.Ad)
        fmt.Println("Level = ",this.Level)
    }
    func (this Hero) GetName() string{
        return this.Name
    }

    func (this *Hero) SetName(newName string){ // 使用指针保证修改的Hero不是副本(引用传递)
        //
        this.Name = newName
    }

    func main (){
        // 创建一个对象
        hero := Hero{Name:"zhang3",Ad:100,Level:1}
        hero.Show()
    }
    ```

- 继承

    ```go
    package main
    import "fmt"

    type Human struct{
        name string
        sex string
    }
    func (this *Human) Eat(){
        fmt.Println("Human.Eat()...")
    }
    func (this *Human) Walk(){
        fmt.Println("Human.Walk()...")
    }

    type SuperMan struct{
        Human // SuperMan类继承了Human类的方法
        level int
    }

    // 重写父类的方法Eat()
    func (this *SuperMan)Eat(){
        fmt.Println("SuperMan.Eat()...")
    }
    // 子类的新方法
    func (this *SuperMan) Fly(){
        fmt.Println("SuperMan.Fly()...")
    }
    func (this *SuperMan) Print(){
        fmt.Println("name = ",this.name)
        fmt.Println("sex = ",this.sex)
        fmt.Println("level = ",this.level)
    }
    func main(){
        h := Huan{"zhang3","female"}

        h.Eat()
        h.Walk()

        // 定义子类对象
        // s := SuperManP{Human{"li4","female"},88} // 此方式初始化时要先初始化父类的结构体
        // 另一种定义和初始化方式
        var s SuperMan
        s.name = "li4"
        s.sex = "female"
        s.level = 88

        s.Walk()
        s.Eat()
        s.Fly()
        s.Print()
    }
    ```

### 多态

- interface 接口

    ```go

    // 本质是一个指针
    type AnimalInterface interface {
        Sleep()
        GetColor() string
        GetType() string
    }

    // 具体的类
    type Cat struct {
        color string
    }
    func (this *Cat) Sleep(){
        fmt.Println("Cat Sleeping...")
    }
    func (this *Cat) GetColor() string {
        return this.color
    }
    func (this *Cat) GetType() string {
        return "Cat"
    }

    // 具体的类
    type Dog struct {
        color string
    }
    func (this *Dog)Sleep(){
        fmt.Println("Dog Sleeping...")
    }
    func (this *Dog) GetColor() string {
        return this.color
    }
    func (this *Dog) GetType() string {
        return "Dog"
    }

    func showAnimal (animal AnimalInterface){
        animal.Sleep()		// 多态，传入何种类型，就会调用什么类型的Sleep()方法
        fmt.Println("Color = ",animal.GetColor())
        fmt.Println("Kind = ",animal.GetType())
    }
    func main(){
        /*var animal AnimalInterface // 接口的数据类型，父类指针

        animal = &Cat{"Green"}
        animal.Sleep()		// 调用的是Cat的Sleep()方法

        animal = &Dog{"Yellow"}
        animal.Sleep() 		// 调用的是Dog的Sleep()方法
        */
        cat := Cat{"Green"}
        dog := Dog{"Yellow"}

        showAnimal(&Cat)
        showAnimal(&Dog)
    }
    ```

- 空接口万能类型与类型断言机制

    ```go
    // 此处的参数 arg 是万能数据类型 interface{}
    func myFunc(arg interface{}){
        fmt.Println("MyFunc is Called...")
        fmt.Println(arg)
        // interface{} 如何区分此时引用的数据类型具体是什么
        value, ok := arg.(string) // 类型断言机制，此处用法是判断arg是否为string类型
        if !ok{
            fmt.Println("arg is not string type")
        }else{
            fmt.Println("arg is string type, value = ",value)
            fmt.Printf("value type is %T\n",value)
        }
    }

    type Book struct{
        auth string
    }

    func main(){
        book := Book{"Golang"}

        myFunc(book)
        myFunc(100)
        myFunc("abc")
    }
    ```

### 反射机制(reflect)及其应用

- **reflect** 能解析变量的类型和具体值

    ```go
    package main

    import (
        "reflect"
        "fmt"
    )

    func reflectNum(arg interface{}){
        fmt.Println("type: ",reflect.TypeOf(arg))
        fmt.Println("value: ",reflect.ValueOf(arg))
    }

    func main(){
        var num float64 = 1.2345
        reflectNum(num)
    }
    ```

- 使用 **reflect** 分析类的内容

    ```go
    package main

    import (
        "fmt"
        "reflect"
    )

    type User struct {
        Id   int
        Name string
        Age  int
    }

    // User 的成员方法
    func (this User) Call() {
        fmt.Println("user is called...")
        fmt.Printf("%v \n", this)
    }

    func main() {
        user := User{1, "Truman", 25}
        DoFieldAndMethod(user)
    }

    func DoFieldAndMethod(input interface{}) {
        // 获取 input 的type
        inputType := reflect.TypeOf(input)
        fmt.Println("inputType is: ", inputType)
        // 获取 input 的value
        inputValue := reflect.ValueOf(input)
        fmt.Println("inputValue is:", inputValue)

        // 通过inputType获取里面的字段(Field)
        for i := 0; i < inputType.NumField(); i++ {
            field := inputType.Field(i)
            value := inputValue.Field(i).Interface()
            fmt.Printf("%s:%v = %v \n", field.Name, field.Type, value)
        }

        // 通过inputType获取里面的方法
        for i := 0; i < inputType.NumMethod(); i++ {
            m := inputType.Method(i)
            fmt.Printf("%s:%v \n", m.Name, m.Type)
        }
    }
    ```

- 通过reflect解析自定义tag内容

    ```go
    package main
    import (
        "fmt"
        "reflect"
    )

    type resume struct {
        Name string `info:"name" doc:"我的名字"`
        Sex string `info:"sex" doc:"性别"`
    }

    func findTag(str interface{}){
        t := reflect.TypeOf(str).Elem() // Elem()方法获取当前结构体的全部元素

        for i:=0;i<t.NumField();i++{
            taginfo := t.Field(i).Tag.Get("info")
            tagdoc := t.Field(i).Tag.Get("doc")
            fmt.Println("info:",taginfo,"doc:",tagdoc)
        }
    }

    func main(){
        var re resume
        findTag(&re)
    }
    ```

- Json标签解析

    ```go
    package main
    import (
        "fmt"
        "encoding/json"
    )

    type Movie struct {
        Title string	`json:"title"`
        Year int		`json:"year"`
        Price int		`json:"rmb"`
        Actors []string	`json:"actors"`
    }

    func main(){
        movie := Movie{"喜剧之王",2000,10,[]string{"zxx","zhangbozhi"}}
        // 编码过程 结构体 ————>Json
        jsonStr,err := json.Marshal(movie)
        if err != nil{
            fmt.Println("json marshal error",err)
            return
        }
        fmt.Printf("jsonStr = %s \n",jsonStr)

        // 解码过程 jsonStr ————> 结构体
        // jsonStr = {"title":"喜剧之王","year":2000,"rmb":10,"actors":["zxx","zhangbozhi"]}
        myMovie := Movie{}
        err = json.Unmarshal(jsonStr,&myMovie)
        if err!= nil{
            fmt.Println("json unmarshal error ",err)
            return
        }
        fmt.Printf("%v \n",myMovie)
    }
    ```

### goroutine

- 基本模型

    ```go
    package main
    import (
    "fmt"
    "time"
    )

    // 子goroutine
    func newTask(){
    i := 0
    for {
        i++
        fmt.Printf ("New Goroutine :i = %d \n",i)
        time.Sleep(1*time.Second)
    }
    }

    // 主goroutine
    func main(){
    // 创建一个goroutine，去执行newTask()流程
    go newTask()

    /*i := 0
    for {
        i++
        fmt.Printf("main goroutine: i = %d\n",i)
        time.Sleep(1*time.Second)
    }*/ // 此段代码会与子goroutine共同执行

    // 10秒后主goroutine退出，子goroutine也会退出
    time.Sleep(10*time.Second)
    }
    ```

- 另一种展示

    ```go
    package main
    import (
        "fmt"
        "time"
    )
    func main(){

        // go 承载一个形参为空，返回值为空的函数
        go func (){
            defer fmt.Println("A.defer")
            func (){
                defer fmt.Println("B.defer")
                // 退出当前goroutine,整个goroutine将会结束，下方的 A 也不会被打印出来
                runtime.Goexit()
                fmt.Println("B")
            }()
            fmt.Println("A")
        }

        // 有参有返回值的写法示例
        go func (a int,b int)bool{
            fmt.Println("a= ",a,"b = ",b)
            return true
        }(10,20)	// 可以执行也可以随时退出，但获取返回值必须依赖Channel机制

        // 死循环
        for{
            time.Sleep(1*time.Second)
        }
        }
    ```

### channel

- 基本定义

    > 一种用于goroutine之间通信的机制，用一种数据类型承载信息

    ```go
    package main

    func main(){
        // 定义一个无缓冲channel
        c := make(chan int)

        go func (){
            defer fmt.Println("goroutine 结束")
            fmt.Println("goroutine 正在执行")
            c <-666 // 将666发送给c变量
        }()

        num := <-c // 从c中接收数据，并赋值给num
        fmt.Println("num = ",num)
        fmt.Println("main goroutine 结束...")
    }
    ```

    上述示例中主程执行到 num= <- c 时，若暂时读取不到值将会发生阻塞，直到c 中被某个协程（示例中为第main中的子goroutine）发送数据；当c 中已被某个协程发送数据，但始终没被读取，则当前协程也将被阻塞直到c中的数据被读取，这种无缓冲的channel机制能够实现协程之间的同步效果

    ```go
    package main
    import (
        "fmt"
        "time"
    )

    func main(){
        c := make (chan int ,3) // 创建带缓冲的channel
        // len()返回当前元素数量，cap()返回当前容量
        fmt.Println("len(c) = ",len(c),"cap(c) = ",cap(c))

        go func(){
            defer fmt.Println("子go程结束")

            for i:=0; i<4;i++{
                c<-i
                fmt.Println("子go程正在运行 发送元素 i : ",i,"len(c)= ",len(c)," cap(c)= ",cap(c))
            }
        }()

        time.Sleep(2*time.Second)
        for i:=0;i<4;i++{
            num := <-c		// 从c中接收数据，并赋值给num
            fmt.Println("num= ",num)
        }
        fmt.Println("main 结束")
    }
    ```

    对于有缓冲 channel ，若channel已满，再次写入数据则会被阻塞，当channel为空时，从中读取数据也会阻塞。前提是channel未被关闭

- **channel** 的关闭

    ```go
    package main

    import "fmt"

    func main(){
        c := make(chan int)

        go func (){
            for i:=0;i<5;i++{
                c<-i
            }
            // close()可以关闭一个channel
            close(c)
        }()

        for {
            // ok 如果channel已关闭且其中没有数据，返回false
            if data,ok := <-c ; ok{
                fmt.Println(data)
            }else{
                break
            }
        }

        fmt.Println("Main Finished")
    }
    ```

    Tips:
    1. 建议不要频繁关闭channel，只在后续完全不需要发送数据时，或者想显示结束某循环之类的逻辑时将其关闭
    2. 向关闭后的channel发送数据会引发panic错误，导致接收方立刻接收到零值（通常会在编译时报错）
    3. 关闭后的channel仍然可以继续接收数据
    4. 对于nil channel，无论收发都会被阻塞

- channel和range

    引用上一个示例

    ```go
    /*for {
        // ok 如果channel已关闭且其中没有数据，返回false
        if data,ok := <-c ; ok{
            fmt.Println(data)
        }else{
            break
        }
    }*/

    // 上述代码可简写为
    for data := range c{
        fmt.Println(data)
    }
    ```

    ### channel和select

    单流程下一个go程只能监控一个channel的状态，使用select可以完成监控多个channel的状态

    ```go
    package main
    import "fmt"

    func fibnacii(c,quit chan int){
        x,y := 1,1
        for{
            select{
                case c<-x:
                    //如果c可写，就会执行该case
                    x = y
                    y = x + y
                case <-quit: // 如果quit可读，就会执行该case
                    fmt.Println("quit")
                    return
            }
        }
    }

    func main(){
        c := make(chan int)
        quit := make (chan int)

        go func (){
            for i:=0;i<6;i++{
                fmt.Println(<-c) // 读值，但当主程不调用fibonacii()方法为c传值时将被阻塞
            }
            quit<- 0
        }()

        fibonacii(c,quit)
    }
    ```
