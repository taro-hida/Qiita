---
ID: 81f2627a2cbad8e5ce81
Title: smartyの使い方（自分用メモ）
Tags: PHP,Smarty
Author: ""
Private: false
---

職場で使うのでちょっと事前に動作を確認

#smartyをcomposerでrequireする

```
composer require smarty/smarty
```

#templateの作成

```test.tpl
{* コメント *}

{* 変数 *}
{ var }

{*連想配列 *}
{marray.hoge}
{marray.huga}
```

#php側による実装

```run.php
require_once 'vendor/autoload.php';

$smarty = new Smarty();
$smarty->setTemplateDir('./templates/');

$marray = array(
  'hoge' => 'hogeです',
  'huga' => 'hugaです'
);

$smarty->assign('var', 'スマーティてす');
$smarty->assign('marray', $marray);

$smarty->display('test.tpl');
```

#templateの結合

```header.tpl
<title>hogehoge</title>
```

```test2.tpl
{include file = 'header.tpl'}
{var}
{marray.hoge}
{marray.huga}
```
まあ、`{include file = 'path'}`を使えばおｋ
<header>と<body>を分割して使いまわす際とかに使いましょう
