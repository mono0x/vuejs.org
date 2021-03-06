title: フィルタ
type: guide
order: 4
---

## 要約

Vue.js のフィルタは、本質的には「値を取り、加工し、加工した値を返す」関数です。マークアップ内ではパイプ(`|`)で表され 、一つ以上の引数を続けることができます。

``` html
<element directive="expression | filterId [args...]"></element>
```

## 例

フィルタは、 ディレクティブの値の最後に位置しなければなりません。

``` html
<span v-text="message | capitalize"></span>
```

mustache スタイルのバインディング内でも利用することができます。

``` html
<span>{{message | uppercase}}</span>
```

複数のフィルタはお互いに連結できます。

``` html
<span>{{message | lowercase | reverse}}</span>
```

## 引数

いくつかのフィルタはオプションの引数を取ることができます。引数はスペースで区切られています。

``` html
<span>{{order | pluralize 'st' 'nd' 'rd' 'th'}}</span>
```

``` html
<input v-on="keyup: submitForm | key 'enter'">
```

プレーンな文字列引数は、引用符 ('') で囲む必要があります。引用符ではない引数は、現在のデータスコープに対して動的に評価されます。カスタムフィルタのより詳細については後で説明します。

上記の例の明確な利用方法は、[API リファレンス](/api/filters.html)を参照してください。

これで、ディレクティブとフィルタについて知ることができました。では、実際に[リスト表示](/guide/list.html)をやってみましょう。
