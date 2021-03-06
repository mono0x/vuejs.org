title: コンポーネントシステム
type: guide
order: 11
---

## コンポーネントの使用

Vue.js では [Web Components](http://www.w3.org/TR/components-intro/) と類似した概念を持つ再利用可能なコンポーネントとして、polyfill 無しで、拡張された Vue サブクラスを扱うことができます。コンポーネントを作るためには、 `Vue.extend()` を用いて Vue のサブクラスコンストラクタを生成します:

``` js
// 再利用可能なコンストラクタを取得するために Vue を拡張します
var MyComponent = Vue.extend({
  template: '<p>A custom component!</p>'
})
```

Vue のコンストラクタに渡すことができるほとんどのオプションは `Vue.extend()` で利用可能です。しかし、 `data` と `el` は例外ケースです。各 Vue インスタンスは `$data` と `$el` をそれぞれが持つべきであるため、 `Vue.extend()` に渡され、コンストラクタを通じて作られる全てのインスタンスを横断して共有されることは好ましくありません。したがって、コンポーネントを定義する際にデフォルトの `data` や `element` を初期化したい場合は、代わりに関数を渡しましょう:


``` js
var ComponentWithDefaultData = Vue.extend({
  data: function () {
    return {
      title: 'Hello!'
    }
  }
})
```

次に、 `Vue.component()` を使ってコンストラクタを**登録**しましょう:

``` js
// my-component という id でコンストラクタを登録する
Vue.component('my-component', MyComponent)
```

より物事を簡単にするために、コンストラクタの代わりにオプションのオブジェクトを直接渡すこともできます。`Vue.component()` はオブジェクトを受け取った場合、暗黙的に `Vue.extend()` を呼び出します:

``` js
// Note: この関数はグローバルな Vue を返し、
// 登録されたコンストラクタを返すものではありません。
Vue.component('my-component', {
  template: '<p>A custom component!</p>'
})
```

また、登録したコンポーネントを親インスタンスのテンプレート内で使用することもできます（ルートの Vue インスタンスを初期化する**前に**そのコンポーネントが登録されていることを確認してください）:

``` html
<!-- 親テンプレートの内部 -->
<my-component></my-component>
```

レンダリング内容:

``` html
<p>A custom component!</p>
```

毎回グローバルなコンポーネントを登録する必要はありません。`components` オプションでそれを渡すことによって、別のコンポーネントへのコンポーネントの可用性とその子孫を制限することができます (このカプセル化は、このようなディレクティブやフィルタなどのその他のアセットに適用されます):

``` js
var Parent = Vue.extend({
  components: {
    child: {
      // 子は親と親の子孫コンポーネントだけ利用できる
    }
  }
})
```

`Vue.extend()` と `Vue.component()` の違いを理解することは重要です。`Vue` 自身はコンストラクタであるため、`Vue.extend()` は**クラス継承メソッド**です。そのタスクは `Vue` のサブクラスを生成して、そのコンストラクタを返すものです。一方、 `Vue.component()` は**アセット登録メソッド**であり、`Vue.directive()` や `Vue.filter()` と類似しています。そのタスクは与えられたコンストラクタに文字列のIDを関連付けて、 Vue.js がそれをテンプレートの中で利用できるようにするものです。直接 `Vue.component()` にオプションを渡した時は、内部的に `Vue.extend()` が呼ばれます。

Vue.js はコンポーネントの使い方として二つの異なる API スタイルをサポートしています: コンストラクタベースの命令的な API とテンプレートベースの API です。もし混同してしまう場合は、image エレメントを `new Image()` を作るか、 `<img>` タグで作るかということを考えてみてください。どちらもそれ自体で有効的であり、Vue.js は最大限の柔軟性のためにどちらの方式も提供しています。

<p class="tip">`table` 要素は、要素がその内部に表示できるものに制限があるため、カスタム要素が押し上げられてしまい正しくレンダリングされません。これらのケースではコンポーネントディレクティブシンタックスを使うことができます: `<tr v-component="my-component"></tr>`</p>

## データの流れ

### Props による伝達

デフォルトでは、コンポーネントは**隔離されたスコープ (isolated scope) **を持ちます。これが意味するところは、子コンポーネントのテンプレートの中で親データの参照ができないということです。データを隔離されたスコープで子コンポーネントに渡すためには、`props` を利用する必要があります。

"prop" は、親コンポーネントから受信されることを期待されるコンポーネントデータ上のフィールドです。子コンポーネントは、[`props` オプション](/api/options.html#props)を利用して受信することを期待するために、明示的に宣言する必要があります:

``` js
Vue.component('child', {
  // props を宣言
  props: ['msg'],
  // prop は内部テンプレートで利用でき、
  // そして `this.msg` として設定される
  template: '<span>{{msg}}</span>'
})
```

そのとき、以下のようにデータを渡すことができます:

``` html
<child msg="hello!"></child>
```

**結果:**

<div id="prop-example-1" class="demo"><child msg="hello!"></child></div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    child: {
      props: ['msg'],
      template: '<span>{&#123;msg&#125;}</span>'
    }
  }
})
</script>

### キャメルケース vs ハイフン付き

HTML の属性は大文字と小文字を区別しません。キャメルケースされた prop 名を属性として使用するとき、それらハイフン付き相当語句として使用する必要があります:

``` js
Vue.component('child', {
  props: ['myMessage'],
  template: '<span>{{myMessage}}</span>'
})
```

``` html
<!-- 重要: ハイフン付きの名前を使用! -->
<child my-message="hello!"></child>
```

### 動的な Props

親から動的なデータを受け取ることができます。例えば:

``` html
<div>
  <input v-model="parentMsg">
  <br>
  <child msg="{{parentMsg}}"></child>
</div>
```

**結果:**

<div id="demo-2" class="demo"><input v-model="parentMsg"><br><child msg="{&#123;parentMsg&#125;}"></child></div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    parentMsg: 'Inherited message'
  },
  components: {
    child: {
      props: ['msg'],
      template: '<span>{&#123;msg&#125;}</span>'
    }
  }
})
</script>

<p class="tip">prop として `$data` を公開することも可能です。渡される値は、オブジェクトでなければならず、コンポーネントをデフォルト `$data` に置き換えます。</p>

### Props のバインディングタイプ

デフォルトで、全ての props は子プロパティと親プロパティとの間で **one way down** バインディングです。親プロパティが更新するとき子と同期されますが、その逆はありません。このデフォルトは、子コンポーネントが誤ってアプリのデータフローが推理しづらい親の状態の変更しないように防ぐためです。しかしながら、明示的に two-way または one-time バインディングを強いることも可能です: 

シンタックスの比較:

``` html
<!-- デフォルトは one-way-down バインディング -->
<child msg="{{parentMsg}}"></child>
<!-- 明示的な two-way バインディング -->
<child msg="{{@ parentMsg}}"></child>
<!-- 明示的な one-time バインディング -->
<child msg="{{* parentMsg}}"></child>
```

two-way バインディングは子の `msg` プロパティの変更を親の `parentMsg` プロパティに戻して同期します。one-time バインディングは、一度セットアップし、親と子との間では、先の変更は同期しません。

<p class="tip">もし、渡される prop がオブジェクトまたは配列ならば、それは参照で渡されることに注意してください。オブジェクトの変更または配列は、使用しているバインディングのタイプに関係なく、子の内部それ自身は、親の状態に影響を与えます。</p>

### Prop 仕様

コンポーネントは受け取る props に対する必要条件を指定することができます。これは他の人に使用されるために目的とされたコンポーネントを編集するときに便利で、これらの prop 検証要件は本質的にはコンポーネントの API を構成するものとして、ユーザーがコンポーネントを正しく使用しているということを保証します。文字列として定義している props の代わりに、検証要件を含んだオブジェクトを使用できます:

``` js
Vue.component('example', {
  props: {
    // 基本な型チェック (`null` はどんな型でも受け付ける)
    onSomeEvent: Function,
    // 存在チェック
    requiredProp: {
      type: String,
      required: true
    },
    // デフォルト値
    propWithDefault: {
      type: Number,
      default: 100
    },
    // オブジェクト/配列のデフォルトはファクトリ関数から返されるべきです
    propWithObjectDefault: {
      type: Object,
      default: function () {
        return { msg: 'hello' }
      }
    },
    // two-way prop は、もしバインディングの型が一致しない場合は警告を投げます
    twoWayProp: {
      twoWay: true
    },
    // カスタムバリデータ関数
    greaterThanTen: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

`type` は次のネイティブなコンストラクタのいずれかになります:

- String
- Number
- Boolean
- Function
- Object
- Array

加えて、`type` はカスタムコンストラクタ、そして assertion は `instanceof` チェック もできます。

prop 検証が失敗するとき、Vue は値を子コンポーネントへのセットを拒否し、そしてもし開発ビルドを使用している場合は警告します。

### Props としてのコールバックの伝達

メソッドや、子コンポーネントへのコールバックなどのステートメントを渡すのも可能です。これは宣言、切り離された親子間のコミュニケーションを可能にします:

``` js
Vue.component('parent', {
  // ...
  methods: {
    onChildLoaded: function (msg) {
      console.log(msg)
    }
  }
})

Vue.component('child', {
  // ...
  props: ['onLoad'],
  ready: function () {
    this.onLoad('message from child!')
  }
})
```

``` html
<!-- 親のテンプレート -->
<child on-load="{{onChildLoaded}}"></child>
```

### 親スコープの継承

もし必要な場合は `inherit: true` オプションを使用して子コンポーネントに対して、親の全てのプロパティをプロトタイプ継承させることができます:

``` js
var parent = new Vue({
  data: {
    a: 1
  }
})
// $addChild() はインスタンスメソッドで
// プログラムで子インスタンスを生成することができます
var child = parent.$addChild({
  inherit: true,
  data: {
    b: 2
  }
})
console.log(child.a) // -> 1
console.log(child.b) // -> 2
parent.a = 3
console.log(child.a) // -> 3
```

注意点: Vue インスタンスにおける各 data プロパティは getter / setter であるため、`child.a = 2` とセットすることは、親のプロパティをコピーして子に新規プロパティを作成する代わりに、 `parent.a` を変更します:

``` js
child.a = 4
console.log(parent.a) // -> 4
console.log(child.hasOwnProperty('a')) // -> false
```

### スコープに関する注釈

コンポーネントが親テンプレートの中で使用される時、e.g.:

``` html
<!-- 親テンプレート -->
<my-component v-show="active" v-on="click:onClick"></my-component>
```

このディレクティブ (`v-show` と `v-on`) は親のスコープでコンパイルされます。そのため、 `active` という値と `onClick` は親で解決されます。子テンプレート内のいかなるディレクティブや挿入句は子のスコープでコンパイルされます。これによって、親と子のコンポーネント間のクリーンな住み分けが実現できます。

詳細については[コンポーネントのスコープ](/guide/best-practices.html#コンポーネントのスコープ)を読んでください。

## コンポーネントライフサイクル

全てのコンポーネントや Vue インスタンスは自身のライフサイクルを持ちます: created、compiled、attached、detached、と最後に destroyed です。それぞれのキーとなるタイミングでインスタンスは対応したイベントを emit します。また、インスタンスの生成やコンポーネント定義の際に、それぞれのイベントに反応するためのライフサイクル hook 関数を渡すことができます。例えば:

``` js
var MyComponent = Vue.extend({
  created: function () {
    console.log('An instance of MyComponent has been created!')
  }
})
```

[ライフサイクル](/api/options.html#ライフサイクル)で利用可能な API リファレンスを確認してください。

## 動的コンポーネント

予約された `component` 要素を使って、"ページをスワップ" を成し遂げるためにコンポーネントを動的に切り替える仕組みがあります:

``` js
new Vue({
  el: 'body',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component is="{{currentView}}">
  <!-- vm.currentview が変更されると、中身が変更されます! -->
</component>
```

状態を保持したりや再レンダリングを避けたりするために、もし切り替えられたコンポーネントを活性化された状態で保持したい場合は、ディレクティブのパラメータ `keep-alive` を追加することができます:

``` html
<component is="{{currentView}}" keep-alive>
  <!-- 非活性になったコンポーネントをキャッシュします! -->
</component>
```

## トランジション操作

2つの追加の param 属性により、コンポーネントがレンダリングまたはトランジションされるべきかの高度な操作が可能になります。

### `wait-for`

DOM と切り替えられる前に、挿入される子コンポーネントを待つためのイベント名です。トランジションの開始そして空のコンテンツ表示を回避する前に非同期なデータのロードを待つことが可能になります。

この属性は、静的そして動的コンポーネント上の両方で使用できます。動的コンポーネントでは、全てのコンポーネントが潜在的に待機イベントを `$emit` する必要があるためにレンダリングされ、それ以外の場合は、それらは挿入されることはないことに注意してください。

**例:**

``` html
<!-- 静的 -->
<my-component wait-for="data-loaded"></my-component>

<!-- 動的 -->
<component is="{{view}}" wait-for="data-loaded"></component>
```

``` js
// コンポーネントの定義
{
  // compiled フックの中で非同期にデータを取得してイベントを発火します。
  // 例として jQuery を使っています。
  compiled: function () {
    var self = this
    $.ajax({
      // ...
      success: function (data) {
        self.$data = data
        self.$emit('data-loaded')
      }
    })
  }
}
```

### `transition-mode`

`transition-mode` パラメータ属性はどうやって2つの動的コンポーネント間でトランジションが実行されるべきかどうか指定できます。

デフォルトでは、入ってくるコンポーネントと出て行くコンポーネントのトランジションが同時に起こります。この属性によって、2つの他のモードを設定することができます:

- `in-out`: 新しいコンポーネントのトランジションが初めに起こり、そのトランジションが完了した後に現在のコンポーネントの出て行くトランジションが開始します。
- `out-in`: 現在のコンポーネントが出て行くトランジションが初めに起こり、そのトランジションが完了した後に新しいコンポーネントのトランジションが開始します。

**例**

``` html
<!-- 先にフェードアウトし, その後フェードインします -->
<component is="{{view}}"
  v-transition="fade"
  transition-mode="out-in">
</component>
```

## リストとコンポーネント

オブジェクトの配列に対して、コンポーネントと `v-repeat` を併用することができます。その場合、配列の中にあるそれぞれのオブジェクトに対して、そのオブジェクトを `$data` として、また、指定されたコンポーネントをコンストラクタとして扱う子コンポーネントが生成されます。


``` html
<ul id="list-example">
  <user-profile v-repeat="users"></user-profile>
</ul>
```

``` js
var parent2 = new Vue({
  el: '#list-example',
  data: {
    users: [
      {
        name: 'Chuck Norris',
        email: 'chuck@norris.com'
      },
      {
        name: 'Bruce Lee',
        email: 'bruce@lee.com'
      }
    ]
  },
  components: {
    'user-profile': {
      template: '<li>{{name}}  {{email}}</li>'
    }
  }
})
```

**結果:**

<ul id="list-example" class="demo"><user-profile v-repeat="users"></user-profile></ul>
<script>
var parent2 = new Vue({
  el: '#list-example',
  data: {
    users: [
      {
        name: 'Chuck Norris',
        email: 'chuck@norris.com'
      },
      {
        name: 'Bruce Lee',
        email: 'bruce@lee.com'
      }
    ]
  },
  components: {
    'user-profile': {
      template: '<li>{&#123;name&#125;} - {&#123;email&#125;}</li>'
    }
  }
})
</script>

### エイリアスによるコンポーネントの反復処理

エイリアスシンタックスはコンポーネントを使用しているときも動作し、繰り返されるデータは、キーとしてエイリアスを使用するコンポーネントのプロパティとして設定されます:

``` html
<ul id="list-example">
  <!-- データは `this.user` として内部コンポーネントで利用できます -->
  <user-profile v-repeat="user in users"></user-profile>
</ul>
```


<p class="tip">`v-repeat` で一度コンポーネントを利用すると、同じスコーピングルールは、コンポーネントコンテナ要素上の他のディレクティブに適用されることに注意してください。結果として、親テンプレートの `$index` にアクセスすることはできません。コンポーネントの独自テンプレート内部だけで利用できるようになります。<br><br>別な方法としては、中間スコープを作るために `<template>` ブロックを繰り返し使用することができますが、ほとんどの場合は、コンポーネント内部の `$index` を使用することをお勧めします。</p>

## 子の参照

時々、JavaScript でネストした子コンポーネントへのアクセスが必要になる場合があります。それを実現するためには `v-ref` を用いて子コンポーネントに対して参照 ID を割り当てる必要があります。例えば:

``` html
<div id="parent">
  <user-profile v-ref="profile"></user-profile>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// 子コンポーネントへのアクセス
var child = parent.$.profile
```

`v-ref` が `v-repeat` と共に使用された時は、得られる値はそのデータの配列をミラーリングした子コンポーネントが格納されている配列になります。

## イベントシステム

Vue インスタンスの子や親に直接アクセスすることもできますが、コンポーネント間通信のためのビルトインのイベントシステムを使用した方が便利です。また、この仕組みによってコードの依存性を減らし、メンテナンスし易くなります。一度親子の関係が確立されれば、それぞれのコンポーネントの[イベント](/api/instance-methods.html#イベント)を使ったイベントのディスパッチやトリガが可能になります。

``` js
var parent = new Vue({
  template: '<div><child></child></div>',
  created: function () {
    this.$on('child-created', function (child) {
      console.log('new child created: ')
      console.log(child)
    })
  },
  components: {
    child: {
      created: function () {
        this.$dispatch('child-created', this)
      }
    }
  }
}).$mount()
```

<script>
var parent = new Vue({
  template: '<div><child></child></div>',
  created: function () {
    this.$on('child-created', function (child) {
      console.log('new child created: ')
      console.log(child)
    })
  },
  components: {
    child: {
      created: function () {
        this.$dispatch('child-created', this)
      }
    }
  }
}).$mount()
</script>

## プライベートアセット

時々、ディレクティブ、フィルタ、子コンポーネントなどのアセットをコンポーネントが使う必要がでてきます。しかし、コンポーネント自体を他のところでも再利用できるように、カプセル化されたそれらのアセットを保持したいと思うかもしれません。それはインスタンス化時にプライベートアセットのオプションを使用することによって実現できます。プライベートアセットは所有者であるコンポーネント、それから継承するコンポーネント、そして view 階層 にある子コンポーネントのインスタンスからのみアクセス可能なものになります。

``` js
// 全5種類のアセット
var MyComponent = Vue.extend({
  directives: {
    // id : グローバルメソッドと同じ定義のペア
    'private-directive': function () {
      // ...
    }
  },
  filters: {
    // ...
  },
  components: {
    // ...
  },
  partials: {
    // ...
  },
  effects: {
    // ...
  }
})
```

<p class="tip">`Vue.config.strict = true` を設定することによって、子コンポーネントが親コンポーネントのプライベートアセットへのアクセスするのを禁止できます。</p>

別の方法として、グローバルなアセットの登録メソッドと類似したチェーンする API を使用して、プライベートアセットを既存のコンポーネントのコンストラクタに追加することもできます:

``` js
MyComponent
  .directive('...', {})
  .filter('...', function () {})
  .component('...', {})
  // ...
```

### アセットの命名規則

コンポーネントやディレクティブのようなあるアセットは、HTML 属性または HTML カスタムタグの形でテンプレートに表示されます。HTML 属性名とタグ名は**大文字と小文字を区別しない**ため、私達はしばしばキャメルケースの代わりにダッシュケースを使用して私達のアセットに名前をつける必要があります。**0.12.9** 以降では、キャメルケースを使用してアセットに名前をつけるのをサポートし、テンプレートでダッシュケースでそれらを使用します。

**例**

``` js
// コンポーネント定義
components: {
  // キャメルケースを使用して登録
  myComponent: { /*... */ }
}
```

``` html
<!-- テンプレートではダッシュケースを使用 -->
<my-component></my-component>
```

これは [ES6 object literal shorthand](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_6) でうまく動作します: 

``` js
import compA from './components/a';
import compB from './components/b';

export default {
  components: {
    // <comp-a> そして <comp-b> としてテンプレートで使用
    compA,
    compB
  }
}
```

## コンテンツ挿入

再利用可能なコンポーネントを作るときに、コンポーネントの一部ではないホストしている要素 (Angular の "transclusion" の概念に類似したものです。) の中にある元のコンテンツへのアクセスや再利用がしばしば必要です。Vue.js は現在の Web Components の仕様ドラフトと互換性のあるコンテンツ挿入の仕組みを実装しています。元のコンテンツに対する挿入ポイントとして機能する特別な `<content>` 要素を使用します。

<p class="tip">**重要**: "transcluded" されたコンテンツは子のスコープではなく、親コンポーネントのスコープの中でコンパイルされます。</p>

### 単独の挿入位置

何も属性の無い一つの `<content>` タグしか存在しない時は、元のコンテンツ全体が DOM の中のその位置に挿入され、置換します。元々の `<content>` タグの内側のものは全て**フォールバックコンテンツ (fallback content) **として解釈されます。フォールバックコンテンツはホストしている要素が空で挿入されるべきコンテンツがない時にだけ表示されます。

`my-component` のテンプレート:

``` html
<h1>This is my component!</h1>
<content>This will only be displayed if no content is inserted</content>
```

このコンポーネントを使用した親のマークアップ:

``` html
<my-component>
  <p>This is some original content</p>
  <p>This is some more original content</p>
</my-component>
```

レンダリング結果:

``` html
<my-component>
  <h1>This is my component!</h1>
  <p>This is some original content</p>
  <p>This is some more original content</p>
</my-component>
```

### 多数の挿入位置

`<content>` 要素は CSS セレクタを期待する `select` という特殊な属性を持ちます。異なる `select` 属性を用いて複数の  `<content>` の挿入位置を指定することができます。それぞれは元のコンテンツの中でそのセレクタにマッチした要素によって置換されます。

<p class="tip">0.11.6 以降では、`<content>` セレクタは、ホストノードのトップレベルの子だけ一致できます。これは Shadow DOM 仕様の振舞いを保ち、そしてネストされたテンプレートで誤って不要なノードを選択することを回避します。 </p>

例として、以下のテンプレートのような、多数のコンポーネント挿入のテンプレートを持っていると仮定:

``` html
<content select="p:nth-child(3)"></content>
<content select="p:nth-child(2)"></content>
<content select="p:nth-child(1)"></content>
```

親のマークアップ:

``` html
<multi-insertion">
  <p>One</p>
  <p>Two</p>
  <p>Three</p>
</multi-insertion>
```

レンダリングされる結果:

``` html
<multi-insertion>
  <p>Three</p>
  <p>Two</p>
  <p>One</p>
</multi-insertion>
```

コンテンツ挿入の仕組みは、元のコンテンツがどのように組み替えられ、表示されるべきか、という点に関して素晴らしい管理機能を提供します。これによってコンポーネントが非常に柔軟性と再利用性が高いものになります。

## インラインテンプレート

0.11.6 では、コンポーネント向けに特別なパラメータ属性として、`inline-template` というパラメータが導入されます。これのパラメータが提供されるとき、コンポーネントはそれはテンプレートではなくテンプレートコンテンツとして内部コンテンツを使用します。これは、より柔軟なテンプレートオーサリングを可能にします。

``` html
<my-component inline-template>
  <p>These are compiled as the component's own template</p>
  <p>Not parent's transclusion content.</p>
</my-component>
```

## 非同期コンポーネント

<p class="tip">非同期コンポーネントは 0.12.0 以降のみサポートされます。</p>

大規模アプリケーションでは、実際に必要になったとき、サーバからコンポーネントをロードするだけの、アプリケーションを小さい塊に分割する必要があるかもしれません。それを簡単にするために、Vue.js はコンポーネント定義を非同期的に解決するファクトリ関数としてあなたのコンポーネントを定義することができます。Vue.js はコンポーネントが実際に描画が必要になったときファクトリ関数のみトリガし、そして将来の再描画のために結果をキャッシュします。例えば:

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

ファクトリ関数は `resolve` コールバックを受け取り、その引数はサーバからあなたのコンポーネント定義を取り戻すときに呼ばれるべきです。ロードが失敗したことを示すために、`reject(reason)` も呼び出すことができます。ここでは `setTimeout` はデモとしてシンプルです。どうやってコンポーネントを取得するかどうかは完全にあなた次第です。1つ推奨されるアプローチは [Webpack のコード分割機能](http://webpack.github.io/docs/code-splitting.html)で非同期コンポーネントを使うことです。

``` js
Vue.component('async-webpack-example', function (resolve) {
  // この特別な require シンタックスは、
  // 自動的に ajax リクエストでロードされているバンドルで、
  // あなたのビルドコードを自動的に分割するために
  // webpack で指示しています。
  require(['./my-async-component'], resolve)
})
```

次: [トランジション](/guide/transitions.html)
