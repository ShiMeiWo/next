---
title: API
layout: detail
class: apidoc
---


## コア API

### riot.mount

`riot.mount(selector: string, props?: object, componentName?: string): [RiotComponent]`

1. `selector` はページから要素を選択し、それらをカスタムコンポーネントとともにマウントします。選択したエレメントの名前は、カスタムタグの名前と一致する必要があります。is属性を持つDOMノードも自動マウントできます。

2. 消費するために `props` というオプショナルなオブジェクトがコンポーネントに渡されます。これには、単純なオブジェクトから完全なアプリケーション API まで、あらゆるものを使用できます。もしくは Flux ストアの場合もあります。実際には、クライアント側のアプリケーションをどのように構成するかによって異なります。*注意* タグに設定した属性は、`props` 引数で同じ名前を指定した属性よりも優先されます。

3. `componentName` は、マウントするノードが riot によって自動マウントされない場合のオプショナルなコンポーネントの名前です。

<strong>@returns: </strong>マウントされた [コンポーネントオブジェクト](#コンポーネントオブジェクト) の配列


例:

``` js
// 選択し、そしてページ上の全ての <pricing> タグをマウント
const components = riot.mount('pricing')

// クラス名が .customer である全てのタグをマウント
const components = riot.mount('.customer')

// <account> タグをマウントし、オプションとして API オブジェクトを渡す
const components = riot.mount('account', api)

// 既に登録された `app` コンポーネントを使用して <div id="root"> タグに API オブジェクトを渡す
const components = riot.mount('#root', api, 'app')
```

メモ: [インブラウザコンパイル](/compiler/#インブラウザコンパイル) を利用しているユーザーは、`riot.mount` メソッドをコールする前にコンポーネントのコンパイルを待つ必要があるでしょう。

```javascript
(async function main() {
  await riot.compile()

  const components = riot.mount('user')
}())
```

`props` 引数には、複数のタグインスタンス間で同じオブジェクトを共有することを避けるために関数を指定することも可能です。[riot/2613](https://github.com/riot/riot/issues/2613)

``` js
riot.mount('my-component', () => ({
  custom: 'option'
}))
```
### riot.unmount

`riot.unmount(selector: string): [HTMLElement]`

1. `selector` はページから要素を選択肢し、それらが既にマウントされていた場合はアンマウントします。

```js
// 全ての <user> タグを選択し、それらをアンマウントする
riot.unmount('user')
```

<strong>@returns: </strong>マウントされた [コンポーネントオブジェクト](#コンポーネントオブジェクト) の配列

### riot.component

`riot.component(component: RiotComponentShell): function`

1. `component` - [コンポーネントシェルオブジェクト](#コンポーネントシェルインターフェース)

<strong>@returns: </strong>[コンポーネントオブジェクト](#component-object) 生成のための関数

`riot.component` メソッドを使用すると、コンポーネントをグローバルに登録せずにコンポーネントを生成、およびマウントできます:

```js
import * from riot from 'riot'
import App from './app.riot'

const createApp = riot.component(App)

const app = createApp(document.getElementById('root'), {
  name: 'This is a custom property'
})
```

### riot.install

`riot.install(plugin: function): Set`

1. `plugin` - 生成された任意のコンポーネントの [コンポーネントオブジェクト](#コンポーネントオブジェクト) を受け取る関数

<strong>@returns: </strong> インストールされた全てのプラグイン関数を含む javascript `Set`

一度インストールされたプラグイン関数 は生成された任意の Riot.js コンポーネントに対しコールされます

```js
import { install } from 'riot'

let id = 0

// このプラグインは生成された任意の riot コンポーネントに uid 属性を追加します
install(function(component) {
  component.uid = id++

  return component
})
```

### riot.uninstall

`riot.uninstall(plugin: function): Set`

1. `plugin` - 既にインストールされた関数プラグイン

<strong>@returns: </strong> インストールされた残りのプラグイン関数を含む javascript `Set`

プラグインはインストールおよびアンインストールできます:

```js
import { install, uninstall } from 'riot'
import uid from './riot-uid.js'

install(uid)

// プラグインがもう不要となった場合アンインストールする
uninstall(uid)
```

### riot.register

`riot.register(name: string, component: RiotComponentShell): Map`

1. `name` - コンポーネント名
2. `component` - [コンポーネントシェルオブジェクト](#component-shell-interface)

<strong>@returns: </strong> 全ての登録済みコンポーネントのファクトリ関数を含む javascript `Map`

```js
import { register, mount } from 'riot'
import MyComponent from './my-component.riot'

// my-component をグローバルコンポーネントとして登録
register('my-component', MyComponent)

// `<my-component>` を呼ばれるすべての DOM ノードを検索し
// 既に登録したコンポーネントと一緒にマウントします
mount('my-component')
```

### riot.unregister

`riot.unregister(name: string): Map`

1. `name` - the component name

<strong>@returns: </strong> アンマウント されていない残りのコンポーネントから生成された関数を含む javascript の `Map`

既にコンパイラか `riot.register()` を介して生成されたタグの登録を解除します。
このメソッドは、例えばアプリケーションをテストする必要があり、かつ同じ名前を使用して複数のタグを生成したい時などに有効かもしれません。

```js
import { register, unregister } from 'riot'
import TestComponent from './test-component.riot'
import TestComponent2 from './test-component2.riot'

// test コンポーネントを生成
register('test-component', TestComponent)

// マウント
const [component] = mount(document.createElement('div'), 'test-component')
expect(component.root.querySelector('p')).to.be.ok

// tag を登録解除
unregister('test-component')

// 異なるテンプレートを利用して同じコンポーネントを再生性
register('test-component', TestComponent2)
```

### riot.version

`riot.version(): string`

<strong>@returns: </strong> 現在使用しているの riot のバージョンを文字列として返す

## コンポーネントオブジェクト

各 Riot.js コンポーネントは軽量なオブジェクトとして生成されます。`export default` を使用してエクスポートするオブジェクトには以下のプロパティがあります:

- Attributes
  - `props` - オブジェクトとして受けとる props
  - `state` - 現在のステートオブジェクト
  - `root` - ルート DOM ノード
- [生成と破壊](#生成と破壊)
  - `mount` - コンポーネントの初期化
  - `unmount` - DOM から コンポーネントを壊し、取り除く
- [ステートハンドリング](#ステートハンドリング) メソッド
  - `update` - コンポーネントのステートを更新するメソッド
  - `shouldUpdate` - コンポーネントレンダリングを一時停止するメソッド
- [ライフサイクルコールバック](#ライフサイクル)
  - `onBeforeMount` - コンポーネントがマウントされる前にコールされる
  - `onMounted` - コンポーネントのレンダリングが完了した後にコールされる
  - `onBeforeUpdate` - コンポーネントの更新前にコールされる
  - `onUpdated` - コンポーネントの更新が完了した後にコールされる
  - `onBeforeUnmount` - コンポーネントが削除される前にコールされる
  - `onUnmounted` - コンポーネントの削除が完了した後にコールされる
- [ヘルパー](#ヘルパー)
  - `$` - `document.querySelector` に似たメソッド
  - `$$` - `document.querySelectorAll` に似たメソッド


### コンポーネントインターフェース

TypeScript に詳しい人は、ここに書かれているように Riot.js コンポーネントインタフェースがどのように見えるを読むことができます:

```ts
// このインターフェースはただ後悔されているのみで、どんな Riot コンポーネントも以下のプロパティを受け取ります
interface RiotCoreComponent {
  // 任意のコンポーネントインスタンスで自動的に生成
  props: object;
  root: HTMLElement;
  mount(
    element: HTMLElement,
    initialState?: object,
    parentScope?: object
  ): RiotComponent;
  update(
    newState?:object,
    parentScope?: object
  ): RiotComponent;
  unmount(keepRootElement: boolean): RiotComponent;

  // ヘルパー
  $(selector: string): HTMLElement;
  $$(selector: string): [HTMLElement];
}

// パブリックな RiotComponent インターフェースのプロパティはすべてオプショナルです
interface RiotComponent extends RiotCoreComponent {
  // コンポーネントオブジェクトでオプショナル
  state?: object;

  // 子コンポーネントの名前をマップするオプションの別名
  components?: object;

  // ステートハンドリングメソッド
  shouldUpdate?(newProps:object, currentProps:object): boolean;

  // ライフサイクルメソッド
  onBeforeMount?(currentProps:object, currentState:object): void;
  onMounted?(currentProps:object, currentState:object): void;
  onBeforeUpdate?(currentProps:object, currentState:object): void;
  onUpdated?(currentProps:object, currentState:object): void;
  onBeforeUnmount?(currentProps:object, currentState:object): void;
  onUnmounted?(currentProps:object, currentState:object): void;
}
```

HTML と JavaScript コードの両方で任意のコンポーネントプロパティを使用できます。例:


``` html
<my-component>
  <h3>{ props.title }</h3>

  <script>
    export default {
      onBeforeMount() {
        const {title} = this.props
      }
    }
  </script>
</my-component>
```

コンポーネントスコープには任意のプロパティを自由に設定でき、HTML に式で使用できます。例:

``` html
<my-component>
  <h3>{ title }</h3>

  <script>
    export default {
      onBeforeMount() {
        this.title = this.props.title
      }
    }
  </script>
</my-component>
```

メモ: いくつかのグローバル変数がある場合、HTML と JavaScript コードの両方でこれらの参照を使用することもできます:

```js
window.someGlobalVariable = 'Hello!'
```

```html
<my-component>
  <h3>{ window.someGlobalVariable }</h3>

  <script>
    export default {
      message: window.someGlobalVariable
    }
  </script>
</my-component>
```

<aside class="note note--warning">:warning: コンポーネントでグローバル変数を使用すると、おそらくサーバサイドレンダリングが損なわれる可能性があるため注意してください。そして、推奨しません。</aside>


### 生成と破壊

#### component.mount

`component.mount(element: HTMLElement, initialState?: object, parentScope?: object): RiotComponent;`

テンプレートをインタラクティブにレンダリングするために、任意のコンポーネントオブジェクトが DOM ノードにマウントされます。

`component.mount` メソッドを自分で呼び出すことはまずありませんが、代わりに [riot.mount](#riotmount) または [riot.component](#riotcomponent) を使用します。

#### component.unmount

`component.mount(keepRoot?: boolean): RiotComponent`

カスタムコンポーネントとその子をページから切り離します。
ルートノードを削除せずにタグをアンマウントしたい場合は、unmountメソッドに `true` を渡す必要があります。

タグをアンマウントし、DOM からテンプレートを削除します:

``` js
myComponent.unmount()
```

ルートノードを DOM に保持したままコンポーネントをアンマウントします:

``` js
myComponent.unmount(true)
```


### ステートハンドリング

#### component.state

Any Riot.js component created has a `state` object property. The `state` object is meant to store all the mutable component properties. For example:

```html
<my-component>
  <button>{ state.message }</button>

  <script>
    export default {
      // initial component state
      state: {
        message: 'hi'
      }
    }
  </script>
</my-component>
```

In this case the component is created with an initial state that can be modified internally via `component.update`.

You should avoid to store nested javascript objects into the state property because their references will be shared across multiple component and might generate side effects. To avoid undesired surprises you can create your components also using a factory function

```html
<my-component>
  <button>{ state.message }</button>

  <script>
    export default function MyComponent() {
      // remember to return an object
      return {
        // the initial state will be always fresh created avoiding surprises
        state: {
          nested: {
            properties: 'are ok now'
          },
          message: 'hi'
        }
      }
    }
  </script>
</my-component>
```

Riot.js will automatically call the function anytime a new component will be mounted.

#### component.components

If you want to avoid registering global Riot.js components you can map your children components directly on your component object. For example:

```html
<my-component>
  <!-- this component is only available in `<my-component>` -->
  <my-child/>

  <!-- this component is named differently and was aliased -->
  <aliased-name/>

  <!-- this component was already registered via riot.register -->
  <global-component/>

  <script>
    import MyChild from './my-child.riot'
    import User from './user.riot'

    export default {
      components: {
        MyChild,
        'aliased-name': User
      }
    }
  </script>
</my-component>
```

<div class="note note--info">
  For this example we assume that you are bundling your application via webpack, rollup, parcel or browserify
</div>

#### component.update

`component.update(newState?:object, parentScope?: object): RiotComponent;`

Updates the component `state` object re-rendering all its expressions. This method can be usually called every time an event handler is dispatched when the user interacts with the application.

``` html
<my-component>
  <button onclick={ onClick }>{ state.message }</button>

  <script>
    export default {
      state: {
        message: 'hi'
      },
      onClick(e) {
        this.update({
          message: 'goodbye'
        })
      }
    }
  </script>
</my-component>
```

You can call this method also manually whenever you need to update your components UI. This typically happens after some non-UI related event: after `setTimeout`, AJAX call or on some server event. For example:

``` html
<my-component>

  <input name="username" onblur={ validate }>
  <span class="tooltip" if={ state.error }>{ state.error }</span>

  <script>
    export default {
      async validate() {
        try {
          const {username} = this.props
          const response = await fetch(`/validate/username/${username}`)
          const json = response.json()
          // do something with the response
        } catch (error) {
          this.update({
            error: error.message
          })
        }
      }
    }
  </script>
</my-component>
```

On above example the error message is displayed on the UI after the `update()` method has been called.

If you want to have more control over your tags DOM updates you can set rely on the `shouldUpdate` function return.
Riot.js will update your component only if that function will return `true`

``` html
<my-component>
  <button onclick={ onClick }>{ state.message }</button>

  <script>
    export default {
      state: {
        message: 'hi'
      },
      onClick(e) {
        this.update({
          message: 'goodbye'
        })
      },
      shouldUpdate(newProps, currentProps) {
        // do not update
        if (this.state.message === 'goodbye') return false
        // if this.state.message is different from 'goodbye' we could update the component
        return true
      }
    }


  </script>
</my-component>
```

The `shouldUpdate` method will always receive 2 arguments: the first one contains the new component properties and the second argument the current ones.

``` html
<my-component>
  <child-tag message={ state.message }></child-tag>
  <button onclick={ onClick }>Say goodbye</button>

  <script>
    export default {
      state: {
        message = 'hi'
      },
      onClick(e) {
        this.update({
          message: 'goodbye'
        })
      }
    }
  </script>
</my-component>

<child-tag>
  <p>{ props.message }</p>

  <script>
    export default {
      shouldUpdate(newProps, currentProps) {
        // update the DOM depending on the new properties received
        return newProps.message !== 'goodbye'
      }
    }
  </script>
</child-tag>
```



###  スロット

The `<slot>` tag is a special Riot.js core feature that allows you to inject and compile the content of any custom component inside its template in runtime.
For example using the following riot tag `my-post`

``` html
<my-post>
  <h1>{ props.title }</h1>
  <p><slot/></p>
</my-post>
```

anytime you will include the `<my-post>` tag in your app

``` html
<my-post title="What a great title">
  My beautiful post is <b>just awesome</b>
</my-post>
```

once mounted it will be rendered in this way:

``` html
<my-post>
  <h1>What a great title</h1>
  <p>My beautiful post is <b>just awesome</b></p>
</my-post>
```

The expressions in slot tags will not have access to the properties of the components in which they are injected

``` html
<!-- This tag just inherits the yielded DOM -->
<child-tag>
  <slot/>
</child-tag>

<my-component>
  <child-tag>
    <!-- here the child-tag internal properties are not available -->
    <p>{ message }</p>
  </child-tag>
  <script>
    export default {
      message: 'hi'
    }
  </script>
</my-component>
```

#### Named Slots

The `<slot>` tag provides also a mechanism to inject html in specific sections of a component template

For example using the following riot tag `my-other-post`

``` html
<my-other-post>
  <article>
    <h1>{ opts.title }</h1>
    <h2><slot name="summary"/></h2>
    <article>
      <slot name="content"/>
    </article>
  </article>
</my-other-post>
```

anytime you will include the `<my-other-post>` tag in your app

``` html
<my-other-post title="What a great title">
  <span slot="summary">
    My beautiful post is just awesome
  </span>
  <p slot="content">
    And the next paragraph describes just how awesome it is
  </p>
</my-other-post>
```

once mounted it will be rendered in this way:

``` html
<my-other-post>
  <article>
    <h1>What a great title</h1>
    <h2><span>My beautiful post is just awesome</span></h2>
    <article>
      <p>
        And the next paragraph describes just how awesome it is
      </p>
    </article>
  </article>
</my-other-post>
```

### ライフサイクル

Each component object can rely on the following callbacks to handle its internal state:

  - `onBeforeMount` - called before the component will be mounted
  - `onMounted` - called after the component has rendered
  - `onBeforeUpdate` - called before the component will be updated
  - `onUpdated` - called after the component has been updated
  - `onBeforeUnmount` - called before the component will be removed
  - `onUnmounted` - called once the component has been removed

For example:

```html
<my-component>
  <p>{ state.message }</p>

  <script>
    export default {
      onBeforeMount() {
        this.state = {
          message: 'Hello there'
        }
      }
    }
  </script>
</my-component>
```

All the lifecycle methods will receive 2 arguments `props` and `state`, they are aliases of the `this.props` and `this.state` component attributes.

```html
<my-component>
  <p>{ state.message }</p>

  <script>
    export default {
      state: {
        message: 'Hello there'
      },
      onBeforeMount(props, state) {
        console.assert(this.state === state) // ok!
        console.log(state.message) // Hello there
      }
    }
  </script>
</my-component>
```

### ヘルパー

Any Riot.js component provides two helpers to query DOM nodes contained in its rendered template.

 - `component.$(selector: string): HTMLElement` - returns a single node located in the component markup
 - `component.$$(selector: string): [HTMLElemet]` - returns all the DOM nodes matched by the selector containing the component markup

You can use the component helpers for doing simple DOM queries:

```html
<my-component>
  <ul>
    <li each={ item in items }>
      { item }
    </li>
  </ul>

  <script>
    export default {
      onMounted() {
        // queries
        const ul = this.$('ul')
        const lis = this.$$('li')

        // do something with the DOM nodes
        const lisWidths = lis.map(li => li.offsetWidth)
        const {top, left} = ul.getBoundingClientRect()
      }
    }
  </script>
</my-component>
```

### 手動でのタグ構築

Riot.js components are meant to be compiled to javascript via [@riotjs/compiler](/compiler). However you can build them manually with any rendering engine you like.

#### Component shell interface

The Riot.js compiler just creates a shell object that will be transformed internally by riot to create the [component object](#component-interface). If want to build this shell object manually it's worth to understand its interface first:

```ts
interface RiotComponentShell {
  css?: string;
  exports?: object|function|class;
  name?: string;
  template(): RiotComponentTemplate;
}
```

The `RiotComponentShell` object consists of 4 properties:

- `css` - the component css as string
- `exports` - the component `export default` public API
- `name` - the component name
- `template` - the factory function to manage the component template

#### Template interface

The template function should return an interface compatible to the following one:

```ts
interface RiotComponentTemplate {
  update(scope: object): RiotComponentTemplate;
  mount(element: HTMLElement, scope: object): RiotComponentTemplate;
  createDOM(element: HTMLElement): RiotComponentTemplate;
  unmount(scope: object): RiotComponentTemplate;
  clone(): RiotComponentTemplate;
}
```

The `RiotComponentTemplate` is an object and it's responsible to handle the component rendering:

- `update` - method that will receive the component data and must be used to update the template
- `mount` - method that must be used to connect the component template to a DOM node
- `createDOM` - factory function might be needed to create the template DOM structure only once
- `unmount` - method to clean up the component DOM
- `clone` - method might be needed to clone the original template object in order to work with multiple instances

#### Examples

This example uses the [@riotjs/dom-bindings (riot core template engine)](https://github.com/riot/dom-bindings)

```js
import { template, expressionTypes } from '@riotjs/dom-bindings'

riot.register('my-component', {
  css: ':host { color: red; }',
  name: 'my-component',
  exports: {
    onMounted() {
      console.log('I am mounted')
    }
  },
  template() {
    return template('<p><!----></p>', [{
      selector: 'p',
      expressions: [
        {
          type: expressionTypes.TEXT,
          childNodeIndex: 0,
          evaluate: scope => `Hello ${scope.greeting}`
        }
      ]
    }])
  }
})
```

Read about the [template engine API](https://github.com/riot/dom-bindings)

You can also use any other kind of template engine if you like.
This example uses [lit-html](https://lit-html.polymer-project.org/) as template engine

```js
import {html, render} from 'lit-html'

riot.register('my-component', {
  css: ':host { color: red; }',
  name: 'my-component',
  exports: {
    onMounted() {
      console.log('I am mounted')
    }
  },
  template() {
    const template = ({name}) => html`<p>Hello ${name}!</p>`

    return {
      mount(element, scope) {
        this.el = element

        this.update(scope)
      },
      update(scope) {
        render(template(scope), this.el)
      },
      unmount() {
        this.el.parentNode.removeChild(this.el)
      },
      // the following methods are not necessary for lit-html
      createDOM(element) {},
      clone() {}
    }
  }
})
```

#### Tags without template

You can also create "wrapper tags" without any template as follows:

``` js
riot.register('my-component', {
  name: 'my-component',
  exports: {
    onMounted() {
      console.log('I am mounted')
    }
  }
})

```

In this case anytime you will mount a tag named `my-component` riot will leave the component markup as it is without parsing it:

```html
<html>
<body>
  <my-component>
    <p>I want to say Hello</p>
  </my-component>
</body>
</html>
```

This technique might be used to enhance serverside rendered templates.
