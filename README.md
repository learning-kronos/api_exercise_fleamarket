# Java API演習 フリマアプリ開発

## 処理概要

コンソール画面で動作するフリマアプリを作成します。次のプロジェクト（Zipファイル）をインポートして開発を進めてください。

[演習用プロジェクト](./src/fleamarket-api.zip)

> Eclipseでのインポート手順<br>①インポート -> フォルダまたはアーカイブからプロジェクト -> [アーカイブ]ボタン -> Zipファイルを選択<br>②フォルダーのリスト内の一番上にある「fleamarket-api.zip_expanded」のチェックを外して[完了]ボタンをクリック

<br>

画面レイアウトなどのベースとなるコードやDTOクラス（Itemクラスなど）は実装済みです。MainクラスやDAOクラスを修正し、以下の機能を実装してください。

<br>

**1. ログイン機能**
```
**********************
**  みんなのフリマ  **
**********************
【ログイン】
メールアドレス：ohagi@example.com
パスワード：ohagi

ようこそ！おはぎさん
```

<br>

**2. 商品一覧表示**　※実装済み
```
【商品一覧】
  ID | 商品名                                             |     価格 | 状態 |                 更新日 |
-----------------------------------------------------------------------------------------------------
   1 | ノートPC KroBook メモリ32GB SSD512GB               |   200000 | A    |  2022-05-10 10:18:32.0 |
   2 | 実践SQL                                            |     1200 | D    |  2022-06-05 14:23:09.0 |
   3 | レンズ 一眼レフカメラ用 50mm F1.4                  |    45000 | A    |  2022-05-15 07:03:54.0 |
   4 | 黒いタブレット SIMフリー 64GB                      |    32000 | B    |  2022-05-19 11:32:11.0 |
   5 | タブレット 9.7インチ シリコンケース                |     3800 | S    |  2022-05-19 16:51:10.0 |
   6 | Javaプログラミング入門 第3版                       |     2800 | B    |  2022-06-10 14:09:49.0 |
   7 | 黒い粉                                             |      300 | S    |  2022-05-25 17:12:44.0 |
   8 | 受講者Xの健診                                      |      900 | A    |  2022-05-30 12:45:37.0 |
   9 | 消せるボールペン                                   |      300 | S    |  2022-06-04 09:51:56.0 |
  10 | ミラーレス一眼カメラ K02X                          |    98000 | A    |  2022-06-08 19:20:54.0 |
  11 | 新！Javaのすべてがわかる本                         |     1500 | C    |  2022-06-08 22:07:08.0 |
  12 | プレミアム万年筆                                   |     5000 | S    |  2022-06-09 15:43:11.0 |
  13 | なぜSQLが使われるのか 第12版                       |     4000 | S    |  2022-06-10 08:12:59.0 |
  14 | ノートパソコン SiroBook メモリ32GB SSD256GB        |   128000 | B    |  2022-06-15 10:50:38.0 |
  15 | MySQL 8.0 マスター                                 |      800 | E    |  2022-06-15 16:20:19.0 |

***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
```

<br>

**3. 検索機能（商品名による部分一致検索）**
```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
1

-----------------------------------------------
検索キーワードで商品名の部分一致検索をします。
※未入力の場合は全件検索します。
-----------------------------------------------
検索キーワード：java

【商品一覧】
  ID | 商品名                                             |     価格 | 状態 |                 更新日 |
-----------------------------------------------------------------------------------------------------
   6 | Javaプログラミング入門 第3版                       |     2800 | B    |  2022-06-10 14:09:49.0 |
  11 | 新！Javaのすべてがわかる本                         |     1500 | C    |  2022-06-08 22:07:08.0 |
```

<br>

**4. 出品機能（商品登録機能）**

```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
2

【出品】
------ 出品する商品情報を入力してください ------
商品名：JDBCのコツが何となくわかる本
価格：2500
状態：A

商品を登録しました。

（商品一覧表示）
```

<br>

**5. 商品削除機能**

```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
3

【商品削除】
------ 削除するIDを入力してください ------
ID：16

商品を削除しました。

（在庫一覧表示）
```

指定したIDが存在しない場合

```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
3

【商品削除】
------ 削除するIDを入力してください ------
ID：999

指定したIDの商品が見つかりません。

（在庫一覧表示）
```

<br>

**6. 商品更新機能**

※更新内容はすべて入力される想定

```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
4

【商品更新】
------ 更新するIDを入力してください ------
ID：16

------ 更新内容を入力してください --------
商品名：JDBCのコツが何となくわかる本
価格：2000
状態：B

商品を更新しました。

（商品一覧表示）
```

指定したIDが存在しない場合

```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
4

【商品更新】
------ 更新するIDを入力してください ------
ID：999

指定したIDの商品が見つかりません。

（商品一覧表示）
```

<br>

**7. ログアウト機能**　※実装済み

```
***** メニュー *****
1:検索　2:出品　3:削除　4:更新　9:ログアウト
9

ログアウトしました。
```

<br>

## システム構成

### データ

フリマアプリで使用するデータ（ユーザ情報、商品情報）は、Listで取り扱います。<br>DataSourceクラスの各メソッド（後述）を使ってデータを取得し、データの加工や更新をしてください。

**ユーザ情報**

| # | 項目名 | データ型 | 備考 |
|:-:|:------|:-------|:---|
| 1 | ユーザID | Integer |  |
| 2 | メールアドレス | String |  |
| 3 | パスワード | String |  |
| 4 | ニックネーム | String |  |
| 5 | 発送元地域 | String | 本演習では使用しない |

**商品情報**

| # | 項目名 | データ型 | 備考 |
|:-:|:------|:-------|:---|
| 1 | 商品ID | Integer |  |
| 2 | 商品名 | String |  |
| 3 | 価格 | Integer |  |
| 4 | 状態 | String |  |
| 5 | 出品者ユーザID | Integer |  |
| 6 | 購入者ユーザID | Integer | 本演習では使用しない |
| 7 | 購入日時 | LocalDateTime | 本演習では使用しない |
| 8 | 更新日時 | LocalDateTime |  |

<br>

### 機能仕様

**ItemDao.java（パッケージ：jp.kronos.dao）**

| 可視性 | メソッド名 | 戻り値 | 引数 | 説明 |
|-------|------------|-------|------|---------|
| public | findAll  | List\<Item>    | なし | 商品データを全件取得する。 |
| public | findByKeyword  | List\<Item>    | String | 商品名による部分一致検索で商品データを取得する。 |
| public | findById | Item | int | 引数のIDに紐付く商品データを取得する。<br>データが存在しない場合はDataNotFoundExceptionをスローする。 |
| public | create   | なし | Item | 引数のItemクラスをもとに商品データを登録する。 |
| public | update   | なし | Item | 引数のItemクラスをもとに商品データを更新する。 |
| public | remove   | なし | int | 引数のIDに紐づく商品データを削除する。<br>データが存在しない場合はDataNotFoundExceptionをスローする。 |

> ソースコード内のコメントの「TODO」をもとに修正してください。

<br>

**UserMain.java（パッケージ：jp.kronos.main）**

| 可視性 | メソッド名 | 戻り値 | 引数 | 説明 |
|-------|------------|-------|------|---------|
| public | login  | なし | なし | ログイン機能 |
| public | showItems  | なし | keyword | 商品一覧表示 / 検索機能 |
| public | create | なし | なし | 出品処理（商品登録処理） |
| public | update   | なし | なし | 商品更新処理 |
| public | remove   | なし | なし | 商品削除処理 |
| public | main   | なし | String[] | 入力されたメニューにより処理を分ける |

> ソースコード内のコメントの「TODO」をもとに修正してください。

<br>

**DataSource.java（パッケージ：jp.kronos）**　※実装済み

| 可視性 | メソッド名 | 戻り値 | 引数 | 説明 |
|-------|------------|-------|------|---------|
| public | getUsers | List\<User> | なし | ユーザ情報を取得する。 |
| public | getItems | List\<Item> | なし | 商品情報を取得する。 |

<br><br><br>

[解答例](./ans/ans.md)

<hr>
<br>

## 改修（オプション）

作成したフリマアプリに対して以下の改修を施してみましょう。

- メニュー番号や価格の入力で整数以外が入力された場合に適切なメッセージを表示する
    - 例）入力されたメニュー番号が不正です。
- 出品時や商品更新時に「S、A、B、C、D、E」以外の状態を入力できないようにする
- 他者が出品した商品は削除できないようにする
- 他者が出品した商品は更新できないようにする
