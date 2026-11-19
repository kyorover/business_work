# コンポーネントの分離によるuseEffectの除去を推奨

分類: 設計・ツール

※基本的に利用しなくてよいuseEffectはなるべく利用しないようにしていく

**■ IndexコンポーネントとContainerコンポーネントに分けることで、副作用を局所的にしていく**

- Indexコンポーネント: 外部からimportされるコンポーネント。すべての親
    - ここで、外部通信など副作用が大きな処理, 初期描画で必要なデータフェッチを終わらせておく
- Containerコンポーネント: 必要なロジックを集めて、Presenter (描画用コンポーネント) に受け渡すコンポーネント
    - Index から初期値を受け取り、各ロジックに初期値を受け渡す
    - useForm や useState, useAction など、具体のロジックを集約させる
        - 各ロジックは、form, state(Reducer), action に記載する (各切り出しを参照)

以下、サンプル

[](https://github.com/MedUpInc/refery/blob/develop/apps/web/src/features/gst/SetupSettings/index.tsx)

```tsx
// 初期描画で必要となるデータフェッチ等は、コンポーネント毎切り分けておく
export const SetupSettings: React.FC = () => {
	// 一部省略している

  const { data, error } = useFetch('/gst/my-guest', () =>
    guestApi.getMyGuest(),
  );
  if (error) throw error;

  return (
    <Loading data={data}>{(loaded) => <Container data={loaded} />}</Loading>
  );
};

type ContainerProps = {
  data: GetMyGuestResponse;
};

// 初期データは、Propsで受け取り、こちらでは、各フォームやアクション、state管理ロジックを注入して、Presenterに渡す
export const Container: React.FC<ContainerProps> = (props) => {
  const { form } = useSetupForm(props.data);
  const { save } = useAction();

  return (
    <SetupSettingsPresenter
      form={form}
      profile={props.data}
      onClickNext={form.handleSubmit(save(form))}
    />
  );
};
```

```tsx
// 左のサンプルが、すべて同じコンポーネント内になっているケース
export const SetupSettings: React.FC = () => {
	// 一部省略している

  const { data, error } = useFetch('/gst/my-guest', () =>
    guestApi.getMyGuest(),
  );
  if (error) throw error;

  // data は初期値では undefined になってしまうため、初期化はuseEffectを使用する必要が出てくる
	const { form } = useSetupForm();
  useEffect(() => {
    if (data) {
      // 省略 form.setValues
    }
  }, [data]);

  const { save } = useAction();

  return (
    <SetupSettingsPresenter
      form={form}
      profile={props.data}
      onClickNext={form.handleSubmit(save(form))}
    />
  );
};

```

上記サンプルのように、データフェッチの結果でハンドリングするようなものが増えてくると(今回で言うformの初期化)、副作用(useEffect)が増え、どんどん見通しが悪くなってくるので、コンポーネント単位で責務を分離することを推奨している