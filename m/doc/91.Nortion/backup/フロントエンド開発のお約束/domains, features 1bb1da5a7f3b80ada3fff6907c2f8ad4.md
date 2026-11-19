# domains, features

分類: ディレクトリ構成

### ディレクトリ構成

```markdown

├── domains(features)
│   ├── Tasks // 各機能が並ぶイメージ。ここではサンプルとしてタスク機能をあげている
│   │   ├── __tests__ // 各テストの置き場
│   │   ├── index.tsx // IndexコンポーネントとContainerコンポーネントの置き場
│   │   ├── presenter.tsx // Presenterコンポーネントの置き場
│   │   ├── presenter.sp.tsx // SP用Presenterコンポーネントの置き場
│   │   ├── styles.module.scss // スタイル
│   │   ├── form.ts // フォームロジック
│   │   ├── action.ts // アクションロジック
│   │   ├── reducer.ts // state管理ロジック
│   │   ├── selector.ts // 描画ロジック
│   │   ├── mapper.ts // 値の変換ロジック（描画に関連するものを除く）
│   │   └── components // Presenterから分割したサブコンポーネント（※）
│   │        ├── Table.tsx
│   │        ├── Table.module.scss
│   │        ├── Table.module.scss.d.ts
│   │        ├── ...
```

※内部に複雑なstateやformを持つ必要があるModalやSidePeakは別途ルール化します

**コンポーネントの種類**

- Indexコンポーネント: 外部からimportされるコンポーネント。すべての親
    - 外部通信など副作用が大きな処理, 初期描画で必要なデータフェッチを行い、Containerコンポーネントに引き継ぐ
- Containerコンポーネント: 必要なロジックを集めて、Presenter (描画用コンポーネント) に受け渡すコンポーネント
    - Index から初期値を受け取り、各ロジックに初期値を受け渡す
    - 切り出された useForm や useState, useAction など、具体のロジックを集約させる
    
    ※ データフェッチ等が存在しない機能では、Containerコンポーネントをexportして、外部から参照可能にする(Indexを省略する)
    
- Presenterコンポーネント: 描画用コンポーネント
    - Container から受け取ったロジックを実際の描画に適応させる
    - 表示ロジックが複雑な場合は、selectorに切り出す

**※ indexコンポーネントの責務もContainerコンポーネントの責務も不要な場合は、index.tsxとして直接Presenterコンポーネントを組み込む形でOK！**

【参考】

[https://www.yuuniworks.com/blog/presentational-component-and-container-component](https://www.yuuniworks.com/blog/presentational-component-and-container-component)

[https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

**■ 切り出す関心事**

- useXXXForm (ex: useEditFormなど) → form.ts
    - フォームロジックの切り出し
        - → フォームスキーマや初期化を受け持つ
    - 具体の実装としては、zod, react-hook-form を採用している

[](https://github.com/MedUpInc/refery/blob/develop/apps/web/src/features/gst/SetupSettings/form.ts)

- useAction → action.ts
    - ユーザーアクションロジックの切り出し
        - → save、csv出力等の非同期通信を伴うユーザーアクションの切り出し

[](https://github.com/MedUpInc/refery/blob/develop/apps/web/src/features/gst/SetupSettings/action.ts)

- useXXXState (ex: useTaskStateなど) → reducer.ts
    - 複雑なstate管理の切り出し
        - stateの更新に合わせて、他のstateを更新するなど、複雑なstate管理が必要な場合は、切り出して管理する
        - ↑ このケースでは、useStateとuseEffectの組み合わせではなく、useReducer を推奨している

[](https://github.com/MedUpInc/refery/blob/develop/apps/web/src/features/tnt/WorkShift/reducer.ts)

- selector → selector.ts
    - Presenterコンポーネント内で描画する際に、描画ロジックが複雑なものを関数に切り出しておく
        - → 単体テストを書きやすくして保守性を上げる狙い
        - & 全体の見通しが良くなる

[](https://github.com/MedUpInc/refery/blob/develop/apps/web/src/features/tnt/WorkShift/selector.ts)

- mapper → mapper.ts
    - APIリクエストパラメーターの組み立てなど、描画に関係がない値の変換ロジックを関数に切り出しておく
- fetcher → src/app/hooks/fetcher/getXxxxx.ts
    - データフェッチは常にグローバルのhooks/fetcherに切り出しておく

**■SP版の表示切り替えについて**

SP表示に対応している画面では、Presenter（およびサブコンポーネント）でウィンドウサイズを判定して表示の切り替えを行う。切り替え単位は以下の3パターンとし、切り分けの範囲によって使い分ける

1. Presenter内でuseResponsiveを利用して一部分の表示を切り替える
2. Presenter自体をpresenter.tsx or presenter.sp.tsxとして切り分ける
3. Presenterから分離したサブコンポーネントを ListItem or ListItemSpのように分ける