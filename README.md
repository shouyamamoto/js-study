# もりけん塾勉強会資料 モンハンで理解する非同期処理の書き方

## 同期処理とは

まず、非同期処理を理解するために、同期処理を説明します。<br>
同期処理とはつまり、順序を守って処理されるということです。一つの処理が完了するまで次の処理には進みません。<br>
処理の順番は決して変わることはなく、上から順に実行されます。<br>
ということは、もし上に重い処理があったときには、次の処理が行われないということです。<br>
例えば、重い処理とは外部からファイルを取得する処理などです。

![重たい同期処理](https://github.com/shouyamamoto/js-study/blob/images/image01.jpg)

## 非同期処理とは
