# GRDM Link
GakuNin RDMメタデータに対応したアイテム登録更新機能

### 目次
- [目的・用途](#目的用途)
- [機能内容](#機能内容)
- [API仕様](#api仕様)
- [処理概要](#処理概要)
- [エラーメッセージ](#エラーメッセージ)
- [現在の設定値](#現在の設定値)


## 目的・用途

GakuNin RDM(以下、GRDM)から送られてくる、JSON-LD形式のメタデータが付与されたファイルを、SWORDv3プロトコルに従い、WEKO3に登録および更新する。  
GRDMから送られてくるファイルは、RO-Crate+BagIt形式あるいはSWORDBagIt形式のZIPファイルを想定する。
TSVおよび、CSV形式のメタデータを登録する機能は、[API_06_sword_api](./API_06_sword_api.md)を参照。

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
<tr class="even">
<td>
<p>1</p></td>
<td><p>POST /sword/service-document</p>
</td>
<td>
<p>JSON-LD形式のメタデータを、SWORDv3プロトコルに従い、WEKO3にアイテム登録する。</p>
</td>
</tr>
<tr class="odd">
<td>
<p>2</p>
</td>
<td>
<p>PUT /sword/deposit/&lt;recid&gt;</p>
</td>
<td>
<p>recidを指定してリポジトリ上に存在するアイテムに対して、JSON-LD形式のメタデータで更新する。</p>
</td>
</tbody>
</table>

### CURLでのリクエスト実行例：
各APIのリクエスト仕様の詳細は後述。

#### POST /sword/service-document
```
$ curl -X POST -s -k https://192.168.56.101/sword/service-document -F "file=@crate.zip;type=application/zip" -H "Authorization:Bearer Dp85qdLJefoKZ9AuUeIVCqL0Zj9lHxulU1ZSqWGZKI0xJUfxA4wKFnWgztEo" -H "Content-Disposition:attachment; filename=crate.zip" -H "Packaging:http://purl.org/net/sword/3.0/package/SimpleZip -H "On-Behalf-Of:Weko Taro; mail=weko.taro@nii.ac.jp"
```

## 利用可能なロール

| ロール | システム管理者 | リポジトリ管理者 | コミュニティ管理者 | 登録ユーザー | 一般ユーザー | ゲスト(未ログイン) |
| ------ | -------------- | ---------------- | ------------------ | ------------ | ------------ | ------------------ |
|   〇   |       〇       |        〇        |         〇         |      〇      |      〇      |        -           |


## 機能内容

### アイテム登録機能

- リクエストをチェックする
- ファイル内容に不備が無いかのチェックを行う
- マッピング定義に基づいてメタデータのマッピングを行う
- 登録処理を行う


## API仕様

### アイテム登録機能：POST /sword/service-document

- エンドポイント : POST /sword/service-document
- 概要 : RO-Crate+BagIt形式のZIPファイルをWEKO3のアイテムとして登録する。

#### リクエスト

##### ヘッダー

| フィールド          | 必須 | 説明                                                                                                                                                                                                                                                                                   | 例                                                                                                                                           |
| ------------------- | ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Authorization       | ○   | 操作するWEKOユーザーのOAuth認証情報。<br/>“Bearer”+” (半角スペース)”+“アクセストークン”の形式で指定する。                                                                                                                                                                        | “Bearer xxxxxxx”                                                                                                                           |
| On-Behalf-Of        | -    | 代理投稿ユーザーのパーソナルアクセストークンまたはePPNが入る。                                                                                                                                                                                                                         | パーソナルアクセストークン: “aB3dE5Fg7Hj9Kl1Mn2Pq4Rs6Tu8Vw0XyZ1aB3cD5eF7gH9iJ2kL4mN6oP8qR0sT1uV3wX5yZ7”<br>ePPN: "sample@sampleuniv.ac.jp" |
| Content-Disposition | ○   | リクエストボディに付加したファイルのファイル名を指定する。                                                                                                                                                                                                                             | “attachment; filename=example.zip”                                                                                                         |
| Content-Length      | ○   | リクエストボディに付加したファイルサイズを指定する。<br/>※現在は指定していなくてもエラーとならない。                                                                                                                                                                                  | 1024000                                                                                                                                      |
| Content-Type        | ○   | リクエストボディにファイルを付加するため multipart/form-data を指定する。                                                                                                                                                                                                              | multipart/form-data; boundary=xxxxxxxx                                                                                                       |
| Packaging           | ○   | パッケージフォーマットと指定する。<br/>SWORDでは以下の3つのパッケージフォーマットが定義されている。<br/>http://purl.org/net/sword/3.0/package/Binary<br/>http://purl.org/net/sword/3.0/package/SimpleZip<br/>http://purl.org/net/sword/3.0/package/SWORDBagIt<br/>※現在Binaryは未対応 | “http://purl.org/net/sword/3.0/package/SimpleZip”                                                                                          |
| Digest              | △   | リクエストボディに付加したファイルのハッシュ値を指定する。<br/>ダイジェスト検証設定([設定値:17](#conf17))が有効の場合、BugIt形式のファイルを登録するときに必須。                                                                                                                                  | "SHA-256=a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b56c86b3e0aeea5f1"                                                                   |

##### ボディ

| フィールド | 必須 | 説明                                                                                                               | 例                                                          |
| ---------- | ---- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| file       | ○   | form-data 形式でボディにZIPファイルを付加する。<br/>ファイルのContent-Type には“application/zip”を指定すること。 | "file=@project/post_files/example.zip;type=application/zip" |

#### レスポンス

| コード | ドキュメント           | 説明                                                                                                              |
| ------ | ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| 200    | ステータスドキュメント | 登録されたアイテムのステータスドキュメントを返す。                                                                |
| 400    | エラードキュメント     | リクエスト内容に何らかの不備がある場合。                                                                          |
| 401    | エラードキュメント     | リクエストでAuthorization ヘッダーーが提供されない場合。                                                          |
| 403    | エラードキュメント     | 認証に失敗した場合。<br/>認証したOAuthトークンが「deposit:write」スコープを持っていない場合。                     |
| 404    | エラードキュメント     | 登録されたアイテムが見つからない場合。                                                                            |
| 412    | エラードキュメント     | サーバー側がOn-Behalf-Of をサポートしていないにもかかわらず、リクエストでOn-Behalf-Of ヘッダーーが提供された場合。|
| 413    | エラードキュメント     | 送信されたファイルのサイズがサーバーに設定されたmaxUploadSizeを超えている場合。                                   |
| 415    | エラードキュメント     | ヘッダーーまたはボディに付加されたファイルのContent-Typeがサーバー側でサポートされていない場合。                  |
| 500    | エラードキュメント     | サーバー内部エラーが発生した場合。                                                                                |

## ファイル形式

## 処理概要
使用する設定値は[現在の設定値](#現在の設定値)、エラーメッセージは[エラーメッセージ](#エラーメッセージ)を参照。

### アイテム登録機能
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
      ファイルサイズ検証設定（[設定値:20](#conf20)）が有効であれば、`Content-Length`ヘッダーが不正な場合エラーとする。  
      `Content-Length`ヘッダーの値あるいはファイルサイズがアップロードのサイズ上限（[設定値:21](#conf21)）を上回っていればえエラーとする。
    - **`Content-Type`** ヘッダーを検証する。  
      現在の実装では、ヘッダーがなくてもエラーとならない。
    - **`Content-Disposition`** ヘッダーを解析する。  
      値が`attachment`かつオプションにファイル名が指定されているかを確認し、満たさない場合はエラーとする。
    - **`Content-Type`** ヘッダーをもとに送付されたファイルを検証する。  
      ヘッダーの値が`application/zip`でなければ、エラーとする。  
      ファイルの有無や上記のファイル名の合致を確認し、問題があればエラーとする。
    - **`Packaging`** ヘッダーを検証する。  
      メタデータのファイル形式が、TSV/CSV、XML、JSON-LD（RO-Crate+BagIt あるいは SWORDBagIt）のいずれかであることを判定する。
    - メタデータ形式がJSON-LD、かつダイジェスト検証設定（[設定値:17](#conf17)）が有効であれば、Digestとリクエストボディのハッシュ値が一致しなければエラーとする。

    ※ SWORD APIでは使用可能なエラータイプが定められているため、適切なエラータイプが存在しない場合はBadRequest(エラーコード400)とし、エラードキュメントにエラー原因を記述し返却する。

2. ファイル内容に不備が無いかのチェックを行う

    Zipファイルを展開し、必要なファイルが含まれているか確認する。メタデータファイル形式によって必須事項が異なる。

    **TSV/CSV形式**
    - TSV/CSVファイルが含まれていなければエラーとする。

    **XML形式**
    - XMLファイルが含まれていなければエラーとする。

    **JSON-LD形式**
    - JSON-LDファイルが含まれていなければエラーとする。  
        ファイル名は、RO-Crate+BagIt形式の場合は`ro-crate-metadata.json`、SWORDBagIt形式の場合は`metadata/sword.json`とする。
    - 登録対象のファイルそれぞれのハッシュ値が`manifest-sha256.txt` に記載されている値と一致しなければエラーとなる。

3. 登録の前処理を行う

   メタデータファイル形式がXMLおよびJSON-LDであれば、メタデータのマッピングを行う。

    **XML形式**
    - メタデータのマッピングを行う

    **JSON-LD形式**
    - アクセストークンから、マッピング定義、マッピング先アイテムタイプ、登録方式を取得する  
        マッピング定義またはマッピング先のアイテムタイプが存在しない場合はエラーとする
    - マッピング定義に基づいてメタデータのマッピングを行う
    - `On-Behalf-Of`ヘッダーが存在する場合、取得しアイテムのコントリビュータ情報とする。

4. 登録処理を行う

    メタデータのファイル形式と登録方式によって処理を分岐する。  
    TSV/CSV形式の場合は直接登録、XML形式およびJSON-LD形式の場合はワークフロー登録に固定される。  
    一方、RO-Crate+BagIt形式およびSWORDBagIt形式のZIPファイルの場合は、連携設定から取得した登録方式に従う。

    **TSV/CSV・JSON-LDで直接登録の場合**
    - zip形式によるインポート機能を使用してインポート処理を行う。

    **XML・JSON-LDでワークフロー登録の場合**
    - 新しいワークフローを作成する。
    - マッピングしたメタデータをもとにDBのテーブル「workflow_activity」のメタデータを更新し、承認前までデータの登録を行う。
    - 承認不要のワークフローの場合はワークフローを最後まで実行し、承認が必要なワークフローの場合は承認の直前まで進める。
    - 必須のメタデータが存在しない場合はエラーとし、どのメタデータが必須かJSON-LD形式で返却する。
    - マッピング先が無いメタデータは不可視のテキストエリアに保存する。


## エラーメッセージ

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
    WEKO_SWORDSERVER_METADATA_FILE_ROCRATE = ["manifest-sha256.txt", "ro-crate-metadata.json"]
    ```

27. SWORDBagItのメタデータファイル名

    ```python
    WEKO_SWORDSERVER_REQUIRED_FILES_SWORD = ["manifest-sha256.txt", "metadata/sword.json"]
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


---
|    日付    | 更新内容                          |
| ---------- | --------------------------------- |
| 2024/11/21 | 初版作成                          |
