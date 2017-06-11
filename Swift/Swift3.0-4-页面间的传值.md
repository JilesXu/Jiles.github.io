# 页面间的传值
页面间传值的方式有很多种，正向传值不必多说，反向传值包括了协议代理和闭包。

举个例子，两个类之间的传值，类A调用类B的方法，类B在执行过程中遇到问题通知类A，这时候我们需要用到代理。

当然闭包也是可以解决此类的问题。下面通过一个例子，将闭包和协议代理的回调传值做一个对比。

- 一个tableView，其中的每个cell都包含一个label，点击cell后可以进入下一级页面，页面2中有一个textField，textField中显示有之前的label值，更改textField中的值后返回tabelView，此时对应cell中的label值随之变化。

这次我们只关注页面间的传值，其他的一笔带过就好。

首先初始化页面的各项功能，把`tableView`的代理设置为本viewController
```
    @IBOutlet var myTableView: UITableView!
    public var dataSource: [Dictionary<String, Any>]?
    public var selectIndexPath: IndexPath?
    override func viewDidLoad() {
        super.viewDidLoad()
        self.createDataSource()
        self.selectIndexPath = IndexPath()
        self.myTableView.delegate = self
        self.myTableView.dataSource = self
    }
    public func createDataSource() {
        self.dataSource = [Dictionary]()
        self.dataSource?.append(["name":"Jiles"])
        self.dataSource?.append(["name":"Amy"])
        self.dataSource?.append(["name":"Herry"])
    }
```    
实现`tableView`的各项代理
```
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.dataSource!.count
    }
    
    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell: DelegateTableViewCell = self.myTableView.dequeueReusableCell(withIdentifier: "DelegateCell" , for: indexPath) as! DelegateTableViewCell
        let tempItem: Dictionary? = self.dataSource![indexPath.row]
        if tempItem != nil {
            let personName:String = tempItem!["name"] as! String
            cell.myLabel.text = personName
        }
        return cell
    }
```    
## 下面进入重点<协议代理>
 首先说协议代理的使用方法，
 
 页面A需要遵循页面B的协议，并且告诉B，他的协议，A来代理，即
 ```
 editViewController.delegate = self
 ```
 再实现B定义的方法
 ```
 public func fetchPersonName(name: String)
 ```
 ```
     func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        self.selectIndexPath? = indexPath
        let currentSelectCell: DelegateTableViewCell? = self.myTableView.cellForRow(at: selectIndexPath!) as? DelegateTableViewCell
        let editViewController: DelegateEditViewController = UIStoryboard.init(name: "Main", bundle: Bundle.main).instantiateViewController(withIdentifier: "DelegateEditViewController") as! DelegateEditViewController
        
        /*****方法一：协议代理*****/
        editViewController.delegate = self

        if currentSelectCell != nil {
            print(currentSelectCell!.myLabel.text!)
            
            editViewController.personOldName = currentSelectCell!.myLabel.text!
        self.navigationController?.pushViewController(editViewController, animated: true)
       }
    }
    public func fetchPersonName(name: String) {
        if selectIndexPath != nil {
            //获取当前点击Cell的索引
            let index = (selectIndexPath?.row)!
            print(self.dataSource!)
            //更新数据源中相应的数据
            self.dataSource![index]["name"] = name
            //重载TableView
            self.myTableView.reloadData()
        }
    }
```    
在B页面中需要定义相关的协议让其他的页面去遵循
```
protocol EditViewControllerDelegate {
    func fetchPersonName(name: String) -> Void
}
var delegate: EditViewControllerDelegate?
```
然后在返回上一级页面的地方
```
    public func backToPreview() -> Void {
        let name:String! = self.textField.text
        /*****方法一：协议代理*****/
        if  name != "" {
            if delegate != nil {
                delegate!.fetchPersonName(name: name)
            }
        }
        self.navigationController!.popViewController(animated: true)
    }
```
这样就可以实现相应的功能。

## 闭包
页面A
```
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        self.selectIndexPath? = indexPath
        let currentSelectCell: DelegateTableViewCell? = self.myTableView.cellForRow(at: selectIndexPath!) as? DelegateTableViewCell
        let editViewController: DelegateEditViewController = UIStoryboard.init(name: "Main", bundle: Bundle.main).instantiateViewController(withIdentifier: "DelegateEditViewController") as! DelegateEditViewController

        /*****方法二：闭包*****/
        editViewController.setNameByClosureType(inputClosure: {
            (inputText: String) -> Void in
            self.fetchPersonName(name: inputText)
        })
        
        if currentSelectCell != nil {
            print(currentSelectCell!.myLabel.text!)
            
            editViewController.personOldName = currentSelectCell!.myLabel.text!
        self.navigationController?.pushViewController(editViewController, animated: true)
        }
    }
``` 
页面B
```
    typealias InputClosureType = (String)->Void
    var backClosure: InputClosureType?
    
    public func setNameByClosureType(inputClosure: @escaping InputClosureType) {
        self.backClosure = inputClosure
    }
    
    public func backToPreview() -> Void {
        let name:String! = self.textField.text

        /*****方法二：闭包*****/
        if self.backClosure != nil {
            if name != nil {
                backClosure?(name)
            }
        }
        
        self.navigationController!.popViewController(animated: true)
    }
```
