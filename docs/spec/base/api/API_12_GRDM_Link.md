# GRDM Link
GakuNin RDMメタデータに対応したアイテム登録更新機能

## 目的・用途

GakuNin RDM(以下、GRDM)から送られてくるJSON-LD形式のメタデータを、SWORDv3プロトコルに従い、WEKO3に登録および更新する。

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

## 処理概要

### アイテム登録機能
#### GRDM側
1. GRDMでインポート画面を表示する際、WEKOのAPIを使用しユーザーの選択可能な登録先を取得する
2. GRDMでインポートするデータを指定し、パッケージ化する
3. WEKOのエンドポイントPOST /sword/service-documentにパッケージを送信する  
  この時、リクエストヘッダにアクセストークンと登録先を付加する

#### WEKO3側
1. リクエストをチェックする
    - Authorizationヘッダーに記載されたアクセストークンを使用しWEKO3にログインする
    - 認証に使用されたOAuthトークンのScopeを確認する
    - On-Behalf-Ofヘッダーが存在する場合、サーバー設定を確認する
    - 送付されたファイルの有無を確認する
    - ファイルサイズを確認する
    - Content-Typeを確認する
    - Packagingを確認する
    - Digestとリクエストボディのハッシュ値が一致するか確認する
    - SWORD APIでは使用可能なエラータイプが定められているため、適切なエラータイプが存在しない場合はBadRequest(エラーコード400)とし、エラードキュメントにエラー原因を記述し返却する

2. ファイル内容に不備が無いかのチェックを行う
    - Zipを展開し、必要なファイルが含まれているか確認する
    - 登録するファイルが`manifest-sha256.txt` に記載されているハッシュ値と一致するか確認する

3. アクセストークンから、マッピング定義、登録先情報を取得する
    - アクセストークンからクライアントIDを取得する
    - クライアントIDから設定画面で事前に指定したマッピング定義、登録方式、アイテムタイプを取得する
    - マッピング定義またはマッピング先のアイテムタイプが存在しない場合はエラーとする

4. マッピング定義に基づいてメタデータのマッピングを行う

5. 登録処理を行う

    **直接登録の場合**
    - zip形式によるインポート機能を使用してインポート処理を行う。

    **ワークフロー登録の場合**
    - 新しいワークフローを作成する。
    - マッピングしたメタデータをもとにDBのテーブル「workflow_activity」のメタデータを更新し、承認前までデータの登録を行う。
    - 承認不要のワークフローの場合はワークフローを最後まで実行する。
    - 必須のメタデータが存在しない場合はエラーとし、どのメタデータが必須かJSON-LD形式で返却する
    - マッピング先が無いメタデータは不可視のテキストエリアに保存する

