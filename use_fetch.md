今回はNuxt3で詰まった部分・困った部分を紹介します。
内容は**useFetch**です。
___
## useFetchでのCORSエラー(開発環境)
Nuxtのuse~シリーズで頻出なのが`useFetch`。最近作ったポートフォリオにZennの記事一覧を載せようと思い、Zenn dev APIの記事とuseFetchのドキュメントを読みいざコードを書いてみると...

:::message alert
Access to fetch at 'https://zenn.dev/api/articles?username=uyuy_create' from origin '(ローカルサーバー)' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
:::

なんかエラーを吐きました。リロードするとエラーは消えるのですが、コードを書いて保存するとまた同じエラーが。

まずは[Google AI Studio](https://aistudio.google.com/prompts/new_chat)に聞いてみました。すると

`1. Zenn APIのCORS対応を待つ (理想的だが、実現するとは限らない):`

いきなり非現実的な回答をしてきました。ほかにも解決策をいくつか提示してはくれたものの、結局解決には至らず...。
___

## 解決 ... ?
いろいろどっこいしょあって、ついに原因がわかりました。

**useFetchは通常サーバーサイドでfetchして値をクライアントサイドに渡すのでCORSのエラーは回避するが、ホットリロードされるとfetchがクライアントサイドで行われてしまいCORSに引っかかるから。**

まさかの**仕様**。どうすればいいんだ...。
しかし調べてみると、回避できる方法があるらしい。
___

## 解決。
その方法とは、**server/apiディレクトリにAPIルートを作成する**というもの。
たしかに、プロジェクト直下に`server`というフォルダがありました。

```js script:server/api/zenn.ts
import { defineEventHandler } from 'h3'

export default defineEventHandler(async (event) => {
  const username = getQuery(event).username
  const url = `https://zenn.dev/api/articles?username=${username}`
  const response = await fetch(url)
  return await response.json()
})
```
`server/api`に上記のようなコードを設置し、それをコンポーネントで呼び出す。
これが一般的な解決策のようです。
___

## 結論
serverディレクトリの使用方法を学びました。
本番環境で同じエラーを吐いたらどこかしらミスがあるのでしょうが、開発環境ではuseFetchを使用しているfetchのエラーについては少し寛容になる必要があるかもしれません。

Author: uyuyu