# もりけん塾勉強会資料

モンハンで理解する非同期処理の書き方<br>
<br>

## 同期処理とは

まず、非同期処理を理解するために、同期処理を説明します。<br>
同期処理とはつまり、順序を守って処理されるということです。一つの処理が完了するまで次の処理には進みません。<br>
処理の順番は決して変わることはなく、上から順に実行されます。<br>
これは JavaScript がシングルスレッドであることが関係しています。<br>
<br>
ということは、もし上に重い処理があったときには、**次の処理が行われない**ということです。<br>
例えば、重い処理とは外部から値を取得する処理などです。<br>
<br>

![重たい同期処理](https://github.com/shouyamamoto/js-study/blob/images/image01.jpg)<br>
<br>

## 非同期処理とは

上記のように、次の処理が行われないことを防ぐために非同期処理が存在します。<br>
非同期処理の書き方は 3 種類あり、<br>
**コールバック関数**、**Promise**、**async, await**です。<br>

まずは非同期処理の書き方のコードをみていきます。

- コールバックを使った非同期処理の書き方

```javascript
setTimeout(() => {
  console.log(5);
  setTimeout(() => {
    console.log(4);
    setTimeout(() => {
      console.log(3);
      setTimeout(() => {
        console.log(2);
        setTimeout(() => {
          console.log(1);
          setTimeout(() => {
            console.log(0);
          }, 1000);
        }, 1000);
      }, 1000);
    }, 1000);
  }, 1000);
}, 1000);

// 出力結果 => 5, 4, 3, 2, 1, 0
```

コールバック関数で書くとネストが深くなり、可読性が悪いです。<br>
<br>

- Promise を使った書き方

```javascript
const promiseFactory = (num) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(num);
      resolve();
    }, 1000);
  });
};

promiseFactory(5)
  .then(() => promiseFactory(4))
  .then(() => promiseFactory(3))
  .then(() => promiseFactory(2))
  .then(() => promiseFactory(1))
  .then(() => promiseFactory(0));

// 出力結果 => 5, 4, 3, 2, 1, 0
```

さきほどのコールバックを使った書き方と比べると、実行している順序や場所がわかりやすくなりました。<br>
<br>

- async await を使った書き方

```javascript
const promiseFactory = (num) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(num);
      resolve();
    }, 1000);
  });
};

const putsNum = async () => {
  await promiseFactory(5);
  await promiseFactory(4);
  await promiseFactory(3);
  await promiseFactory(2);
  await promiseFactory(1);
  await promiseFactory(0);
};
putsNum();

// 出力結果 => 5, 4, 3, 2, 1, 0
```

async await を使うと、**直感的に書ける**ようになり、Promise を使うよりもさらにコードの可読性があがったと思います。<br>
<br>

## 書き方はわかったけど、中身の処理が理解できない...

という方も多いと思います。ここからは絵も取り入れながら、まずはイメージを掴んでもらおうと思います。<br>
今回は「モンハン」を題材にして、解説します。<br>
<br>

モンハンではクエストを受け、モンスターを倒したり、薬草を採取したりします。<br>
今回は、**「回復薬」**を作ることをクエスト達成の条件としたいと思います。<br>
イメージはこんな感じです。<br>
<br>

![クエスト依頼](https://github.com/shouyamamoto/js-study/blob/images/image02.jpg)<br>
