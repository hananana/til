心がUnityの話をしたがってるんだ

## はじめに
今年3月より約2年ぶりに仕事でUnityを使うようになりました。
Versionが上がっていることはもちろん、いろいろと面白いAssetが出たりなどしていたので
実験的にプロジェクトに取り込んでみて思うことなどをつらつらと書いてみます。

実際にいろいろやってみたプロジェクトのリポジトリは[こちら](https://github.com/ignis-ltd/CubicDancers)

## Messagingで悩む
Unityで悩ましい問題となるのが各コンポーネントとの橋渡し、Messagingです。
例えばGameというクラスを作成してゲームのロジックを実装したとしましょう。このクラスで算出された結果をViewに反映しなければなりませんが、SpriteRendererやRigidBodyへの参照をModelが持ってしまうのは嫌な臭いがします。じゃあどうする？という話なのですがメジャーなところでは以下のような案があるかと思います

- SendMessage系

```cs
//メッセージを発射するクラス
public class HogeModel : MonoBehaviour {
    void Start(){
        SendMessage("Shou");//stringでメソッド名を指定するというアレな処理
    }
}

//メッセージを受け取るクラス
public class Message : MonoBehaviour {
    public void Show(){
        //something
    }
}
```

UnityManualにある手法ですが、実際のプロジェクトでこれをメインで使ってる例を見たことがないw  

やっぱりstringでメソッド名を指定して処理させるっていう芋臭さがいかんともしがたいのではないでしょうか？さらには上記意図的にtypoしてますがstringなんでコンパイル通ってしまいます。あとコードの見た目も実に良くない＆普通の設定のIDEだと補完が効かないし（vimだったら補完効くけどな！！！）
しかもその上処理が重いとくれば……

- Delegate&Event系

```cs
public class Game : MonoBehavior {
    //下記のおまじない的な書き方が正直冗長と感じている
    public delegate void DidChangeScore(int score);
    static public event DidChangeScore didChangeScore;

    private int score = 0;

    void Hoge(){
        score++;
        didChangeScore(score);
    }
}

public class ScoreText : MonoBehavior {
    void OnEnable(){
        Game.didChangeScore += Fuga;
    }

    void Fuga(int score){
        //something
    }
}
```

大分洗練された雰囲気のコードになり、SendMessageより軽い処理になりました。
ですがDelegate&Eventの宣言は２行に渡る冗長なコードなので、上記Sampleではまだマシですが実際に使ってみるとclassの冒頭がDelegate&Eventまみれになります。ヒッジョーに視認性が悪いです。実際に書いてみると下記のように…

```cs
public class Status {
    //もうカンベンしてくれえええ！
    public delegate void DidChangeLevel(int level);
    static public event DidChangeLevel didChangeLevel;
    public delegate void DidChangeHp(int hp);
    static public event DidChangeHp didChangeHp;;
    public delegate void DidChangeItem(Item item);
    static public event DidChangeItem didChangeItem;;
    public delegate void DidChangeExperience(int experience);
    static public event DidChangeExperience didChangeExperience;
    public delegate void DidChangeScore(int score);
    static public event DidChangeScore didChangeScore;
}
```

またunityの文化的にstaticでeventを定義するのですが、それってそもそもどうなんだろうと思っていたりします。
さらにはエラーが発生した際にログが何やってんだか読みづらいという隠れた問題もあります。

上記のような感じで実際に両方の手法を自分のプロジェクトでやってみたんですが、見事にDisばっかりですねw 
だけどDisったのは当然このやり方よりももっとスマートなやり方があるよ！という話なのです。

## MV(R)Pという考え方
![20150708130424.png](https://qiita-image-store.s3.amazonaws.com/0/41782/16742a42-aeee-2128-11c9-4fc6d7a715d7.png "20150708130424.png")

![MVP_Pattern.png](https://qiita-image-store.s3.amazonaws.com/0/41782/bdaa7ac0-50cc-b883-c672-60452b9dd673.png "MVP_Pattern.png")

[UniRx](http://u3d.as/7tT)というAssetがあります。c#のRxをUnity向けに特化して実装されたものです。(UniRxの詳細は上記URLやggってください。UniRxについてはモチベがあったら別途記事にします)このUniRxの作者neueccさんが提唱するのがUniRxをフル活用したMV(R)Pパターンという設計です。説明画像のように各ComponentをPresenterが仲介し、ReactivePropertyやIObservableなどを用いてMessagingするというものですね。

```cs
//MV(R)Pの実装例
public class Presenter : PresenterBase {

    private GameModel model;
    private ScoreView view;

    protected override IPresenter[] Children(){
        get { return EmptyChildren; }
    }

    protected override void BeforeInitialize(){
        //something
    }

    protected override void Initialize(){
        model.Score.Subscribe(score => view.OnAddScore(score));
    }
}

public class GameModel {

    public IntReactiveProperty Score;

    void AddScore(){
        Score++;
    }
}

public class ScoreView : MonoBehaviour {

    public void OnAddScore(int score){
        //something
    }
}

```

上記のようにModel,Viewは自分より外のクラスの振る舞いを気にせずに済みます。PresenterはIObservable<T>をViewやModelを流し込む橋渡しをするだけです。この思想でイケてるなーと私的に考えられる点の一つめは、Unity独特のHierarchyやMonoBehaviourのことも設計に織り込まれていることです。

![スクリーンショット 2016-05-10 10.25.46.png](https://qiita-image-store.s3.amazonaws.com/0/41782/c4684ff4-1f05-dac3-81ab-cdbbfdaf522a.png "スクリーンショット 2016-05-10 10.25.46.png")  
（Hirarchyはこんな感じでGUIでオブジェクトの関係性を階層で表現したものです)  


地味に厄介な問題としてUnityはSceneが読み込まれた後、GameObjectが初期化される順番は一定ではありません。適当にStart()内でObjectを参照してメソッド起動！とかやると**たまにnull**ってしまい、hageることになります。そこでUniRx。PresenterBaseというクラスにはHierarchy下層にいるPresenterの実行順を制御する仕組みがあります。BeforeInitialize()とInitialize()というprotectedなメソッドを通じてChildrenに安全にアクセスすることができるのです。これでオブジェクトが未生成でうっかりnullになることが無くMessagingしていくことができます。

またもう一つのイケてる点はModelとViewの仲介役としてPresenterクラスが設けられている点です。Modelが直接ViewへEventを流し込む作りは好きではありませんし、SendMessageは~~クソ~~ヒューマンエラーしそうです。Delegate&Eventは冗長過ぎます。またUnityのAPIはbackgroundでは動かないという仕様があるので、Modelクラスはいつでもbackgroundで切り離せるようにUnityのAPIとはなるべく関係無くしておきたいのもあります。Presenterを仲介させるならばModel層からUnityAPIから排除でき、海は干上がり山を崩れ落ちて世界は平和になるのではないでしょうか？

## 参考
http://warapuri.com/post/28972633000/unity%E9%96%8B%E7%99%BA%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B50%E3%81%AEtips-%E3%83%99%E3%82%B9%E3%83%88%E3%83%97%E3%83%A9%E3%82%AF%E3%83%86%E3%82%A3%E3%82%B9%E7%BF%BB%E8%A8%B3
http://tamilabo.blog.fc2.com/blog-entry-21.html
http://www.slideshare.net/torisoup/unirxmvrp
http://neue.cc/2015/04/13_510.html
