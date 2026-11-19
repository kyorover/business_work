# foro-outpatient-web 編

## 前準備

[medup core service 編](https://app.notion.com/p/medup-core-service-f33ab51be0d044e689adcfc008107b51?pvs=21) が完了している前提

## 環境変数の設定

- /apps/foro-outpatient-web に以下の .env を設置する
    
    [1Password — Password Manager for Teams, Businesses, and Families](https://my.1password.com/vaults/all/tags/lrl7iwhu56zycoke4xunetyshp/rxbgur66rl65rjlxbjpn3hvo4u)
    
- 直リンクが行けなかったら、1pass ログイン後の検索で `foro-outpatient-web` を検索

## 起動

- /apps/foro-outpatient-web 配下で以下のコマンドを実行
    - もしくは [一括起動](https://app.notion.com/p/2957138305f74f828baefceed0402c80?pvs=21) も可能

```bash
$ npm run dev
```

## ログイン

- [http://localhost:3004](http://localhost:3004) へアクセス

![Untitled](Untitled.png)

ログインアカウントはステージング環境と共有している。

実際にログインされるユーザーは foro-query-service の `.env` で設定している user になる。

## 開発のTips

- バックエンドとの通信は tRPC のAPIを採用しているので、swagger 等のスキーマファイルを生成する必要なし

[https://www.figma.com/file/RqQ6pyy0o0l5DY7m7N3SSX/%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E6%A7%8B%E6%88%90?type=whiteboard&node-id=0-1&t=i1YWDbqxPpdTuPkU-0](https://www.figma.com/file/RqQ6pyy0o0l5DY7m7N3SSX/%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E6%A7%8B%E6%88%90?type=whiteboard&node-id=0-1&t=i1YWDbqxPpdTuPkU-0)

- 認証周りは azure b2c 上で foro crm と同じユーザープールを使用している。

[https://www.figma.com/file/KeW8bsofBhMpuPu0lOz3JD/%E8%AA%8D%E8%A8%BC%E5%91%A8%E3%82%8A%E3%81%AE%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF?type=whiteboard&node-id=0-1&t=vbUw3uaKPrd644nM-0](https://www.figma.com/file/KeW8bsofBhMpuPu0lOz3JD/%E8%AA%8D%E8%A8%BC%E5%91%A8%E3%82%8A%E3%81%AE%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF?type=whiteboard&node-id=0-1&t=vbUw3uaKPrd644nM-0)