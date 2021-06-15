
## Day1

**（１）モバイルアプリを開発する上で、設計上留意すべき点はどこになるか、サーバサイドやフロントエンドとの違いの観点から説明してください。**  
→[ANSWER]アプリのライフサイクルのKnowledgeと全体の設計を分かるのは大事なことです.  
Design:  
- Decide about app architecture, eg: MVC vs MVVM, swiftui vs uikit  

FrontEnd:  
- The goal to create a design that is both visually appealing and easy to navigate/intuitive. UI and UX of the app should also comply with Apple’s guidelines.  
- While creating the view using auto-layout, consider different size of devices so that it the elements place perfectly for smaller screen(iPhone5) to bigger screen (iPhone12proMax)
- Reduce loading time of elements (eg. rows of tableView)by considering caching, memory management and optimizing backEnd  
- Don't use any method that is announced to be deprecated after certain time

BackEnd:
- Initially create different target for different environment, eg: DEV, STG, PROD  
- Optimize Data receiving process from API  
- Encrypt saved data in local storage, eg: Login Credentials, Api Keys  
- While using other 3rd party libraries, specify version to avoid uncessary error of updated version
- Don't use any method that is announced to be deprecated after certain time

Reduce/Optimize App Size: This is also very important factor  
- Delete unused assets.
- Reduce Used assets size without considering quality. For example: For images it's recommended to use 8-bit PNGs, for Audios you may use lower bit rate audios.  
- Don’t make unnecessary modifications to files while App Updates.  
- Adopt On-Demand Resources to avoid downloading all assets at first download.  

**（２）ViewControllerへの過度な依存や類似/同一コードの重複を避けるため、コード設計上どのような対策をとることが望ましいか、プレゼンテーション層（View）と処理・ビジネスロジック（Controller）それぞれの観点から、実際のコード例を挙げて説明してください。**  
→[ANSWER]We can utilize Service/Helper/Utility for reducing the duplication of same codes.  
[Answer]Service/Helper/Utility を利用して、類似のコードの重複を減らすことができます。
```
class Helper {
      static func someTask() {
      }    
}
// in controllers
Helper.someTask()
```  
プレゼンテーション層（View）  
Considering the code snippet in below, It’s getting the user input and showing using table view
```
class ViewController: UIViewController {

  @IBOutlet weak var tableView: UITableView!

  override func viewDidLoad() {
    super.viewDidLoad()
    self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
  }
}

extension ViewController: UITableViewDataSource {

  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return 100
  }

  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: "cell") else { fatalError() }
    cell.textLabel?.text = "Row \(indexPath.item + 1)"
    return cell
  }
}

extension ViewController: UITableViewDelegate {

  func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("Selected row \(indexPath.item + 1)")
    tableView.deselectRow(at: indexPath, animated: true)
  }
}
```  
We can create an adapter named TableViewAdapter.swift and move delegate and data source code there.  
```
final class TableViewAdapter: NSObject, UITableViewDataSource {

  let tableView: UITableView

  init(tableView: UITableView) {
    self.tableView = tableView
    super.init()

    self.tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")

    self.tableView.dataSource = self
    self.tableView.delegate = self
  }

  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return 100
  }

  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: "cell") else { fatalError() }
    cell.textLabel?.text = "Row \(indexPath.item + 1)"
    return cell
  }
}

extension TableViewAdapter: UITableViewDelegate {

  func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    print("Selected row \(indexPath.item + 1)")
    tableView.deselectRow(at: indexPath, animated: true)
  }
}
```
What we have doen here is -  
1. We added a tableView reference in the adapter file.
2. We added an initializer to get the tableView as the adapter initializes.  

Now we will simply add the adapter in view controller, and the view controller will be so neat and clean.
```
class ViewController: UIViewController {

  @IBOutlet weak var tableView: UITableView!

  private var adapter: TableViewAdapter!

  override func viewDidLoad() {
    super.viewDidLoad()
    self.adapter = TableViewAdapter(tableView: self.tableView)
  }
}
```
## Day2

**（３）以下のコードのTODO箇所を埋めて、SnapKitによるレイアウトを実現するコードを書いてください。その際、下記の条件を満たすこと。**

> - 左右に20ポイントマージンを取って、あとは横幅いっぱいにすること
> - 上下は親ビューの中央に配置されるようにすること
> - 高さは150ポイントとすること

```swift
import UIKit
import SnapKit

class SampleViewController: UIViewController {
    private lazy var sampleView: UIView = {
        let sampleView = UIView()
        sampleView.backgroundColor = .red
        view.addSubView(sampleView)
        return sampleView
    }()
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // TODO: ここでSnapKitを使ってレイアウト制約をつけてください
        //ANSWER:2
        sampleView.snp.makeConstraints { make in
            make.left.equalTo(view).offset(20)
            make.right.equalTo(view).offset(-20)
            make.centerY.equalToSuperview()
            make.height.equalTo(150) 
        }
    }
}
```

**（４）以下のコードを埋めて、プロパティ`view`がタップされたときのタッチイベントをGestureRecognizerで処理するコードを書いてください。その際、下記の条件を満たすこと。**

> - クラスが`view`を認識したときにGestureRecognizerを設定すること
> - タップされたら`print`でログを出力すること

```swift
import UIKit

class SampleView: UIView {
    @IBOutlet private dynamic weak var view: UIView!

    override init() {
        super.init(frame: frame)
        let gestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(changeImage))
        sampleView.addGestureRecognizer(gestureRecognizer)
    }
    
    @objc func changeImage() {
        print("something")
    }
}
```

**（５）あるビューやクラスを別のクラスのプロパティとして持つ場合、どんなときにlazyを使えば良いのか、またlazyを使うことでどんなメリットがあるのか、講座を通じて覚えたことや自分なりの考察を踏まえて説明してください。**

```
どんなときにlazyを使えるか？
-> when the view is not initially needed - ビューが最初から必要ない場合
-> when the initial value depend on other factors - 初期値が他の要因に依存する場合
-> to reduce the memory allocation for the memory heavy application - メモリ負荷の高いアプリケーションのメモリ割り当てを減らす
```


## Day3

**（６）以下のコードを埋めて、UIViewController（呼び出し側）とカスタムビューとの間での処理のやりとりを実現するコードを書いてください。その際、下記の条件を満たすこと。**

> - `SampleCustomView`に`delegate`を設定して、ボタンがタップされた際にUIViewController側でイベントを受け取るようにすること
> - `SampleCustomView`にデータ（`SampleData`）を設定したら、`nameLabel`にデータの`name`を表示させること
> - `SampleCustomView`の`update()`関数を呼び出したら、`nameLabel`に表示される情報を更新すること

```swift
import UIKit

class SampleData {
    var name: String = ""
}

//Answer:6
protocol SampleCustomViewDelegate {
    func buttonTapped(_ view: SampleCustomView)
}
//Answer:6
class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var customView: SampleCustomView!

    override func viewDidLoad() {
        super.viewDidLoad()
        let data = SampleData()
        data.name = "テストデータ"
        customView.data = data
    }
}
extension SampleViewController: SampleCustomViewDelegate {
    func updateData(view: SampleCustomView) {
        // button event
    }
}

class SampleCustomView: UIView {
    var data: SampleData? {
        didSet {
            update()
        }
    }

    func update() {
        // TODO
        //Answer:6
        nameLabel.text = data.name ?? ""
    }
    @IBOutlet private dynamic weak var nameLabel: UILabel!
    @IBOutlet private dynamic weak var button: UIButton!
    var delegate: SampleCustomViewDelegate?
    @IBAction private func buttonTouchUpInside(_ sender: UIButton) {
        // TODO
        //Answer:6
        delegate.buttonTapped(self)
    }
}
```

**（７）下記の要件に従って、以下のコードのTODO箇所を埋めて、コード中の各ImageViewに、指定された方法で画像を表示させるコードを書いてください。**

> - `imageView1`に画像Assetから "icon" を設定すること
> - `imageView2`にアプリのバンドルリソースから "icon2.png" を設定すること
> - `imageView3`にenum`BrandIcon`を使ってアイコンフォントの画像を設定すること
> - `imageView4`にOSS`Kingfisher`を使って、画像URL "https://sample.com/sample.jpg" を読み込むこと

```swift
import UIKit
import Kingfisher

class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var imageView1: UIImageView!
    @IBOutlet private dynamic weak var imageView2: UIImageView!
    @IBOutlet private dynamic weak var imageView3: UIImageView!
    @IBOutlet private dynamic weak var imageView4: UIImageView!
    override func viewDidLoad() {
        super.viewDidLoad()
        // TODO
        //Answer:7
        let imgUrl = "https://sample.com/sample.jpg"
        let brandIcon = BrandIcon.twitter
        imageView1.image = UIImage(named: “icon”)
        imageView2.image = UIImage(named: “icon2.png”, in: Bundle(for: type(of:self)))
        imageView3.image = UIImage.brandIcon(icon: brandIcon, color: brandIcon.color)
        imageView4.kf.setImage(with: imgUrl)
    }
}
enum BrandIcon {
    case twitter
    var text: String {
        switch self {
        case .twitter: return "\u{f099}"
        }
    }
    var color: UIColor {
        let code: Int
        switch self {
        case .twitter: code = 0x55acee
        }
        return UIColor.hexColor(code)
    }
    var name: String {
        switch self {
        case .twitter: return "Twitter"
        }
    }
}
extension UIImage {
    class func brandIcon(icon: BrandIcon, color: UIColor, fontSize: CGFloat, size: CGSize? = nil) -> UIImage? {
        let font = UIFont.faBrand(fontSize)
        return fontImage(font: font, name: icon.text, color: color, fontSize: fontSize, size: size)
    }
    class func fontImage(
        font: UIFont,
        name: String,
        color: UIColor,
        fontSize: CGFloat,
        size: CGSize? = nil
        ) -> UIImage? {
        var imageSize: CGSize = CGSize.zero
        if let size = size {
            imageSize = size
        } else {
            imageSize = CGSize(width: fontSize, height: fontSize)
        }
        UIGraphicsBeginImageContextWithOptions(imageSize, false, 0.0)
        let attributes: [NSAttributedString.Key: AnyObject] = [
            NSAttributedString.Key.backgroundColor: UIColor.clear,
            NSAttributedString.Key.font: font,
            NSAttributedString.Key.foregroundColor: color
        ]
        let attributeString = NSAttributedString(string: name, attributes: attributes)
        let ctx = NSStringDrawingContext()
        let boundingRect = attributeString.boundingRect(with: CGSize(width: fontSize, height: fontSize), options: .usesLineFragmentOrigin, context: ctx)
        attributeString.draw(in: CGRect(
            x: (imageSize.width/2.0) - boundingRect.size.width/2.0,
            y: (imageSize.height/2.0) - boundingRect.size.height/2.0,
            width: imageSize.width,
            height: imageSize.height
            )
        )
        if let iconImage: UIImage = UIGraphicsGetImageFromCurrentImageContext() {
            UIGraphicsEndImageContext()
            return iconImage
        }
        return nil
    }
}
```

**（８）下記の要件に従って、コード内のTODO箇所に、UILabelにBonMotで装飾テキストを設定するコードを書いてください。**

> - 文言は "xxx" のアカウントを使ってログインする" とすること
> - 文言の "xxx" は、enum`BrandIcon`の`twitter`を使ってアイコンフォントの画像と、enum要素の名前を表示すること
> - 文言の"ログイン"は、赤字で表示すること

```swift
import UIKit
import BonMot

class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var textLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // TODO
        // Answer:8
        let twitter = BrandIcon.twitter
        let loginStyle = StringStyle(
                .color(.red)
            )
        let text =  "<twitter>\(twitter.text)</twitter>のアカウントを使って<loginStyle>ログイン</loginStyle>する"

    }
}
extension UIFont {
    class func `default`(_ ofSize: CGFloat) -> UIFont {
        return UIFont(name: "HiraginoSans-W3", size: ofSize)!
    }
    class func defaultBold(_ ofSize: CGFloat) -> UIFont {
        return UIFont(name: "HiraginoSans-W6", size: ofSize)!
    }
    class func faBrand(_ ofSize: CGFloat) -> UIFont {
        return UIFont(name: "FontAwesome5Brands-Regular", size: ofSize)!
    }
}
enum BrandIcon {
    case twitter
    var text: String {
        switch self {
        case .twitter: return "\u{f099}"
        }
    }
    var color: UIColor {
        let code: Int
        switch self {
        case .twitter: code = 0x55acee
        }
        return UIColor.hexColor(code)
    }
    var name: String {
        switch self {
        case .twitter: return "Twitter"
        }
    }
}
extension UIImage {
    class func brandIcon(icon: BrandIcon, color: UIColor, fontSize: CGFloat, size: CGSize? = nil) -> UIImage? {
        let font = UIFont.faBrand(fontSize)
        return fontImage(font: font, name: icon.text, color: color, fontSize: fontSize, size: size)
    }
    class func fontImage(
        font: UIFont,
        name: String,
        color: UIColor,
        fontSize: CGFloat,
        size: CGSize? = nil
        ) -> UIImage? {
        var imageSize: CGSize = CGSize.zero
        if let size = size {
            imageSize = size
        } else {
            imageSize = CGSize(width: fontSize, height: fontSize)
        }
        UIGraphicsBeginImageContextWithOptions(imageSize, false, 0.0)
        let attributes: [NSAttributedString.Key: AnyObject] = [
            NSAttributedString.Key.backgroundColor: UIColor.clear,
            NSAttributedString.Key.font: font,
            NSAttributedString.Key.foregroundColor: color
        ]
        let attributeString = NSAttributedString(string: name, attributes: attributes)
        let ctx = NSStringDrawingContext()
        let boundingRect = attributeString.boundingRect(with: CGSize(width: fontSize, height: fontSize), options: .usesLineFragmentOrigin, context: ctx)
        attributeString.draw(in: CGRect(
            x: (imageSize.width/2.0) - boundingRect.size.width/2.0,
            y: (imageSize.height/2.0) - boundingRect.size.height/2.0,
            width: imageSize.width,
            height: imageSize.height
            )
        )
        if let iconImage: UIImage = UIGraphicsGetImageFromCurrentImageContext() {
            UIGraphicsEndImageContext()
            return iconImage
        }
        return nil
    }
}
```

## Day4

**（９）下記の各データを保存するとき、どのような手法を用いて要件を満たせば良いか、その理由も含めて説明してください。**

①　アプリのインストール後チュートリアルの閲覧が完了したかどうかのフラグ  
ANSWER: FlagはBoolean(小さい値)なので、UserDefaultに保存します. UserDefaultから早くにAccessすることもできます

②　ログインが必要なアプリで、次回起動時にログインを省略するためのAPIキー  
ANSWER: Keychainを使います。UserDefaultをじゃなくてキーチェーンの理由はキーチェーンはユーザーのデフォルトよりも安全です

③　マスターデータ  
ANSWER: CoreData, Because the size of the data might be big, and as CoreData is a graph diagram, if the database is huge than CoreData will allow us to establish relations between entities (one-to-one, one-to-many, etc.)

④　ユーザーまたはアプリ運営者が継続的に投稿しているコンテンツ  
ANSWER: CoreData, Because the size of the data might be big, and as CoreData is a graph diagram, if the database is huge than CoreData will allow us to establish relations between entities (one-to-one, one-to-many, etc.)

⑤　④のコンテンツのキャッシュ  
ANSWER: Realm, If there is cache is not expired than it will use the cache or otherwise API will be called

**（１０）以下の要件を満たすデータ群を、enumを用いて実際のコードで書いてください。**

> - 名前は`Gender`とすること
> - enumの共通init関数である`init(rawValue:Int)`の引数に「ID」を設定することでenum変数を作れるようにすること
> - `name`プロパティを呼ぶことで「名称」を呼び出せること
> - `convert()`関数を呼ぶことで、「男性」の場合は「女性」、「女性」の場合は「男性」、「不明」の場合は「不明」のenum変数を返すようにすること
> - 値の仕様は以下のテーブルの通り

|  | 男性 | 女性 | 不明 |
|---:|:---|:---:|---:|
|ID|1 |2 |0 |
|名前 |男性 |女性 |不明 |

```swift
enum Gender: Int {
      case unknown = 0
      case male    = 1
      case female  = 2
      var genderId: Int

      var name: String {
        convert(genderId)
      }

      func convert(id: Gender) -> String {
          switch genderId {
              case .unknown: return "不明"
              case .male:    return "女性"
              case .female:  return "男性"
          } 
      }
}

```

**（１１）データの取得と加工を、それが必要とする箇所（ViewController側など）ではなく、仲介クラスやメソッド（講座では`Service`というクラスを使った）を使って行ったほうが良い理由をわかりやすく説明してください。**  
→[Answer]Previously in another question, it is asked that how to reduce same code in multiple viewControllers. Service Class would be so handy in that case. We can write common logics/functions in Service class so that we can use that from view controllers. <ServiceClass> n ----------- 1 <ViewControllers>  
For Example to handle rather than writing similar request code in every view controller we can make a Service class named Network and use the the methods dynamically from any view controllers
```
import Foundation
import Alamofire
import SwiftyJSON

typealias CbNet<T> = (T?, AppError?) -> Void

class Network {
    static let TIMEOUT = 90 //in Seconds
    
    static func get<T: ApiModel>(_ url: String, headerMap: [String: Any], cb: @escaping CbNet<T>) {
        //code for get req
    }
    
    static func post<T: ApiModel>(_ url: String, headerMap: [String: Any], params: [String: Any] = [:], encoding: String = "utf8", cb: @escaping CbNet<T>) {
        //Code for post req
    }
    
    private static func request<T: ApiModel>(_ req: URLRequest, cb: @escaping CbNet<T>) {
        //AlamoFire Request...
        
    }
}

```

**（１２）サーバAPIからJSONデータを取得して、モデルクラスの形で呼び出し元まで返す過程を、準備のための実装フローも含めて、箇条書きでできるだけ詳しく、ロジックフローで説明してください。なお、ライブラリはAlamofire, Moya, SwiftyJson, ObjectMapperを使うものとします。**
>     Step1: Make Model following the api response - そのResponseのようにモデリを作る
>     Step2: Request using Alamofire and get response - As using alamofire, JSONSerialization.jsonObject and do-catch is not needed - Alamofireを使ってRequestをしてResponseをもらう
>     Step3: Parse the response using SwiftyJson - As SwiftyJson doesn't require Casting - SwiftJsonを使ってResponseをParseする
>     Step4: After Parsing return as model - モデルにそのResponseをParseする


## Day5

**（１３）複数の異なる型を持つデータ群を１つの配列で持ちたいとする。それらのデータをサーバAPIから取得した場合、どのように実装をすればいいのか、以下のコードのTODO箇所を埋める形でコードを書いてください。なお、ライブラリはAlamofire, Moya, SwiftyJson, ObjectMapperを使うものとします。**

```swift
import Foundation
import Moya
import SwiftyJSON
import Alamofire
import ObjectMapper

enum FeedContentType: Int {
    case dog = 1, cat = 2
}
protocol Feedable: class {
    var feedContentType: FeedContentType { get }
}
class Dog: Mappable, Feedable {
    var id: Int = 0
    var name: String = ""
    // Feedable
    var feedContentType: FeedContentType {
        return .dog
    }
    required convenience init?(map: Map) {
        self.init()
    }
    func mapping(map: Map) {
        id <- map["id"]
        name <- map["name"]
    }
}
class Cat: Mappable, Feedable {
    var id: Int = 0
    var name: String = ""
    // Feedable
    var feedContentType: FeedContentType {
        return .cat
    }
    required convenience init?(map: Map) {
        self.init()
    }
    func mapping(map: Map) {
        id <- map["id"]
        name <- map["name"]
    }
}

enum SampleAPI {
    case getList
}
extension SampleAPI: TargetType {
    var headers: [String: String]? {
        return nil
    }
    var baseURL: URL {
        return URL(string: "http://cs267.xbit.jp/~w065038/app/winas/day5")!
    }
    var path: String {
        switch self {
        case .getList: return "/list.json"
        }
    }
    var method: Moya.Method {
        return .get
    }
    var parameters: [String: Any]? {
        switch self {
        case .getList: return nil
        }
    }
    var sampleData: Data {
        return Data()
    }
    var task: Moya.Task {
        if let parameters = self.parameters {
            return .requestParameters(parameters: parameters, encoding: self.parameterEncoding)
        } else {
            return .requestPlain
        }
    }
    var multipartBody: [Moya.MultipartFormData]? {
        return nil
    }
    var parameterEncoding: Moya.ParameterEncoding {
        return URLEncoding.default
    }
}
struct SampleNetwork {
    static let queue = DispatchQueue(label: "com.winas-lesson.ios.exam.request", attributes: .concurrent)
    static let plugins: [PluginType] = [
        NetworkLoggerPlugin(configuration: NetworkLoggerPlugin.Configuration())
    ]
    static var provider = MoyaProvider<SampleAPI>(plugins: plugins)
    static func request(
        target: SampleAPI,
        success successCallback: @escaping (_ json: JSON?, _ allHeaderFields: [AnyHashable : Any]?) -> Void,
        error errorCallback: @escaping (_ statusCode: Int) -> Void,
        failure failureCallback: @escaping (Moya.MoyaError) -> Void
        ) -> Cancellable
    {
        return provider.request(target, callbackQueue: self.queue) { result in
            switch result {
            case let .success(response):
                let headerFields = response.response?.allHeaderFields
                do {
                    let res = try response.filterSuccessfulStatusAndRedirectCodes()
                    if (res.statusCode >= 300) {
                        successCallback(nil, headerFields)
                    } else {
                        let json = try JSON(response.mapJSON())
                        //let content = try response.mapString()
                        successCallback(json, headerFields)
                    }
                }
                catch let error {
                    if response.statusCode == 200 {
                        successCallback(nil, headerFields)
                    } else {
                        switch error as! Moya.MoyaError {
                        case .statusCode(let response):
                            if let statusCode = StatusCode(rawValue: response.statusCode) {
                                errorCallback(statusCode.rawValue)
                            } else {
                                failureCallback(error as! MoyaError)
                            }
                        default: failureCallback(error as! Moya.MoyaError)
                        }
                    }
                }
            case let .failure(error): failureCallback(error)
            }
        }
    }
}
class ContentService {
    func getList(
        completion: ((_ dataArray: [Feedable]) -> Void)? = { _ in },
        failure: ((_ error: NSError?, _ statusCode: Int?) -> Void)? = { _, _ in }
        ) {
        _ = SampleNetwork.request(
            target: .getList,
            success: { json, _ in
                guard let safeJson = json else { return }
                // run in main => UI thread
                DispatchQueue.main.async {
                    //TODO
                    //Answer:13
                    var dataArray = [Feedable]()

                    safeJson.arrayValue.forEach { jsonObject in
                        guard 
                            let id = jsonObject["content_type"].int,
                            let contentType = FeedContentType(rawValue: id) 
                        else {
                            return
                        }
                        switch contentType {
                        case .dog:
                            guard let dog = Mapper<Dog>().map(JSONObject: jsonObject.dictionaryObject) else {
                                return
                            }
                            dataArray.append(dog)
                        }
                        case .cat:
                            guard let cat = Mapper<Cat>().map(JSONObject: jsonObject.dictionaryObject) else {
                                return
                            }
                            dataArray.append(cat)
                    }
                    completion?(dataArray)
                }
            },
            error: { statusCode in
                failure?(nil, statusCode)
            },
            failure: { error in
                failure?(nil, nil)
            }
        )
    }
}
```

**（１４）あるカスタムビュークラスが、プロパティのデータの型によって異なる表示を行うものとします。その際、ジェネリクスを使った場合の実装、使わない場合の実装のコード例を、以下のコードを埋める形でそれぞれ書いてください。その際、下記の条件を満たすこと。**

> - `SampleCustomView`では、`data`がセットされたときに`nameLabel`に「Dog」または「Cat」の`name`の文字列を表示させること

```swift
import UIKit

class Dog: Animal {
    override var name: String = "いぬ🐶"
}
class Cat: Animal {
    override var name: String = "ねこ🐱"
}
protocol Animal {
    var name: String { get set }
}
class SampleViewController: UIViewController {
    private lazy var customView: SampleCustomView = {
        let customView = SampleCustomView()
        view.addSubview(customView)
        return customView
    }()
    override func viewDidLoad() {
        super.viewDidLoad()
        customView.snp.makeConstraints { make in
            make.top.equalTo(view).offset(0)
            make.left.equalTo(view).offset(0)
            make.right.equalTo(view).offset(0)
            make.bottom.equalTo(view).offset(0)
        }
        let Dog = Dog()
        customView.data = dog //error
    }
}
class SampleCustomView: UIView {
    var data: Animal {
          didSet {
             guard let name = data.name else {return}
             nameLabel.text = name
          }
      }
    @IBOutlet private dynamic weak var nameLabel: UILabel!
}
```

**（１５）あるUIViewControllerが、プロパティが指定されたモデルデータに基づいてUIを構築・表示する役割を持っていた場合について考えます。**

①　その「モデルデータ」がnilになるケースとしてどのようなケースが考えられるか。わかりやすく説明してください。

②　モデルデータがnilであった場合、どのようにして代替処理を行うか、以下のコードのTODO箇所を埋める形でコードを書いてください。

```swift
import UIKit
import ObjectMapper

class Content: Mappable {
    var id: Int = 0
    var name: String = ""
    required convenience init?(map: Map) {
        self.init()
    }
    func mapping(map: Map) {
        id <- map["id"]
        name <- map["name"]
    }
}
class ContentViewController: UIViewController {
    var content: Content?
    // TODO : contentが設定されない場合の代替処理
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // TODO
    }
    private func updateView() {
        // TODO : viewの更新処理（nameLabelのテキストにcontentの`name`を設定すること）
    }
    @IBOutlet fileprivate dynamic weak var nameLabel: UILabel!
}
class ContentService {
    func getContent(
        contentId: Int,
        completion: ((_ content: Content) -> Void)? = { _ in },
        failure: ((_ error: NSError?, _ statusCode: Int?) -> Void)? = { _, _ in }
        )
    {
        // API通信でのデータ取得処理は省略
        let content = Content()
        content.id = 1234
        content.name = "テストコンテンツ"
        completion?(hospital)
    }
}
```

## Day6

**（１６）以下の各ケースでメモリリークが発生しないよう、コードを変更してください。**

ケース①
```swift
import UIKit

class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var customView: SampleCustomView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        customView.viewController = self
    }
    func onCustomViewChanged() {
        print("onCustomViewChanged!!")
    }
}
class SampleCustomView: UIView {
    var viewController: SampleViewController? // Answer:16 = Make it weak var
    @IBOutlet private dynamic weak var button: UIButton!
    @IBAction private func buttonTouchUpInside(_ sender: UIButton) {
        viewController?.onCustomViewChanged()
    }
}
```

ケース②
```swift
import UIKit

class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var customView: SampleCustomView!
    override func viewDidLoad() {
        super.viewDidLoad()
        header.closure = {
            view.backgroundColor = .black
        }
    }
}
class SampleCustomView: UIView {
    var closure: (() -> Void)?
}
```

ケース③
```swift
import UIKit

class SampleViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillShow(_:)), name: UIResponder.keyboardWillShowNotification, object: nil)
    }
    @objc private func keyboardWillShow(_ notification: Foundation.Notification) {
        print("keyboardWillShow!!")
    }
}
```

**（１７）UICollectionViewのカスタムセルのサイズが可変な場合（複数行のテキストが入る場合など）、そのサイズをあらかじめ求める方法を以下のコードのTODOを埋める形で実装してください。**

```swift
import UIKit

class Content {
    var id: Int = 0
    var name: String = ""
}
class SampleViewController: UIViewController {
    var contents = [Content]()
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    @IBOutlet private dynamic weak var collectionView: UICollectionView! {
        didSet {
            collectionView.clipsToBounds = true
            collectionView.delegate = self
            collectionView.dataSource = self
            collectionView.backgroundColor = .clear
            collectionView.register(UINib(nibName: "ContentCollectionViewCell", bundle: nil), forCellWithReuseIdentifier: "ContentCollectionViewCell")
        }
    }
}
extension SampleViewController: UICollectionViewDataSource {
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 1
    }
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return contents.count
    }
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: "ContentCollectionViewCell", for: indexPath
        ) as! ContentCollectionViewCell
        cell.content = contents[indexPath.row]
        return cell
    }
}
extension SampleCollectionViewController: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {}
}
extension SampleCollectionViewController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
    let content = contents[indexPath.row]
        return ContentCollectionViewCell.sizeForItem(content: content, width: collectionView.bounds.width)
    }
}
class ContentCollectionViewCell: UICollectionViewCell {
    // TODO
    class func sizeForItem(content: Content, width: CGFloat) -> CGSize {
        // TODO
        return .zero
    }
    var content: Content?
    private func updateView(_ content: Content) {
        nameLabel.text = content?.name ?? ""
        layoutIfNeeded()
    }
    @IBOutlet private dynamic weak var nameLabel: UILabel!
}
```
