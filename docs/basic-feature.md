# 要件定義

## ユースケース

* 認証基盤系
  * ルートユーザ
  * IAM
    * ユーザ
    * API
* リソース操作系
  * VPC
  * EC2
  * RDS

localhostにDNSを立てて、hoge.localのようなドメイン名でEC2およびRDSに接続できるようにする。

### 認証基盤系
#### ユーザ認証

##### ルートユーザ登録機能

ユーザから受け取ったEメールおよびパスワードを保存する機能

###### Input

* Eメール
  RFC準拠っぽくなってればいい(言語・フレームワークについているやつでいいよ)
* パスワード
  8文字以上
  半角英数字のみ

###### Output

RDB上のaccountsテーブル

* アカウントID

  12桁であること,

  0-9の数字だけを持つこと,

  各アカウントでユニークであること

* Eメール

* パスワード

  保存時にハッシュ化すること(BCryptPasswordEncoderを使用すること)

#### IAMユーザ登録機能

ルートユーザもしくは他のIAMユーザからIAMユーザ名とパスワードを受け取り保存する。
その際付随情報として、コンソールへのログインが必要か？も受け取ることが可能にする。(デフォルトではログインが必要なユーザが作成される。)

##### Input

* IAMユーザ名
* パスワード
* コンソールへのログインが必要か？(プログラムのみから接続するIAMユーザか？)

##### Output

###### RDB

RDB上のusersテーブルに以下の情報を保存する。

* IAMユーザ名
* access_key_id

  ASIAから始まる大文字英字および数字の混合20文字のランダム文字列

* secret_access_key

  英数字記号含めた40文字のランダム文字列

###### APIの返答

* IAMユーザ名
* access_key_id
* secret_access_key

#### ログイン機能

##### Input

ルートユーザの場合

* Eメールアドレス
* パスワード

IAMユーザの場合

* アカウントID 
* IAMユーザ名
* パスワード

###### RDB

ルートユーザの場合、accountsテーブル
IAMユーザの場合、usersテーブルを参照し
それに紐づくポリシーを取得し、セッショントークンを生成すること。

##### Output

###### APIの返答

```json
{
  "access_token": "{アクセストークン}",
  "refresh_token": "{リフレッシュトークン}"
}
```

###### アクセストークン

  JWTを使用する。

  JWT内部に認可情報を埋め込みその内容をフロントエンド側で使用する想定

  * 認可される対象リソースはARN準拠とし、その操作に関してはIAM Policy準拠とする。
    https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference-arns.html

  * ルートユーザの認可情報は*のような特殊文字により、制限がないことを表現できること。

###### リフレッシュトークン

  ランダム文字列を生成する。
  255バイトのランダムな文字列

###### RDB

APIの返答に作成されたリフレッシュトークンは、ハッシュ化した状態でrefresh_tokensテーブルに保存する。
