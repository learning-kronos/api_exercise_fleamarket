## 解答例

**jp.kronos.dao.UserDao.java**

```java
package jp.kronos.dao;

import java.sql.SQLException;

import jp.kronos.DataNotFoundException;
import jp.kronos.DataSource;
import jp.kronos.dto.User;

public class UserDao {
    
    /**
     * 一件検索
     * @param email メールアドレス
     * @param passwd パスワード
     * @return ユーザ情報
     * @throws SQLException
     */
    public User findByEmailAndPassword(String email, String password) throws DataNotFoundException {
        /*
         * TODO
         * DataSourceクラスからユーザ情報（List）を取得し、引数のメールアドレスとパスワードに一致する情報を探す
         * 一致するユーザ情報がある場合：
         * 　一致したユーザ情報を返す
         * 一致するユーザ情報がない場合：
         * 　DataNotFoundExceptionをスローする
         * 　メッセージ「メールアドレスまたはパスワードが正しくありません。」
         */
        for (User user : DataSource.getUsers()) {
            if (email.equals(user.getEmail()) && password.equals(user.getPassword())) {
                return user;
            }
        }
        
        throw new DataNotFoundException("メールアドレスまたはパスワードが正しくありません。");
    }
}
```

<br>

**jp.kronos.dao.ItemDao.java**

```java
package jp.kronos.dao;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

import jp.kronos.DataNotFoundException;
import jp.kronos.DataSource;
import jp.kronos.dto.Item;

public class ItemDao {
    /**
     * 全件検索
     * @return 商品情報（複数）
     */
    public List<Item> findAll() {
        return DataSource.getItems();
    }
    
    /**
     * 商品名による部分一致検索
     * @param keyword 検索キーワード（商品名）
     * @return 商品情報（複数）
     */
    public List<Item> findByKeyword(String keyword) {
        /*
         * TODO
         * DataSourceクラスから商品情報（List）を取得し、商品名に引数のキーワードを含む情報を探す
         * ヒットした商品情報が格納されたListを返す
         */
        List<Item> foundItems = new ArrayList<>();
        for (Item item : DataSource.getItems()) {
            if (item.getName().toUpperCase().contains(keyword.toUpperCase())) {
                foundItems.add(item);
            }
        }
        return foundItems;
    }
    
    /**
     * 一件検索
     * @param id 検索ID
     * @throws DataNotFoundException
     */
    public void findById(int id) throws DataNotFoundException {
        /*
         * TODO
         * 引数のIDをもとに商品情報を取得する
         * データが存在する場合：
         * 　何もしない
         * データが存在しない場合：
         * 　DataNotFoundExceptionをスローする
         * 　メッセージ「指定したIDの商品が見つかりません。」
         */
        List<Item> items = DataSource.getItems();
        for (int i = 0; i < items.size(); i++) {
            if (id == items.get(i).getId()) {
                return;
            }
        }
        
        throw new DataNotFoundException("指定したIDの商品が見つかりません。");
    }
    
    /**
     * 商品登録
     * @param item 登録する商品情報
     */
    public void create(Item item) {
        /*
         * TODO
         * DataSourceクラスから商品情報（List）を取得する
         * 引数の商品情報（インスタンス）に以下の内容をセットし、Listに追加する
         * 　ID　　   : Listの要素数 + 1
         * 　更新日時 : 現在日時
         */
        List<Item> items = DataSource.getItems();
        item.setId(items.size() + 1);
        item.setUpdatedDt(LocalDateTime.now());
        items.add(item);
    }
    
    /**
     * 商品更新
     * @param item 更新する商品情報
     */
    public void update(Item item) {
        /*
         * TODO
         * DataSourceクラスから商品情報（List）を取得し、引数の商品情報のIDに一致する情報を探す
         * 該当する商品のインスタンスに更新情報をセットする
         * 　商品名   : 引数の商品情報の商品名
         * 　価格     : 引数の商品情報の価格
         * 　状態     : 引数の商品情報の状態
         * 　更新日時 : 現在日時
         */
        List<Item> items = DataSource.getItems();
        for (int i = 0; i < items.size(); i++) {
            if (item.getId() == items.get(i).getId()) {
                Item targetItem = items.get(i);
                targetItem.setName(item.getName());
                targetItem.setPrice(item.getPrice());
                targetItem.setState(item.getState());
                targetItem.setUpdatedDt(LocalDateTime.now());
            }
        }
    }

    /**
     * 削除処理
     * @param id 削除する商品のID
     * @throws DataNotFoundException
     */
    public void remove(int id) throws DataNotFoundException {
        /*
         * TODO
         * DataSourceクラスから商品情報（List）を取得し、引数のIDに一致する情報を探す
         * 一致する商品情報がある場合：
         * 　Listから該当する商品情報を削除する
         * 一致する商品情報がない場合：
         * 　DataNotFoundExceptionをスローする
         * 　メッセージ「指定したIDの商品が見つかりません。」
         */
        List<Item> items = DataSource.getItems();
        for (int i = 0; i < items.size(); i++) {
            if (id == items.get(i).getId()) {
                items.remove(i);
                return;
            }
        }
        
        throw new DataNotFoundException("指定したIDの商品が見つかりません。");
    }
}
```

<br>

**jp.kronos.main.UserMain.java**

```java
package jp.kronos.main;

import java.nio.charset.Charset;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

import jp.kronos.DataNotFoundException;
import jp.kronos.dao.ItemDao;
import jp.kronos.dao.UserDao;
import jp.kronos.dto.Item;
import jp.kronos.dto.User;

public class UserMain {
    // ログインユーザ情報
    private static User user = null;

    // 標準入力を取得するクラス（Scanner）
    private static Scanner scan = new Scanner(System.in);

    /**
     * ログイン処理
     */
    private static void login() {
        while (true) {
            System.out.println("【ログイン】");
            System.out.print("メールアドレス：");
            String email = scan.next();
            System.out.print("パスワード：");
            String password = scan.next();

            // ユーザ情報の取得
            UserDao userDao = new UserDao();
            
            try {
                user = userDao.findByEmailAndPassword(email, password);
                System.out.println();
                System.out.println("ようこそ！" + user.getNickname() + "さん");
                System.out.println();
                break;
            } catch (DataNotFoundException e) {
                System.out.println();
                System.err.println(e.getMessage());
                System.out.println();
            }
        }
    }

    /**
     * コンソール表示時の整形用
     * @param target 表示対象の文字列
     * @param length 文字列の長さ
     * @param alignLight 右寄せ
     * @return
     */
    private static String format(String target, int length, boolean alignLight) {
        int byteDiff = (target.getBytes(Charset.forName("UTF-8")).length - target.length()) / 2;
        return String.format("%" + (alignLight ? "" : "-") + (length - byteDiff) + "s", target);
    }

    /**
     * 商品一覧表示
     * @param keyword 検索キーワード
     */
    static void showItems(String keyword) {
        List<Item> items = new ArrayList<>();

        ItemDao itemDao = new ItemDao();
            
        // 商品情報を取得する
        if (keyword == null || keyword.isEmpty()) {
            // 全件検索
            items = itemDao.findAll();
        } else {
            // 曖昧検索（部分一致検索）
            /*
             * TODO
             * ItemDaoのfindByKeywordメソッドで曖昧検索（部分一致検索）をする
             */
            items = itemDao.findByKeyword(keyword);
        }

        System.out.println("【商品一覧】");
        System.out.println(format("ID", 4, true) + " | " +
                format("商品名", 50, false) + " | " +
                format("価格", 8, true) + " | " +
                format("状態", 4, false) + " | " +
                format("更新日", 22, true) + " |");
        System.out.println(
                "-----------------------------------------------------------------------------------------------------");
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
        for (Item item : items) {
            System.out.println(format(item.getId().toString(), 4, true) + " | " +
                    format(item.getName(), 50, false) + " | " +
                    format(item.getPrice().toString(), 8, true) + " | " +
                    format(item.getState(), 4, false) + " | " +
                    format(item.getUpdatedDt().format(formatter), 22, true) + " |");
        }
    }

    /**
     * 出品処理
     */
    static void create() {
        System.out.println();
        System.out.println("【出品】");
        System.out.println("------ 出品する商品情報を入力してください ------");

        System.out.print("商品名：");
        String name = scan.nextLine();
        System.out.print("価格：");
        int price = scan.nextInt();
        System.out.print("状態：");
        String state = scan.next();

        /*
         *  TODO
         *  Itemクラスのインスタンスを生成し、登録内容をセットする
         *  商品名  : 入力した商品名
         *  価格    : 入力した価格
         *  状態    : 入力した状態
         *  出品者ID: ログインユーザのID
         */
        Item item = new Item();
        item.setName(name);
        item.setPrice(price);
        item.setState(state);
        item.setSellerId(user.getId());
        
        /*
         * TODO
         * ItemDAOのcreateメソッドを実行する
         */
        ItemDao itemDao = new ItemDao();
        itemDao.create(item);

        System.out.println();
        System.out.println("商品を登録しました。");
        System.out.println();
    }

    /**
     * 商品削除処理
     */
    static void remove() {
        System.out.println();
        System.out.println("【商品削除】");
        System.out.println("------ 削除するIDを入力してください ------");

        System.out.print("ID：");
        int id = scan.nextInt();

        try {
            /*
             * TODO
             * ItemDaoのremoveメソッドを実行する
             * 正常終了時：
             * 　「商品を削除しました。」
             * DataNotFoundException発生時：
             * 　「指定したIDの商品は見つかりません。」
             */
            ItemDao itemDao = new ItemDao();
            itemDao.remove(id);
        
            System.out.println();
            System.out.println("商品を削除しました。");
            
        } catch (DataNotFoundException e) {
            System.out.println();
            System.err.println(e.getMessage());
        }
        
        System.out.println();
    }
    
    /**
     * 商品更新処理
     */
    static void update() {
        System.out.println();
        System.out.println("【商品更新】");
        System.out.println("------ 更新するIDを入力してください ------");

        System.out.print("ID：");
        int id = scan.nextInt();
        scan.nextLine();  // 残存する改行コードを処理するため

        try {
            /*
             * TODO
             * ItemDaoのfindByIdメソッドを実行して指定したIDに紐づく商品データが存在するかチェックする
             * DataNotFoundException発生時：
             * 　「指定したIDの商品は見つかりません。」
             */
            ItemDao itemDao = new ItemDao();
            itemDao.findById(id);
            
            System.out.println();
            System.out.println("------ 更新内容を入力してください --------");
            System.out.print("商品名：");
            String name = scan.nextLine();
            System.out.print("価格：");
            int price = scan.nextInt();
            System.out.print("状態：");
            String state = scan.next();
    
            /*
             * TODO
             * Itemクラスのインスタンスを生成し、更新内容をセットする
             * ItemDaoのupdateメソッドを実行する
             * 成功終了時：
             *   「商品を更新しました。」
             */
            Item targetItem = new Item();
            targetItem.setId(id);
            targetItem.setName(name);
            targetItem.setPrice(price);
            targetItem.setState(state);
            
            itemDao.update(targetItem);
            System.out.println();
            System.out.println("商品を更新しました。");
            
        } catch (DataNotFoundException e) {
            System.out.println();
            System.out.println(e.getMessage());
        }
            
        System.out.println();
    }

    /**
     * 主処理
     * @param args
     */
    public static void main(String[] args) {

        String keyword = "";

        System.out.println("**********************");
        System.out.println("**  みんなのフリマ  **");
        System.out.println("**********************");

        // ログイン処理
        login();

        loop:
        while (true) {
            // 商品一覧表示
            showItems(keyword);
            keyword = "";

            System.out.println();
            System.out.println("***** メニュー *****");
            System.out.println("1:検索　2:出品　3:削除　4:更新　9:ログアウト");
            int selected = 0;
            selected = scan.nextInt();
            scan.nextLine();  // 残存する改行コードを処理するため

            switch (selected) {
            case 1:
                // 検索処理
                System.out.println();
                System.out.println("-----------------------------------------------");
                System.out.println("検索キーワードで商品名の部分一致検索をします。");
                System.out.println("※未入力の場合は全件検索します。");
                System.out.println("-----------------------------------------------");
                System.out.print("検索キーワード：");
                keyword = scan.nextLine();
                System.out.println();
                // ※keyword値は次のループでshowItems()メソッドに渡される
                
                break;
            case 2:
                // 出品処理
                create();
                break;
            case 3:
                // 商品削除処理
                remove();
                break;
            case 4:
                // 商品更新処理
                update();
                break;
            case 9:
                System.out.println();
                System.out.println("ログアウトしました。");
                break loop;
            default:
                System.out.println();
                System.out.println("入力されたメニュー番号が不正です。");
                System.out.println();
                continue;
            }
        }
    }
}
```
