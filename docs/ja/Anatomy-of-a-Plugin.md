# ScriptCraftプラグインの分析

Javaで書かれたプラグインでできるすべてのことは、より早くより簡単にScriptCraftプラグインのJavascriptで行うことが出来ます。これを説明するために、一般的に使用されているMODであるhomesをJavascriptで再実装してみました。[homes][homes]プラグインとはゲーム内のコマンドからプレイヤーに場所をhomeとして登録させ、その場所に戻れるようにするものです。プレイヤーは他のプレイヤーのhomeに行くことが出来ます。これは、ScriptCraft内で以下の2つの機能をデモする単純なプラグインです。

 * 永続性
 * 一般プレイヤ用のコマンド追加

[homes]:  /src/main/js/plugins/homes/homes.js

さて、プレイヤーにゲーム内チャットの文字色を変更する他のプラグインもご案内します。

## 永続性
まず、永続性について説明します。永続性とはサーバがシャットダウンされ再び起動した後でも状態を保存しておける能力のことです。内蔵されている`persist()`関数を使用することにより、シャットダウン時に保存されサーバ起動時に再読込されたJavascriptのオブジェクトを作成できます。

```javascript
// file: scriptcraft/plugins/my-first-plugin.js
var prefs = persist('myprefs', {});
...
prefs.color = 'black';
```

上記の例では、新しい空のオブジェクトを作成し`myprefs-store.json`ファイルに保存しています。`persist()`関数はそのファイルにデータが何も記録されていなかったり、ファイルが存在しなかったりした場合は空のオブジェクトが返し、そのオブジェクトのすべての変更はサーバがシャットダウンされるときにファイルに保存されます。

JSON形式はいくらか人間にとって読みやすいので、JSON形式で保存されます。新しいプラグインを宣言するのは簡単です。そこで、ゲーム内のチャットウィンドウからチャットの文字のデフォルトの色をプレイヤーが変更できる"チャット"という新しいプラグインを作ろうと思います。

```javascript
var store = persist('chat-colors', {players: {}});
exports.chat = { 
  setColor: function(player,chatColor) { 
    store.players[player.name] = chatColor;
  }
}
```

上記のコードはオペレータにプレイヤーの色の選択を設定させる以外には何もしません。( `/js chat.setColor(self, 'green')` )他のプレイヤーとチャットする際にチャットの文字色を変更できるようにするには、もう少しだけコードに変更が必要です。しかし、上記のコードは最低限プレイヤーの色設定を保存することを保証します。以下のコードを追加すると、プレイヤーがチャットした際に、チャットの文字が各々のプレイヤーが選択した色で表示されるようになります。

```javascript
var colors = ['black', 'blue', 'darkgreen', 'darkaqua', 'darkred',
              'purple', 'gold', 'gray', 'darkgray', 'indigo',
              'brightgreen', 'aqua', 'red', 'pink',
              'yellow', 'white'];
var colorCodes = {};
var COLOR_CHAR = '\u00a7';
for (var i =0;i < colors.length;i++) 
  colorCodes[colors[i]] = i.toString(16);

var addColor = function( evt ) {
  var player = evt.player;
  var playerChatColor = store.players[ player.name ];
  if ( playerChatColor ) {
    evt.message = COLOR_CHAR + colorCodes[ playerChatColor ] + evt.message;
  }
};

if (__plugin.bukkit) {
  events.asyncPlayerChat(addColor);
} else if (__plugin.canary) {
   events.chat(addColor);
};
```

次のステップは色と名前の対応表を定義し、チャットを検知し色指定コードをプレイヤーのチャットメッセージに追加するイベントハンドラ追加することです。

## 新しいプレイヤーコマンドの追加
ScriptCraftの他のコマンドは`/jsp`コマンドです。&ndash;このコマンドはオペレーターに一般プレイヤーが使用できるようにプラグインを公開します。誤解のないようにすると`/jsp`コマンドはJavascriptの評価を一切行わず、Javascriptのプラグインに渡すパラメータを伝えるだけです。今までのところお分かりの通り現在このサンプルプラグインでは一般のプレイヤーに選んだチャットの色に設定されず、オペレータのみが`js chat.setColor(...)`というJavascript式を使うことでチャットの色を設定することが可能です。API全てにJavascriptを使用してプレイヤーがアクセスできるようにするのはよい考えではないということを明らかにしましょう。では、どうすれば安全にチャットメッセージの色を選ばせることができるでしょうか？Javascript関数を書いてプレイヤーがこの関数を使用できるようにしたいのであればこのように新しい`command()`関数を使用し機能を公開します。

```javascript
function chat_color( params, sender ){
  var color = params[0];
  if (colorCodes[color]){
    chat.setColor(sender,color);
  }else{
    echo(sender, color + ' is not a valid color');
    echo(sender, 'valid colors: ' + colors.join(', '));
  }
}
command(chat_color, colors);
```

上記のコードに`/jsp`コマンドの*サブコマンド*を追加したり、プレイヤーが`TAB`キーを押した時のオートコンプリートのオプション(コマンドの最後のパラメータである色)も明示します。これでプレイヤー自身でチャットの色を以下の様なコマンドで変更できるようになりました。

    /jsp chat_color yellow

プレイヤーにチャットの文字色を選択させることが出来、サーバがシャットダウンし起動した時設定を保存することができるようになりました。`chat_color`という新しい`jsp`のサブコマンドも追加ました。これはプレイヤーがチャットの文字色を変更するために使用することが出来ます。プラグインのソースコードは僅かな行数ですが、十分にプラグインが動作します。

```javascript
var store = persist('chat-colors', {players: {}});
exports.chat = { 
  setColor: function(player,chatColor) { 
    store.players[player.name] = chatColor;
  }
}
var colors = ['black', 'blue', 'darkgreen', 'darkaqua', 'darkred',
              'purple', 'gold', 'gray', 'darkgray', 'indigo',
              'brightgreen', 'aqua', 'red', 'pink',
              'yellow', 'white'];
var colorCodes = {};
var COLOR_CHAR = '\u00a7';
for (var i =0;i < colors.length;i++) 
  colorCodes[colors[i]] = i.toString(16);

var addColor = function( evt ) {
  var player = evt.player;
  var playerChatColor = store.players[ player.name ];
  if ( playerChatColor ) {
    evt.message = COLOR_CHAR + colorCodes[ playerChatColor ] + evt.message;
  }
};

if (__plugin.bukkit) {
  events.asyncPlayerChat(addColor);
} else if (__plugin.canary) {
   events.chat(addColor);
};

function chat_color( params, sender ){
  var color = params[0];
  if (colorCodes[color]){
    chat.setColor(sender,color);
  }else{
    echo(sender, color + ' is not a valid color');
    echo(sender, 'valid colors: ' + colors.join(', '));
  }
}

command(chat_color, colors);
```
    
![Chat Color plugin][1]

これは、最低限実行可能なプラグインでScriptCraftのいくつかの新しい機能を証明します。その機能とは、自動的な永続性、イベントハンドリング、プレイヤーが`/jsp`コマンドを使用することで公開される新しい機能のことです。これはMinecraftの潜在的なMOD作成者のニーズにふさわしいゲームの変更を簡単にすることが出来るでしょう。



[1]: ../img/scriptcraft-chat-color.png

