---
ID: 583095d671e38ada7b99
Title: 【PHP】JSONを使って、LINEbotから情報を保存/読み込みする
Tags: PHP,Heroku,JSON,linebot,LINEmessagingAPI
Author: ""
Private: false
---

#はじめに
Heroku上でLINEMessagingAPIを使って遊んでいる者です。
php初心者向けの内容になりますが、LINEbotでJsonファイルを使って、情報を保存し、再度利用する方法を共有します。

内容としてはほとんどphpでjsonファイルを操作する方法の紹介となりますが、実例も添えてシェアしてみようと思います。自分で使ってみて面白かったので、ぜひみなさんも使ってみてください。

#コード
例えばこのような形で実装します。
input.phpとpreference.jsonは同じ階層に置いてあげてください。

```preference.json
{
  "mode":"normal",
  "time":"1518609396",
  "song":0
}
```

```index.php
//preference.jsonの読み込み
$url = "preference.json";
$raw_json = file_get_contents($url);
if($raw_json === false){
    error_log('failed to load json file.');
}
//オブジェクトとしてデコードしたpreference.jsonを変数$pref_jsonに代入します。
$pref_json = json_decode($raw_json);

//今回はpreference.jsonのキー「mode」を書き換えます。
$pref_json->mode = "書き換えたい語句";
//preference.jsonの書き換え
$w_json = fopen("preference.json", "w+b");
fwrite($w_json, json_encode($pref_json, JSON_UNESCAPED_UNICODE));
fclose($w_json);
```

これで、jsonfileを書き換えることができます。そして、再度アプリで読み込んでやれば、LINEbotの記録場所として利用することができます。

#使用例
###1.botに歌ってもらう
こちらから「歌って」と連呼することで歌ってもらいます。

```index.php
            if($pref_json->tonnel === 0){
                $bot->replyText($event->getReplyToken(), "はっぴばーすでい、てゅーゆ～♪");
            }elseif($pref_json->tonnel === 1){
                $bot->replyText($event->getReplyToken(), "はっぴばーすでい、てゅーゆ～♪");
            }elseif($pref_json->tonnel === 2){
                $bot->replyText($event->getReplyToken(), "はっぴばーすでい、でぃあ〇〇～♪");
            }elseif($pref_json->tonnel === 3){
                $bot->replyText($event->getReplyToken(), "はっぴばーすでいてゅーゆ～♪");
                $pref_json->tonnel = -1;
            }
            $pref_json->tonnel += 1;
            $w_json = fopen("./json/preference.json", "w+b");
            fwrite($w_json, json_encode($pref_json));
            fclose($w_json);
            exit;
            break;
```

![qiita1.gif](https://qiita-image-store.s3.amazonaws.com/0/124948/c66532e4-227c-11fe-c9a3-d9bedd293392.gif)

###2.BOTに語句を覚えさせる

```index.php
      default :
      if(preg_match("/#/u", $text)){
        $text = mb_substr($text,1);
        //memory.jsonを検索
        //* するためにmemory.jsonを読み込み
        $url = "./json/memory.json";
        $raw_json = file_get_contents($url);
        if($raw_json === false){
            error_log('failed to load json file(cash.json).');
        }
        $memory_json = json_decode($raw_json);
        //*keyを元に検索
        $key = array_search($text, $memory_json);
        $tail = ["", "！", "笑", "ｗｗｗｗ", "↑↑"];
        if(($key)!= FALSE){
            $bot->replyText($event->getReplyToken(), $memory_json[++$key].selectArray($tail));
            exit;
        }
      //preference.json->modeを書き換え
        $pref_json->mode = "input";
        $w_json = fopen("./json/preference.json", "w+b");
        fwrite($w_json, json_encode($pref_json));
        fclose($w_json);
        //cash.jsonをデコード
        $url = "./json/cash.json";
        $raw_json = file_get_contents($url);
        if($raw_json === false){
          error_log('failed to load json file(cash.json).');
        }
        $cash_json = json_decode($raw_json);
        //cash.jsonを書き換え
        $cash_json->pre_message = $text;
        $w_json = fopen("./json/cash.json", "w+b");
        fwrite($w_json, json_encode($cash_json, JSON_UNESCAPED_UNICODE));
        fclose($w_json);
        $word = ["どういう意味？", "なんて意味？","それはどういうこと？"];
        replyTextMessage($bot, $event->getReplyToken(), selectArray($word));
        goto label_end;
    break;

    }	

label_input:
if($pref_json->mode === "input"){
      foreach ($events as $event) {
//メッセージ以外の場合にエラーを吐いてcontinueで抜け
        if (!($event instanceof \LINE\LINEBot\Event\MessageEvent)) {
            error_log('Non message event has come');
            continue;
        }
//テキスト以外の場合もエラー吐いてcontinueで抜け
        if (!($event instanceof \LINE\LINEBot\Event\MessageEvent\TextMessage)) {
            error_log('Non text message has come');
            continue;
        }
        if(preg_match("/だまれ/u", $event->getText())){
            $pref_json->mode = "normal";
            $w_json = fopen("./json/preference.json", "w+b");
            fwrite($w_json, json_encode($pref_json, JSON_UNESCAPED_UNICODE));
            fclose($w_json);
            $bot->replyText($event->getReplyToken(), "ごめんなさい");
            goto label_end;
        }
        //まずpreference.jsonを書き換えて保存する
        $pref_json->mode = "normal";
        $w_json = fopen("./json/preference.json", "w+b");
        fwrite($w_json, json_encode($pref_json, JSON_UNESCAPED_UNICODE));
        fclose($w_json);
        //memory.jsonをデコード
        $url = "./json/memory.json";
        $raw_json = file_get_contents($url);
        if($raw_json === false){
          error_log('failed to load json file(memory.json).');
        }
        $memory_json = json_decode($raw_json, true);
        //cash.jsonをデコード
        $url = "./json/cash.json";
        $raw_json = file_get_contents($url);
        if($raw_json === false){
          error_log('failed to load json file(cash.json).');
        }
        $cash_json = json_decode($raw_json);
        //memory.jsonを書き換え
        $memory_json[] = $cash_json->pre_message;
        $memory_json[] = $event->getText();
        $w_json = fopen("./json/memory.json", "w+b");
        fwrite($w_json, json_encode($memory_json, JSON_UNESCAPED_UNICODE));
        fclose($w_json);
        $answord = array("なるほど","理解した","ご教授いただきありがとうございます");
            $word = selectArray($answord);
        $bot->replyText($event->getReplyToken(), $word);
    }
    goto label_end;
}

      default :
      if(preg_match("/#/u", $text)){
        $text = mb_substr($text,1);
        //memory.jsonを検索
        //* するためにmemory.jsonを読み込み
        $url = "./json/memory.json";
        $raw_json = file_get_contents($url);
        if($raw_json === false){
            error_log('failed to load json file(cash.json).');
        }
        $memory_json = json_decode($raw_json);
        //*keyを元に検索
        $key = array_search($text, $memory_json);
        $tail = ["", "！", "笑", "ｗｗｗｗ", "↑↑"];
        if(($key)!= FALSE){
            $bot->replyText($event->getReplyToken(), $memory_json[++$key].selectArray($tail));
            exit;
        }
      //preference.json->modeを書き換え
        $pref_json->mode = "input";
        $w_json = fopen("./json/preference.json", "w+b");
        fwrite($w_json, json_encode($pref_json));
        fclose($w_json);
        //cash.jsonをデコード
        $url = "./json/cash.json";
        $raw_json = file_get_contents($url);
        if($raw_json === false){
          error_log('failed to load json file(cash.json).');
        }
        $cash_json = json_decode($raw_json);
        //cash.jsonを書き換え
        $cash_json->pre_message = $text;
        $w_json = fopen("./json/cash.json", "w+b");
        fwrite($w_json, json_encode($cash_json, JSON_UNESCAPED_UNICODE));
        fclose($w_json);
        $word = ["どういう意味？", "なんて意味？","それはどういうこと？"];
        replyTextMessage($bot, $event->getReplyToken(), selectArray($word));
        goto label_end;
    break;

    }
```

```memory.json
["LINEbot","楽しい","にゃーん","猫の鳴き声"]
```

```cash.json
{
   "pre_message":"none"
}

```
![qiita3.gif](https://qiita-image-store.s3.amazonaws.com/0/124948/c4fe820c-ffda-33bf-5c44-d01eadca45a9.gif)

外部記憶を利用することで、LINE Messaging API開発の幅はぐっと広がります！
外部記憶としてはJsonの他にもテキストファイル、データベースなどの利用が考えられますが、手軽な実装方法として、手始めにJsonを利用してみるのはいかがでしょうか？
使用例で使用したBOTのソースコードはこちらから→ https://github.com/mutsu00062/qiita_LINE

---
---
LINE Messaging APIに関する記事を書いている方々へ↓
私が行ったように、動作例をgifとして保存する場合は[Gifcam](https://forest.watch.impress.co.jp/docs/review/617499.html)が便利です(スクリーンを範囲選択して動画をキャプチャし、それをそのままGifとして保存できます)
