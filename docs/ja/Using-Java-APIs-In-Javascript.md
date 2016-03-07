# JavascriptからJavaのAPIを使用するには

ScriptCraftはJava6移行に含まれているJavascriptengineを使用します。これはすべてのコアJavaのクラスがScriptCraft内から使用できることを意味します。
さらにすべてのBukkit APIもJavascriptから使用できます。
ただ、Javascript内からJavaのクラスを使用するにあたっていくつか頭において置くかなければならないことがあります。
## Java Beansを使用するには

Javaに同梱されたJavascriptエンジンはJava Beansにアクセスしたり修正するのに便利な表記法を備えています。Java Beanとは`get{Property}()`メソッドをオブジェクトのプロパティを得るメソッドとして使用しており、`set{Property}()`メソッドをオブジェクトのプロパティにセットするためのメソッドとして使用してあるJavaのクラスです。[Bukkit API][bukapi]でJava Beanを使用している例として[org.bukkit.entity.Player][bukpl]クラスが挙げられます。このクラスはJava Beanの仕様に従っているメソッドを持っています。

例えば、[Player.getWalkSpeed()][bukplws]メソッドはプレイヤーの歩く速度を得るときに使用します。Javaでは歩く速度を得るために以下のようにコードを書く必要があります。

    float walkingSpeed = player.getWalkSpeed();

しかし、Javascriptでは歩く速度のプロパティにより簡潔にアクセスすることが出来ます。

    var walkingspeed = player.walkSpeed;

お好みであればJavaの用にアクセスすることも出来ます。

    var walkingspeed = player.getWalkSpeed();

個人的には、より簡潔な`player.walkSpeed`のほうが読みやすいので好みです。覚えておくべき大事なことは、JavascriptからJava BeanのBukkit APIやJava APIを呼ぶ時、`propertyName`というプロパティは`getPropertyName()`というgetterと`setPropertyName()`というsetterを呼びます。このルールによりBukkitクラスのプロパティが何かを推測できます。例えば、[Bukkit Player][bukpl]オブジェクトは以下のようなメソッドを持っています。

 * float getWalkSpeed()
 * void setWalkSpeed(float speed)

これからPlayerオブジェクトは読み書き可能な`walkSpeed`プロパティを持っていると推測できます。例えばゲーム内のコマンドプロンプトで以下のようなコマンドを使用すると歩く速度をデフォルトの0.2から3倍にすることが出来ます。

    /js self.walkSpeed = self.walkSpeed * 3;


Javaの表記に制限されているなら`/js self.setWalkSpeed( self.getWalkSpeed() * 3 )`と書くべきです。Bukkit APIの殆どのクラスもJava Beanなのでプロパティなどにアクセスできます。例えば、プレイヤーが居るワールドの名前を取得するためには...

    /js self.location.world.name

と書くと`/js self.getLocation().getWorld().getName()`と書くより簡潔です。
If you're new to Java and the [Bukkit API][bukapi] is the first time
you've browsed Java documentation, you may be wondering where the
`location` property came from - `location`プロパティはPlayerクラスのスーパークラスから継承されています。（スーパークラスとはそのクラスの親・祖先です。）
[Player][bukpl]クラスのJavadocページの
**Methods inherited from interface org.bukkit.entity.Entity** の部分に`getLocation()`メソッドがあります。
## java.langパッケージのクラスを使用するには

Javaでは`user.dir`や`user.timezone`プロパティを表示するために以下のようなコードを書きます。

    System.out.println( System.getProperty( "user.dir" ) );
    System.out.println( System.getProperty( "user.timezone" ) );

Javaでは`java.lang`パッケージのどんなクラスも接頭辞を必要としません。つまり、`java.lang.System`クラスは簡単に`System`と記述することが出来ます。
しかし、`java.lang`パッケージ内のJavascriptクラスは完全修飾名が必要です。つまり、以下のように記述しなければいけません。

    println( java.lang.System.getProperty( "user.dir" ) );
    println( java.lang.System.getProperty( "user.timezone" ) );

`println()`関数はJavascriptエンジンによって提供されているデフォルトの関数の一つです。つまり、クラス名の接頭辞を付ける必要がありません。しかし、他のSystemクラスのメソッドは明示的にパッケージ名をインクルードする必要があります。（例：`java.lang.`）
もし、式の数だけSystemクラスを使用するのであれば、System変数を宣言し完全修飾パッケージ名やクラス名の代わりにしようするとタイプ量を減らすことが出来ます。

    var System = java.lang.System;
    println( System.getProperty( "user.dir" ) );
    println( System.getProperty( "user.timezone" ) );

Javascriptエンジンはパッケージをimport(読み込む)する`importPackage()`関数を提供します。これも、クラス名の前につけるパッケージ名を入力する手間を減らすことが出来ます。
具体的には以下の通りです。

    importPackage(java.util);
    var hMap = new HashMap();
    hMap.put('name','Walter');

これによりJavaライブラリの`java.util`パッケージは`java.util`接頭辞を付けずに使用できます。しかし、`java.util`のクラスのいくつかはJavascriptのオブジェクトのタイプと競合する（例： String, Object）ため`java.util`パッケージをimportすることはおすすめしません。

## 概要

ScriptCraftでモジュールやプラグインを記述する時、JavaBeanのプロパティを読み書きするためにJavaの.get{PropertyName}()や.set{PropertyName}()メソッドを使用する代わりに簡潔な.{propertyName}と記述出来ます。これによりコードをより簡潔に出来ます。この簡潔な表記法はJava6移行に組み込まれているJavascriptエンジンによって提供されています。Javascriptはprivateなメンバーにはアクセスしませんが、なんと.{propertyName}表記は自動的に適切なJavaの.get{PropertyName}()や.set{PropertyName}()に変換されます。

[bukapi]: http://jd.bukkit.org/beta/apidocs/
[bukpl]: http://jd.bukkit.org/beta/apidocs/org/bukkit/entity/Player.html
[bukplws]: http://jd.bukkit.org/beta/apidocs/org/bukkit/entity/Player.html#getWalkSpeed()
[buksrv]: http://jd.bukkit.org/beta/apidocs/org/bukkit/Server.html

