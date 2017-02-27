#Optional
##什么是Optional
Optional是swift中的一种类型，既可以是一个值也可以为空（nil）。他其实是一个enum，包含none和some。
在某些场景Optional会启到很好的作用。

1.当一些类的属性值可以为空时；例如：Person类中的name，age可以为空时，可定义为`var name : string? `

2.当一个方法可以返回空值时；例如：官方的例子
```
//try to convert a String into an Int

let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)

// convertedNumber is inferred to be of type "Int?", or "optional Int"
// 如果possibleNumber 是“hello”,则转换不会成功，就会返回nil
```

##Optional Binding
在swift中Optional值不能被当作boolean值处理
例：这段代码会报错
```
var name : string? = "Jiles"
if name {
  print(name)
}
```
但是可以将Optional值与`nil`做比较
例：
```
if name != nil {
  pring(name!)
} 
```

上述的两个`print`并不相同，一个是`name`一个是`name!`
通过Log可以看出区别：

`print(name)`

打印：Optional("Jiles")

`print(name!)`

打印：Jiles

所以当我们确定name中包含值的时候可以使用`!`强制解包，获取Optional内包含的值。

但是Swift提供了一种更加方便的形式来完成这一过程:Optional Binding
```
if let myName = name {
  print(myName)
}
```
打印：Jiles

##隐式解包Optional
相较于普通的Optional值，隐式解包Optional对它的成员或者方法进行访问时，编译器会自动进行解包。
通过在类型后面添加`!`来告诉编译器这是一个隐式解包Optional：
```
var  name : String! = "Jiles"
print(name)
```
打印：Jiles

隐式解包的Optional与普通的Optional本质上没有差别，只是在访问时，编译器会自动帮我们完成在变量后插入`!`的行为。

那么此处会有一个问题，当访问一个值为空的隐式Optional时，就会遇到一个runtime error。
但是类型安全的swift为什么会有这么一个机智，喵神是这样说的：

>这一切都是历史的锅。因为Object-C中Cocoa的所有类型变量都是可以指向nil的，有一部分Cocoa的API中在参数或者返回时即使被声明为具体的类型，但是还是可能在某些特定的情况下是nil, 而同时也有另一部分API永远不会接受或者返回nil。在Objective-C时，这两种情况并没有加以区别，因为在OC中向nil发送消息是允许的，结果就是什么都不会发生，而在Cocoa API从OC转为Swift的module声明的自动化工具里，是无法判定是否存在nil的可能的，因此也无法决定哪些类型应该是实际的类型，而哪些类型应该声明为Optional。
>在这种自动化转换中，最简单粗暴的应对方式是全部转为 Optional，然后让使用者通过 Optional Binding 来判断并使用。虽然这是最安全的方式，但对使用者来说是一件非常麻烦的事情，我猜不会有人喜欢每次用个 API 就在 Optional 和普通类型之间转来转去。这时候，隐式解包的 Optional 就作为一个妥协方案出现了。使用隐式解包 Optional 的最大好处是对于那些我们能确认的 API 来说，我们可直接进行属性访问和方法调用，会很方便。但是需要牢记在心的是，隐式解包不意味着 “这个变量不会是 nil，你可以放心使用” 这种暗示，只能说 Swift 通过这个特性给了我们一种简便但是危险的使用方式罢了。

##Optional Chaining
我们可以通过一个链来安全的访问一个Optional的属性或者方法。
例：
`myImageView.image?.size`
image的定义
```
//Optional Type
var image : UIimage?
```
size的定义
```
//not Optional Type(普通的类型)
var size : CGSize
```
`myImageView.image?.size`使得整个代码返回的也是Optional Type。
如果image包含值，则获得image并继续下一级取size的值，如果image包含的值为空，则直接返回nil
使用Optional Chaining可以让我们摆脱很多不必要的判断和取值，从而精简代码。

```
// Example 1
var myImageView = UIImageView()
//image 是Option Type，:UIImage?
myImageView.image = UIImage(named:"image1")

//Optional Chaining 返回的也是Optional
var size = anotherImageView.image?.size

//解开Optional Type
if let imageSize = size {
    print("Here's the image size: \(imageSize)")
} else {
    print("This image hasn't been set.")
}
```

##Tips
1.页面间传值时不能带“?”，否则会造成传值不成功
例：当页面A向页面B正向传值时
页面A：
```
        let editViewController: DelegateEditViewController = UIStoryboard.init(name: "Main", bundle: Bundle.main).instantiateViewController(withIdentifier: "DelegateEditViewController") as! DelegateEditViewController
        editViewController.personOldName = "Jiles"
        print(editViewController.personOldName)
        self.navigationController?.pushViewController(editViewController, animated: true)
        打印：Optional("Jiles")
```        
页面B中的定义：
```
        var personOldName:String?
```
end
