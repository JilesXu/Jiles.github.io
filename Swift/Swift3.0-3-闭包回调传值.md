#闭包回调传值
在现实项目中经常会遇到navigation中将secondViewController中的值回传给firstViewController的情况，闭包回调传值是一个不错的方法。

用一个例子来说明这个问题：
有一个navigationcontroller，其中有firstViewController和secondViewController。firstViewController中有一个label和一个button，label显示从secondViewController回传回来的内容，点击button进入secondViewController，secondViewController中有一个textField和一个button
textField中可以输入任何内容，点击button后返回firstViewController并将textfield中的内容带回显示在label上。
```
    @IBAction func goSecondViewBtnPressed(_ sender: UIButton) {
        let secondVC =  UIStoryboard.init(name: "Main", bundle: Bundle.main).instantiateViewController(withIdentifier: "BlockSecondViewController") as! BlockSecondViewController
        secondVC.setBackClosure(tempClosure: {
            (inputText:String)->Void in
            self.showTextLabel.text = inputText
        })
        self.navigationController?.pushViewController(secondVC, animated: true)
    }
```
- 这里secondVC.setBackClosure就是要实现其提供的闭包回调，以便接受回传过来的值。
- 即向secondViewController中传入闭包的具体内容，包体中的内容不立即执行，而是等待下一个页面调用时再执行。
- 形象但是不准确的说就是这里的包体就是一台手机，而secondViewController中定义的函数类型就是手机壳。这里就是将手机放入了手机壳中。
```
    typealias InputClosureType = (String) -> Void
    var backClosure : InputClosureType?
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    ///swift3.0默认闭包不可逃逸,所以需要声明逃逸闭包,闭包setter方法
    func setBackClosure(tempClosure: @escaping InputClosureType) {
        self.backClosure = tempClosure
    }
    @IBAction func backButtonPressed(_ sender: UIButton) {
        if self.backClosure != nil {
            let tempString : String? = self.textField.text
            if tempString != nil {
                backClosure!(tempString!)
            }
        }
        self.navigationController!.popViewController(animated: true)
    }
```
- 这里定义了一个闭包类型（函数类型）typealias InputClosureType = (String) -> Void
- 然后使用这个特定的函数类型声明了一个此函数类型对应的变量 var backClosure : InputClosureType? （即手机壳）
- 通过这个变量来接受上个页面传过来的闭包体，从而把用户输入的内容，通过这个闭包体回传到上个页面。
- backClosure!(tempString!)这里就是调用了上一个页面中包体的内容，将textfield中的内容传给了上一个页面的label
