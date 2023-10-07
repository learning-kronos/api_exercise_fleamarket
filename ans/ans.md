## 解答例

**jp.kronos.dao.ItemDao.java**

```java
package jp.kronos.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import jp.kronos.dto.Item;

public class ItemDao {
    protected Connection con;

    public ItemDao(Connection con) {
        this.con = con;
    }

    /**
     * 全件検索
     * @return 商品情報（複数）
     * @throws SQLException
     */
    public List<Item> findAll() throws SQLException {
        String sql = "SELECT * FROM item";
        try (PreparedStatement ps = con.prepareStatement(sql)) {
            ResultSet rs = ps.executeQuery();
            List<Item> items = new ArrayList<>();
            
            while (rs.next()) {
                Item item = new Item();
                item.setId(rs.getInt("id"));
                item.setName(rs.getString("name"));
                item.setPrice(rs.getInt("price"));
                item.setState(rs.getString("state"));
                item.setUpdatedDt(rs.getTimestamp("updated_dt"));
                items.add(item);
            }
            return items;
        }
    }
    
    /**
     * 商品名による部分一致検索
     * @param keyword 検索キーワード（商品名）
     * @return 商品情報（複数）
     * @throws SQLException
     */
    public List<Item> findByKeyword(String keyword) throws SQLException {
        String sql = "SELECT * FROM item WHERE name LIKE ?";
        try (PreparedStatement ps = con.prepareStatement(sql)) {
            ps.setString(1, "%" + keyword + "%");
            ResultSet rs = ps.executeQuery();
            List<Item> items = new ArrayList<>();
            
            while (rs.next()) {
                Item item = new Item();
                item.setId(rs.getInt("id"));
                item.setName(rs.getString("name"));
                item.setPrice(rs.getInt("price"));
                item.setState(rs.getString("state"));
                item.setUpdatedDt(rs.getTimestamp("updated_dt"));
                items.add(item);
            }
            return items;
        }
    }
    
    /**
     * 一件検索
     * @param id 商品ID
     * @return 商品情報（一件）
     */
    public Item findById(int id) throws SQLException {
        String sql = "SELECT * FROM item WHERE id = ?";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, id);
            ResultSet rs = ps.executeQuery();

            if (rs.next()) {
                Item item = new Item();
                item.setId(rs.getInt("id"));
                item.setName(rs.getString("name"));
                item.setPrice(rs.getInt("price"));
                item.setState(rs.getString("state"));
                item.setSellerId(rs.getInt("seller_id"));
                item.setUpdatedDt(rs.getTimestamp("updated_dt"));
                return item;
            }
            return null;
            
        }
    }
    
    /**
     * 商品登録
     * @param item 商品情報
     */
    public void create(Item item) throws SQLException {
        String sql = "INSERT INTO item (name, price, state, seller_id) "
                +      "VALUES (?, ?, ?, ?)";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setString(1, item.getName());
            ps.setInt(2, item.getPrice());
            ps.setString(3, item.getState());
            ps.setInt(4, item.getSellerId());

            ps.executeUpdate();

        }
    }
    
    /**
     * 商品更新
     * @param item 商品情報
     */
    public void update(Item item) throws SQLException {
        String sql = "UPDATE item SET name = ?, price = ?, state = ? WHERE id = ?";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setString(1, item.getName());
            ps.setInt(2, item.getPrice());
            ps.setString(3, item.getState());
            ps.setInt(4, item.getId());

            ps.executeUpdate();

        }
    }

    /**
     * 商品削除
     * @param id 削除対象ID
     */
    public void delete(int id) throws SQLException {
        String sql = "DELETE FROM item WHERE id = ?";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, id);
            ps.executeUpdate();

        }
    }

}

```

<br>

**jp.kronos.main.UserMain.java**

```java
package jp.kronos.main;

import java.nio.charset.Charset;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.InputMismatchException;
import java.util.List;
import java.util.Scanner;

import jp.kronos.dao.ItemDao;
import jp.kronos.dao.UserDao;
import jp.kronos.dto.Item;
import jp.kronos.dto.User;

public class UserMain {
    // ログインユーザ情報
    private static User user = null;

    // 標準入力を取得するクラス（Scanner）
    private static Scanner scan = new Scanner(System.in);

    // データベース接続情報
    private final static String URL = "jdbc:mysql://localhost:3306/fleamarket?allowPublicKeyRetrieval=true&useSSL=false";
    private final static String USER = "root";
    private final static String PASSWORD = ""; // TODO パスワードの設定

    /**
     * ログイン
     * @throws SQLException
     */
    private static void login() throws SQLException {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            UserDao userDao = new UserDao(con);

            while (true) {
                System.out.println("【ログイン】");
                System.out.print("メールアドレス：");
                String email = scan.next();
                System.out.print("パスワード：");
                String passwd = scan.next();

                // ユーザ情報の取得
                user = userDao.findByEmailAndPassword(email, passwd);
                if (user == null) {
                    System.out.println("\r\nメールアドレスまたはパスワードが正しくありません。\r\n");
                    continue;
                }
                System.out.println("\r\nようこそ！" + user.getNickname() + "さん\r\n");
                break;
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
     * @throws SQLException
     */
    static void showItems(String keyword) throws SQLException {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            List<Item> items = new ArrayList<>();

            ItemDao itemDao = new ItemDao(con);
            
            // 商品情報を取得する
            if (keyword == null || keyword.isEmpty()) {
                // 全件検索
                items = itemDao.findAll();
            } else {
                // 曖昧検索（部分一致検索）
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
            for (Item item : items) {
                System.out.println(format(item.getId().toString(), 4, true) + " | " +
                        format(item.getName(), 50, false) + " | " +
                        format(item.getPrice().toString(), 8, true) + " | " +
                        format(item.getState(), 4, false) + " | " +
                        format(item.getUpdatedDt().toString(), 22, true) + " |");
            }
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
        String name = scan.next();
        int price = 0;
        while (true) {
            try {
                System.out.print("価格：");
                price = scan.nextInt();
                scan.nextLine();  // 残存する改行コードを処理するため
                if (price < 0) {
                    System.out.println("価格は0円以上で入力してください。");
                    continue;
                }
                break;
            } catch (InputMismatchException e) {
                scan.nextLine();  // 残存する改行コードを処理するため
                System.out.println("価格は整数で入力してください。");
            }
            
        }
        System.out.print("状態：");
        String state = scan.next();

        /*
         *  TODO Itemクラスのインスタンスを生成し、登録内容をセットする
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
         * ItemDAOのinsertメソッドを実行する
         * 成功終了時：
         *   「商品を登録しました。」
         * 例外発生時：
         *   「商品の登録に失敗しました。」
         */
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            ItemDao itemDao = new ItemDao(con);
            itemDao.create(item);
            System.out.println("\r\n商品を登録しました。");
        } catch (SQLException e) {
            System.out.println("\r\n商品の登録に失敗しました。");
            System.out.println(e.getMessage());
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

        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            ItemDao itemDao = new ItemDao(con);
            
            /*
             * TODO
             * 指定したIDが存在するかチェックする
             * 存在しない場合はメッセージを表示して更新処理を終了する
             * 「指定したIDの商品は見つかりません。」
             */
            Item oldItem = itemDao.findById(id);
            if (oldItem == null) {
                System.out.println("\r\n指定したIDの商品は見つかりません。");
                System.out.println();
                return;
            }

            System.out.println();
            System.out.println("------ 更新内容を入力してください --------");
            System.out.print("商品名：");
            String name = scan.next();
            System.out.print("価格：");
            int price = scan.nextInt();
            System.out.print("状態：");
            String state = scan.next();
    
            /*
             * TODO
             * Itemクラスのインスタンスを生成し、更新内容をセットする
             * ItemDAOのupdateメソッドを実行する
             * 成功終了時：
             *   「商品を更新しました。」
             * 例外発生時：
             *   「商品の更新に失敗しました。」
             */
            Item targetItem = new Item();
            targetItem.setId(id);
            targetItem.setName(name);
            targetItem.setPrice(price);
            targetItem.setState(state);
            
            itemDao.update(targetItem);
            System.out.println("\r\n商品を更新しました。");
        } catch (SQLException e) {
            System.out.println("\r\n商品の更新に失敗しました。");
            System.out.println(e.getMessage());
        }
            
        System.out.println();
    }

    /**
     * 商品削除処理
     */
    static void delete() {
        System.out.println();
        System.out.println("【商品削除】");
        System.out.println("------ 削除するIDを入力してください ------");

        System.out.print("ID：");
        int id = scan.nextInt();

        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            ItemDao itemDao = new ItemDao(con);
            
            /*
             * TODO
             * ItemDAOのfindByIdメソッドを実行して指定したIDに紐づく商品データが存在するかチェックする
             * 存在しない場合はメッセージを表示して更新処理を終了する
             * 「指定したIDの商品は見つかりません。」
             */
            Item oldItem = itemDao.findById(id);
            if (oldItem == null) {
                System.out.println("\r\n指定したIDの商品は見つかりません。");
                System.out.println();
                return;
            }
    
            /*
             * TODO
             * ItemDAOのupdateメソッドを実行する
             * 成功終了時：
             *   「商品を削除しました。」
             * 例外発生時：
             *   「商品の削除に失敗しました。」
             */
            itemDao.delete(id);
            System.out.println("\r\n商品を削除しました。");
        } catch (SQLException e) {
            System.out.println("\r\n商品の削除に失敗しました。");
            System.out.println(e.getMessage());
        }
        
        System.out.println();
    }

    public static void main(String[] args) {

        String keyword = new String("");

        System.out.println("**********************");
        System.out.println("**  みんなのフリマ  **");
        System.out.println("**********************");

        // ログイン処理
        try {
            login();
        } catch (SQLException e) {
            System.out.println("\r\nシステムエラーが発生しました。");
            System.out.println(e.getMessage());
            return;
        }

        loop:
        while (true) {
            // 商品一覧表示
            try {
                showItems(keyword);
                keyword = "";
            } catch (SQLException e) {
                System.out.println("システムエラーが発生しました。");
                System.out.println(e.getMessage());
                return;
            }

            System.out.println("\r\n***** メニュー *****");
            System.out.println("1:検索　2:出品　3:更新　4:削除　9:ログアウト");
            int selected = 0;
            try {
                selected = scan.nextInt();
            } catch (InputMismatchException e) {
                scan.nextLine();  // 残存する改行コードを処理するため
                System.out.println("\r\n入力された処理番号が不正です。\r\n");
                continue;
            }
            scan.nextLine();

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
                // 商品更新処理
                update();
                break;
            case 4:
                // 商品削除処理
                delete();
                break;
            case 9:
                System.out.println("\r\nログアウトしました。");
                user = null;
                break loop;
            default:
                System.out.println("\r\n入力された処理番号が不正です。\r\n");
                continue;
            }
        }
    }
}

```