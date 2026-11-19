<!DOCTYPE html>

<html lang="ja">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>foro CRM Eメール認証コード</title>
    <meta name="ROBOTS" content="NOINDEX, NOFOLLOW">
    <style>
        body {
            font-family: -apple-system,"BlinkMacSystemFont","Hiragino Kaku Gothic Pro","Noto Sans JP",sans-serif;
            font-size: 14px;
            color: #08151a;
        }
    </style>
</head>
<body>
    <div>
        <p>
            foro CRMをご利用いただきありがとうございます。<br>
            パスワードの再設定のための認証コードをお送りします。<br>
            <br>
            以下の6桁のコードをコピーしてブラウザの入力欄に貼り付けてください。
        </p>
        <br>
        <p style="font-weight: bold">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence2">認証コード:&nbsp;&nbsp;{{otp}}</span>
        </p>
        <br>
        <p>
            このURLの有効期限は20分です。<br>
            有効期限を過ぎてしまった場合は以下のURLの<br>
            "パスワードをお忘れの方はこちら"から再度パスワードの再設定を依頼ください。<br>
            https://stg-foro-crm.com
        </p>
        <p>
            このメールに覚えのない場合は、このメールを無視するか削除していただきますようお願いいたします。<br><br>
            ※このメールは送信専用のため返信はできません。<br>
            ご不明な点等ございましたら、foro CRM事務局までお知らせください。
        </p>
        <hr>
        <p>
            foro CRM事務局 &lt;cs@medup.jp&gt;<br>
        </p>
    </div>
</body>
</html>