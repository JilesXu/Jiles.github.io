#函数和闭包
##函数
- 使用`func`声明一个函数。通过函数名称和参数调用一个函数。使用`->`区分参数名和函数返回的类型。

例：
```
    var name : String?
    func printName(myName:String?) -> Void {
        print("Hello \(myName!)")
    }
    name = "Jiles"
    printName(myName: name)
```
    
 - 函数类型
 
每个函数都有类型，其类型是又函数的参数和返回值决定，用上面的例子
他的函数类型就是`(String)->(void)`
函数类型在构建函数的时候当成参数类型或者返回值类型来使用
    
- 函数声明中包含外部参数名和内部参数名，外部参数名是用来说明用的(不必须)，内部参数名是函数内部使用的(必须)，“_”可以表示没有外部参数名称。

如果开发者不设置函数中参数的外部名称，则全部参数都默认外部名称和内部名称相同.

例：
  var name : String?
  var age : Int?
  func printNameAndAge(with myName:String?, _ myAge:Int?) -> Void {
        print("\(myName!) is \(myAge!) years old.")
  }
  name = "Jiles"
  age = 20
  printNameAndAge(with: name, age)
  注：swift3.0开始函数的第一个内部参数不再可以省略，在函数调用时参数必须写全
  
3.当函数需要返回多个值的时候，可以使用元组来返回
  func caculateStatistics(scores:[Int])->(min:Int,max:Int,sum:Int) {
        var min = scores[0]
        var max = scores[0]
        var sum = 0
        
        for score in scores {
            if score > max {
                max = score
            } else if score < min {
                min = score
            }
            sum += score
        }
        return (min,max,sum)
  }
  let statistics = caculateStatistics(scores:[5,3,100,3,9])
  print(statistics.min)
  print(statistics.1)
  打印：3 100

4.一个函数可以将另一个函数作为返回值返回
  func makeIncrementer() -> ( (Int) -> Int ) {
       //(Int) -> Int 表示返回函数的参数类型和返回类型
       func addOne(number:Int) ->Int {
           return 1+number
       }
       return addOne
  }
  let number = makeIncrementer() // 返回的是addOne函数
  let res = number(7)
  print(res)
  打印：8

5.一个函数可以用其他函数作为参数
    func hasAnyMatches(list: [Int], condition:(Int)->Bool) -> Bool {
        for item in list {
            if condition(item) {
                return true
            }
        }
        return false
    }
    func lessThanTen(number:Int) -> Bool {
        return number < 10
    }
    let numbers = [20, 19, 7, 12]
    //只要number数组有一个元素小于10就返回true
    let isLessTen = hasAnyMatches(list:numbers,condition:lessThanTen)
    print(isLessTen) 
    打印：true

6.可变参数函数
可变参数函数是指函数可以接收不固定个参数。
在参数类型后面添加 … 来标记这个参数为可变参数。
可以在函数中像访问数组一样访问可变参数。
例：
    func foo(names:String...) ->() {
        for name in names {
            print("\(name)")
        }
    }
    foo(names: "zhao","zhang","wang")
    
 7.In-out 参数函数
 传入函数的参数值只能在函数域内改变。
 参数传递是值传递，也就是说我们在函数内部修改了一个参数的值，在函数结束后，函数外部访问到的参数值还是传入函数之前的值，它并没有随着函数内的修改而改变。
 我们需要inout 关键字来标记要修改的参数。
 例：
    func swapTwoInts( a: inout Int, b: inout Int) {
        let temporaryA = a
        a = b
        b = temporaryA
    }
    var firstNumber = 3
    var secondNumber = 107
    swapTwoInts(a: &firstNumber, b: &secondNumber)
    print(firstNumber)
    print(secondNumber)
    打印：107 3
    
/**********闭包**********/
闭包其实就是匿名函数，函数是闭包的一种特殊形式
简书上作者sipdar 是这样说的
1.全局函数是一个有名字但不会捕获任何值的闭包
2.内嵌函数是一个有名字可以捕获到所在的函数域内值的闭包
3.闭包表达式是一个没有名字的可以捕获上下文中的变量或者常量的闭包

闭包的声明
    var myClosure : ((Int, Int) -> Int)?
给闭包变量赋值，其实就是把一个函数体赋值给一个函数类型的变量，和函数的定义区别不大。
但是给闭包变量赋值的函数体中含有参数列表，并且参数列表和真正的函数体之间使用关键字in来分割。
闭包可选变量的调用方式与普通函数没什么两样，唯一不同的是这个函数需要用!来强制打开才可以使用。例：   
    myClosure = { (num1:Int, num2:Int) -> Int in
        return num1 + num2
    }
    print(myClosure!(1, 2))
    打印：3
    
简而言之闭包是以{}包围，函数类型()->()定义的代码模块，并以关键字in来隔离包头和包体
以数组map为例：
    let names = ["zhao","wang","Li"]
    let array = names.map({
        (name:String) -> String in
        "\(name) has been map !"
    })
    print(array)
    打印：["zhao has been map !", "wang has been map !", "Li has been map !"]

已知参数类型的闭包
map的闭包是作为函数参数传入的，Swift可以做类型推断。
所以就不需要在闭包中在描述闭包的函数类型
也就是可以省略 (String) -> (String) 部分，来简写闭包表达式
用上面的例子：
    let array = names.map({
        name in
        "\(name) has been map !"
    })
    
闭包跟函数是引用类型    
无论将函数/闭包赋值给一个常量还是变量，实际上都是将常量/变量的值设置为对应函数/闭包的引用。这也意味着如果将闭包赋值给了两个不同的常量/变量，两个值都会指向同一个闭包.

/**********尾随闭包**********/
如果需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用尾随闭包来增强函数的可读性。
尾随闭包是一个书写在函数括号之后的闭包表达式，函数支持将其作为最后一个参数调用。
在使用尾随闭包时，你不用写出它的参数标签。
例：
正常闭包：
        secondVC.setBackClosure(tempClosure: {
            (inputText:String)->Void in
            self.showTextLabel.text = inputText
        })
尾随闭包：
        secondVC.setBackClosure() {
            (inputText:String)->Void in
            self.showTextLabel.text = inputText
        }
end
