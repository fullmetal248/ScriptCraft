## ScriptCraftから他のプラグインを使用するには
これらの質問が多く見受けられたので回答を以下に示します:

> PermissionEX APIのような他のプラグインのAPIを利用する方法は?  
> 以下のJavaプログラムで権限のグループのチェックができますがJavaScriptでは出来ません:  
> ru.tehkode.permissions.bukkit.PermissionsEx.getUser(player).inGroup("moderator");  
> -- [Bukkitフォーラムの質問より][1]  

[1]: http://dev.bukkit.org/bukkit-plugins/scriptcraft/?page=2#c48

この質問はCraftBukkitでのScriptCraftの使用に関わることですので最初にお答えします。

PermissionsExのインスタンスは以下のように取得します。 (他のBukkitプラグインについても同様です。)
```javascript
var pex = server.pluginManager.getPlugin('PermissionsEx');
if (pex.getUser(player).inGroup('moderator') ) {
...
}
```
一般的に他のプラグインのAPIを使用したいときは名前でプラグインオブジェクトを取得しメソッドを呼びます。以上の例では`pex`という変数が前述の`PermissionsEx`プラグインの参照を持っています。参照を取得すると、Javaと同じようにプラグインのメソッドを呼ぶことができます。
面倒な点は`server.pluginManager.getPlugin()`でプラグインの参照を取得する所です。

CanaryModで他のプラグインのAPIへの参照は同じ原理で取得します。
仮にScriptCraftとdConomyプラグインをインストールしたとします:

```javascript
var Canary = Packages.net.canarymod.Canary;
var pluginMgr = Canary.pluginManager();
var dConomy = pluginMgr.getPlugin('dConomy');
var dConomyServer = dConomy.modServer;
// ここからdConomyServerのオブジェクトのすべてのメソッドなどにアクセスできます。
// 例： dConomyServer.newTransaction()
```

BukkitとCanaryModの違いはプラグインの参照の取得方法だけです。Bukkitは以下のように行います。

```javascript
var otherPlugin = server.pluginManager.getPlugin('プラグイン名');
```

CanaryModでは以下のように行います。

```javascript
var Canary = Packages.net.canarymod.Canary;
var otherPlugin = Canary.pluginManager().getPlugin('プラグイン名');
```
