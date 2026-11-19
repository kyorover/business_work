# TypeScriptガイドライン

分類: コーディング

### 変数を持たせる場合は、意味を持てるものを用意する

例えばこの `nowDate` が、2箇所以上発生する場合では、変数化することで同じ概念を扱うことを明記します

```tsx
const nowDate = New Date();

if (nowDate < hoge) {
  console.log(nowDate);
}
```

### propsは原則、分割代入して使用しない

```tsx
// ❌
const {hogera, fugara} = props.hoge;

return <Piyo hogera={hogera} fugara={fugara} />

// ------------

// ✅　　
return <Piyo {...props.hoge} />
```

分割代入だと

- シンプル
- デフォルト値が使える

展開しない形だと

- propsということがわかりやすい
- スプレッド構文で子のReact要素に渡しやすい場合がある

と、特徴がそれぞれあるが、**メダップではpropsは原則、分割代入して使用しない**ことにします。

参考

[https://twitter.com/takepepe/status/1258025262949691392?s=20](https://twitter.com/takepepe/status/1258025262949691392?s=20)

ただし例外として **初期値を設定する必要がある場合だけ許容** とします。

```tsx
const obj = { bar: 2 };
const { bar, foo: foo = 'foo' } = obj;
```

注意: この項目は配列(タプル)の分割代入についてではなく、オブジェクトの分割代入についての内容です。

### useFormWrapperは、分割代入しない

```jsx
// ❌
const { register, watch, errors } = useFormWrapper<HogeMap>(...);

// ✅
const form = useFormWrapper<HogeMap>(...);
```

※「propsは原則、分割代入して使用しない」と同じですが念のため記入

### () ⇒ void ではなく、VoidFunction で設定する

型で `() => void`となる時は `VoidFunciton` を使いましょう。

（※ `(v: hoge) => void`となる場合はそのままでOK）

### form 関係の props は、form 属性にまとめる

```jsx
// ❌
return (
  <LoginFormPresenter
    {...props}
    login={{ ...form.register('login'), errorMessages: [] }}
    password={{ ...form.register('password'), errorMessages: [] }}
    isDisabled={checkIsSubmitDisabled(inputMap)}
    onSubmit={form.handleSubmit(onSubmit)}
  />
);

// ✅
return (
  <LoginFormPresenter
    {...props}
    form={{
      login: { ...form.register('login'), errorMessages: [] },
      password: { ...form.register('password'), errorMessages: [] },
      isDisabled: checkIsSubmitDisabled(inputMap),
      onSubmit: form.handleSubmit(onSubmit),
    }}
  />
);
```

### ビュー上で「データが無い」を表す “-” の記述方法について

データが無い場合をブラウザ上で表示させる際は、原則として `-` を表示させます。

その際は、src/utils/emptyText を使用してください。

```tsx
// ❌
<div>名前: -</div>

// ✅
<div>名前: {emptyText()}</div>
```