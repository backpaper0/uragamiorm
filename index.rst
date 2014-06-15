JavaのORM、Domaの話 +α
===========================

.. revealjs:: JavaのORM、Domaの話 +α
   :subtitle: @backpaper0

   .. rv_small::

      2014-06-14 `#uragamiorm <http://connpass.com/event/3583/>`_

.. revealjs:: @backpaper0とは

   * うらがみです。
   * ｴｽｱｲﾔｰでｼﾞｬﾊﾞ使ってｳｪｯﾌﾞやってます。
   * ｱﾝﾖﾖｲﾖも少しやってます。仕事で。
   * Doma歴4年ぐらい。

.. revealjs:: +αの話から

.. revealjs:: みなさんORM使っていますか？
   :title-heading: h3

.. revealjs:: backpaper0が知ってるORM

   * JPA (EclipseLink, Hibernate)
   * S2JDBC
   * S2Dao
   * ActiveObjects
   * Iciql (JaQu)
   * MyBatis (旧iBatis)
   * Mirage
   * Doma

.. revealjs::

   .. revealjs:: JPA

      * Java EEに含まれるORMフレームワーク。
      * オブジェクトとリレーションの柔軟なマッピング。
      * JPQLでSQLの方言を吸収。
      * 生SQLも書ける。
      * Criteria API + Metamodel API = 型安全なクエリ。
        ただしコード地獄になりがち。
        ていうかなる。
        クエリを再利用できるメリットある？
      * 少数精鋭向け？
      * QueryDSLを併用すれば幸せになれる？

   .. revealjs:: JPAのエンティティ

      .. rv_code::

         @Entity
         public class Book {

             @Id
             public String isbn;

             public String title;

             ...

   .. revealjs:: JPAのクエリ

      .. rv_code::

         Book book = em.createQuery("SELECT b.* FROM Book b WHERE b.isbn = :isbn", Book.class)
                       .setParameter("isbn", "978-4-488-10118-3")
                       .getSingleResult();

   .. revealjs:: Metamodel API

      .. rv_code::

         @StaticMetamodel(Book.class)
         public class Book_ {

             public static SingularAttribute< Book, String> isbn;

             public static SingularAttribute< Book, String> title;

             public static SingularAttribute< Book, String> author;
         }

   .. revealjs:: Criteria API

      .. rv_code::

         CriteriaBuilder builder = em.getCriteriaBuilder();
         CriteriaQuery< Book> criteria = builder.createQuery(Book.class);
         Root< Book> from = criteria.from(Book.class);
         criteria.select(from);
         criteria.where(builder.equal(from.get(Book_.isbn), "978-4-488-10118-3"));
         Book book = em.createQuery(criteria).getSingleResult();

.. revealjs::

   .. revealjs:: S2JDBC

      * Seasar2に含まれるORM。
      * JPAのアノテーションを利用する。
      * `@ManyToMany` には対応していない。
      * Namesクラスで型安全なクエリ。
      * 生SQLも書ける。
      * Seasar2を利用しない場合、設定(というかJdbcManagerインスタンスのセットアップ)がめんどい。
      * JTA必須っぽい。

   .. revealjs:: S2JDBCのエンティティ

      .. rv_code::

         @Entity
         public class Book {

             @Id
             public String isbn;

             public String title;

             ...

   .. revealjs:: S2JDBCのクエリ

      .. rv_code::

         Book book = jdbcManager.from(Book.class)
                                .where(eq(isbn(), "978-4-488-10118-3"))
                                .getSingleResult();

.. revealjs::

   .. revealjs:: S2Dao

      * Daoインターフェースを用意して実装は動的に生成する。
      * Seasar2必須と思う。
      * getBookByAuthorPublisherという風にメソッド名をもとにクエリを組み立てる。
      * SQLファイルを使用したりアノテーションにクエリ書いたりもできるっぽい。

.. revealjs::

   .. revealjs:: ActiveObjects

      * エンティティはインターフェースでEntityインターフェースをextendsする。
      * アクセサっぽいメソッドを定義する。
      * ダイナミックプロキシで実装を生成している。
      * **2009年頃から更新されていないプロジェクトなので使ってはいけない**

   .. revealjs:: ActiveObjectsのエンティティ

      .. rv_code::

         public interface Book extends Entity {

             String getIsbn();

             void setIsbn(String isbn);

             ...


   .. revealjs:: ActiveObjectsの永続化

      .. rv_code::

         net.java.ao.EntityManager em = ...
         Book book = em.create(Book.class);
         book.setIsbn("978-4-488-10118-3");
         book.save();

.. revealjs::

   .. revealjs:: Iciql

      * H2Databaseに付属のJaQuというORMが元になっている
      * 言葉では説明しづらい変わった方法でクエリを組み立てる

   .. revealjs:: Iciqlのエンティティ

      .. rv_code::

         public class Book {

             @IQColumn(primaryKey = true)
             public String isbn;

             public String title;

             ...

   .. revealjs:: Iciqlのクエリ

      .. rv_code::

         Book b = new Book();
         Book book = db.from(b)
                       .where(b.isbn).is("978-4-488-10118-3")
                       .selectFirst();

   .. revealjs:: Iciqlで結合

      .. rv_code::

         Book b = new Book();
         Author a = new Author();
         List< BookView> books = db.from(b)
                 .innerJoin(a)
                 .on(a.id).is(b.authorId)
                 .select(new BookView() {{
                     title = b.title;
                     author = a.name;
                 }});

   .. revealjs:: Iciqlのもうひとつのクエリ

      whereメソッドに渡したFilterの匿名サブクラスのバイトコードを解析してクエリを組み立てる。

      .. rv_code::

         List< Book> books = db.from(b).where(new Filter() {

             @Override
             public boolean where() {
                 return b.isbn.equals("978-4-488-10118-3");
             }
         }).select();

 
.. revealjs::

   .. revealjs:: MyBatis

      * こざけさんが説明してくれるはず。

.. revealjs::

   .. revealjs:: Mirage

      * `GitBucket <https://github.com/takezoe/gitbucket>`_ の `@takezoen <https://twitter.com/takezoen>`_ さんが作成されているORM。
      * S2JDBCを手軽にした感じ？
      * でもタイプセーフクエリは無い。
      * mirage-scalaというのもある。

.. revealjs:: Doma

.. revealjs:: Domaとは

   * JavaのORM。
   * Object ResultSet Mapper (※個人の感想です)。
   * `Pluggable Annotation Processing API <https://www.jcp.org/en/jsr/detail?id=269>`_ を使用している。
   * その仕組み上、Scalaなど他の言語で書くことはできない。
   * 特定のアノテーションを付けたクラスやインターフェースをもとに補助クラスや実装クラスをコンパイル時にモリモリ生成

.. revealjs:: エンティティ

   .. rv_code::

      @Entity
      public class Book {

          @Id
          public String isbn;

          public String title;

          public String author;

          ...

.. revealjs:: Daoインターフェース

   .. rv_code::

      @Dao(config = MyConfig.class)
      public interface BookDao {

          @Select
          List< Book> select(String title, String author);

.. revealjs:: SQLファイル

   META-INF/app/dao/BookDao/select.sql

   .. rv_code::

      SELECT /*%expand*/*
        FROM book
       WHERE title = /* title */'x'
         /*%if author != null */
         AND author = /* author */'y'
         /*%end*/

.. revealjs:: コンパイル時に色々検出

   * @Selectを付けたメソッドに対応するSQLファイルがないと **コンパイルエラー**
   * Daoクラスのメソッドに@Selectや@Insertなどのアノテーションが付いていないと **コンパイルエラー**
   * SQLファイルの中身が空っぽだと **コンパイルエラー**
   * メソッドの引数がSQLファイル内で使用されていないと **コンパイルエラー**
   * SQLファイル内の `/\*%if ...\*/` や `/\*%end\*/` が変な位置にあると **コンパイルエラー**

.. revealjs:: ドメインクラス

   エンティティのフィールドやDaoのメソッドの引数、戻り値にStringなどの基本型ではなくてユーザー定義のクラスを利用できる仕組み。

   .. rv_code::

      @Domain(valueType = String.class, factoryMethod = "of")
      public class Isbn {

          private final String value;
      
          private Isbn(String value) {
              this.value = value;
          }
      
          public String getValue() {
              return value;
          }

          public static Isbn of(String value) {
              return Optional.ofNullable(value).map(Isbn::new).orElse(null);
          }
      }

.. revealjs:: エンティティ + ドメインクラス
   :title-heading: h3

   .. rv_code::

      @Entity
      public class Book {

          @Id
          public Isbn isbn;

          public Title title;

          public Author author;

          ...

.. revealjs:: Dao + ドメインクラス
   :title-heading: h3

   .. rv_code::

      @Dao(config = MyConfig.class)
      public interface BookDao {

          @Select
          List< Book> select(Title title, Author author);

          @Select
          Title selectTitle(Isbn isbn);

.. revealjs:: SQLファイル + ドメインクラス
   :title-heading: h3

   SQLファイルはドメインクラスを使用しない場合と何も変わらない。

   .. rv_code::

      SELECT /*%expand*/*
        FROM book
       WHERE title = /* title */'x'
         /*%if author != null */
         AND author = /* author */'y'
         /*%end*/

.. revealjs:: ドメインクラスの有無を比較
   :title-heading: h3

   .. rv_code::

      @Select
      List< Book> select(Title title, Author author);

      @Select
      List< Book> select(String title, String author);

   ドメインクラスを使用していると `dao.select(author, title)` はコンパイルエラーになる。

.. revealjs:: ジェネリックなドメインクラス
   :title-heading: h3

   例えばサロゲートキーを表すドメインクラスがあったとする。

   .. rv_code::

      @Domain(valueType = Long.class)
      public class SurrogateKey< T> {
      
          private final Long value;
      
          public SurrogateKey(Long value) {
              this.value = value;
          }
      
          public Long getValue() {
              return value;
          }
      }

.. revealjs:: ジェネリックなドメインクラス
   :title-heading: h3

   型変数にはエンティティをバインドする。

   .. rv_code::

      @Entity
      public class Book {

          @Id
          @GeneratedValue(strategy = GenerationType.IDENTITY)
          public SurrogateKey< Book> id;

   .. rv_code::

      @Entity
      public class Author {

          @Id
          @GeneratedValue(strategy = GenerationType.IDENTITY)
          public SurrogateKey< Author> id;

   .. rv_code::

      author.id = book.id; //コンパイルエラー

.. revealjs:: StreamやCollectorへの対応

   .. rv_code::

      @Select
      List< Book> select();
  
      @Select(strategy = SelectType.STREAM)
      < R> R select(Function< Stream< Book>, R> fn);
  
      @Select(strategy = SelectType.COLLECT)
      < R> R select(Collector< Book, ?, R> collector);

   .. rv_code::

      //タイトルをカンマ区切りで並べる
      String titleList = select(s -> s.map(book -> book.title).collect(Collectors.joining(", ")));

.. revealjs:: Optionalへの対応
   
   .. rv_code::

      @Select
      Optional< String> selectTitle(Isbn isbn);
  
      @Select
      String selectTitle(Optional< Isbn> isbn);

   エンティティのフィールドにもOptionalは使える。

.. revealjs:: その他の機能

   * イミュータブルなエンティティ。
   * エンティティ作成の支援ツールdoma-gen。
     テーブル定義からエンティティを生成する。

     * SQLファイルの実行結果からも生成できる？
       試してない。

   * ローカルトランザクション。
   * 外部ドメイン。
     既に存在しており変更できないクラスをドメインとして扱う。
   * クエリビルダ。
     やむを得ず動的にSQLを組み立てるための補助的なクラス。
   * `Date and Time API <https://jcp.org/en/jsr/detail?id=310>`_ への対応。

.. revealjs:: 公式ドキュメントなど

   Doma http://doma.readthedocs.org/

   ( Doma 1.x http://doma.seasar.org/ )

   作者: `@nakamura_to <https://twitter.com/nakamura_to>`_ さん

.. revealjs:: ☃

   おわり。
