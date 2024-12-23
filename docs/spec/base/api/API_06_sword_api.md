# SWORD API

### 目次
- [目的・用途](#目的用途)
- [利用方法](#利用方法)
- [利用可能なロール](#利用可能なロール)
- [機能内容](#機能内容)
- [API仕様](#api仕様)
- [ドキュメント仕様](#ドキュメント仕様)
- [エラータイプ](#エラータイプ)
- [関連モジュール](#関連モジュール)
- [処理概要](#処理概要)
- [エラーメッセージ](#エラーメッセージ)
- [サーバー設定値](#サーバー設定値)
- [更新履歴](#更新履歴)

## 目的・用途

クライアントからSWORD v3プロトコルに従いリポジトリ上のアイテム操作を実現する。  
アイテム登録には、TSV/CSV、XML、あるいはJSON-LD形式のメタデータを含むZIPファイルを用いる。

## 利用方法

APIの認証にはOAuth2を利用する。  
アクセストークンの発行は[API-1:OAuth2](./API_01_Oauth2.md#oauth2)を参照。

### Scope：
TSV/CSVおよびJSON-LD形式のメタデータを含むZIPファイルを直接登録するためには以下のスコープが必要となる。
- deposit: write
XMLおよびJSON-LD形式のメタデータを含むZIPファイルをワークフロー登録するためには以下のスコープが必要となる。
- deposit: write
- user:activity

### エンドポイント：

| 項番 | HTTP request                  | 内容                                                                                                          |
| ---- | ----------------------------- | ------------------------------------------------------------------------------------------------------------- |
|  1   | GET /sword/service-document   | リポジトリのサービスドキュメントを取得する。                                                                  |
|  2   | POST /sword/service-document  | WEKO3の一括登録フォーマットを用いて、アイテムを登録する。                                                     |
|  3   | GET /sword/deposit/<recid>    | recidを指定してリポジトリ上に存在するアイテムのステータスドキュメントを取得する。                             |
|  4   | PUT /sword/deposit/<recid>    | recidを指定してリポジトリ上に存在するアイテムに対して、JSON-LD形式のメタデータで更新する。<br/>※現在は未実装 |
|  5   | DELETE /sword/deposit/<recid> | recidを指定してアイテムを削除する。                                                                           |


### CURLでのリクエスト実行例：

各APIのリクエスト仕様の詳細は後述。

#### GET /sword/service-document
##### リクエスト

```shell
$ curl -X GET https://192.168.56.101/sword/service-document \
    -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo"
```

  - -H オプション

    - リクエストにカスタムヘッダーを追加する
    - Authorization は "Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する

##### レスポンス

```json
{
  "@context": "https://swordapp.github.io/swordv3/swordv3.jsonld",
  "@id": "https://192.168.56.101/sword/service-document",
  "@type": "ServiceDocument",
  "accept": [
    "*/*"
  ],
  "acceptArchiveFormat": [
    "application/zip"
  ],
  "acceptDeposits": true,
  "acceptMetadata": [
    "https://github.com/JPCOAR/schema/blob/master/2.0/jpcoar_scm.xsd",
    "https://w3id.org/ro/crate/1.1/"
  ],
  "acceptPackaging": [
    "*"
  ],
  "authentication": [
    "OAuth"
  ],
  "byReferenceDeposit": false,
  "collectionPolicy": {},
  "dc:title": "WEKO3",
  "dcterms:abstract": "",
  "digest": [
    "SHA-256",
    "SHA",
    "MD5"
  ],
  "maxAssembledSize": 30000000000000,
  "maxByReferenceSize": 30000000000000000,
  "maxSegments": 1000,
  "maxUploadSize": 16777216000,
  "onBehalfOf": true,
  "root": "https://192.168.56.101/sword/service-document",
  "staging": "",
  "stagingMaxIdle": 3600,
  "treatment": {},
  "version": "http://purl.org/net/sword/3.0"
}
```

#### POST /sword/service-document
##### リクエスト

```shell
$ curl -X POST -s -k https://192.168.56.101/sword/service-document -F "file=@import.zip;type=application/zip" \
    -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo" \
    -H "Content-Disposition:attachment; filename=import.zip" -H "Packaging:http://purl.org/net/sword/3.0/package/SimpleZip"
```

##### レスポンス

```json
{
  "@context": "https://swordapp.github.io/swordv3/swordv3.jsonld",
  "@id": "https://192.168.56.101/sword/deposit/96568",
  "@type": "Status",
  "actions": {
    "appendFiles": false,
    "appendMetadata": false,
    "deleteFiles": false,
    "deleteMetadata": false,
    "deleteObject": true,
    "getFiles": false,
    "getMetadata": false,
    "replaceFiles": false,
    "replaceMetadata": false
  },
  "eTag": "5",
  "fileSet": {},
  "links": [
    {
      "@id": "https://weko3.ir.rcos.nii.ac.jp/records/96568",
      "contentType": "text/html",
      "rel": [
        "alternate"
      ]
    },
    {
      "@id": "http://hdl.handle.net/20.500.12465/0000096568",
      "contentType": "text/html",
      "rel": [
        "alternate"
      ]
    }
  ],
  "metadata": {},
  "service": "/sword/service-document",
  "state": [
    {
      "@id": "http://purl.org/net/sword/3.0/state/ingested",
      "description": ""
    }
  ]
}
```

  - -F オプション
      - POSTするファイルを指定する。自動的にContent-Typeは"multipart/form-data"となる
      - boundaryやContent-Lengthは自動で付加されるため自前で指定しなくてもよい
      - ファイル名の先頭には@を付加すること
      - ファイルのContent-Typeを"application/zip"とするため、ここでtypeを指定する（指定しないと application/octet-stream となってしまう）

  - -H オプション
      - Authorization は "Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する
      - Content-Disposition の filename は -Fオプションで指定したファイルのファイル名と一致させる
      - Packaging は "http://purl.org/net/sword/3.0/package/SimpleZip" を指定
      - 必須の Content-Length および Content-Type については前述の通り、-Fオプションにて自動付加されるため-Hオプションでの指定は不要


#### GET /sword/deposit/\<recid\>

```
curl -X GET https://192.168.56.101/sword/deposit/1 -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo"
```

  - H オプション
      - Authorization は "Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する

#### DELETE /sword/deposit/\<recid\>

```
curl -X DELETE https://192.168.56.101/sword/deposit/1 -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo"
```

  - H オプション
      - Authorization は "Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する


## 利用可能なロール

|  ロール  | システム管理者 | リポジトリ管理者 | コミュニティ管理者 | 登録ユーザー | 一般ユーザー | ゲスト(未ログイン) |
| -------- | -------------- | ---------------- | ------------------ | ------------ | ------------ | ------------------ |
| 利用可否 |       〇       |        〇        |         〇         |      〇      |      〇      |        ×          |


## 機能内容

- 各APIへのリクエストに応じて処理を実行しレスポンスを返す
    - OAuthアクセストークンによるユーザー認証を必須とする

- アイテム登録機能で登録に使用するZIPファイルは、メタデータのファイルがTSV/CSV形式、XML形式、あるいはJSON-LD形式である必要がある。
    - TSV/CSV形式のメタデータを含むZIPファイルの詳細は [ADMIN-2-4:インポート](../admin/ADMIN_2_4.md#インポート) を参照
    - JSON-LD形式のメタデータを含むZIPファイルは、RO-Crate+BagItまたはSWORDBagItに準拠したZIPファイルである必要がある。

- XMLおよびJSON-LD形式のメタデータは、マッピング機能をもちいてWEKO3のアイテムタイプへ変換されメタデータの登録および更新に使用される。  
  また、データセット登録設定([設定:30](#conf30))が有効の場合、ZIPファイルそのものもアイテムのファイルの一つとして保存する。

## API仕様

### サービスドキュメント取得機能：GET /sword/service-document
リポジトリのサービスドキュメントを取得する。

#### エンドポイント
GET /sword/service-document

#### リクエストヘッダー

| ヘッダー      | 必須 | 説明                                                                                                                  | 例                                      |
| ------------- | ---- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Authorization |  ○  | 操作するWEKOユーザーのOAuth認証情報。アクセストークンを用いる。<br/>"Bearer" + " (半角スペース)" + "トークン"の形式。 | "Bearer fVzaeTNY5PCHsNS3rZOARrYR7kPBl4" |
| On-Behalf-Of  |  -   | 代理投稿ユーザーのメールアドレス、ePPNなどを指定する。                                                                |                                         |

#### レスポンスコード

| コード | ドキュメント         | 説明                                                                                                                  |
| ------ | -------------------- | --------------------------------------------------------------------------------------------------------------------- |
| 200    | サービスドキュメント | サーバーのサービスドキュメントを返す。                                                                                |
| 400    | エラードキュメント   | リクエスト内容に何らかの不備がある場合。                                                                              |
| 401    | エラードキュメント   | リクエストでAuthorization ヘッダーが提供されない場合。                                                                |
| 403    | エラードキュメント   | 認証に失敗した場合。                                                                                                  |
| 412    | エラードキュメント   | サーバー側がOn-Behalf-Of をサポートしていないにもかかわらず、<br/>リクエストでOn-Behalf-Of ヘッダーが提供された場合。 |
| 500    | エラードキュメント   | サーバー内部エラーが発生した場合。                                                                                    |


### アイテム登録機能：POST /sword/service-document
一括登録用のZIPファイルを用いてアイテムを新規登録する。

#### エンドポイント
POST /sword/service-document

#### リクエストヘッダー

| フィールド          | 必須   | 説明                                                                                                                                                                                                                                                                                   | 例                                                                                                         |
| ------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Authorization       | ○     | 操作するWEKOユーザーのOAuth認証情報。アクセストークンを用いる。<br/>"Bearer" + " (半角スペース)" + "トークン"の形式。                                                                                                                                                                  | "Bearer fVzaeTNY5PCHsNS3rZOARrYR7kPBl4"                                                                    |
| On-Behalf-Of        | -      | 代理投稿ユーザーのメールアドレス、パーソナルアクセストークンまたはePPNが入る。                                                                                                                                                                                                         | パーソナルアクセストークン: <br>　　"e0Pke8qpzEkkGjPE1RoSqNw7qu3tH4..."<br>ePPN: "sample@sampleuniv.ac.jp" |
| Content-Disposition | ○     | リクエストボディに付加したファイルのファイル名を指定する。                                                                                                                                                                                                                             | "attachment; filename=example.zip"                                                                         |
| Content-Length      | ※     | リクエストボディに付加したファイルサイズを指定する。<br/>※ファイルサイズ検証設定([設定値:29](#conf29))が有効の場合、必須。                                                                                                                                                            | 1024000                                                                                                    |
| Content-Type        | ○     | リクエストボディにファイルを付加するため "multipart/form-data" を指定する。                                                                                                                                                                                                            | multipart/form-data; boundary=xxxxxxxx                                                                     |
| Packaging           | ○     | パッケージフォーマットと指定する。<br/>SWORDでは以下の3つのパッケージフォーマットが定義されている。<br/>http://purl.org/net/sword/3.0/package/Binary<br/>http://purl.org/net/sword/3.0/package/SimpleZip<br/>http://purl.org/net/sword/3.0/package/SWORDBagIt<br/>※現在Binaryは未対応 | "http://purl.org/net/sword/3.0/package/SimpleZip"                                                          |
| Digest              | ※     | ボディに付加したファイルのハッシュ値を指定する。<br/>※ダイジェスト検証設定([設定値:28](#conf28))が有効の場合、メタデータのファイルがJSON-LD形式であるときに必須。                                                                                                                     | "SHA-256=e0Pke8qpzEkkGjPE1RoSqNw7qu3tH4..."                                                                |


#### ボディ

| フィールド | 必須 | 説明                                                                                                               | 例                                                          |
| ---------- | ---- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| file       | ○   | form-data 形式でボディにZIPファイルを付加する。<br/>ファイルのContent-Type には"application/zip"を指定すること。   | "file=@project/post_files/example.zip;type=application/zip" |

#### レスポンスコード

| コード | ドキュメント           | 説明                                                                                                                   |
| ------ | ---------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 200    | ステータスドキュメント | 登録されたアイテムのステータスドキュメントを返す。                                                                     |
| 400    | エラードキュメント     | リクエスト内容に何らかの不備がある場合。                                                                               |
| 401    | エラードキュメント     | リクエストでAuthorization ヘッダーが提供されない場合。                                                                 |
| 403    | エラードキュメント     | 認証に失敗した場合。<br/>認証したOAuthトークンが「deposit:write」スコープを持っていない場合。                          |
| 404    | エラードキュメント     | 登録されたアイテムが見つからない場合。                                                                                 |
| 412    | エラードキュメント     | サーバー側がOn-Behalf-Of をサポートしていないにもかかわらず、<br/>リクエストでOn-Behalf-Of ヘッダーーが提供された場合。|
| 413    | エラードキュメント     | 送信されたファイルのサイズがサーバーに設定されたmaxUploadSizeを超えている場合。                                        |
| 415    | エラードキュメント     | ヘッダーまたはボディに付加されたファイルのContent-Typeがサーバー側で<br/>サポートされていない場合。                    |
| 500    | エラードキュメント     | サーバー内部エラーが発生した場合。                                                                                     |


### アイテム状態取得機能：GET /sword/deposit/\<recid\>

#### エンドポイント
GET /sword/deposit/\<recid\>

#### リクエストヘッダー

| フィールド         | 必須 | 説明                                                                                                                  | 例                                      |
| ------------------ | ---- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Authorization      | ○   | 操作するWEKOユーザーのOAuth認証情報。アクセストークンを用いる。<br/>"Bearer" + " (半角スペース)" + "トークン"の形式。 | "Bearer fVzaeTNY5PCHsNS3rZOARrYR7kPBl4" |
| On-Behalf-Of       | -    | 操作するWEKOユーザーのOAuth認証情報。<br/>"Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する。         | "user@example.com"                      |


#### パスパラメータ

| キー        | 必須 | 説明                    | 例                   |
| ----------- | ---- | ----------------------- | -------------------- |
| \<recid\>   | ○   | レコードID              | 20000021             |


#### レスポンスコード

| コード | ドキュメント           | 説明                                                                                                                       |
| ------ | ---------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| 200    | ステータスドキュメント | 指定されたアイテムのステータスドキュメントを返す。                                                                         |
| 400    | エラードキュメント     | リクエスト内容に何らかの不備がある場合。                                                                                   |
| 401    | エラードキュメント     | リクエストでAuthorization ヘッダーが提供されない場合。                                                                     |
| 403    | エラードキュメント     | 認証に失敗した場合。                                                                                                       |
| 404    | エラードキュメント     | 指定したrecidに該当するアイテムが存在しない（削除されている）場合。                                                        |
| 412    | エラードキュメント     | サーバー側がOn-Behalf-Of をサポートしていないにもかかわらず、<br/>リクエストでOn-Behalf-Of ヘッダーが提供された場合。      |
| 500    | エラードキュメント     | サーバー内部エラーが発生した場合。                                                                                         |


### アイテム削除機能：DELETE /sword/deposit/\<recid\>

#### エンドポイント
DELETE /sword/deposit/\<recid\>

#### リクエストヘッダー

| フィールド         | 必須 | 説明                                                                                                                  | 例                                      |
| ------------------ | ---- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Authorization      | ○   | 操作するWEKOユーザーのOAuth認証情報。アクセストークンを用いる。<br/>"Bearer" + " (半角スペース)" + "トークン"の形式。 | "Bearer fVzaeTNY5PCHsNS3rZOARrYR7kPBl4" |
| On-Behalf-Of       | -    | 操作するWEKOユーザーのOAuth認証情報。<br/>"Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する。         | "user@example.com"                      |


#### パスパラメータ

| キー        | 必須 | 説明                    | 例                   |
| ----------- | ---- | ----------------------- | -------------------- |
| \<recid\>   | ○   | レコードID              | 20000021             |


#### レスポンスコード

| コード | ドキュメント           | 説明                                                                                                                       |
| ------ | ---------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| 204    | -                      | 空のレスポンスを返す。                                                                                                     |
| 400    | エラードキュメント     | リクエスト内容に何らかの不備がある場合。                                                                                   |
| 401    | エラードキュメント     | リクエストでAuthorization ヘッダーが提供されない場合。                                                                     |
| 403    | エラードキュメント     | 認証に失敗した場合。                                                                                                       |
| 404    | エラードキュメント     | 指定したrecidに該当するアイテムが存在しない（削除されている）場合。                                                        |
| 412    | エラードキュメント     | サーバー側がOn-Behalf-Of をサポートしていないにもかかわらず、<br/>リクエストでOn-Behalf-Of ヘッダーが提供された場合。      |
| 500    | エラードキュメント     | サーバー内部エラーが発生した場合。                                                                                         |


## ドキュメント仕様

### サービスドキュメント
サーバー全体の機能と操作パラメータを定義したドキュメント

| 項目                         | 型      | 説明                                                                                                                                                       |
| ---------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| @context                     | string  | "https://swordapp.github.io/swordv3/swordv3.jsonld" を固定で出力。                                                                                        |
| @id                          | string  | "[WEKO3のURL]/sword/service-document"を出力。                                                                                                              |
| @type                        | string  | "ServiceDocument"を固定で出力。                                                                                                                            |
| accept                       | array   | サーバーに受け入れられるコンテンツタイプのリスト。"\*/\*"を出力する。[設定値:5](#conf05)                                                                   |
| acceptArchiveFormat          | array   | サーバーが解凍できるアーカイブ形式のリスト。現状"application/zip"のみ対応。[設定値:6](#conf06)                                                                                |
| acceptDeposits               | boolean | サーバーがデポジットを受け入れるか否か。[設定値:7](#conf07)                                                                                                |
| acceptMetadata               | array   | サーバーで受け入れ可能なメタデータ形式のリスト。[設定値:8](#conf08)                                                                                        |
| acceptPackaging              | array   | サーバーで受け入れ可能なパッケージ形式のリスト。<br/>現状すべての形式を受け入れるが、アイテム登録はSimpleZip/SWORDBagIt形式でのみ可能。[設定値:9](#conf09) |
| authentication               | Array   | サーバーでサポートされている認証スキームのリスト。現状”OAuth”のみ対応。[設定値:17](#conf17)                                                              |
| byReferenceDeposit           | boolean | サーバーがbyReferenceDepositをサポートしているか否か。現状未対応のためFalseを出力。[設定値:14](#conf14)                                                    |
| collectionPolicy             | object  | コレクションポリシーを示すオブジェクト。[設定値:10](#conf10)                                                                                               |
| collectionPolicy.@id         | string  | コレクションポリシーのURL。                                                                                                                                |
| collectionPolicy.description | string  | コレクションポリシーの説明。                                                                                                                               |
| dc:title                     | string  | リポジトリの名称を出力。"WEKO3"を固定で出力。                                                                                                              |
| dcterms:abstract             | string  | リポジトリの説明。未設定。[設定値:4](#conf04)                                                                                                              |
| digest                       | array   | サーバーが受け入れるdigest形式のリスト。<br/>現状digestはメタデータのファイルがJSON-LD形式であるときのみ、SHA-256の検証が可能。[設定値:16](#conf16)        |
| maxAssembledSize             | integer | Segmented File Upload時のファイル合計最大サイズ（単位：byte）。[設定値:21](#conf21)                                                                        |
| maxByReferenceSize           | integer | By-Reference Deposit時のファイル最大サイズ（単位：byte）。[設定値:20](#conf20)                                                                             |
| maxSegmentSize               | integer | Segmented File Upload時の１ファイルの最大サイズ（単位：byte）。<br/>現時点では出力していない。                                                             |
| maxSegments                  | integer | Segmented File Upload時のセグメントの最大数。[設定値:22](#conf22)                                                                                          |
| maxUploadSize                | integer | アップロードされるファイルの最大サイズ（単位：byte）。[設定値:19](#conf19)                                                                                 |
| minSegmentSize               | integer | Segmented File Upload時の１ファイルの最小サイズ（単位：byte）。                                                                                            |
| onBehalfOf                   | boolean | 代理投稿をサポートしているか否か。<br/>現状メタデータのファイルがJSON-LD形式であるときのみ対応している。[設定値:15](#conf15)                               |
| root                         | string  | サービスドキュメントのルートURL。                                                                                                                          |
| services                     | array   | 親サービスに含まれるサービスのリスト。現状未対応。[設定値:18](#conf18)                                                                                     |
| staging                      | string  | Segmented File Upload時にコンテンツをステージング先URL。現状未対応のため空文字を出力。[設定値:12](#conf12)                                                 |
| stagingMaxIdle               | integer | ステージングされたファイルの最小保持時間。[設定値:13](#conf13)                                                                                             |
| treatment                    | object  | デポジット時に期待される処理のURLと説明を示すオブジェクト。[設定値:11](#conf11)                                                                            |
| treatment.@id                | string  | 処理のURL。                                                                                                                                                |
| treatment.description        | string  | 処理の説明。                                                                                                                                               |
| version                      | string  | サポートしているSWORDバージョン。"http://purl.org/net/sword/3.0"を出力。[設定値:3](#conf03)                                                                |


### ステータスドキュメント
アイテムの内容と現在の状態に関する詳細情報を示すドキュメント

| 項目                         | 型      | 説明                                                                                                                                   |
| ---------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| @context                     | string  | "https://swordapp.github.io/swordv3/swordv3.jsonld" を固定で出力。                                                                     |
| @id                          | string  | "[WEKO3のURL]/sword/deposit/[アイテムのrecid]"を出力。                                                                                 |
| @type                        | string  | "ServiceDocument"を固定で出力。                                                                                                        |
| actions                      | object  | アイテムに対してSWORDで使用可能なアクション。  現時点では deleteObject のみTrueを返し、それ以外はFalseを返すようになっている。         |
| actions. appendFiles         | boolean | ファイル追加要求が発行可能か否か。                                                                                                     |
| actions.appendMetadata       | boolean | メタデータ追加要求が発行可能か否か。                                                                                                   |
| actions. deleteFiles         | boolean | ファイル削除要求が発行可能か否か。                                                                                                     |
| actions. deleteMetadata      | boolean | メタデータ削除要求が発行可能か否か。                                                                                                   |
| actions. deleteObject        | boolean | アイテム削除要求が発行可能か否か。                                                                                                     |
| actions. getFiles            | boolean | ファイル取得要求が発行可能か否か。                                                                                                     |
| actions. getMetadata         | boolean | メタデータ取得要求が発行可能か否か。                                                                                                   |
| actions. replaceFiles        | boolean | ファイル置き換え要求が発行可能か否か。                                                                                                 |
| actions. replaceMetadata     | boolean | メタデータ置き換え要求が発行可能か否か。                                                                                               |
| eTag                         | string  | アイテムのeTag。  WEKOではアイテムのリビジョン番号を返す。                                                                             |
| fileSet                      | object  | ファイルセットを示すオブジェクト。  現時点では空オブジェクトを返す。                                                                   |
| fileSet.@id                  | string  | ファイルセットのURL。                                                                                                                  |
| fileSet.eTag                 | string  | ファイルセットのeTag。                                                                                                                 |
| links                        | array   | アイテムのリンクを示すオブジェクト。  現時点ではアイテム詳細ページのURLを出力する。またDOIやCNRIハンドルを持つ場合も同様に出力する。   |
| links[].@id                  | string  | リソースのURL。                                                                                                                        |
| links[].byReference          | string  | byReference deposit の際の参照元URL。                                                                                                  |
| links[].contentType          | string  | リソースのコンテンツタイプ。                                                                                                           |
| links[].dcterms:isReplacedBy | string  | 同じオブジェクト内のファイルの新しいバージョンへのURL。                                                                                |
| links[].dcterms:relation     | string  | 非SWORDアクセスポイントへのURL。                                                                                                       |
| links[].dcterms:replaces     | string  | 同じオブジェクト内の古いバージョンのファイルへのURL。                                                                                  |
| links[].depositedBy          | string  | アイテム登録を行ったユーザーの識別子。                                                                                                 |
| links[].depositedOn          | string  | アイテム登録日時のタイムスタンプ。                                                                                                     |
| links[].depositedOnBehalfOf  | string  | 代理投稿により登録を行ったユーザーの識別子。                                                                                           |
| links[].derivedFrom          | string  | 現在のリソースが派生したリソースのURLへの参照。                                                                                        |
| links[].eTag                 | string  | リソースのeTag。                                                                                                                       |
| links[].log                  | string  | クライアントが知っておくべきデポジットに関連する情報。                                                                                 |
| links[].packaging            | string  | リソースがパッケージである場合、パッケージ形式の識別子を示す。                                                                         |
| links[].rel                  | string  | リソースとオブジェクトの関係。  以下の何れかの文字列を持つ。<ul><li>alternate</li><li>packaging</li><li>depositedOn</li><li>depositedOnBehalfOf</li><li>status</li><li>log</li><li>dcterms:relation</li><li>dcterms:replaces</li><li>dcterms:isReplacedBy</li><li>versionReplaced</li><li>eTag</li><li>byReference</li><li>derivedFrom</li><li>metadataFormat</li></ul> |
| links[].status               | string  | 取り込みに関するリソースのステータス。                                                                                                 |
| links[].versionReplacedOn    | string  | 現在のリソースが新しいリソースに置き換えられた日付。                                                                                   |
| metadata                     | object  | メタデータを示すオブジェクト。  現時点では空オブジェクトを返す。                                                                       |
| metadata.@id                 | string  | メタデータのURL。                                                                                                                      |
| metadata.eTag                | string  | メタデータのeTag。                                                                                                                     |
| service                      | string  | サービスドキュメントのURL。                                                                                                            |
| state                        | array   | アイテムがサーバー上にある状態のリスト。                                                                                               |
| state[].@id                  | string  | 状態の識別子。現状では"http://purl.org/net/sword/3.0/state/ingested"を固定で出力。                                                     |
| state[].description          | string  | 状態の説明                                                                                                                             |

### エラードキュメント
エラー内容を表すドキュメント

| 項目      | 型     | 説明                                                                |
| --------- | ------ | ------------------------------------------------------------------- |
| @context  | string | "https://swordapp.github.io/swordv3/swordv3.jsonld"を固定で出力。   |
| @type     | string | エラータイプを示す文字列。[エラータイプ](#エラータイプ) を参照。    |
| error     | string | エラー内容の説明。                                                  |
| log       | string | より詳細なエラー内容。現在は出力していない。                        |
| timestamp | string | エラー発生時のタイムスタンプ。                                      |

## エラータイプ

| エラータイプ文字列           | エラーコード | エラー原因等                                                                           |
| ---------------------------- | ------ | -------------------------------------------------------------------------------------------- |
| AuthenticationFailed         | 403    | 認証に失敗。                                                                                 |
| AuthenticationRequired       | 401    | 認証情報が不足。                                                                             |
| BadRequest                   | 400    | リクエストに何らかの不備がある。                                                             |
| ByReferenceFileSizeExceeded  | 400    | サーバーの制限を超えるファイルをデポジットしようとした。                                     |
| ByReferenceNotAllowed        | 412    | サーバーが By-Reference deposit をサポートしていない。                                       |
| ContentMalformed             | 400    | リクエスト本文の内容に不正がある。                                                           |
| ContentTypeNotAcceptable     | 415    | サーバーで許可されていないコンテンツタイプをリクエストした。                                 |
| DigestMismatch               | 412    | リクエストヘッダーによって提供されたdigestがサーバーで受け取ったコンテンツと一致していない。 |
| ETagNotMatched               | 412    | リクエストヘッダーによって提供されたIf-Matchの値が更新対象コンテンツのeTagと一致していない。 |
| ETagRequired                 | 412    | リクエストヘッダーにIf-Matchの値が指定されていない。                                         |
| Forbidden                    | 403    | サーバーによって許可されていない操作をリクエストした。                                       |
| FormatHeaderMismatch         | 415    | サーバーがサポートしていない形式のコンテンツがリクエストされた。                             |
| InvalidSegmentSize           | 400    | Segmented File Upload時のファイルサイズが範囲外。                                            |
| MaxAssembledSizeExceeded     | 400    | Segmented File Upload時の合計ファイルサイズが最大値を超えている。                            |
| MaxUploadSizeExceeded        | 413    | アップロードされたコンテンツサイズが最大値を超えている                                       |
| MetadataFormatNotAcceptable  | 415    | サーバーがサポートしていない形式のMetadata-Formatがリクエストされた。                        |
| MethodNotAllowed             | 405    | メソッドへのアクセスが許可されていない。                                                     |
| OnBehalfOfNotAllowed         | 412    | サーバーが On-Behalf-Of をサポートしていない。                                               |
| PackagingFormatNotAcceptable | 415    | サーバーがサポートしていない形式のPackagingフォーマットがリクエストされた。                  |
| SegmentedUploadTimedOut      | 410    | Segmented File Upload先のURLにアクセスできない。                                             |
| SegmentLimitExceeded         | 400    | セグメント数が最大値を超えている。                                                           |
| UnexpectedSegment            | 400    | サーバーが予期していないセグメントを受信した。                                               |
| **Additional ErrorType**     |        |                                                                                              |
| NotFound                     | 404    | リクエストされたリソースが存在しない。                                                       |
| ServerError                  | 500    | サーバー内部エラーが発生した。                                                               |


## 関連モジュール


  - invenio_oauth2server：OAuthトークンによるユーザー認証を行う

  - invenio_deposit：OAuthトークンが参照するデポジット操作スコープを定義している

  - weko_swordserver：リクエストの処理を行う

  - weko_records_ui：レコード情報の取得、アイテムの削除を実行する

  - weko_search_ui：インポート処理を実行する

  - weko_workflow：ワークフロー経由でインポート処理を実行する

## 関連テーブル

  - sword_clients：SWORD連携設定情報を保持する

    - id：ID
    - client_id：クライアントID
    - registration_type_id：登録方式区分
    - mapping_id：マッピング定義ID
    - workflow_id：ワークフローID

  - sword_item_type_mappings：マッピング定義設定を保持する

    - id：マッピング定義ID
    - name：マッピング定義名
    - mapping：マッピング定義(JSON)
    - item_type_id：アイテムタイプID
    - version_id：バージョンID
    - is_delete：論理削除フラグ

  - oauth2server_token：クライアントが使用できるトークンを保持する

    - id：オブジェクトID
    - client_id：クライアントID
    - user_id：ユーザーID
    - token_type：トークンタイプ
    - access_token：アクセストークン
    - refresh_token：リフレッシュトークン
    - expires：有効期限
    - _scopes：スコープ
    - is_personal：個人用トークンフラグ
    - is_internal：内部用トークンフラグ


## 処理概要
使用する設定値は[サーバー設定値](#サーバー設定値)、エラーメッセージは[エラーメッセージ](#エラーメッセージ)を参照。

### サービスドキュメント取得機能：GET /sword/service-document

- リクエストをチェックする
    - Authorizationヘッダーに記載されたOAuth認証情報を使用しWEKOにログインする
    - On-Behalf-Ofヘッダーが存在する場合、サーバー設定を確認する
- サーバー設定値を参照し、サービスドキュメントを生成する
- サービスドキュメントを返却する

### アイテム登録機能：POST /sword/service-document
#### GRDM側(想定)
1. GRDMでインポート画面を表示する際、WEKOのAPIを使用しユーザーの選択可能な登録先を取得する
2. GRDMでインポートするデータを指定し、パッケージ化する
3. WEKOのエンドポイントPOST /sword/service-documentにパッケージを送信する  
  この時、リクエストヘッダーにアクセストークンと登録先を付加する

#### WEKO3側
1. リクエストをチェックする
    - **`Authorization`** ヘッダーに記載されたアクセストークンを使用しユーザーを認証する。  
      アクセストークンのScopeを確認し、`deposit:write`が与えられていなければエラーとする。
    - **`On-Behalf-Of`** ヘッダーが存在する場合、`On-Behalf-Of`許容設定([設定値:15](#conf15))が無効であればエラーとする。
    - **`Content-Length`** ヘッダーおよびファイルサイズを検証する。  
      ファイルサイズ検証設定（[設定値:29](#conf29)）が有効であれば、`Content-Length`ヘッダーと実際のファイルサイズを比較し、不一致であればエラーとする。  
      `Content-Length`ヘッダーの値あるいはファイルサイズがアップロードのサイズ上限（[設定値:19](#conf19)）を上回っていればエラーとする。
    - **`Content-Disposition`** ヘッダーを解析する。  
      値が`attachment`かつオプションにファイル名が指定されているかを確認し、満たさない場合はエラーとする。
      リクエストのファイルの有無や実際のファイルと上記のファイル名の合致を確認し、問題があればエラーとする。
    - **`Content-Type`** ヘッダーをもとに送付されたファイルを検証する。  
      ヘッダーの値が`application/zip`でなければ、エラーとする。  
    - **`Packaging`** ヘッダーを検証する。  
      値の末尾が`SWORDBagIt`のとき、`metadata`フォルダ内に`sword.json`ファイルが存在すればSWORDBagIt形式と判定し、なければエラーとする。  
      値の末尾が`SimpleZip`のとき、`ro-crate-metadata.json`ファイルが存在すればRO-Crate+BagIt形式と判定し、なければTSV/CSVあるいはXML形式と判定する。  
      値がその他の場合はエラーとする。
    - **`Digest`** ヘッダーを検証する。  
      メタデータ形式がJSON-LD、かつダイジェスト検証設定（[設定値:28](#conf28)）が有効であるとき、Digestとリクエストボディのハッシュ値が一致しなければエラーとする。

    ※ SWORD APIでは使用可能なエラータイプが定められているため、適切なエラータイプが存在しない場合はBadRequest(エラーコード400)とし、エラードキュメントにエラー原因を記述し返却する。

2. ファイル内容に不備が無いかのチェックを行う

    Zipファイルを展開し、必要なファイルが含まれているか確認する。メタデータファイル形式によって必須事項が異なる。

    **TSV/CSV形式**
    - TSV/CSVファイルが含まれていなければエラーとする。

    **XML形式**
    - XMLファイルが含まれていなければエラーとする。

    **JSON-LD形式**
    - 登録対象のファイルそれぞれのハッシュ値が`manifest-sha256.txt` に記載されている値と一致しなければエラーとなる。

3. 登録の前処理を行う

   メタデータファイル形式がXMLおよびJSON-LDであれば、メタデータのマッピングを行う。

    **XML形式**
    - メタデータのマッピングを行う

    **JSON-LD形式**
    - アクセストークンから、マッピング定義、マッピング先アイテムタイプ、登録方式を取得する  
        マッピング定義またはマッピング先のアイテムタイプが存在しない場合はエラーとする
    - JSONファイルからメタデータを取得し、マッピング定義に基づいてメタデータのマッピングを行う  
        マッピング処理の詳細については、[メタデータマッピング機能](#メタデータマッピング機能)を参照
    - マッピング結果に基づいて、メタデータのバリデートや必須項目のチェックを行い、問題があればエラーとする
    - 登録先の状態やアイテムの公開ステータスのチェックを行い、問題があればエラーとする
    - `On-Behalf-Of`ヘッダーが存在する場合、取得しアイテムのコントリビュータ情報とする
    - データセット登録設定（[設定値:30](#conf30)）が有効であれば、メタデータを含むZIPファイルそのものもアイテムの
      一部として登録するようにメタデータを作成し追加する

4. 登録処理を行う

    メタデータのファイル形式と登録方式によって処理を分岐する。  
    TSV/CSV形式の場合は直接登録、XML形式およびJSON-LD形式の場合はワークフロー登録に固定される。  
    一方、RO-Crate+BagIt形式およびSWORDBagIt形式のZIPファイルの場合は、連携設定から取得した登録方式に従う。

    **TSV/CSV・JSON-LDで直接登録の場合**
    - ZIPによるインポート機能を使用してインポート処理を行う。

    **XML・JSON-LDでワークフロー登録の場合**
    - 新しいアクティビティを作成する。
    - マッピングしたメタデータをもとにDBのテーブル「workflow_activity」のメタデータを更新し、承認前までデータの登録を行う。
    - 承認不要のワークフローの場合はワークフローを最後まで実行し、承認が必要なワークフローの場合は承認の直前まで進める。
    - 必須のメタデータが存在しない場合はエラーとし、どのメタデータが必須かJSON-LD形式で返却する。
    - マッピング先が無いメタデータは不可視のテキストエリアに保存する。


### アイテム状態取得機能：GET /sword/deposit/\<recid\>

- リクエストをチェックする
    - Authorizationヘッダーに記載されたOAuth認証情報を使用しWEKOにログインする
    - On-Behalf-Ofヘッダーが存在する場合、サーバー設定を確認する
- 指定されたrecidからアイテム情報を取得する
- 取得したアイテム情報からステータスドキュメントを生成する
- ステータスドキュメントを返却する

### アイテム削除機能：DELETE /sword/deposit/\<recid\>

- リクエストをチェックする
    - Authorizationヘッダーに記載されたOAuth認証情報を使用しWEKOにログインする
    - On-Behalf-Ofヘッダーが存在する場合、サーバー設定を確認する
- 指定されたrecidを引数にsoft_delete処理を実行する
- 空のレスポンスを返却する

### メタデータマッピング機能
- JSON-LD形式のメタデータを、マッピング定義をもとにWEKO3のアイテムタイプにマッピングする機能を提供する。
- 現時点でRO-Crate+BagIt形式のメタデータのマッピングにのみ対応している。
- この処理にはJSON形式で記述されたマッピング定義と、アイテムタイプのスキーマ定義を使用する。  
  マッピング定義には、JSON-LD形式のメタデータのキーと、WEKO3のアイテムタイプのプロパティ名を対応付ける情報が記述されている。  
  そして、アイテムタイプのスキーマ定義をもとに登録に適した構造のメタデータを構築する。
- アイテムタイプにマッピング定義にないメタデータを保持するプロパティが存在する場合、マッピング先のないメタデータはそこに格納される。

### 処理に関するエトセトラ

- ZIPの展開に使用しているライブラリ：zipfile
- ハッシュ値の計算に使用しているライブラリ：hashlib
- Bagの整合性検証に使用しているライブラリ：[bagit v1.7.0](https://github.com/LibraryOfCongress/bagit-python/tree/v1.7.0)
- アイテムをインポートする際に作成するテンポラリファイルは以下のように生成する

    /tmp/weko_import_YYYYMMDDhhmmss


## エラーメッセージ

## サーバー設定値

1. アプリケーションのデフォルト値

    ```python
    WEKO_SWORDSERVER_DEFAULT_VALUE = "foobar"
    ```

2. デモページのデフォルトの基本テンプレート

    ```python
    WEKO_SWORDSERVER_BASE_TEMPLATE = "weko_swordserver/base.html"
    ```

3. サーバーがサポートするSWORDプロトコルのバージョン<span id="conf03">

    ```python
    WEKO_SWORDSERVER_SWORD_VERSION = "http://purl.org/net/sword/3.0"
    ```

4. サービスの説明<span id="conf04">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ABSTRACT = ""
    ```

5. サーバーが受け入れられるコンテンツタイプのリスト<span id="conf05">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT = ["*/*"]
    ```

6. サーバーが解凍できるアーカイブ形式のリスト<span id="conf06">

    サーバーが異なるフォーマットでパッケージを送信した場合、サーバーはそれをバイナリファイルとして扱うことができる。  
    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_ARCHIVE_FORMAT = ["application/zip"]
    ```

7. ファイルの登録を受け付けるかどうか<span id="conf07">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_DEPOSITS = True
    ```

8. サーバーが受け入れられるメタデータ形式のリスト<span id="conf08">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_METADATA = [
        "https://github.com/JPCOAR/schema/blob/master/2.0/jpcoar_scm.xsd",
        "https://w3id.org/ro/crate/1.1/",
    ]
    ```

9. サーバーで受け入れられるパッケージ形式のリスト<span id="conf09">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_PACKAGING = ["*"]
    """
    ["*"] or List of Packaging Formats URI
    - http://purl.org/net/sword/3.0/package/Binary
    - http://purl.org/net/sword/3.0/package/SimpleZip
    - http://purl.org/net/sword/3.0/package/SWORDBagIt
    """
    ```

10. サーバーの収集ポリシーのURLと説明<span id="conf10">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_COLLECTION_POLICY = {}
    """
    example:
    {
        "@id" : "http://www.myorg.ac.uk/collectionpolicy",
        "description" : "...."
    }
    """
    ```

11. デポジット時に期待される処理のURLと説明を示すオブジェクト<span id="conf11">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_TREATMENT = {}
    ```

12. セSegmented File Upload時のステージング先URL<span id="conf12">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_STAGING = ""
    ```

13. Segmented File Upload時のステージングしたファイルを保持する最小時間<span id="conf13">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_STAGING_MAX_IDLE = 3600
    ```

14. 参照によるデポジットをサポートするか<span id="conf14">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_BY_REFERENCE_DEPOSIT = False
    ```

15. 他のユーザーに代わっての代理投稿をサポートするか<span id="conf15">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ON_BEHALF_OF = True
    ```

16. サーバーが受け入れるダイジェスト形式のリスト<span id="conf16">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_DIGEST = ["SHA-256", "SHA", "MD5"]
    ```

17. サポートする認証方式のリスト<span id="conf17">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_AUTHENTICATION = ["OAuth"]
    ```

18. 親サービスに含まれるサービスのリスト<span id="conf18">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_SERVICES = []
    ```

19. アップロードされるファイルの最大サイズ (整数) (バイト単位)<span id="conf19">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_UPLOAD_SIZE = 16777216000
    ```

20. 参照によってアップロードされたファイルの最大サイズ (バイト単位)<span id="conf20">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_BY_REFERENCE_SIZE = 30000000000000000
    ```

21. Segmented File Uploadの合計サイズの最大サイズ (整数) (バイト単位)<span id="conf21">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_ASSEMBLED_SIZE = 30000000000000
    ```

22. Segmented File Uploadがサポートされている場合、サーバーがひとつのアップロードで受け入れるセグメントの最大数<span id="conf22">

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_SEGMENTS = 1000
    ```

23. 登録方式の列挙型クラス

    ```python
    WEKO_SWORDSERVER_REGISTRATION_TYPE = SwordClientModel.RegistrationType
    ```

    - `Direct` (1): Direct registration.
    - `Workfolw` (2): Workflow registration.

24. RO-Crate+BagItのメタデータファイル名

    ```python
    WEKO_SWORDSERVER_METADATA_FILE_ROCRATE = "ro-crate-metadata.json"
    ```

25. SWORDBagItのメタデータファイル名

    ```python
    WEKO_SWORDSERVER_REQUIRED_FILES_SWORD = "metadata/sword.json"
    ```

26. データセット識別子に付与するプレフィックス

    ```python
    WEKO_SWORDSERVER_DATASET_PREFIX = "weko-"
    ```

27. データセット識別子の置換設定

    ```python
    WEKO_SWORDSERVER_DATASET_ROOT = {
        "": "./",
        "enc": base64.b64encode(f"{WEKO_SWORDSERVER_DATASET_PREFIX}./".encode("utf-8")).decode("utf-8")
    }
    ```

28. クライアントにダイジェストを送信することを要求するか<span id="conf28">

    ```python
    WEKO_SWORDSERVER_DIGEST_VERIFICATION = True
    ```

29. リクエストに Content-Length ヘッダーを要求するか<span id="conf29">

    ```python
    WEKO_SWORDSERVER_CONTENT_LENGTH = False
    ```

30. データセットのzipファイルをアイテムとして登録するか<span id="conf30">

    ```python
    WEKO_SWORDSERVER_DEPOSIT_DATASET = False
    ```


## 更新履歴

| 日付       | GitHubコミットID                           | 更新内容                              |
| ---------- | ------------------------------------------ | ------------------------------------- |
| 2022/06/13 | e6db31c99d459605f5bc09f15c4abd07ea573428   | 初版作成                              |
| 2023/08/31 | 353ba1deb094af5056a58bb40f07596b8e95a562   | ADMIN-2-4へのリンクを追加             |
| 2024/11/26 |                                            | JSON-LD形式のメタデータ登録機能を追加 |
