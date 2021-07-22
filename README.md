# もりけん塾勉強会資料

モンハンで理解する非同期処理の書き方<br>
<br>

### この記事の対象者

- 非同期処理って聞くけど何？
- Promise を調べてみたけど、あまり理解できていない
- 簡単なコードの例ばかりで、実際の使い方がわからない
  <br>

### この記事を読むにあたって身につけて置いた方がよい知識

- 引数、コールバック関数の書き方
- アロー関数の書き方
- 簡単な Promise の知識(みたことあるレベルで可。なくても良いですが、あるとスムーズ)
  <br>

### この記事を呼んで理解できること

- 同期処理とは、非同期処理とは
- Promise 実行時のイメージ
- Promise チェーンや Promise を使った並列処理
  <br>

## 同期処理とは

まず、非同期処理を理解するために、同期処理を説明します。<br>
同期処理とはつまり、順序を守って処理されるということです。一つの処理が完了するまで次の処理には進みません。<br>
処理の順番は決して変わることはなく、上から順に実行されます。<br>
これは JavaScript がシングルスレッドであることが関係しています。<br>
<br>
ということは、もし上に重い処理があったときには、次の処理が行われないということです。<br>
例えば、重い処理とは外部から値を取得する処理などです。<br>
<br>

![重たい同期処理](https://github.com/shouyamamoto/js-study/blob/images/image01.jpg)<br>
<br>

## 非同期処理とは

上記のように、次の処理が行われないことを防ぐために非同期処理が存在します。<br>
非同期処理の書き方は 3 種類あり、<br>
**コールバック関数**、**Promise**、**async, await**です。<br>

まずは非同期処理の書き方をコードでみていきます。

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

コールバックを使って非同期処理を順次実行しようとすると、ネストが深くなってしまい、よく見る「コールバック地獄」というコードが出来上がります。<br>
<br>
そこで、ES6(2015 年)から Promise が使えるようになりました。Promise を使って書くことでネストが深くならず、見通しの良いコードがかけるようになります。

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

さらに Promise よりもさらに直感的にかけるように async await も ES8(2017)から使えるようになりました。<br>
`then()`と囲う必要がないことでさらにコードがすっきりします。<br>
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

<br>
Promise や async await を使うとネストが深くならず、可読性があがるというメリットがあります。<br>
非同期処理を書く際には Promise または async await を使うことになると思います。<br>
今回は Promise の書き方にフォーカスして解説していきます。<br>
Promiseを理解することで、async awaitの理解や、fetchを使うときにも理解が早くなると思います。<br>
<br>

## ざっくり書き方はわかったけど、中身の処理が理解できない...

という方も多いと思います。ここからは絵も取り入れながら、まずはイメージを掴んでもらおうと思います。<br>
今回は「モンハン」を題材にして、解説します。<br>
<br>

モンハンでは依頼者からクエストを受け、モンスターを倒したり、薬草を採取したりなどしてストーリーを進めていきます。<br>
今回は、「回復薬」を作ることをクエスト達成の条件としたいと思います。<br>
イメージはこんな感じです。<br>
<br>

![クエスト依頼](https://github.com/shouyamamoto/js-study/blob/images/image02.jpg)<br>

回復薬を作るためには、「薬草」と「アオキノコ」が必要で、「密林」にいく必要があります。<br>

### クエスト開始！！

さきほど、JavaScript はシングルスレッドであると説明しました。シングルスレッドは直訳で「一本の糸」という意味で、一度に一つの処理しかできません。<br>
クエストを受注したハンターは防具をつけ、武器を選び、モンスターと闘うことも想定していろいろな道具を持っていくなど、クエストにいくまでにはたくさんの準備が必要です。<br>

![ハンター一人でクエストに挑む](https://github.com/shouyamamoto/js-study/blob/images/image03.jpg)<br>
<br>
村を出たハンターは「密林」に到着し、「薬草」「アオキノコ」を採取、そして 2 つを調合して「回復薬」を作り、無事クエスト依頼者に納品しました。
<br>

### 新しい仲間を手に入れる

回復薬を作るだけでもかなり大変なクエストだと感じたハンターは、どうにかして楽をできないかを考えました。そこで、ネコを雇うことに。<br>
![アイルー](https://github.com/shouyamamoto/js-study/blob/images/image04.png)<br>
<br>
モンハンではアイルーというオトモネコを仲間にすることができます。(一緒にクエストいくと可愛い。🐱)<br>
このアイルー(猫)は働き者で、かならず約束を守るイイやつだったのでハンターはこのアイルーに「プロミス」という名前をつけました。<br>
(今後、JS での説明では Promise、モンハンのアイルーはプロミスと分けて記載しています。)<br>
<br>
プロミスはクエストが完了すると、クエスト成功したか失敗したかを教えてくれます。また、成功した時・失敗した時に次の手順を決めてくれます。<br><br>
![プロミスはきちんと報告してくれる](https://github.com/shouyamamoto/js-study/blob/images/image06.jpg)<br>
<br>
プロミスには今回、薬草とアオキノコを採取してきてもらうクエストを任せました。<br>
ハンターは準備やクエストに向かう必要がなくなったので、その時間で回復薬を調合する準備をすることができます。さらに空いた時間で釣りをすることもできました。<br><br>
そしてプロミスはクエストを終え次第、ハンターに「クエストの結果」を知らせ、もしクエストを達成した場合には「薬草」と「アオキノコ」を渡します。<br>
そして、プロミスからクエストの結果と素材を受け取ったハンターは、プロミスのいう通りに回復薬を調合します。<br>
ハンターは調合の準備ができているので、すぐに回復薬を作り依頼者に納品します。<br>

![プロミスにクエストを頼む](https://github.com/shouyamamoto/js-study/blob/images/image05.jpg)<br>
<br>
こうしてハンターはうまく時間を使えるようになりました。<br>
ここで重要なのは、`プロミス（アイルー）がクエストに向かっているときにもハンターの行動が制限されることがない`というところです。<br>
JavaScript に話を戻すと、重い処理を実行していてもクリックやスクロールができるので、ユーザ体験がよくなるということです。

### コードとモンハンを混ぜて解説

まずは基本的な Promise の書き方についてみていきましょう。

```javascript
new Promise((resolve, reject) => {
  // 同期処理 ここでresolveかrejectを実行する
})
  .then() // resolveが実行されたら実行される処理を書く
  .catch(); // rejectが実行されたら実行される処理を書く
```

まず、Promise を使うときには`new Promise`として、Promise から Promise オブジェクトを生成します。<br>
続けて`.then()`や`.catch()`と書くことができます。これらは Promise の処理結果によって実行される処理が変わります。<br>
<br>
ここからは実際のコードと先ほどのモンハンの解説を混ぜて説明します。<br>

```javascript
const restorativeItem = ["薬草", "アオキノコ"];

const getRestorativeItem = (questResult) => {
  return new Promise((resolve, reject) => {
    if (questResult) {
      resolve(restorativeItem);
    } else {
      reject();
    }
  });
};

const createRestorative = (item) => {
  const yakusou = item[0];
  const aokinoko = item[1];
  console.log(`${yakusou} + ${aokinoko} を調合して回復薬を作った！`);
};

getRestorativeItem(true)
  .then((item) => createRestorative(item))
  .catch(() => console.error("クエストに失敗しました。"));
```

まず、1 行目

```js
const restorativeItem = ["薬草", "アオキノコ"];
```

は、回復薬を作るために必要なアイテムを配列に格納しています。<br>

```js
const getRestorativeItem = (questResult) => {
  return new Promise((resolve, reject) => {
    if (questResult) {
      resolve(restorativeItem);
    } else {
      reject();
    }
  });
};
```

次に`new Promise()`としてプロミスを呼び出しています。これからこのプロミスに何をして欲しいかを指示します。（プロミス（アイルー）を呼んでハンターが何をして欲しいかのメモを渡すイメージ）<br>
ここでは`questResult`の結果によって、`resolve`か`reject`を実行するように伝えています。<br>
そして、`resolve`を実行したときには、`restorativeItem`を渡すようにしました。<br>
<br>
次に進みます。

```javascript
getRestorativeItem(true)
  .then((item) => createRestorative(item))
  .catch(() => console.error("クエストに失敗しました。"));
```

先ほど、`.then()`や`.catch()`は Promise の処理結果によって実行される処理が変わるとお伝えしました。<br>
`then()`の第一引数に与えた関数に`item`が渡ってきます。これはどこから渡ってくるのでしょうか？<br>
<br>
答えは、Promise の中にある`resolve(restorativeItem)`から渡ってきています。<br>
このように、`resolve()`は`.then()`と、`reject()`は`.catch()`とペアになっていることがわかります。<br>
(ここではわかりやすいようにペアと説明していますが、厳密には違います。細かい挙動に関しては　[promises-book](https://azu.github.io/promises-book)などをご参照ください)<br>
一連の流れを図解します。<br>
<br>

![プロミスを図解](https://github.com/shouyamamoto/js-study/blob/images/image08.jpg)<br>
<br>

![プロミスを図解](https://github.com/shouyamamoto/js-study/blob/images/promise.gif)<br>

### Promise チェーン

ここまでで、基本的なプロミスの書き方についてはある理解できたかと思います。次は Promise チェーンを解説していきます。<br>
Promise チェーンとは、Promise を使って非同期処理を順次実行していくことです。<br>
またモンハンで例えながら説明していきます。<br>

![新しいクエスト](https://github.com/shouyamamoto/js-study/blob/images/image09.jpg)<br>
<br>
さきほどの回復薬から一つグレードアップした回復薬グレートが欲しい様子です。素材は「薬草」「アオキノコ」「ハチミツ」が必要です。<br>
今回は「薬草」「アオキノコ」は密林のエリア 1 に、「ハチミツ」は密林のエリア 2 にあることを想定します。<br>
なので、まずは密林についたら「薬草」「アオキノコ」を先に採取して、次に「ハチミツ」を採取することにします。<br>

![クエストの達成イメージ](https://github.com/shouyamamoto/js-study/blob/images/image10.jpg)<br>
<br>

では、コードを書いてみましょう。

```javascript
const pocket = [];
const restorativeItem = ["薬草", "アオキノコ"]; // 例えば密林のエリア1にある
const greatRestorativeItem = ["ハチミツ"]; // 例えば密林のエリア2にある
const getRestorativeItem = (targetItem, questResult) => {
  return new Promise((resolve, reject) => {
    if (questResult) {
      setTimeout(() => {
        insertPocket(targetItem);
        resolve(pocket);
      }, 2000);
    } else {
      reject();
    }
  });
};

getRestorativeItem(restorativeItem, true) // getRestorativeItem(採取しにいく素材, 採取できたかどうか)
  .then((pocketItem) => {
    return getRestorativeItem(greatRestorativeItem, true); // getRestorativeItem(採取しにいく素材, 採取できたかどうか)
  })
  .then((pocketItem) => {
    createRestorativeItem(pocketItem); // 調合する処理
  })
  .catch(() => console.log("クエスト失敗..."))
  .finally(() => console.log("クエスト終了"));

const insertPocket = (items) => {
  items.forEach((item) => {
    pocket.push(item);
  });
};

const createRestorativeItem = (pocketItem) => {
  if (pocketItem.length === 3) {
    const yakusou = pocketItem[0];
    const aokinoko = pocketItem[1];
    const hachimitu = pocketItem[2];
    console.log(
      `${yakusou}と${aokinoko}と${hachimitu}を調合して回復薬グレートを作った！！`
    );
  } else {
    console.log(`素材が不足している...調合できずにクエスト失敗`);
  }
};
```

<br>

まずはじめに`getRestorativeItem`を実行し、「薬草」「アオキノコ」を採取しにいきます。<br>
その後に`.then()`と繋いで再度`getRestorativeItem`を実行して「ハチミツ」を採取しにいきます。<br>
<br>
`return`としている理由は、Promise チェーンを繋ぐ場合には then メソッドのコールバック関数の`return`に Promise のインスタンスを設定する必要があるからです。<br>
こうしないと Promise のチェーンが切れてしまい意図した挙動になりません。<br>
<br>
例えば以下のように`return`を書かない場合には、2 回目の`getRestorativeItem`の処理を待たずに次の`createRestorativeItem`が実行されてしまい、`createRestorativeItem`での処理が失敗します。（ハチミツが不足している状態）

```javascript
getRestorativeItem(restorativeItem, true)
  .then((pocketItem) => {
    getRestorativeItem(greatRestorativeItem, true); // return がない場合
  })
  .then((pocketItem) => {
    createRestorativeItem(pocketItem); // ハチミツが足りない状態で回復薬グレートを作り始めてしまう。　-> 素材が不足している...調合できずにクエスト失敗が出力される
  })
  .catch(() => console.log("クエスト失敗..."))
  .finally(() => console.log("クエスト終了"));
```

```js
return がない場合の出力結果;
// ["薬草", "アオキノコ"]
// 何かが足りないようだ...調合失敗
// クエスト終了
// ["ハチミツ"]
```

このように、Promise チェーンを順次実行する際には、「`コールバック関数のreturnにPromiseのインスタンスを設定する`」ことを覚えておきましょう。<br>
<br>

### Promise を並列処理する方法

Promise は並列処理を行うこともできます。並列処理を行うためには`Promise.all`を使います。（他にも`Promise.race`や`Promise.allSettled`などありますが、説明は割愛します。）
では先ほどのコードを`Promise.all`を用いて書き直してみます。

```js
const pocket = [];
const restorativeItem = ["薬草", "アオキノコ"]; // 例えば密林のエリア1にある
const greatRestorativeItem = ["ハチミツ"]; // 例えば密林のエリア2にある

const getRestorativeItem = (targetItem, questResult) => {
  return new Promise((resolve, reject) => {
    if (questResult) {
      setTimeout(() => {
        insertPocket(targetItem);
        resolve(pocket);
      }, 2000);
    } else {
      reject();
    }
  });
};

const insertPocket = (items) => {
  items.forEach((item) => {
    pocket.push(item);
  });
};

const createRestorativeItem = (pocketItem) => {
  if (pocketItem.length === 3) {
    const yakusou = pocketItem[0];
    const aokinoko = pocketItem[1];
    const hachimitu = pocketItem[2];
    console.log(
      `${yakusou}と${aokinoko}と${hachimitu}を調合して回復薬グレートを作った！！`
    );
  } else {
    console.log(`素材が不足している...調合できずにクエスト失敗`);
  }
};

Promise.all([
  getRestorativeItem(restorativeItem, true),
  getRestorativeItem(greatRestorativeItem, true),
])
  .then(() => {
    createRestorativeItem(pocket);
  })
  .catch(() => {
    console.log("クエスト失敗...");
  })
  .finally(() => {
    console.log("クエスト終了");
  });
```

`Promise.all`の中に反復可能オブジェクトを設定し、その中で Promise のインスタンスを格納します。<br>

```js
Promise.all([
  getRestorativeItem(restorativeItem, true),
  getRestorativeItem(greatRestorativeItem, true),
]);
```

今回は反復可能オブジェクトとして配列を使用し、配列に Promise のインスタンスを返す`getRestorativeItem`を 2 つ格納しています。<br>
`Promise.all`を使うと、反復可能オブジェクトに格納した Promise のインスタンスの状態が全て`fulfilled`になるまで次の`.then()`が実行されません。<br>
Promise のインスタンスの状態が`fulfilled`になるということは、全ての処理で`resolve()`が実行されるまでという考え方もできます。<br>
<br>

この`Promise.all()`を使うことで Promise オブジェクトの配列を並列に実行することができるため、同じ処理を直列で行うよりも処理が早くなります。<br>

- https://azu.github.io/promises-book/#ch2-promise-all

例えば、先ほどであれば直列に`.then`を実行していたので、2 秒、2 秒で合計 4 秒かかっていましたが、<br>
`Promise.all`を使うことで 2 秒で処理を終わらせることができます。<br>
<br>
イメージとしてはアイルーを 2 匹雇って、別々のステージに行かせてクエストを早く終わらせるといった感じです。<br>

![ Promise.allのイメージ](https://github.com/shouyamamoto/js-study/blob/images/image11.jpg)<br>
