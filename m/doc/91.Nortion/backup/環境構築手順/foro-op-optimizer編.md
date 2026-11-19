# foro-op-optimizer編

## 前準備

[medup core service 編](https://app.notion.com/p/medup-core-service-f33ab51be0d044e689adcfc008107b51?pvs=21) が完了している前提

## 環境変数の設定

apps/foro-op-optimizer-web 直下に次の .env を設置する

https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=y6el7oewq7uoczknd7gpb34fui&h=medupinc.1password.com

## 起動

単体で起動

```bash
# medup-core-service直下で実行
$ npm run dev -w apps/foro-op-optimizer-web
```

関連サービスとまとめて起動

```bash
# medup-core-service直下で実行
$ turbo run dev \
	--filter=./apps/foro-query-service \
	--filter=./apps/foro-command-service \
	--filter=./apps/foro-op-optimizer-web
```

 [一括起動](https://app.notion.com/p/2957138305f74f828baefceed0402c80?pvs=21) も可能

## ログイン

[http://localhost:3005](http://localhost:3005)へアクセス

- ログインアカウントはステージング環境と共有している。
- 実際にログインされるユーザーは foro-query-service の `.env` で設定している user になる。