# AppStoreでのアプリのサイズ計算のやり方

```sh
$ cd path/to/ipa
$ unzip hoge.ipa
$ cd Payload/hoge.app
$ otool -l hoge | grep cryptsize
  cryptsize 200000
  cryptsize 210000
```

** 最悪のケースを想定した場合 **
ipaのサイズ + 200000 + 210000 = AppStoreでのサイズ
