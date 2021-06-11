
## Day1

**（１）モバイルアプリを開発する上で、設計上留意すべき点はどこになるか、サーバサイドやフロントエンドとの違いの観点から説明してください。**

Answer: Memory Management (ARC), アプリのLifeCycle

Answer: 
#### Security
- User information seccurity must be considered. 
- Securing API Key, user email, password using encryption.
#### Permission
- Make sure that the app asks for permission to use users bluetooth, mic, photo gallary, camera etc. Double check the plist file to make sure that permission text are enlisted.
#### Be careful when using 3rd Party Library 
- While using any third party library, use the most reliable library. 
#### Server Side
- Create different ```target``` from the beginning. 
- Server side is responsible for providing data. If you receives so many data then load them in a batch. But make sure, this doesn't have any effect of use experience.
#### Front End
- Consider different screen size from the beginning & use auto layout. So, the app looks same on various devices.
- If you use tableView/CollectionView, consider the scroll performance. Use image cache, row cache etc for faster loading.

翻訳：
#### セキュリティ
- 利用者情報の機密性を考慮しなければならない。
- 暗号化を使用して API キー、ユーザーのメール、パスワードを保護します。
#### 許可
- アプリがユーザーの Bluetooth、マイク、フォト ギャラリー、カメラなどの使用許可を求めていることを確認します。plist ファイルを再確認して、許可テキストが含まれていることを確認します。
#### サードパーティのライブラリを使用する場合は注意してください
- サードパーティのライブラリを使用する場合は、最も信頼できるライブラリを使用してください。
#### サーバ側
- 最初から別の ```target``` を作成します。
- サーバー側はデータを提供する責任があります。 大量のデータを受信する場合は、それらをバッチでロードします。 ただし、これは使用経験に影響を与えないことを確認してください。
#### フロントエンド
- 最初から違う画面サイズを考え、オートレイアウトを利用。 そのため、アプリはさまざまなデバイスで同じように見えます。
- tableView/CollectionView を使用する場合は、スクロール性能を考慮してください。 読み込みを高速化するには、画像キャッシュ、行キャッシュなどを使用します。

**（２）ViewControllerへの過度な依存や類似/同一コードの重複を避けるため、コード設計上どのような対策をとることが望ましいか、プレゼンテーション層（View）と処理・ビジネスロジック（Controller）それぞれの観点から、実際のコード例を挙げて説明してください。**

Answer:

First of all consider the following code snippets.
翻訳：まず、次のコード スニペットを検討してください。

```swift
// ViewController.swift
import UIKit

class ViewController: UITableViewController {
    
    var projects = [Project]()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        title = "Winas"
        navigationController?.navigationBar.prefersLargeTitles = true

        guard let url = Bundle.main.url(forResource: "projects", withExtension: "json") else {
            fatalError("Failed to locate projects.json in app bundle.")
        }

        guard let data = try? Data(contentsOf: url) else {
            fatalError("Failed to load projects.json in app bundle.")
        }

        let decoder = JSONDecoder()

        guard let loadedProjects = try? decoder.decode([Project].self, from: data) else {
            fatalError("Failed to decode projects.json from app bundle.")
        }

        projects = loadedProjects
    }

    override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return projects.count
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        let project = projects[indexPath.row]
        cell.textLabel?.attributedText = makeAttributedString(title: project.title, subtitle: project.subtitle)
        return cell
    }

    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let project = projects[indexPath.row]

        guard let detailVC = storyboard?.instantiateViewController(withIdentifier: "DetailViewController") as? DetailViewController else {
            return
        }

        detailVC.project = project
        navigationController?.pushViewController(detailVC, animated: true)
    }

    func makeAttributedString(title: String, subtitle: String) -> NSAttributedString {
        let titleAttributes = [NSAttributedString.Key.font: UIFont.preferredFont(forTextStyle: .headline), NSAttributedString.Key.foregroundColor: UIColor.purple]
        let subtitleAttributes = [NSAttributedString.Key.font: UIFont.preferredFont(forTextStyle: .subheadline)]

        let titleString = NSMutableAttributedString(string: "\(title)\n", attributes: titleAttributes)
        let subtitleString = NSAttributedString(string: subtitle, attributes: subtitleAttributes)

        titleString.append(subtitleString)

        return titleString
    }
}

// Project.swift
struct Project: Codable {
    var number: Int
    var title: String
    var subtitle: String
    var topics: String
}
```
## Centralizing code
This is a massive `viewController`. Let’s start by looking for common functionality that we can move from view controllers.What you’ll see is that viewDidLoad() has a big chunk of code for loading JSON from the bundle, which doesn’t need to be there – this is the kind of thing you might want to do in lots of places.

We can create an `extension` for proccessing & loading JSON from the bundle. Let\`s create a file called `Bundle+JSON.swift`. Now remove the following content from `viewDidLoad()` & insert the into the extension.

翻訳：
## コードの集中化
これは巨大な「viewController」です。 ビューコントローラーから移動できる一般的な機能を探すことから始めましょう。表示されるのは、viewDidLoad() にはバンドルから JSON をロードするためのコードの大きなチャンクがあり、そこにある必要はありません。 いろいろなところでやりたいこと。

バンドルから JSON を処理およびロードするための「拡張機能」を作成できます。 「Bundle+JSON.swift」というファイルを作成しましょう。 次に、「viewDidLoad()」から次のコンテンツを削除し、拡張機能に挿入します。

```swift
extension Bundle {

    func JSONDecode<T: Decodable>(for fileName: String) -> T {
        
        guard let url = Bundle.main.url(forResource: fileName, withExtension: nil) else {
            fatalError("Failed to locat \(fileName) in app bundle.")
        }

        guard let data = try? Data(contentsOf: url) else {
            fatalError("Failed to locat \(fileName) in app bundle.")
        }

        let decoder = JSONDecoder()

        guard let loaded = try? decoder.decode(T.self, from: data) else {
            fatalError("Failed to locat \(fileName) in app bundle.")
        }

        return loaded
    }
}
```
Back in view controller, delete this line from viewDidLoad():
翻訳：ビュー コントローラーに戻り、viewDidLoad() から次の行を削除します。

```swift projects = loadedProjects```
Instead, we can call our new decode() method straight from the property definition:

翻訳：代わりに、プロパティ定義から直接新しい decode() メソッドを呼び出すことができます。

```swift let projects: [Project] = Bundle.main.decode(from: "projects.json")```

Not only has that removed a lot of code from our view controller, but it also turned projects from a variable to a constant – a small but important win.

翻訳：これにより、View Controller から多くのコードが削除されただけでなく、プロジェクトが変数から定数に変わりました。これは、小さいながらも重要な勝利です。

## Model to model
viewController.swift contains some other shared functionality: the ```makeAttributedString()``` method. This is designed to take the title and subtitle of a project, and return them as an attributed string that can be used in table view cells.

Now cut the ```makeAttributedString()``` method to your clipboard, then paste it into the Project model. I don’t think this feels quite right as a method, so I would convert it to a computed property like this:

翻訳：
viewController.swift には、他のいくつかの共有機能が含まれています。```makeAttributedString()`` メソッドです。 これは、プロジェクトのタイトルとサブタイトルを取得し、それらをテーブル ビュー セルで使用できる属性付き文字列として返すように設計されています。

次に、```makeAttributedString()`` メソッドをクリップボードに切り取り、Project モデルに貼り付けます。 これはメソッドとしてはあまり適切ではないと思うので、次のように計算されたプロパティに変換します。

```swift var attributedTitle: NSAttributedString { ```

Now change the following line .　
翻訳：次に、次の行を変更します。

```swift cell.textLabel?.attributedText = makeAttributedString(title: project.title, subtitle: project.subtitle)```
To
```swift cell.textLabel?.attributedText = project.attributedTitle```

### Carving off data sources
At this point, ViewController.swift is responsible for only two major things: acting as a table view data source, and acting as a table view delegate. Of the two, the data source is always an easier target for refactoring because it can be isolated neatly.

So, we’re going to take all the data source code out of ViewController.swift, and instead put it into a dedicated class. This allows us to re-use that data source class elsewhere if needed, or to switch it out for a different data source type as appropriate.

Go to the File menu and choose New > File. Create a new Cocoa Touch Class subclassing NSObject, and name it it “ProjectDataSource”. We’re going to paste a good chunk of code into here in just a moment, but first we need to do two things:

- Add import UIKit to the top of the file, if it isn’t there already.
- Make this new class conform to UITableViewDataSource.

It should look like this:

翻訳：
この時点で、ViewController.swift は、テーブル ビュー データ ソースとして機能することと、テーブル ビュー デリゲートとして機能することの 2 つの主要な処理のみを担当します。 この 2 つのうち、データ ソースはきれいに分離できるため、常にリファクタリングの対象として簡単です。

そのため、ViewController.swift からすべてのデータ ソース コードを取り出し、代わりに専用のクラスに入れます。 これにより、必要に応じてそのデータ ソース クラスを別の場所で再利用したり、必要に応じて別のデータ ソース タイプに切り替えることができます。

[ファイル] メニューに移動し、[新規] > [ファイル] を選択します。 NSObject をサブクラス化する新しい Cocoa Touch クラスを作成し、「ProjectDataSource」という名前を付けます。 すぐに大量のコードをここに貼り付けますが、最初に 2 つのことを行う必要があります。

- ファイルの先頭にインポート UIKit を追加します (まだ存在しない場合)。
- この新しいクラスを UITableViewDataSource に準拠させる。

次のようになります。

```swift
import UIKit

class ProjectDataSource: NSObject, UITableViewDataSource {

}
```
Now for the important part: we need to move numberOfSections, numberOfRows, and cellForRowAt into there, from ViewController.swift. You can literally just cut them to your clipboard then paste them inside ProjectDataSource, but you will need to remove the three override keywords.

Now move the ```projects``` property from ViewController to ProjectDataSource. That will get rid of most, but not all, of the errors.

Create this new property in ViewController.swift:

翻訳：

次に重要な部分です。ViewController.swift から、numberOfSections、numberOfRows、および cellForRowAt をそこに移動する必要があります。 文字通りクリップボードに切り取って ProjectDataSource 内に貼り付けることができますが、3 つのオーバーライド キーワードを削除する必要があります。

次に、```projects``` プロパティを ViewController から ProjectDataSource に移動します。 これにより、ほとんどのエラーが解消されますが、すべてではありません。

ViewController.swift にこの新しいプロパティを作成します。


```swift
let dataSource = ProjectDataSource()
```
Now add this to its ```viewDidLoad()``` method:

```swift
tableView.dataSource = dataSource
```

Add this to ProjectDataSource:
```swift
func project(at index: Int) -> Project {
    return projects[index]
}
```

Now back in the ```didSelectRowAt``` method we can write this:
翻訳：```didSelectRowAt``` メソッドに戻って、次のように記述できます。

```swift
let project = dataSource.project(at: indexPath.row)
```

Now your viewController looks very neat & clean. You can also use viewModel for data proccessing & update . Also you can use coordinator pattern to reduce the responsibility of viewController.

翻訳：
これで、あなたの viewController はとてもきれいできれいに見えます。 データの処理と更新に viewModel を使用することもできます。 また、コーディネーター パターンを使用して、viewController の責任を軽減することもできます。

### View Layer
The view layer has two important tasks:

- presenting data to the user
- and handling user interaction

Views are dumb objects. They only know how to present data to the user. They don't know or understand what they are presenting.

翻訳：

### ビューレイヤー
ビューレイヤーには2つの重要なタスクがあります。

- ユーザーにデータを提示する
- そしてユーザーインタラクションの処理

ビューはダム オブジェクトです。 彼らはユーザーにデータを提示する方法しか知りません。 彼らは彼らが何を提示しているのかを知らないか理解していません

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
        
        // Answer: ③
        sampleView.snp.makeConstraints { make in
            make.leading.equalToSuperview().offset(20)
            make.trailing.equalToSuperview().offset(-20)
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

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupGesture()
    }
    
    required init?(coder aDcoder: NSCoder) {
        super.init(coder: aDcoder)
    }
    
    func setupGesture() {
        let gestureRecognizer: UITapGestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(didTapOnView))
        view.addGestureRecognizer(gestureRecognizer)
    }
    
    @objc private func didTapOnView() {
        print("Tap")
    }
}


```

**（５）あるビューやクラスを別のクラスのプロパティとして持つ場合、どんなときにlazyを使えば良いのか、またlazyを使うことでどんなメリットがあるのか、講座を通じて覚えたことや自分なりの考察を踏まえて説明してください。**

Answer: Lazy properties allow you to create certain parts of a Swift type when needed, rather than doing it as part of its initialization process. This can be useful in order to avoid optionals or to improve performance when certain properties might be expensive to create. It can also help with keeping initializers more clean, since you can defer some of the setup of your types until later in their lifecycle.

There are a few advantages in having a lazy property instead of a stored property.
- The closure associated to the lazy property is executed only if you read that property. So if for some reason that property is not used (maybe because of some decision of the user) you avoid unnecessary allocation and computation.
- You can populate a lazy property with the value of a stored property.
- You can use ```self``` inside the closure of a lazy property

翻訳：

遅延プロパティを使用すると、初期化プロセスの一部として実行するのではなく、必要に応じて Swift 型の特定の部分を作成できます。 これは、オプションを回避したり、特定のプロパティの作成にコストがかかる可能性がある場合にパフォーマンスを向上させるために役立ちます。 また、ライフサイクルの後半まで型のセットアップの一部を延期できるため、初期化子をよりクリーンに保つのにも役立ちます。

格納されたプロパティの代わりに遅延プロパティを使用することには、いくつかの利点があります。
- 遅延プロパティに関連付けられたクロージャは、そのプロパティを読み取った場合にのみ実行されます。 したがって、何らかの理由でそのプロパティが使用されない場合 (おそらくユーザーの何らかの決定のため)、不必要な割り当てと計算を避けることができます。
- 遅延プロパティに保存済みプロパティの値を設定できます。
- 遅延プロパティのクロージャ内で ```self``` を使用できます


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
}

extension SampleViewController: SampleCustomViewDelegate {
    func didTapOnButton(_ view: SampleCustomView) {
        // button pressed
    }
}

protocol SampleCustomViewDelegate {
    func didTapOnButton(_ view: SampleCustomView)
}

class SampleCustomView: UIView {

    var data: SampleData? {
        didSet {
            update()
        }
    }
    
    weak var delegate: SampleCustomViewDelegate?
    
    func update() {
        // TODO
        // answer: 6
        nameLabel.text = data.name
    }
    
    @IBOutlet private dynamic weak var nameLabel: UILabel!
    @IBOutlet private dynamic weak var button: UIButton!
    
    @IBAction private func buttonTouchUpInside(_ sender: UIButton) {
        // TODO
        Answer: 6
        delegate?.didTapOnButton(self)
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
        // answer: 7
        let brandIcon = BrandIcon.twitter
        let imageURL = "https://sample.com/sample.jpg"
        imageView1.image = UIImage(named: "icon")
        imageView2.image = UIImage(named: "icon2.png", in: Bundle(for: type(of:self)), compatibleWith: nil)
        imageView3.image = UIImage.brandIcon(icon: brandIcon, color: brandIcon.color, fontSize: 128)
        imageView4.setImage(with: imageURL)
        
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
    
    func setImage(with urlString: String) {
        guard let url = URL.init(string: urlString) else {
            return
        }
        let resource = ImageResource(downloadURL: url, cacheKey: urlString)
        var kf = self.kf
        kf.indicatorType = .activity
        self.kf.setImage(with: resource)
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
        // answer
        let brand = BrandIcon.twitter
        let text =  "<brand>\(brand.text)</brand>のアカウントを使って<login>ログイン</login>する"
        let attributedString: NSAttributedString = text.styled(with: StringStyle(
            .font(UIFont.default(20)),
            .color(.black),
            .lineSpacing(6),
            .xmlRules([
                .style("login", StringStyle(
                    .color(.red)
                )),
                .style("brand", StringStyle(
                    .font(UIFont.faBrand(20)),
                    .color(brand.color)
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

**（９）下記の各データを保存するとき、どのような手法を用いて要件を満たせば良いか、その理由も含めて説明してください。**

①　アプリのインストール後チュートリアルの閲覧が完了したかどうかのフラグ

Answer: ```swift UserDefaults```
This is just a flag. So, we don\`t need any external or heavy DB in this case. We can simply use the standard userDefaults. 
翻訳：これはただのフラグです。 したがって、この場合、外部または重い DB は必要ありません。 標準の userDefaults を使用するだけです。

②　ログインが必要なアプリで、次回起動時にログインを省略するためのAPIキー

Answer: ```swift UserDefaults or KeyChain```

*** APIキーはログイン情報なので、encryptして保存した方がいいと思います。あとは、UserDefaultsはKeyChainほうど安全ではありません。この場合、Keychainは適切なです。
③　マスターデータ

Answer: ```swift CoreData or Realm``` . 

The ammount of master data might be large. So, we can use core data or Realm. If the data is really large, the its wise to use Realm. Because Realm uses its own engine, simple and fast. Thanks to its zero-copy design, Realm is much faster than ORM, and often faster than SQLite either. Its also cross-platform. So you can use the same DB in both iOS and Android or others. 

翻訳：
マスターデータの量が多い可能性があります。 したがって、コアデータまたはRealmを使用できます。 データが非常に大きい場合は、Realm を使用するのが賢明です。 Realm は独自のエンジンを使用しているため、シンプルで高速です。 ゼロコピー設計のおかげで、Realm は ORM よりもはるかに高速であり、多くの場合 SQLite よりも高速です。 また、クロスプラットフォームです。 そのため、iOS と Android またはその他の両方で同じ DB を使用できます。

④　ユーザーまたはアプリ運営者が継続的に投稿しているコンテンツ
answer: ```swift Realm``` 
⑤　④のコンテンツのキャッシュ
answer: ```swift Realm```

有効期限内のキャッシュがあったらキャッシュを使う、なければAPIを叩く

```swift
    func fetch(_ data: data_id) -> Single<[ArticleEntity]> {
        do {
            let realm = try Realm()
            // キャッシュがあり、有効期限が現在時刻より未来だったらキャッシュを使う
            if let cache = realm.object(ofType: Feed.self, forPrimaryKey: data_id.rawValue),
                cache.expiredDate > now { // <- 日付の比較はよしなに
                let entities = Array(cache.recentArticles)
                return Single<[ArticleEntity]>.just(entities)
            }
        } catch {
            logger?.error(error.localizedDescription)
        }
        // キャッシュがなければAPIを叩いたものを使う
        return fetchLatest(data_id) // APIをヒット
    }
```
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
    case unknown = 0, male = 1, female = 2
    var name: String {
        selectGender(genderId: self)
    }
    
    func convert() -> String {
        selectGender(genderId: self, isToggle: true)
    }
    
    func selectGender(genderId: Gender, isToggle: Bool = false) -> String {
        switch self {
            case .unknown: return "不明"
            case .male: return (isToggle ? "女性" : "男性")
            case .female: return (isToggle ? "男性" : "女性")
        }
    }
}
```

**（１１）データの取得と加工を、それが必要とする箇所（ViewController側など）ではなく、仲介クラスやメソッド（講座では`Service`というクラスを使った）を使って行ったほうが良い理由をわかりやすく説明してください。**

Answer: A ```service``` class is very useful when you want to use the same chunk of function or logics in several places. it centralizes the code and handle logic from one single unit. It reduces the code redundancy. For example-

翻訳：

```service``` クラスは、関数やロジックの同じチャンクを複数の場所で使用したい場合に非常に便利です。 コードと処理ロジックを 1 つのユニットから一元化します。 コードの冗長性を減らします。 例えば-

```swift
class OperationService {
    
    static let numberService = NumberService()

    static func sum(a: Int, b: Int) -> Int {
        return a + b
    }

    static func isDecimal() -> Double {
        // code ....
    }
}

class NumberService {
    
    func convertToDecimal(with number: Int) -> Double {
        // .....
    }
}

```
Now we can use the same ```OperationService``` in various class .
翻訳：
これで、さまざまなクラスで同じ「OperationService」を使用できます。

```swift
class ViewController1: UIViewController {

    //// .....
    let isDecimal: Double = OperationService.isDecimal()
    let convertToDecimal = OperationService.numberService.convertToDecimal(with: 10)
}

class ViewController2: UIViewController {

    //// .....
    let isDecimal: Double = OperationService.isDecimal()
    let convertToDecimal = OperationService.numberService.convertToDecimal(with: 10)
}
```
**（１２）サーバAPIからJSONデータを取得して、モデルクラスの形で呼び出し元まで返す過程を、準備のための実装フローも含めて、箇条書きでできるだけ詳しく、ロジックフローで説明してください。なお、ライブラリはAlamofire, Moya, SwiftyJson, ObjectMapperを使うものとします。**
> 例：　Step1: XXクラスをXXライブラリの仕様に沿うよう、XXする。  
>　　　Step2: XXデータをXXライブラリのXXメソッドを使ってXXする。

Answer: 

#### Step: 1
According to the JSON response , create the model class. For example, consider the following JSON response.
翻訳：
JSONレスポンスに従って、モデルクラスを作成します。 たとえば、次の JSON 応答について考えてみましょう。

```json
{
    "data": [
        {
            "title": "Title", 
            "description": "Description", 
        },
    ], 
    "result": true
}
```

```swift

struct SampleResponse: Decodable {
    var data: [SampleDataModel]?
    var result: Bool?

    enum CodingKeys: String, CodingKey { 
        case data = "data"
        case result = "result"
    }

}

struct SampleDataModel: Decodable {

    var title: String?
    var description: String?

    enum CodingKeys: String, CodingKey { 
        case title = "title"
        case description = "description"
    }
}
```

#### Step: 2
Now will create a Network layer using Alamofire. 

First define the possible errors case using ```enum```

翻訳：
次に、Alamofire を使用してネットワーク レイヤーを作成します。

まず、```enum``` を使用して、起こりうるエラーのケースを定義します。

```swift
// HTTPError.swift

enum HTTPError: String, Error {
    case unAuthorized  // Status code 401
    case forbidden // Status code 403
    case notFound  // Status code 404
    case conflict  // Status code 409
    case internalServerError  // Status code 500
    
    static func getErrorMessage(_ error: HTTPError) -> String {
        switch error {
            case unAuthorized:
                return "Error Found : You must be Authenticated"
            case forbidden:
                return "Error Found : Forbidden"
            case notFound:
                return "Error Found : URL not found"
            case conflict:
                return "Error Found : Conflict occurs"
            case internalServerError:
                return "Error Found : Internal Server Error"
        }
    }
}
```
Now we will prepare our Request Class.

翻訳：
次に、リクエスト クラスを準備します。

```swift

// APIRequest.swift

import Alamofire

class APIRequest<T: Decodable>: URLRequestConvertible {
    
    var path: String { fatalError("上書き必須") }
    var method: HTTPMethod { fatalError("上書き必須") }
    var parameters: Parameters? {
        return nil
    }
    
    var encoding: ParameterEncoding = {
        return JSONEncoding.prettyPrinted
    }()
    
    var headers: HTTPHeaders {
        let loginData = String(format:"%@:%@", "basicAuthUserId", "basicAuthPassword").data(using: String.Encoding.utf8)!
        let base64LoginString = loginData.base64EncodedString()
        let fields: HTTPHeaders = [
            "Accept": "application/json",
            "Cache-Control": "no-cache",
            "Content-Type": "application/json",
            "Authorization" : "Basic \(base64LoginString)",
        ]
        return fields
    }

    func asURLRequest() throws -> URLRequest {
        let url = "BASE URL" // not string , must be URL
        var urlRequest = URLRequest(url: url.addPath(path))
        // Method
        urlRequest.httpMethod = method.rawValue
        // Headers
        urlRequest.headers = headers
        return try encoding.encode(urlRequest, with: parameters)
    }
}

```
Now its time to create API Client class. I will use RxSwift for reactive behaviour.
翻訳：次に、API クライアント クラスを作成します。 リアクティブな動作には RxSwift を使用します。

```swift
// APIClient.swift

import Alamofire
import RxSwift

final class APIClient {
    
    static let `default`: APIClient = .init()
    
    func send<T: Decodable>(_ request: APIRequest<T>) -> Single<T> {
        return self.request(request)
    }
    //MARK: - The request function to get results in an Observable
    private func request<T: Decodable> (_ urlConvertible: URLRequestConvertible) -> Single<T> {
        return Single.create { event in
            let task = AF.request(urlConvertible)
                .responseDecodable { (response: DataResponse<T, AFError>) in
                switch response.result {
                    case .success(let result):
                        event(.success(result))
                    case .failure(let error):
                        switch response.response?.statusCode {
                            case 401:
                                event(.error(HTTPError.unAuthorized))
                            case 403:
                                event(.error(HTTPError.forbidden))
                            case 404:
                                event(.error(HTTPError.notFound))
                            case 409:
                                event(.error(HTTPError.conflict))
                            case 500:
                                event(.error(HTTPError.internalServerError))
                            default:
                                event(.error(error))
                        }
                }
            }
            return Disposables.create {
                task.cancel()
            }
        }
    }
}

```

#### Step-3
Our Network Layer is completed. Now we send a accual API request.
Now write a Sample Request class that extends the ```APIRequest``` class.

翻訳：

ネットワーク層が完成しました。 次に、実際の API リクエストを送信します。
次に、```API Request``` クラスを拡張する Sample Request クラスを作成します。

```swift

// SampleRequest.swift

class SampleRequest: APIRequest<SampleResponse> {
    
    override var method: HTTPMethod {
        return .post
    }

    override var path: String {
        return "/article"
    }
    
    // MARK: - Parameters
    var apiToken: String
    
    override var parameters: Parameters? {
        let params: [String: Any] = [
        ]
        return params
    }
    
    // MARK: Initializers
    init(apiToken: String) {
        self.apiToken = apiToken
    }
}
```
Use this request in viewModel/Any where.
翻訳：このリクエストを viewModel/Any where で使用します。

```swift
// viewModel.swift
import RxSwift

let disposeBag = DisposeBag()

@discardableResult
func getSampleData() -> Single<[SampleDataModel]> {
    let request = SampleRequest(apiToken: "API Token")
    let task = APIClient.default.send(request)
        .asObservable()
        .share()
        .asSingle()
    task.subscribe().disposed(by: disposeBag)
    return task
}

```

Now call your ```getSampleData()``` and subscribe the response.
翻訳：次に、```getSampleData()``` を呼び出して、応答をサブスクライブします。
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
                    if let dataArray = Mapper<Feedable>().mapArray(JSONObject: safeJson.arrayObject) {
                        completion?(dataArray)
                    } else {
                        failure?(nil, nil)
                    }
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
protocol Animal {
    var name: String { get set }
}

class Dog: Animal {
    var name: String = "Dog"
}

class Cat: Animal {
    var name: String = "Cat"
}

class SampleViewController: UIViewController {
    @IBOutlet private dynamic weak var customView: SampleCustomView!
    
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
    var data: Animal? {
        didSet {
            guard let name = data.name else { return }
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
        getContent(contentId: 1) { [weak self] data in
            guard let self = self else { return }
            self.content = data 
            /// reload views ???
            /// or save data to local ???
        } 
        
    }
    private func updateView() {
        guard let content = self.content else {
            return 
        }
        nameLabel.text = content.name
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
        completion?(hospital) // *** 「hospital」？？
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
    weak var viewController: SampleViewController?
    @IBOutlet private dynamic weak var button: UIButton!
    
    @IBAction private func buttonTouchUpInside(_ sender: UIButton) { [weak self] in
        guard let self = self else { return }
        self.viewController.onCustomViewChanged()
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
            DispatchQueue.main.async { [weak self] in
                guard let self = self else { return }
                self.view.backgroundColor = .black
            }
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

    // Answer: 
    denit() {
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

    class func sizeForItem(content: Content, width: CGFloat) -> CGSize {
        if (content.name == "") {
            return .zero
        }
        let boundingHeight = content.name.width(withConstrainedWidth: width, font: UIFont.systemFont(ofSize: 14.0))
        let boundingWidth = content.name.width(withConstrainedHeight: boundingHeight, font: UIFont.systemFont(ofSize: 14.0))
        return CGSize(width: boundingWidth + 40.0, height: boundingHeight)
    }
    var content: Content?
    private func updateView(_ content: Content) {
        nameLabel.text = content?.name ?? ""
        layoutIfNeeded()
    }
    @IBOutlet private dynamic weak var nameLabel: UILabel!
}

extension String {
    func height(withConstrainedWidth width: CGFloat, font: UIFont) -> CGFloat {
		let constraintRect = CGSize(width: width, height: .greatestFiniteMagnitude)
		let boundingBox = self.boundingRect(with: constraintRect, options: .usesLineFragmentOrigin, attributes: [NSAttributedString.Key.font: font], context: nil)
		return ceil(boundingBox.height)
	}

    func width(withConstrainedHeight height: CGFloat, font: UIFont) -> CGFloat {
		let constraintRect = CGSize(width: .greatestFiniteMagnitude, height: height)
		let boundingBox = self.boundingRect(with: constraintRect, options: .usesLineFragmentOrigin, attributes: [NSAttributedString.Key.font: font], context: nil)
		
		return ceil(boundingBox.width)
	}
	
}

```

