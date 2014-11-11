knockoutjs-doc-ja
=================
[![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/sukobuto/knockoutjs-doc-ja?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

# KnockoutJS の非公式日本語ドキュメントです。

http://kojs.sukobuto.com

Markdown に対応しました。
docs/articles/*.md が記事ファイルです。
PULL REQ 送っていただければ反映いたします！

# フォーマット

一応、本家と同様に以下のスタイルを一貫させていますが、
特に気にせず書いていただければこちらで整えます。

## ページタイトル... H1

```
# タイトル
```

## 節見出し... H3

節見出しには、ハッシュリンク用の ID を付加します。
Markdown を拡張した記法で、`{#some_id}` のように記述できます。

```
### 見出し {#heading_id}
### 使い方 {#how_to_use}
```

本ドキュメントでは、本家のハッシュリンクID（アンカーネーム）をそのまま用いています。
理由は別の場所からのリンクの際に本家と同じハッシュリンクが使える、いちいち確認する手間を省けるためです。
可能であれば Dev Tools などで見出しのハッシュリンクIDをコピーして使ってください。

## コード

### HTML

	```html
	<p>hello world!</p>
	```

### JavaScript

	```javascript
	alert('hello world!');
	```

