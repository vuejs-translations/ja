# \<script setup> {#script-setup}

`<script setup>` は単一ファイルコンポーネント（SFC）内で Composition API を使用するコンパイル時のシンタックスシュガー（糖衣構文）です。SFC と Composition API の両方を使うならば、おすすめの構文です。これは通常の `<script>` 構文よりも、多くの利点があります:

- ボイラープレートが少なくて、より簡潔なコード
- 純粋な TypeScript を使って props と発行されるイベントを宣言する機能
- 実行時のパフォーマンスの向上（テンプレートは中間プロキシなしに同じスコープ内のレンダー関数にコンパイルされます）
- IDE で型推論のパフォーマンス向上（言語サーバーがコードから型を抽出する作業が減ります）

## 基本の構文 {#basic-syntax}

この構文を導入するには、`setup` 属性を `<script>` ブロックに追加します:

```vue
<script setup>
console.log('hello script setup')
</script>
```

内部のコードは、コンポーネントの `setup()` 関数の内容としてコンパイルされます。これはつまり、通常の `<script>` とは違って、コンポーネントが最初にインポートされたときに一度だけ実行されるのではなく、`<script setup>` 内のコードは **コンポーネントのインスタンスが作成されるたびに実行される** ということです。

### トップレベルのバインディングはテンプレートに公開 {#top-level-bindings-are-exposed-to-template}

`<script setup>` を使用する場合、`<script setup>` 内で宣言されたトップレベルのバインディング（変数、関数宣言、インポートを含む）は、テンプレートで直接使用できます:

```vue
<script setup>
// 変数
const msg = 'Hello!'

// 関数
function log() {
  console.log(msg)
}
</script>

<template>
  <button @click="log">{{ msg }}</button>
</template>
```

インポートも同じように公開されます。これはつまり、インポートされたヘルパー関数を `methods` オプションで公開することなくテンプレート内の式で直接使用できます:

```vue
<script setup>
import { capitalize } from './helpers'
</script>

<template>
  <div>{{ capitalize('hello') }}</div>
</template>
```

## リアクティビティー {#reactivity}

リアクティブな状態は [リアクティビティー API](./reactivity-core) を使って明示的に作成する必要があります。`setup()` 関数から返された値と同じように、テンプレート内で参照されるときに ref は自動的にアンラップされます:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

## コンポーネントの使用 {#using-components}

`<script setup>` のスコープ内の値は、カスタムコンポーネントのタグ名としても直接使用できます:

```vue
<script setup>
import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```

`MyComponent` を変数として参照していると考えてください。JSX を使ったことがあれば、このメンタルモデルは似ています。ケバブケースの `<my-component>` も同じようにテンプレートで動作します。しかし、一貫性を保つために、パスカルケースのコンポーネントタグを強く推奨します。これはネイティブのカスタム要素と区別するのにも役立ちます。

### 動的コンポーネント {#dynamic-components}

コンポーネントは、文字列キーで登録されるのではなく変数として参照されるため、`<script setup>` 内で動的コンポーネントを使う場合は、動的な `:is` バインディングを使う必要があります:

```vue
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```

三項演算子で変数としてコンポーネントをどのように使うことができるかに注意してください。

### 再帰的コンポーネント {#recursive-components}

SFC はそのファイル名を介して、暗黙的に自身を参照できます。例えば、`FooBar.vue` というファイル名は、そのテンプレート内で `<FooBar/>` として自身を参照できます。

これはインポートされたコンポーネントよりも優先度が低いことに注意してください。コンポーネントの推論された名前と競合する名前付きインポートがある場合、インポートでエイリアスを作成できます:

```js
import { FooBar as FooBarChild } from './components'
```

### 名前空間付きコンポーネント {#namespaced-components}

`<Foo.Bar>` のようにドット付きのコンポーネントタグを使って、オブジェクトプロパティの下にネストしたコンポーネントを参照できます。これは単一のファイルから複数のコンポーネントをインポートするときに便利です:

```vue
<script setup>
import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

## カスタムディレクティブの使用 {#using-custom-directives}

グローバルに登録されたカスタムディレクティブは通常通りに動作します。`<script setup>` ではローカルのカスタムディレクティブは明示的に登録する必要はありませんが、`vNameOfDirective` という命名規則に従う必要があります:

```vue
<script setup>
const vMyDirective = {
  beforeMount: (el) => {
    // 要素を使って何かする
  }
}
</script>
<template>
  <h1 v-my-directive>This is a Heading</h1>
</template>
```

他の場所からディレクティブをインポートする場合、必須の命名規則に合うようにリネームすることができます:

```vue
<script setup>
import { myDirective as vMyDirective } from './MyDirective.js'
</script>
```

## defineProps() & defineEmits() {#defineprops-defineemits}

完全な型推論のサポートつきで `props` と `emits` のようなオプションを宣言するために、`defineProps` と `defineEmits` の API を使用できます。これらは `<script setup>` の中で自動的に利用できるようになっています:

```vue
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
// セットアップのコード
</script>
```

- `defineProps` と `defineEmits` は、`<script setup>` 内でのみ使用可能な**コンパイラーマクロ**です。インポートする必要はなく、`<script setup>` が処理されるときにコンパイルされます。

- `defineProps` は `props` オプションと同じ値を受け取り、`defineEmits` は `emits` オプションと同じ値を受け取ります。

- `defineProps` と `defineEmits` は、渡されたオプションに基づいて、適切な型の推論を行います。

- `defineProps` および `defineEmits` に渡されたオプションは、setup のスコープからモジュールのスコープに引き上げられます。そのため、オプションは setup のスコープで宣言されたローカル変数を参照できません。参照するとコンパイルエラーになります。しかし、インポートされたバインディングはモジュールのスコープに入っているので、参照できます。

### 型のみの props/emit 宣言<sup class="vt-badge ts" /> {#type-only-props-emit-declarations}

`defineProps` や `defineEmits` にリテラル型の引数を渡すことで、純粋な型の構文を使って props や emits を宣言することもできます:

```ts
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 3.3+: より簡潔な代替構文
const emit = defineEmits<{
  change: [id: number] // 名前付きタプル構文
  update: [value: string]
}>()
```

- `defineProps` または `defineEmits` は、実行時の宣言か型宣言のどちらかしか使用できません。両方を同時に使用すると、コンパイルエラーになります。

- 型宣言を使用する場合は、同等の実行時宣言が静的解析から自動的に生成されるため、二重の宣言を行う必要がなくなり、実行時の正しい動作が保証されます。

  - 開発モードでは、コンパイラーは型から対応する実行時バリデーションを推測しようとします。上記の例では `foo: string` という型からは `foo: String` が推測されます。もし型がインポートされた型への参照である場合、コンパイラーは外部ファイルの情報を持っていないので、推測される結果は `foo: null`（`any` 型と同じ）になります。

  - プロダクションモードでは、バンドルサイズを小さくするために、コンパイラーが配列形式の宣言を生成します（上記の props は `['foo', 'bar']` にコンパイルされます）。

- バージョン 3.2 以下では、`defineProps()` の型引数はローカルの型への参照か、型リテラルのどちらかに制限されていました。

  この制限は 3.3 で解決されました。Vue の最新バージョンは型引数の位置でインポートされた複雑な型の限定されたセットを参照することをサポートしています。しかし、型からランタイムへの変換は依然として AST ベースであるため、実際の型解析を必要とするいくつかの複雑な型、例えば条件型など、はサポートされていません。条件型は単一の props の型には使用できますが、props オブジェクト全体の型には使用できません。

### リアクティブな props の分割代入 <sup class="vt-badge" data-text="3.5+" /> {#reactive-props-destructure}

Vue 3.5 以降、`defineProps` の戻り値から分割代入された変数はリアクティブです。同じ `<script setup>` ブロック内で `defineProps` から分割代入された変数にアクセスするコードがあると、Vue のコンパイラーは自動的に `props.` を先頭に追加します:

```ts
const { foo } = defineProps(['foo'])

watchEffect(() => {
  // 3.5 以前は 1 回だけ実行されます。
  // 3.5 以降は "foo" が変更されるたびに再実行されます。
  console.log(foo)
})
```

上記のコードがコンパイルされると以下のようになります:

```js {5}
const props = defineProps(['foo'])

watchEffect(() => {
  // `foo` はコンパイラーによって `props.foo` に変換されました。
  console.log(props.foo)
})
```

さらに、JavaScript のネイティブなデフォルト値構文を使用して、props のデフォルト値を宣言できます。これは型ベースの props 宣言を使用する場合に特に便利です:

```ts
interface Props {
  msg?: string
  labels?: string[]
}

const { msg = 'hello', labels = ['one', 'two'] } = defineProps<Props>()
```

### 型宣言を使用時のデフォルトの props 値 <sup class="vt-badge ts" /> {#default-props-values-when-using-type-declaration}

3.5 以降では、リアクティブな props の分割代入を使用すると、自然な形でデフォルト値を宣言できます。しかし、3.4 以前では、リアクティブな props の分割代入はデフォルトで有効になっていません。型ベースの宣言で props のデフォルト値を宣言するには、`withDefaults` コンパイラーマクロが必要です:

```ts
interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

これは、同等なランタイム props の `default` オプションにコンパイルされます。さらに、`withDefaults` ヘルパーは、デフォルト値の型チェックを行います。また、返される `props` の型が、デフォルト値が宣言されているプロパティに対して、省略可能フラグが削除されていることを保証します。

:::info
変更可能な参照型（配列やオブジェクトなど）のデフォルト値は、偶発的な変更や外部からの副作用を避けるために `withDefaults` を使う時は、関数でラップする必要があることに注意してください。こうすることで、各コンポーネントのインスタンスがデフォルト値のコピーを取得することが保証されます。これは分割代入でデフォルト値を使う時は**不要**です。
:::

## defineModel() <sup class="vt-badge" data-text="3.4+" /> {#definemodel}

このマクロは親コンポーネントから `v-model` 経由で使用できる双方向バインディングの props を宣言するために使用できます。使用例は、[コンポーネントの `v-model`](/guide/components/v-model) のガイドでも説明されています。

このマクロは内部でモデルの props と、それに対応する値更新イベントを宣言します。第一引数がリテラル文字列の場合、props の名前として使用されます。それ以外の場合、props 名はデフォルトで `"modelValue"` になります。どちらの場合も、props のオプションやモデル ref の値変換オプションを含んだ追加のオブジェクトを渡すこともできます。

```js
// 親から v-model 経由で使用される、"modelValue" props を宣言する
const model = defineModel()
// もしくは: オプション付きで "modelValue" props を宣言する
const model = defineModel({ type: String })

// 変更された時に "update:modelValue" イベントを発行
model.value = 'hello'

// 親から v-model:count 経由で使用される、"count" props を宣言する
const count = defineModel('count')
// もしくは: オプション付きで "count" props を宣言する
const count = defineModel('count', { type: Number, default: 0 })

function inc() {
  // 変更された時に "update:count" イベントを発行
  count.value++
}
```

:::warning
もし `defineModel` props に `default` 値を指定し、親コンポーネントからこの props に何も値を与えなかった場合、親と子のコンポーネント間で同期が取れなくなる可能性があります。以下の例では、親コンポーネントの `myRef` は undefined ですが、子コンポーネントの `model` は 1 です:

```js
// 子コンポーネント:
const model = defineModel({ default: 1 })

// 親コンポーネント:
const myRef = ref()
```

```html
<Child v-model="myRef"></Child>
```

:::

### 修飾子と変換 {#modifiers-and-transformers}

`v-model` ディレクティブで使われる修飾子にアクセスするには、`defineModel()` の戻り値を次のように分割代入します:

```js
const [modelValue, modelModifiers] = defineModel()

// v-model.trim に該当
if (modelModifiers.trim) {
  // ...
}
```

修飾子が存在する場合、親から読み込んだ値や親に同期して返す値を変換する必要があることが多いです。それを実現するには `get` と `set` 変換オプションを使用します:

```js
const [modelValue, modelModifiers] = defineModel({
  // ここでは必要ないので get() は省略されている
  set(value) {
    // .trim 修飾子が使われた場合、トリムした値を返す
    if (modelModifiers.trim) {
      return value.trim()
    }
    // それ以外は値をそのまま返す
    return value
  }
})
```

### TypeScript での使用 <sup class="vt-badge ts" /> {#usage-with-typescript}

`defineProps` や `defineEmits` と同様に、`defineModel` も型引数を受け取ることができ、モデルの値や修飾子の型を指定できます:

```ts
const modelValue = defineModel<string>()
//    ^? Ref<string | undefined>

// オプション付きのデフォルトモデル。required は undefined になりうる値を除去する
const modelValue = defineModel<string>({ required: true })
//    ^? Ref<string>

const [modelValue, modifiers] = defineModel<string, 'trim' | 'uppercase'>()
//                 ^? Record<'trim' | 'uppercase', true | undefined>
```

## defineExpose() {#defineexpose}

`<script setup>` を使用したコンポーネントは、**デフォルトで閉じられています**。つまり、テンプレート参照や `$parent` チェーンを介して取得されるコンポーネントのパブリックインスタンスは、`<script setup>` 内で宣言されたバインディングを公開**しません**。

`<script setup>` コンポーネントのプロパティを明示的に公開するには、`defineExpose` コンパイラーマクロを使用します:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

親がテンプレート参照を介してこのコンポーネントのインスタンスを取得すると、取得されたインスタンスは `{ a: number, b: number }` という形状になります（ref は通常のインスタンスと同様、自動的にアンラップされます）。

## defineOptions() <sup class="vt-badge" data-text="3.3+" /> {#defineoptions}

このマクロは、`<script>` ブロックを別途使用することなく、`<script setup>` 内で直接コンポーネントオプションを宣言するために使用できます:

```vue
<script setup>
defineOptions({
  inheritAttrs: false,
  customOptions: {
    /* ... */
  }
})
</script>
```

- 3.3 以上でのみサポートされています。
- これはマクロです。オプションはモジュールスコープに巻き上げられ、`<script setup>` 内のリテラル定数でないローカル変数にはアクセスできません。

## defineSlots() <sup class="vt-badge ts"/> {#defineslots}

このマクロは、スロット名と props の型チェックのために IDE に型ヒントを提供するために使用することができます。

`defineSlots()` は型パラメーターのみを受け取り、実行時引数はありません。型パラメーターは、プロパティキーがスロット名で、値の型がスロット関数である型リテラルでなければなりません。関数の最初の引数はスロットが受け取ることを期待する props で、その型はテンプレート内のスロット props に使用されることになります。戻り値の型は現在無視されており、`any` を指定することができますが、将来的にはスロットの内容チェックのために活用するかもしれません。

また、`slots` オブジェクトを返します。これはセットアップコンテキストで公開されている `slots` オブジェクト、または `useSlots()` が返す `slots` オブジェクトと同じものです。

```vue
<script setup lang="ts">
const slots = defineSlots<{
  default(props: { msg: string }): any
}>()
</script>
```

- 3.3 以上でのみサポートされています。

## `useSlots()` & `useAttrs()` {#useslots-useattrs}

`<script setup>` 内で `slots` や `attrs` を使用することは比較的少ないはずです。なぜなら、テンプレート内で `$slots` と `$attrs` として直接アクセスできるからです。万が一、必要になった場合には、それぞれ `useSlots` と `useAttrs` ヘルパーを使用してください:

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```

`useSlots` と `useAttrs` は、`setupContext.slots` と `setupContext.attrs` と同等のものを返す実際のランタイム関数です。これらは通常の Composition API の関数内でも使用できます。

## 通常の `<script>` との併用 {#usage-alongside-normal-script}

`<script setup>` は、通常の `<script>` と一緒に使うことができます。次のことが必要な場合は、通常の `<script>` が必要になることがあります:

- `inheritAttrs` や、プラグインで有効になるカスタムオプションなど、`<script setup>` では表現できないオプションを宣言する（3.3+ では [`defineOptions`](/api/sfc-script-setup#defineoptions) で置き換え可能）
- 名前付きのエクスポートを宣言する
- 副作用を実行したり、一度しか実行してはいけないオブジェクトを作成する

```vue
<script>
// 通常の <script>、モジュールのスコープで実行される（1 回だけ）
runSideEffectOnce()

// 追加のオプションを宣言
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// setup() のスコープで実行される（インスタンスごとに）
</script>
```

同じコンポーネント内で `<script setup>` と `<script>` を組み合わせることは、上記のシナリオに限定してサポートします。具体的には:

- `props` や `emits` のような `<script setup>` で定義できるオプションは、`<script>` セクションで定義**しない**でください。
- `<script setup>` 内で作成された変数は、コンポーネントインスタンスのプロパティとして追加されないので、Options API からはアクセスできません。このように API を混在させることは、強くお勧めしません。

もし、サポートされていないシナリオに遭遇した場合は、`<script setup>` の代わりに、明示的な [`setup()`](/api/composition-api-setup) 関数に切り替えることを検討する必要があります。

## トップレベルの `await` {#top-level-await}

`<script setup>` の中ではトップレベルの `await` を使うことができます。その結果、コードは `async setup()` としてコンパイルされます:

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

さらに、await の対象の式は、`await` 後の現在のコンポーネントのインスタンスのコンテキストを保持する形式で自動的にコンパイルされます。

:::warning Note
`async setup()` は、現在まだ実験的な機能である [`Suspense`](/guide/built-ins/suspense.html) と組み合わせて使用する必要があります。将来のリリースで完成させてドキュメント化する予定ですが、もし今興味があるのであれば、その[テスト](https://github.com/vuejs/core/blob/main/packages/runtime-core/__tests__/components/Suspense.spec.ts)を参照することで、どのように動作するかを確認できます。
:::

### ジェネリクス <sup class="vt-badge ts" /> {#generics}

`<script>` タグの `generic` 属性を使ってジェネリック型パラメーターを宣言できます:

```vue
<script setup lang="ts" generic="T">
defineProps<{
  items: T[]
  selected: T
}>()
</script>
```

`generic` の値は、TypeScript の `<...>` 間のパラメーターリストと全く同じ働きをします。例えば、複数のパラメーター、`extends` 制約、デフォルトの型、インポートされた型を参照できます:

```vue
<script
  setup
  lang="ts"
  generic="T extends string | number, U extends Item"
>
import type { Item } from './types'
defineProps<{
  id: T
  list: U[]
}>()
</script>
```

`ref` でジェネリックコンポーネントへの参照を使用する場合、`InstanceType` は動作しないので、[`vue-component-type-helpers`](https://www.npmjs.com/package/vue-component-type-helpers) ライブラリーを使用する必要があります。

```vue
<script
  setup
  lang="ts"
>
import componentWithoutGenerics from '../component-without-generics.vue';
import genericComponent from '../generic-component.vue';

import type { ComponentExposed } from 'vue-component-type-helpers';

// ジェネリクスのないコンポーネントでは動作します
ref<InstanceType<typeof componentWithoutGenerics>>();

ref<ComponentExposed<typeof genericComponent>>();
```

## 制限 {#restrictions}

- モジュールの実行セマンティクスの違いにより、`<script setup>` 内のコードは、SFC のコンテキストに依存しています。外部の `.js` や `.ts` ファイルに移動すると、開発者とツールの両方に混乱を招く可能性があります。そのため、**`<script setup>`** は、`src` 属性と一緒に使うことはできません。
- `<script setup>` は、DOM 内のルートコンポーネントテンプレートをサポートしていません（[関連する議論](https://github.com/vuejs/core/issues/8391)）。
