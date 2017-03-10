# さらに進化する万能.global_ycm_extra_conf.py


iOSのframeworkおよび、自作クラス、podでimportしたクラスを自動でflagsに追加スクリプトを改善した。
今回のケースではGoogleAnalyticsのGAIDictionaryBuilderなどの補完が効かず、フォルダを調べてみたことろ下記であった。

```
├── GoogleAnalytics
│   └── Library
│       ├── GAI.h
│       ├── GAIDictionaryBuilder.h
│       ├── GAIEcommerceFields.h
│       ├── GAIEcommerceProduct.h
│       ├── GAIEcommerceProductAction.h
│       ├── GAIEcommercePromotion.h
│       ├── GAIFields.h
│       ├── GAILogger.h
│       ├── GAITrackedViewController.h
│       └── GAITracker.h
├── GoogleTagManager
│   └── Library
│       ├── TAGContainer.h
│       ├── TAGContainerOpener.h
│       ├── TAGDataLayer.h
│       ├── TAGLogger.h
│       └── TAGManager.h
├── Readme.txt
└── libGoogleAnalyticsServices.a
```

ヘッダーから隠蔽した.aファイルを呼ぶパターン。
現状では.mm, .m, .cppなどのファイルを検索してflagsに追加していたが.hについては失念していた。
下記のスクリプトを.global_ycm_extra_conf.pyに追加して無事バリバリ補完が効くようになった。


```python
  header_candidates = []
  for root, dirs, files in os.walk(dir):
    for file in files:
      path, ext = os.path.splitext(file)
      if ext == ".h":
        header_candidates.append(root)
    uniq_headers = list(set(header_candidates))

  for f in uniq_headers:
    flags += ['-isystem']
    flags += [f]
```
