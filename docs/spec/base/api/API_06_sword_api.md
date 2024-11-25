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
- [現在の設定値](#現在の設定値)

## 目的・用途

クライアントからSWORDv3プロトコルに従いリポジトリ上のアイテム操作を実現する。  
アイテムを登録には、TSV/CSV、XML、あるいはJSON-LD形式のメタデータを含むZIPファイルを用いる。

## 利用方法

APIの認証にはOAuth2を利用する。  
アクセストークンの発行は[API-1:OAuth2](./API_01_Oauth2.md#oauth2)を参照。

### Scope：
deposit: write

### エンドポイント：

<table>
<thead>
<tr class="header">
<th>
<p>項番</p>
</th>
<th>
<p>HTTP request</p>
</th>
<th>
<p>内容</p>
</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>
<p>1</p>
</td>
<td>
<p>GET /sword/service-document</p>
</td>
<td>
<p>リポジトリのサービスドキュメントを取得する。</p>
</td>
</tr>
<tr class="even">
<td>
<p>2</p></td>
<td><p>POST /sword/service-document</p>
</td>
<td>
<p>WEKO3の一括登録フォーマットを用いて、アイテムを登録する。</p>
</td>
</tr>
<tr class="odd">
<td>
<p>3</p>
</td>
<td>
<p>GET /sword/deposit/&lt;recid&gt;</p>
</td>
<td>
<p>recidを指定してリポジトリ上に存在するアイテムのステータスドキュメントを取得する。</p>
</td>
</tr>
<tr class="odd">
<td>
<p>4</p>
</td>
<td>
<p>PUT /sword/deposit/&lt;recid&gt;</p>
</td>
<td>
<p>
recidを指定してリポジトリ上に存在するアイテムに対して、JSON-LD形式のメタデータで更新する。<br/>
※現在は未実装
</p>
</td>
</tr>
<tr class="even">
<td>
<p>5</p>
</td>
<td>
<p>DELETE /sword/deposit/&lt;recid&gt;</p>
</td>
<td>
<p>recidを指定してアイテムを削除する。</p>
</td>
</tr>
</tbody>
</table>

### CURLでのリクエスト実行例：

各APIのリクエスト仕様の詳細は後述。

#### GET /sword/service-document

```
$ curl -X GET https://192.168.56.101/sword/service-document -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo"
```

  - -H オプション

    - リクエストにカスタムヘッダーを追加する
    - Authorization は "Bearer" + " (半角スペース)" + "アクセストークン"の形式で指定する


#### POST /sword/service-document
レスポンスの例も示す。

```
$ curl -X POST -s -k https://192.168.56.101/sword/service-document -F "file=@import.zip;type=application/zip" -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo" -H "Content-Disposition:attachment; filename=import.zip" -H "Packaging:http://purl.org/net/sword/3.0/package/SimpleZip" | jq .
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

<table>
<thead>
<tr class="header">
<th>ロール</th>
<th>システム<br />
管理者</th>
<th>リポジトリ<br />
管理者</th>
<th>コミュニティ<br />
管理者</th>
<th>登録ユーザー</th>
<th>一般ユーザー</th>
<th>ゲスト<br />
(未ログイン)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>利用可否</td>
<td>○</td>
<td>○</td>
<td>○</td>
<td>○</td>
<td>○</td>
<td>×</td>
</tr>
</tbody>
</table>

## 機能内容

  - 各APIへのリクエストに応じて処理を実行しレスポンスを返す
      - OAuthアクセストークンによるユーザー認証を必須とする

  - アイテム登録機能で登録に使用するZIPファイルはインポートで使用するものと同様の形式のみ使用できる。
      - TSV/CSV形式のメタデータを含むZIPファイルの詳細は [ADMIN-2-4:インポート](../admin/ADMIN_2_4.md#インポート) を参照

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
| 401    |                      | リクエストでAuthorization ヘッダーが提供されない場合。                                                                |
| 403    |                      | 認証に失敗した場合。                                                                                                  |
| 412    |                      | サーバー側がOn-Behalf-Of をサポートしていないにもかかわらず、<br/>リクエストでOn-Behalf-Of ヘッダーが提供された場合。 |
| 500    |                      | サーバー内部エラーが発生した場合。                                                                                    |


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
| Content-Length      | ※     | リクエストボディに付加したファイルサイズを指定する。<br/>※ファイルサイズ検証設定([設定値:20](#conf20))が有効の場合、必須。                                                                                                                                                            | 1024000                                                                                                    |
| Content-Type        | ○     | リクエストボディにファイルを付加するため "multipart/form-data" を指定する。                                                                                                                                                                                                            | multipart/form-data; boundary=xxxxxxxx                                                                     |
| Packaging           | ○     | パッケージフォーマットと指定する。<br/>SWORDでは以下の3つのパッケージフォーマットが定義されている。<br/>http://purl.org/net/sword/3.0/package/Binary<br/>http://purl.org/net/sword/3.0/package/SimpleZip<br/>http://purl.org/net/sword/3.0/package/SWORDBagIt<br/>※現在Binaryは未対応 | "http://purl.org/net/sword/3.0/package/SimpleZip"                                                          |
| Digest              | ※     | ボディに付加したファイルのハッシュ値を指定する。<br/>※ダイジェスト検証設定([設定値:17](#conf17))が有効の場合、BugIt形式のファイルを登録するときに必須。                                                                                                                               | "SHA-256=e0Pke8qpzEkkGjPE1RoSqNw7qu3tH4..."                                                                |


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

| キー        | 必須 | 説明                    |                      |
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

| キー        | 必須 | 説明                    |                      |
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

<table>
<thead>
<tr class="header">
<th>項目</th>
<th>型</th>
<th>説明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>@context</td>
<td>string</td>
<td>“https://swordapp.github.io/swordv3/swordv3.jsonld”を固定で出力。</td>
</tr>
<tr class="even">
<td>@id</td>
<td>string</td>
<td>"[WEKO3のURL]/sword/service-document"を出力。</td>
</tr>
<tr class="odd">
<td>@type</td>
<td>string</td>
<td>"ServiceDocument"を固定で出力。</td>
</tr>
<tr class="even">
<td>accept</td>
<td>array</td>
<td>サーバーに受け入れられるコンテンツタイプのリスト。<br />
”*/*”を出力する。</td>
</tr>
<tr class="odd">
<td>acceptArchiveFormat</td>
<td>array</td>
<td>サーバーが解凍できるアーカイブ形式のリスト。<br />
現状"application/zip"のみ対応。</td>
</tr>
<tr class="even">
<td>acceptDeposits</td>
<td>boolean</td>
<td>サーバーがデポジットを受け入れるか否か。</td>
</tr>
<tr class="odd">
<td>acceptMetadata</td>
<td>array</td>
<td>サーバーで受け入れ可能なメタデータ形式のリスト。<br />
現状では出力内容にかかわらず、Metadataの受け入れには対応していない。</td>
</tr>
<tr class="even">
<td>acceptPackaging</td>
<td>array</td>
<td>サーバーで受け入れ可能なパッケージ形式のリスト。<br />
現状すべての形式を受け入れるが、アイテム登録はWEKOの一括登録用ZIP形式でのみ可能。</td>
</tr>
<tr class="odd">
<td>authentication</td>
<td>Array</td>
<td>サーバーでサポートされている認証スキームのリスト。<br />
現状”OAuth”のみ対応。</td>
</tr>
<tr class="even">
<td>byReferenceDeposit</td>
<td>boolean</td>
<td>サーバーがbyReferenceDepositをサポートしているか否か。現状未対応のためFalseを出力。</td>
</tr>
<tr class="odd">
<td>collectionPolicy</td>
<td>object</td>
<td>コレクションポリシーを示すオブジェクト。</td>
</tr>
<tr class="even">
<td>collectionPolicy.@id</td>
<td>string</td>
<td>コレクションポリシーのURL。</td>
</tr>
<tr class="odd">
<td>collectionPolicy.description</td>
<td>string</td>
<td>コレクションポリシーの説明。</td>
</tr>
<tr class="even">
<td>dc:title</td>
<td>string</td>
<td>リポジトリの名称を出力。</td>
</tr>
<tr class="odd">
<td>dcterms:abstract</td>
<td>string</td>
<td>リポジトリの説明。</td>
</tr>
<tr class="even">
<td>digest</td>
<td>array</td>
<td>サーバーが受け入れるdigest形式のリスト。<br />
現状digestの検証は未対応。</td>
</tr>
<tr class="odd">
<td>maxAssembledSize</td>
<td>integer</td>
<td>Segmented File Upload時のファイル合計最大サイズ（単位：byte）。</td>
</tr>
<tr class="even">
<td>maxByReferenceSize</td>
<td>integer</td>
<td>By-Reference Deposit時のファイル最大サイズ（単位：byte）。</td>
</tr>
<tr class="odd">
<td>maxSegmentSize</td>
<td>integer</td>
<td>Segmented File Upload時の１ファイルの最大サイズ（単位：byte）。</td>
</tr>
<tr class="even">
<td>maxSegments</td>
<td>integer</td>
<td>Segmented File Upload時のセグメントの最大数。</td>
</tr>
<tr class="odd">
<td>maxUploadSize</td>
<td>integer</td>
<td>アップロードされるファイルの最大サイズ（単位：byte）。</td>
</tr>
<tr class="even">
<td>minSegmentSize</td>
<td>integer</td>
<td>Segmented File Upload時の１ファイルの最小サイズ（単位：byte）。</td>
</tr>
<tr class="odd">
<td>onBehalfOf</td>
<td>boolean</td>
<td>代理投稿をサポートしているか否か。<br />
現状未対応のためfalseを出力。</td>
</tr>
<tr class="even">
<td>root</td>
<td>string</td>
<td>サービスドキュメントのルートURL。</td>
</tr>
<tr class="odd">
<td>services</td>
<td>array</td>
<td>親サービスに含まれるサービスのリスト。<br />
現状未対応。</td>
</tr>
<tr class="even">
<td>staging</td>
<td>string</td>
<td>Segmented File Upload時にコンテンツをステージング先URL。現状未対応のため空文字を出力。</td>
</tr>
<tr class="odd">
<td>stagingMaxIdle</td>
<td>integer</td>
<td>ステージングされたファイルの最小保持時間。</td>
</tr>
<tr class="even">
<td>treatment</td>
<td>object</td>
<td>デポジット時に期待される処理のURLと説明を示すオブジェクト。</td>
</tr>
<tr class="odd">
<td>treatment.@id</td>
<td>string</td>
<td>処理のURL。</td>
</tr>
<tr class="even">
<td>treatment.description</td>
<td>string</td>
<td>処理の説明。</td>
</tr>
<tr class="odd">
<td>version</td>
<td>string</td>
<td>サポートしているSWORDバージョン。<br />
"http://purl.org/net/sword/3.0"を出力。</td>
</tr>
</tbody>
</table>

### ステータスドキュメント
アイテムの内容と現在の状態に関する詳細情報を示すドキュメント

<table>
<thead>
<tr class="header">
<th>項目</th>
<th>型</th>
<th>説明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>@context</td>
<td>string</td>
<td>“https://swordapp.github.io/swordv3/swordv3.jsonld”を固定で出力。</td>
</tr>
<tr class="even">
<td>@id</td>
<td>string</td>
<td>"[WEKO3のURL]/sword/deposit/[アイテムのrecid]"を出力。</td>
</tr>
<tr class="odd">
<td>@type</td>
<td>string</td>
<td>"ServiceDocument"を固定で出力。</td>
</tr>
<tr class="even">
<td>actions</td>
<td>object</td>
<td>アイテムに対してSWORDで使用可能なアクション。<br />
現時点では deleteObject のみTrueを返し、それ以外はFalseを返すようになっている。</td>
</tr>
<tr class="odd">
<td>actions. appendFiles</td>
<td>boolean</td>
<td>ファイル追加要求が発行可能か否か。</td>
</tr>
<tr class="even">
<td>actions.appendMetadata</td>
<td>boolean</td>
<td>メタデータ追加要求が発行可能か否か。</td>
</tr>
<tr class="odd">
<td>actions. deleteFiles</td>
<td>boolean</td>
<td>ファイル削除要求が発行可能か否か。</td>
</tr>
<tr class="even">
<td>actions. deleteMetadata</td>
<td>boolean</td>
<td>メタデータ削除要求が発行可能か否か。</td>
</tr>
<tr class="odd">
<td>actions. deleteObject</td>
<td>boolean</td>
<td>アイテム削除要求が発行可能か否か。</td>
</tr>
<tr class="even">
<td>actions. getFiles</td>
<td>boolean</td>
<td>ファイル取得要求が発行可能か否か。</td>
</tr>
<tr class="odd">
<td>actions. getMetadata</td>
<td>boolean</td>
<td>メタデータ取得要求が発行可能か否か。</td>
</tr>
<tr class="even">
<td>actions. replaceFiles</td>
<td>boolean</td>
<td>ファイル置き換え要求が発行可能か否か。</td>
</tr>
<tr class="odd">
<td>actions. replaceMetadata</td>
<td>boolean</td>
<td>メタデータ置き換え要求が発行可能か否か。</td>
</tr>
<tr class="even">
<td>eTag</td>
<td>string</td>
<td>アイテムのeTag。<br />
WEKOではアイテムのリビジョン番号を返す。</td>
</tr>
<tr class="odd">
<td>fileSet</td>
<td>object</td>
<td>ファイルセットを示すオブジェクト。<br />
現時点では空オブジェクトを返す。</td>
</tr>
<tr class="even">
<td>fileSet.@id</td>
<td>string</td>
<td>ファイルセットのURL。</td>
</tr>
<tr class="odd">
<td>fileSet.eTag</td>
<td>string</td>
<td>ファイルセットのeTag。</td>
</tr>
<tr class="even">
<td>links</td>
<td>array</td>
<td>アイテムのリンクを示すオブジェクト。<br />
現時点ではアイテム詳細ページのURLを出力する。またDOIやCNRIハンドルを持つ場合も同様に出力する。</td>
</tr>
<tr class="odd">
<td>links[].@id</td>
<td>string</td>
<td>リソースのURL。</td>
</tr>
<tr class="even">
<td>links[].byReference</td>
<td>string</td>
<td>byReference deposit の際の参照元URL。</td>
</tr>
<tr class="odd">
<td>links[].contentType</td>
<td>string</td>
<td>リソースのコンテンツタイプ。</td>
</tr>
<tr class="even">
<td>links[].dcterms:isReplacedBy</td>
<td>string</td>
<td>同じオブジェクト内のファイルの新しいバージョンへのURL。</td>
</tr>
<tr class="odd">
<td>links[].dcterms:relation</td>
<td>string</td>
<td>非SWORDアクセスポイントへのURL。</td>
</tr>
<tr class="even">
<td>links[].dcterms:replaces</td>
<td>string</td>
<td>同じオブジェクト内の古いバージョンのファイルへのURL。</td>
</tr>
<tr class="odd">
<td>links[].depositedBy</td>
<td>string</td>
<td>アイテム登録を行ったユーザーの識別子。</td>
</tr>
<tr class="even">
<td>links[].depositedOn</td>
<td>string</td>
<td>アイテム登録日時のタイムスタンプ。</td>
</tr>
<tr class="odd">
<td>links[].depositedOnBehalfOf</td>
<td>string</td>
<td>代理投稿により登録を行ったユーザーの識別子。</td>
</tr>
<tr class="even">
<td>links[].derivedFrom</td>
<td>string</td>
<td>現在のリソースが派生したリソースのURLへの参照。</td>
</tr>
<tr class="odd">
<td>links[].eTag</td>
<td>string</td>
<td>リソースのeTag。</td>
</tr>
<tr class="even">
<td>links[].log</td>
<td>string</td>
<td>クライアントが知っておくべきデポジットに関連する情報。</td>
</tr>
<tr class="odd">
<td>links[].packaging</td>
<td>string</td>
<td>リソースがパッケージである場合、パッケージ形式の識別子を示す。</td>
</tr>
<tr class="even">
<td>links[].rel</td>
<td>string</td>
<td><p>リソースとオブジェクトの関係。<br />
以下の何れかの文字列を持つ。</p>
<ul>
<li>
<p>alternate</p>
</li>
<li>
<p>packaging</p>
</li>
<li>
<p>depositedOn</p>
</li>
<li><p>depositedOnBehalfOf</p></li>
<li><p>status</p></li>
<li><p>log</p></li>
<li><p>dcterms:relation</p></li>
<li><p>dcterms:replaces</p></li>
<li><p>dcterms:isReplacedBy</p></li>
<li><p>versionReplaced</p></li>
<li><p>eTag</p></li>
<li><p>byReference</p></li>
<li><p>derivedFrom</p></li>
<li><p>metadataFormat</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>links[].status</td>
<td>string</td>
<td>取り込みに関するリソースのステータス。</td>
</tr>
<tr class="even">
<td>links[].versionReplacedOn</td>
<td>string</td>
<td>現在のリソースが新しいリソースに置き換えられた日付。</td>
</tr>
<tr class="odd">
<td>metadata</td>
<td>object</td>
<td>メタデータを示すオブジェクト。<br />
現時点では空オブジェクトを返す。</td>
</tr>
<tr class="even">
<td>metadata.@id</td>
<td>string</td>
<td>メタデータのURL。</td>
</tr>
<tr class="odd">
<td>metadata.eTag</td>
<td>string</td>
<td>メタデータのeTag。</td>
</tr>
<tr class="even">
<td>service</td>
<td>string</td>
<td>サービスドキュメントのURL。</td>
</tr>
<tr class="odd">
<td>state</td>
<td>array</td>
<td>アイテムがサーバー上にある状態のリスト。</td>
</tr>
<tr class="even">
<td>state[].@id</td>
<td>string</td>
<td>状態の識別子。<br />
現状では"http://purl.org/net/sword/3.0/state/ingested"を固定で出力。</td>
</tr>
<tr class="odd">
<td>state[].description</td>
<td>string</td>
<td>状態の説明</td>
</tr>
</tbody>
</table>

### エラードキュメント
エラー内容を表すドキュメント

<table>
<thead>
<tr class="header">
<th>項目</th>
<th>型</th>
<th>説明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>@context</td>
<td>string</td>
<td>“https://swordapp.github.io/swordv3/swordv3.jsonld”を固定で出力。</td>
</tr>
<tr class="even">
<td>@type</td>
<td>string</td>
<td>エラータイプを示す文字列。<br />
4.3.3エラータイプ を参照。</td>
</tr>
<tr class="odd">
<td>error</td>
<td>string</td>
<td>エラー内容の説明。</td>
</tr>
<tr class="even">
<td>log</td>
<td>string</td>
<td>より詳細なエラー内容。<br />
現在は出力していない。</td>
</tr>
<tr class="odd">
<td>timestamp</td>
<td>string</td>
<td>エラー発生時のタイムスタンプ。</td>
</tr>
</tbody>
</table>

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
| InvalidSegmentSize           | 400    | セグメントアップロード時のファイルサイズが範囲外。                                           |
| MaxAssembledSizeExceeded     | 400    | セグメントアップロード時の合計ファイルサイズが最大値を超えている。                           |
| MaxUploadSizeExceeded        | 413    | アップロードされたコンテンツサイズが最大値を超えている                                       |
| MetadataFormatNotAcceptable  | 415    | サーバーがサポートしていない形式のMetadata-Formatがリクエストされた。                        |
| MethodNotAllowed             | 405    | メソッドへのアクセスが許可されていない。                                                     |
| OnBehalfOfNotAllowed         | 412    | サーバーが On-Behalf-Of をサポートしていない。                                               |
| PackagingFormatNotAcceptable | 415    | サーバーがサポートしていない形式のPackagingフォーマットがリクエストされた。                  |
| SegmentedUploadTimedOut      | 410    | セグメントアップロード先のURLにアクセスできない。                                            |
| SegmentLimitExceeded         | 400    | セグメント数が最大値を超えている。                                                           |
| UnexpectedSegment            | 400    | サーバーが予期していないセグメントを受信した。                                               |

## 関連モジュール


  - invenio\_oauth2server：OAuthトークンによるユーザー認証を行う

  - invenio\_deposit：OAuthトークンが参照するデポジット操作スコープを定義している

  - weko_swordserver：リクエストの処理を行う

  - weko\_records\_ui：レコード情報の取得、アイテムの削除を実行する

  - weko\_search\_ui：インポート処理を実行する


## 処理概要

### サービスドキュメント取得機能：GET /sword/service-document

- リクエストをチェックする
    - Authorizationヘッダーに記載されたOAuth認証情報を使用しWEKOにログインする
    - On-Behalf-Ofヘッダーが存在する場合、サーバー設定を確認する
- サーバー設定値を参照し、サービスドキュメントを生成する
- サービスドキュメントを返却する

### アイテム登録機能：POST /sword/service-document

- リクエストをチェックする
    - Authorizationヘッダーに記載されたOAuth認証情報を使用しWEKOにログインする
    - 認証に使用されたOAuthトークンのScopeを確認する
    - On-Behalf-Ofヘッダーが存在する場合、サーバー設定を確認する
    - 送付されたファイルの有無を確認する
    - ファイルサイズを確認する
    - Content-Typeを確認する
    - Packagingを確認する
- ファイル内容に不備が無いかのチェックを行う
- ファイル内のアイテムが新規登録か否かを確認する
- インポート処理を行う
- 登録したアイテムのステータスドキュメントを生成する
    - インポート処理から返されたrecidからアイテム情報を取得する
    - 取得したアイテム情報からステータスドキュメントを生成する
- ステータスドキュメントを返却する

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
- 指定されたrecidを引数にsoft\_delete処理を実行する
- 空のレスポンスを返却する

### 処理に関するエトセトラ

- zipファイルの展開に使用しているライブラリ：zipfile
- ハッシュ値の計算に使用しているライブラリ：hashlib
- Bagの整合性検証に使用しているライブラリ：[bagit](https://github.com/LibraryOfCongress/bagit-python/tree/v1.7.0)
- アイテムをインポートする際に作成するテンポラリファイルは以下のように生成する

    /home/invenio/.virtualenvs/invenio/var/instance/data/tmp/weko_import_YYYYMMDDhhmmss


## 現在の設定値

1. アプリケーションのデフォルト値

    ```python
    WEKO_SWORDSERVER_DEFAULT_VALUE = "foobar"
    ```

2. デモページのデフォルトの基本テンプレート

    ```python
    WEKO_SWORDSERVER_BASE_TEMPLATE = "weko_swordserver/base.html"
    ```

3. サーバーがサポートするSWORDプロトコルのバージョン

    ```python
    WEKO_SWORDSERVER_SWORD_VERSION = "http://purl.org/net/sword/3.0"
    ```

4. サービスの説明

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ABSTRACT = ""
    ```

5. サーバーが受け入れられるコンテンツタイプのリスト

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT = ["*/*"]
    ```

6. サーバーが解凍できるアーカイブ形式のリスト

    サーバーが異なるフォーマットでパッケージを送信した場合、サーバーはそれをバイナリファイルとして扱うことができる。  
    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_ARCHIVE_FORMAT = ["application/zip"]
    ```

7. ファイルの登録を受け付けるかどうか

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_DEPOSITS = True
    ```

8. サーバーが受け入れられるメタデータ形式のリスト

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_METADATA = []
    ```

9. サーバーで受け入れられるパッケージ形式のリスト

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ACCEPT_PACKAGING = ["*"]
    ```

    ["*"] or List of Packaging Formats URI
    - http://purl.org/net/sword/3.0/package/Binary
    - http://purl.org/net/sword/3.0/package/SimpleZip
    - http://purl.org/net/sword/3.0/package/SWORDBagIt

10. サーバーの収集ポリシーのURLと説明

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_COLLECTION_POLICY = {}
    ```

11. 登録時に期待できる処理内容のURLと説明

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_TREATMENT = {}
    ```

12. セグメント化されたアップロードの場合、クライアントが預け入れ前にコンテンツをステージアップできるURL

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_STAGING = ""
    ```

13. 最後にコンテンツを受信して​​から、サーバーが不完全な分割ファイルのアップロードを削除するまで保持する最小時間

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_STAGING_MAX_IDLE = 3600
    ```

14. 参照によるデポジットをサポートするか

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_BY_REFERENCE_DEPOSIT = False
    ```

15. 他のユーザーに代わっての登録(仲介)をサポートするか<span id="conf15"></span>

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_ON_BEHALF_OF = True
    ```

16. サーバーが受け入れるダイジェスト形式のリスト

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_DIGEST = ["SHA-256", "SHA", "MD5"]
    ```

17. クライアントにダイジェストを送信することを要求するか<span id="conf17"></span>

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_DIGEST_VERIFICATION = True
    ```

18. サポートする認証方式のリスト

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_AUTHENTICATION = ["OAuth"]
    ```

19. 親サービスに含まれるサービスのリスト

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_SERVICES = []
    ```

20. リクエストに Content-Length ヘッダーを要求するか<span id="conf20"></span>

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_CONTENT_LENGTH = False
    ```

21. セグメント化アップロードの合計サイズの最大サイズ (整数) (バイト単位)<span id="conf21"></span>

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_UPLOAD_SIZE = 16777216000
    ```

22. 参照によってアップロードされたファイルの最大サイズ (バイト単位)

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_BY_REFERENCE_SIZE = 30000000000000000
    ```

23. アップロードされるファイルの最大サイズ (整数) (バイト単位)

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_ASSEMBLED_SIZE = 30000000000000
    ```

24. セグメント化されたアップロードがサポートされている場合、サーバーが単一のセグメント化されたアップロードで受け入れるセグメントの最大数

    ```python
    WEKO_SWORDSERVER_SERVICEDOCUMENT_MAX_SEGMENTS = 1000
    ```

25. 登録方式の列挙型クラス

    ```python
    WEKO_SWORDSERVER_REGISTRATION_TYPE = SwordClientModel.RegistrationType
    ```

    - `Direct` (1): Direct registration.
    - `Workfolw` (2): Workflow registration.

26. RO-Crate+BagItのメタデータファイル名

    ```python
    WEKO_SWORDSERVER_METADATA_FILE_ROCRATE = "ro-crate-metadata.json"
    ```

27. SWORDBagItのメタデータファイル名

    ```python
    WEKO_SWORDSERVER_REQUIRED_FILES_SWORD = "metadata/sword.json"
    ```

28. データセット識別子に付与するプレフィックス

    ```python
    WEKO_SWORDSERVER_DATASET_PREFIX = "weko-"
    ```

29. データセット識別子の置換設定

    ```python
    WEKO_SWORDSERVER_DATASET_IDENTIFIER = {
        "": "./",
        "enc": base64.b64encode(f"{WEKO_SWORDSERVER_DATASET_PREFIX}./".encode("utf-8")).decode("utf-8")
    }
    ```



## 更新履歴

<table>
<thead>
<tr class="header">
<th>日付</th>
<th>GitHubコミットID</th>
<th>更新内容</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>
<p>2022/06/13</p>
</td>
<td>e6db31c99d459605f5bc09f15c4abd07ea573428</td>
<td>初版作成</td>
</tr>
<tr class="even">
<td>
<p>2023/08/31</p>
</td>
<td>353ba1deb094af5056a58bb40f07596b8e95a562</td>
<td>ADMIN-2-4へのリンクを追加</td>
</tr>
</tbody>
</table>
