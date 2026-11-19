# state設計の方法

分類: 設計・ツール

## 概要

- フロントエンドの機能追加（※）ではデータフローを整理・把握しておくことが肝になる。
- 機能追加開発をする場合、データフローを明確化する目的で必ず事前にstate設計を行うこととする。

※UIコンポーネント作成の場合はstate設計は重要ではない

ここで指しているstateとは画面が持っている状態からformとAPI取得結果を除いたもの（=reducerで管理する対象）。以下にstate設計の考え方、アウトプット、実施手順について記載する。

## 考え方

- Reactは状態（state + form + API取得結果）をUIに変換する仕組み
- 必要最小限の独立したstateを持たせる
- 他のstateから計算可能な状態はstateとして持たせない
- 他のstate変更時に依存して変更する必要があるstateを整理

## アウトプット

state同士の関係を依存関係グラフとして記載。きれいにしない

[https://www.figma.com/board/sJDbkbVjWFBw8yaJGWuvG9/%5Bstate%E8%A8%AD%E8%A8%88%5D%E9%80%A3%E6%90%BA%E5%85%88%E3%83%95%E3%82%A3%E3%83%BC%E3%83%89%E3%83%90%E3%83%83%E3%82%AF?node-id=0-1&p=f&t=YpFt5vzCoQuAA0Nc-0](https://www.figma.com/board/sJDbkbVjWFBw8yaJGWuvG9/%5Bstate%E8%A8%AD%E8%A8%88%5D%E9%80%A3%E6%90%BA%E5%85%88%E3%83%95%E3%82%A3%E3%83%BC%E3%83%89%E3%83%90%E3%83%83%E3%82%AF?node-id=0-1&p=f&t=YpFt5vzCoQuAA0Nc-0)

## 手順

- 画面で扱うデータの洗い出し
    - UIから以下を抜き出す
    - ユーザーが操作によって変更可能なUI要素
- 分類とデータの依存関係の整理
    - APIから取得した結果（or それを変形したもの） → useSWRで管理
        - APIから取得した結果はuseSWRで管理する
        - ↑を変形したもの → 場合に応じてuseMemoで変形結果をキャッシュ
    - Aを変更したらBも追随して変更することがある
        - Bはユーザーの入力によっても変わる → stateとして持つ
        - Bが他のstateによってのみ決まる → stateとして持つ必要はない
- formを持つModal、SidePeakは切り出し、それ単体でformとstateを持たせる
    - 利用側にはformでのみ利用するstateは持たせない
    - 基本的にformのデフォルト値とonSubmitを入出力とする
    - formが参照するデータ（診療科一覧など）を渡すことは許可