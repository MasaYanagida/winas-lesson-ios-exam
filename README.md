
## Day1

**（１）モバイルアプリを開発する上で、設計上留意すべき点はどこになるか、サーバサイドやフロントエンドとの違いの観点から説明してください。**


A.

------
モバイルアプリを設計する時にはlife cycleを把握して全体を設計する必要がある。
->モバイルアプリを開発は「神様の目」みたいな観点で全体一つ一つの画面の観点ではなく全体的な観点、全てのデータと情報の流れ、ユーザーのアクションを把握する必要がある。
サーバーサイドやフロントエンドはページの間で一部のデータを引き継ぐだけ。

------

**（２）ViewControllerへの過度な依存や類似/同一コードの重複を避けるため、コード設計上どのような対策をとることが望ましいか、プレゼンテーション層（View）と処理・ビジネスロジック（Controller）それぞれの観点から説明してください。**

A.

-----
プレゼンテーション層（View）：
過度な依存や類似/同一コードの重複を避けるためカスタムビューを作って画面内のUI部品として分割して呼び出して処理する。

処理・ビジネスロジック（Controller）：
Service/Manager/Helperなどの部品を呼び出すことで複雑な処理、アプリ内の随所で使う機能を防ぐことでViewControllerへの過度な依存やコードの重複を避ける

-----

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
        sampleView.snp.makeConstraints { make in
        make.leading.equalTo(20) //左20ポイントマージン
        make.trailing.equalTo(-20) //右に20ポイントマージン
        make.height.equalTo(150) //高さは150ポイント
        //make.center.equalTo(self.view) //中央に配置
        make.centerY.equalToSuperview()

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

    override init(frame: CGRect) {
        super.init(frame: frame)
        setGestureRecognizer()
    }
    
    required init?(coder aDcoder: NSCoder) {
        super.init(coder: aDcoder)
    }

    func setGestureRecognizer() {
    let gestureRecognizer: UITapGestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(tappedSampleView))
        view.addGestureRecognizer(gestureRecognizer)
    }
    
    @objc private func tappedSampleView() {
        print("sampleView")
    }
    
}
```

**（５）あるビューやクラスを別のクラスのプロパティとして持つ場合、どんなときにlazyを使えば良いのか、またlazyを使うことでどんなメリットがあるのか、講座を通じて覚えたことや自分なりの考察を踏まえて説明してください。**

A.
------
<!-- 1. varとして使う
2. 最初requestする時初期化されてその後、値を保存します。従って最初の値を維持します。→letとしては使えない
3. structとclassで使う -->

lazyは初めて使うまでには演算してない。つまり、必要などころに呼ばれて無駄なメモリを使うのを抑えられます。そして計算した値を保存して再計算を防止します。

lazyを使う時のメリット
 - lazyを使用した場合接近した要素に対したものだけについて演算を行う→演算コストがたくさん使うクロージャを使って配列の要素を扱う時もっと効率的に管理ができる

------
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

class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var customView: SampleCustomView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        let data = SampleData()
        data.name = "テストデータ"
        customView.data = data
    }
    
    //func updateData() {
       // button event
    //}
}

extension SampleViewController: SampleCustomViewDelegate {
    func updateData(view: SampleCustomView) {
        // button event
    }
}

protocol SampleCustomViewDelegate {
    func updateData(view: SampleCustomView)
}

class SampleCustomView: UIView {
    //var data: SampleData?
    var data: SampleData? {
        didSet {
            update()
        }
    }
    var delegate: SampleCustomViewDelegate?
    
    func update() {
        // TODO
        nameLabel.text = data.name
    }
    @IBOutlet private dynamic weak var nameLabel: UILabel!
    @IBOutlet private dynamic weak var button: UIButton!
    @IBAction private func buttonTouchUpInside(_ sender: UIButton) {
        // TODO
        //if let delegate = delegate {
        //   delegate.updateData()
        //}
        delegate?.updateData(self)
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
        self.imageView1.image = UIImage(named: "icon")
        
        let imagePath = Bundle.main.path(forResource: "icon2", ofType: "png")
        self.imageView2.image = UIImage(contentsOfFile:imagePath!)
        
        let brandIcon = BrandIcon.twitter
        imageView.image = UIImage.(icon: brandIcon, color: brandIcon.color, fontSize: 128)
        
        let url = URL(string: "https://sample.com/sample.jpg")
        imageView4.kf.setImage(with: url)
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
        //let favoriteApp = BrandIcon.twitter
        let brand = BrandIcon.twitter
        let text = "<brand>\(favoriteApp.text)</brand>のアカウントを使って<red>ログイン</red>する"
        let attributedString: NSAttributedString = text.styled(with: StringStyle(
            .font(UIFont.default(20)),
            .color(.black),
            .lineSpacing(6),
            .xmlRules([
                .style("red", StringStyle(
                    .color(.red)
                )),
                .style("brand", StringStyle(
                    .font(UIFont.faBrand(20)),
                    .color(favoriteApp.color)
                ))
            ])
        ))
        textLabel.attributedText = attributedString
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

**（９）下記の各データを保存または取得するとき、どのような手法を用いて要件を満たせば良いか、その理由も含めて説明してください。**

①　アプリのインストール後チュートリアルの閲覧が完了したかどうかのフラグ

userDefault : 簡単にkey-valueを解除できる
-> フラグを使うことならば外部DBを利用する必要ではなく閲覧前後のフラグをvalueとして設定すれば簡単に携帯内部で使える

②　ログインが必要なアプリで、次回起動時にログインを省略するためのAPIキー

keyChain : 
データを暗号化して保存するのでパスワードなどのログイン情報を保存する時保安性が良い

userDefault : 
ユーザー基本設定ような簡単なデータの値を扱いには良い

③　マスターデータ

CoreData/SQLite :
端末に保存して管理ができる
->CRUD機能を通じてデータ修正、保存、削除などの管理ができる

④　ユーザーまたはアプリ運営者が継続的に投稿しているコンテンツ

サーバー通信 :
継続的にデータの更新があっても簡潔にデータを提供することができる
->リアルタイムでデータを送ったり受け取ることができるのでサーバーとの連結が切れることではなければ連続的にデータの処理ができる。

⑤　④のコンテンツのキャッシュ

CoreData/SQLite : 
アプリに対して一時的にキャッシュ作業を行い、関連する機能の取り消しができる

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
// TODO : ここにコードを記述してください
enum Gender: Int {
    case unknown = 0, male = 1, female = 2
    var genderId: Int = Gender.unknown.rawValue

    var name: String {
        convert(genderId)
    }

    func convert(_ genderId: Int) -> String {
        switch genderId {
        case 0: return "不明"
        case 1: return "女性"
        case 2: return "男性"
        }
    }
}
```

**（１１）データの取得と加工を、それが必要とする箇所（ViewController側など）ではなく、仲介クラスやメソッド（講座では`Service`という名前をつけたクラスを作った）を使って行ったほうが良い理由をわかりやすく説明してください。**

A.

-----
同じデータが複数のクラスで必要な場合必要な箇所で毎度同じデータを呼び出すクラスやメソッドを定義すると何度も同じコードを入力して維持保守の側面ですごく悪い、

仲介クラスやメソッドを作って同じデータを提供する機能があればそれを必要な箇所で簡単に呼び出すのがViewControllerの負担も減らすしコード管理側面でも効率的。

-----

**（１２）サーバAPIからJSONデータを取得して、モデルクラスの形で呼び出し元まで返す過程を、準備のための実装フローも含めて、箇条書きでできるだけ詳しく、ロジックフローで説明してください。なお、ライブラリはAlamofire, Moya, SwiftyJson, ObjectMapperを使うものとします。**
> 例：　Step1: XXクラスをXXライブラリの仕様に沿うよう、XXする。  
>　　　Step2: XXデータをXXライブラリのXXメソッドを使ってXXする。

A.

-----
Step1: 列挙型でMoyaライブラリの仕様にそうようにTargetTypeの属性を宣言する。

Step2: 宣言したMoyaライブラリのpathからAPI通信ができるようにAlamofireライブラリ仕様に従って宣言する。

Step3: Alamofireライブラリで通信したデータをJson型で表示するためSwiftyJsonの仕様の通りに宣言する。
->responseで受け取ったデータを別のプロセスが必要なくjson()に入れるだけて自動的に仕組みがparsingできるし必要なkeyだけ読んで使う

Step4: 取得したデータをModelクラスでマッピングするためObjectMapperを使ってJsonデータをModelにマッピングする。

-----

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
                    // TODO
                    var dataArray = [Feedable]()
                    safeJson.arrayValue.forEach { jsonObject in
                        guard let contentTypeId = jsonObject["content_type"].int,
                            let contentType = FeedContentType(rawValue: contentTypeId) else {
                            return
                        }
                        switch contentType {
                        case .dog:
                            if let dog = Mapper<Dog>().map(JSONObject: jsonObject.dictionaryObject) {
                                dataArray.append(dog)
                            }
                        case .cat:
                            if let cat = Mapper<Cat>().map(JSONObject: jsonObject.dictionaryObject) {
                                dataArray.append(cat)
                            }
                        }
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
        let cat = Cat()
        customView.data = cat
    }
}
class SampleCustomView: UIView {
    //var data: ??? // TODO: 型名
    var data: Animal? {
        didSet {
            guard let name = data.name else { return }
            nameLabel.text = name
        }
    }
    //ジェネリクス使った場合
    var data: Anuimal? { _ in
            guard let name = data.name else { return }
            nameLabel.text = name
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
    // var viewController: SampleViewController?
    weak var viewController: SampleViewController?
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
            //view.backgroundColor = .black
                DispatchQueue.main.async { [weak self] in
                 guard let self = self else { return }
                 self.view.backgroundColor = .black
             }
        }
    }
}
class SampleCustomView: UIView {
    var closure: (() -> Void)? 
    //private var closure: (() -> Void)? 
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

    deinit() {
        NotificationCenter.default.removeObserver(self, name: UIResponder.keyboardWillShowNotification, object: nil)
    }

    override func viewWillDisappear(animated: Bool) {
        super.viewWillDisappear(animated)
        NotificationCenter.default.removeObserver(self, name: UIResponder.keyboardWillShowNotification, object: nil)
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
            for _ in 0 ..< 20 {
            contents.append(Content.create())
        }
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
    guard let content = contents[safe: indexPath.row] else { return .zero }
        return ContentCollectionViewCell.sizeForItem(content: content, width: collectionView.bounds.width)
    }
}
class ContentCollectionViewCell: UICollectionViewCell {
    // TODO
        struct Static {
        static var cell = ContentCollectionViewCell.fromNib()
        static let cache = NSCache<NSString, NSValue>()
    }

    class func sizeForItem(content: Content, width: CGFloat) -> CGSize {
        // TODO
        let cacheKey: NSString = "ContentCollectionViewCell:\(content.id)" as NSString
        if let size = Static.cache.object(forKey: cacheKey) as? CGSize {
            return size
        }
        let cell = Static.cell
        cell.content = content
        cell.frame.size = CGSize(width: width, height: 0)
        cell.setNeedsLayout()
        var size = cell.systemLayoutSizeFitting(
            CGSize(width: width, height: 0),
            withHorizontalFittingPriority: .required,
            verticalFittingPriority: .defaultLow
        )
        size.width = width
        Static.cache.setObject(NSValue(cgSize: size), forKey: cacheKey)
        return size
    }

    var content: Content?
    private func updateView(_ content: Content) {
        nameLabel.text = content?.name ?? ""
        layoutIfNeeded()
    }
    @IBOutlet private dynamic weak var nameLabel: UILabel!
}
```

