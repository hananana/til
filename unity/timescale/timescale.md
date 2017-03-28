# Time.timeScaleでハマった話

`UnityEngine`名前空間の`Time.timeScale`なんですが、
ポーズ機能の実装などでよくお世話になっていました。
ところが`UniRx`名前空間にある`Observable.Interval`クラスと併用した際にありゃりゃ？となり、
大分恥をかいてしまったので忘備録として。。

# 問題の状況

```
UnityEngine.Time.timeScale = 2.0f;
```

実にシンプルなコードです。
この`Time`クラスは`UnityEngine`名前空間にあり、
ドキュメントでは

>  realtimeSinceStartup を除いて timeScale はすべての時間とデルタ時間を測定する Time クラスの変数が影響します。

と書かれています。

では、ここでクエスチョン。


```
UnityEngine.Time.timeScale = 2.0f;
UniRx.Observable.Interval(System.Timespan.FromSeconds(10)).First().Subscribe(_ => Debug.logger.Log("hoge"));
```

この処理は一体実時間で何秒後にhogeとログに表示されるでしょうか？

答えは**5秒後**です。
