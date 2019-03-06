---
date: "2019-03-04T13:31:09.129Z"
title: "gqlgenメモ"
tags: ["graphql", "golang", "gqlgen"]
description: ""
---

gqlgenを軽く触ったのでメモを残します。

## コレ何

- Go言語製GraphQLサーバーの最有力実装(たぶん)
- スキーマ駆動開発ができる

## バージョン

現在リリースされているのが`v0.7.2`。もうすぐリリースされる `v0.8.0`から 
[go modulesに対応する](https://github.com/99designs/gqlgen/issues/226)
ようで、始めるにはちょっとタイミングが悪いみたいです。

とか書いてたら `v0.8.0` が[リリース](https://github.com/99designs/gqlgen/releases/tag/v0.8.0)
されました(∩´∀｀)∩ （以下は0.7.2でやったメモです。）

## 試し方

[Getting Started](https://gqlgen.com/getting-started/) を見てやりました。
試す際はそれ以上でもそれ以下でもない感じです。

## 実際に開発の流れを書いてみる

本当に作ってるものはここに書けないので以下のようなモデルのシステムを例として書きます。

- 注文(`Sale`)
  - 注文内容(`Detail`)

1注文に複数の注文内容(商品)が含まれるのを想定した構成です。

### データベース

まずデータベースのスキーマを書きます。
どうでもいい（ことない）んですが、DBの予約語って使いたい単語使えなくていつもモヤっとします。
ORDER...(・ω・)

```sql
-- +migrate Up
CREATE TABLE sale (
  id BIGINT PRIMARY KEY
  , created_at BIGINT NOT NULL DEFAULT 0
);

COMMENT ON TABLE sale IS '受注データ';
COMMENT ON COLUMN sale.created_at IS '作成日時(UNIXTIME)かつ受注日時';

CREATE TABLE detail (
  id BIGINT PRIMARY KEY
  , sale_id BIGINT  NOT NULL DEFAULT 0
  , name    TEXT    NOT NULL DEFAULT ''
  , price   INTEGER NOT NULL DEFAULT 0
  , FOREIGN KEY (sale_id) REFERENCES sale (id)
);

COMMENT ON TABLE  detail         IS '詳細データ';
COMMENT ON COLUMN detail.sale_id IS '親ID';
COMMENT ON COLUMN detail.name    IS '商品名';
COMMENT ON COLUMN detail.price   IS '価格';

-- +migrate Down
DROP TABLE sale;
DROP TABLE product;
```

PostgreSQLを使っています。NULLは許さない派です。
時刻系はいつもUNIXTIMEで保存しています。タイムゾーンとかそういうのはDBが
考えることじゃないと思っています。
デフォルト値は0です。つまり `1970-01-01 00:00:00 +0000 UTC` です。この時間に1秒のずれもなく
なにかしらイベントが発生することは考えにくいので大丈夫だと考えています。フロントエンド側で
**0の場合は非表示** にするとか **0の場合は--表示** にするとかいうのをいつもしています。

`-- +migrate Up` や `-- +migrate Down` は [sql-migrate](https://github.com/rubenv/sql-migrate)
のおまじないです。 `sql-migrate up` でテーブルが作られて、 `sql-migrate down` で
テーブルが削除されます。どのsql文が適用されたとかが管理される超便利な奴です。
がっつりとしたORM(gormとか)は使わずに、こういう軽いツールを組み合わせる派です。

```
xo pgsql://postgres@127.0.0.1:5432/hogedb?sslmode=disable -o ./model --template-path .xo_templates
```

私の大好きツール `xo`により `./model` 以下に `sale.xo.go` と `detail.xo.go` 
が出力されます。 `.xo_templates` は `xo` が使うテンプレートファイル置き場です。
秘伝のタレ(汚い)なので内緒です。デフォルトのstruct tagのjsonタグがsnake_case
で出力されるのをlowerCamelに変換するようになど、独自実装しています。

### やっとgqlgen

ここまでやってから `gqlgen` の世界に入っていきます。

`schema.graphql` を書きます。

```graphql
type PageInfo {
  endCursor: Int        # 現在ページの最終データのIDをもたせる
  hasNextPage: Boolean  # 次のページがあるかどうか？
}

type Detail {
  id: Int!
  saleId: Int!
  name: String!
  price: Int!
  productId: Int!
}

type Sale {
  id: Int!
  createdAt: Int!
  details: [Detail!]!
}

type Sales {
  totalCount: Int!  # 全部で何件あるのか？
  edges: [Sale!]!
  pageInfo: PageInfo
}

type Query {
  sales(
    first: Int,
    after: Int,
    start: Int,
    finish: Int,
    keyword: String,
    orderBy: String
  ): Sales!
}
```

Mutationは今回無しです。データ取得するだけです。
`Detail` と `Sale` は データベースの定義に（ほぼ）合わせます。
変えているところとして、Saleの子として詳細情報の配列（`[Detail!]!`） をもたせました。

親子関係を記述することで、gqlgenが自動的に

```
query {
  sales {
    edges {
      id
      details {
        id
        name
      }
    }
  }
}
```

のようなクエリを投げた時に紐づくdetailを取得する処理を行ってくれます。
なお、この点が **GraphQLはN+1問題に注意** と言われているところで、
このままだと素直にN+1問題が発生するので [dataloader](https://gqlgen.com/reference/dataloaders/)
を利用するようにします。（後述）

その他、 `PageInfo`、`Sales`、`Query` がGraphQLライクなところです。

受注情報はレコード数が大量になることを想定し、 
[ページネーション](https://graphql.github.io/learn/pagination/) の実装を考えての
記述になっています。

`edges` に実体を入れます。`edges` という名前は決まりじゃなくて慣例です。
GitHubのGraphQLもこうなっていますね。

EdgeとNodeについては [GraphQL入門 \- 使いたくなるGraphQL \- Qiita](https://qiita.com/bananaumai/items/3eb77a67102f53e8a1ad)
が解りやすかったです。ありがとうございます。

gqlgen の init します。 `scripts/gqlgen.go` ってなんだよっていう場合は
[Getting Started](https://gqlgen.com/getting-started/) を参照してください。

```
go run scripts/gqlgen.go init
```

色々ファイルが出来ますが、気にすべきは `resolver.go` です。開くと

```go
panic("not implemented")
```

というのが至るところにあるので、そこにDBとのやり取りなどを記載していきます。

この後は `schema.graphql` を更新するたびに `go run scripts/gqlgen.go -v` 
を実行し、出力されたエラーを見ながら `resolver.go` に足りない実装を追加
していくという流れになります。

### dataloader

Salesに対するDetailを要求する際のN+1問題を回避するためのdataloaderです。  
gqlgenがdataloadenを使おうぜと仰っているので従います。

```
go get github.com/vektah/dataloaden
dataloaden -keys int -slice github.com/miiton/hogehoge/model.Detail
```

これで `detailsliceloader_gen.go` というファイルが出来ます。これは触りません。  
dataloaderの実処理を書く `dataloader.go` というファイルを作成します。(名前はなんでも良いです。)

ポイントはコード中の `// NOTE:` に書きました。チュートリアルのサンプルコード
などはDBへの接続を考慮していなかったりするので結局N+1問題が発生していたりしました。
（DBのクエリログでちゃんと確認しましょうね）

```go
package hogehoge

import (
	"context"
	"log"
	"net/http"
	"time"

	"github.com/jmoiron/sqlx"
	"github.com/miiton/hogehoge/model"
	"golang.org/x/xerrors"
)

type ctxKeyType struct{ name string }

var ctxKey = ctxKeyType{"detailLoader"}

func DataloaderMiddleware(db *sqlx.DB, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		wait := 250 * time.Microsecond

		// detail loader - 1:N
		detailLoader := DetailSliceLoader{
			maxBatch: 100,
			wait:     wait,
			fetch: func(keys []int) ([][]model.Detail, []error) {
				// NOTE: keysに最大maxBatch分の親IDが入る（今回の場合は sale_id ）
				// NOTE: wait入れてもどこでwait入るのかいまいちわからんかった

				// NOTE: [バッチ単位の[値のスライス]]を返す
				resultSet := make([][]model.Detail, len(keys))
				errors := make([]error, len(keys))
				var details []model.Detail

				// NOTE: keys分のデータ(= maxBatch = 最大100件)を取得する
				// NOTE: ここはsqlxの力で記述を省力化!!
				query, args, err := sqlx.In("SELECT * FROM detail WHERE sale_id IN (?);", keys)
				query = db.Rebind(query)
				if err != nil {
					log.Fatalf("%+v\n", xerrors.Errorf("detailLoader: ", err))
				}

				err = db.Select(&details, query, args...)
				if err != nil {
					log.Println(query)

					// NOTE: 1クエリで発生したエラーをkeys分複製して返すようにする
					for i, _ := range keys {
						errors[i] = err
					}
				}

				// NOTE: DBから取得したデータを各親項目に分配する
				// メモリ上でやるからはやいぜ
				for i, key := range keys {
					for _, detail := range details {
						if detail.SaleID == key {
							resultSet[i] = append(resultSet[i], detail)
						}
					}
				}

				return resultSet, errors
			},
		}
		ctx := context.WithValue(r.Context(), ctxKey, &detailLoader)
		r = r.WithContext(ctx)
		next.ServeHTTP(w, r)
	})
}

func ctxLoaders(ctx context.Context) *DetailSliceLoader {
	return ctx.Value(ctxKey).(*DetailSliceLoader)
}
```

あとgqlgenのドキュメントには書いてないのですが、 `server/server.go`
をDataloaderMiddlewareを使うように書き換えます。

```go
package main

import (
	"log"
	"net/http"
	"os"

	"github.com/99designs/gqlgen/handler"
	"github.com/miiton/hogehoge"
	"golang.org/x/xerrors"
)

const defaultPort = "8080"

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = defaultPort
	}

	http.Handle("/", handler.Playground("GraphQL playground", "/query"))
	queryHandler := handler.GraphQL(hogehoge.NewExecutableSchema(hogehoge.Config{Resolvers: &hogehoge.Resolver{}}))

	db, err := hogehoge.ConnectDB()
	if err != nil {
		err = xerrors.Errorf("server.go main():", err)
		log.Fatalf("%+v\n", err)
	}
	//                                 ↓ここ大事！
	http.Handle("/query", hogehoge.DataloaderMiddleware(db, queryHandler))

	log.Printf("connect to http://localhost:%s/ for GraphQL playground", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

これでOKです。クエリログで確認すると、 `dataloader.go` に書いた `maxBatch` の
数だけIN句にパラメータが渡されたSQL分が複数回実行されるのを確認できたらOKです。


## 所感

- 子要素（今回の場合はSaleに対するDetail）の取得など、良く考えられてるなーと思いました。
- コードジェネレータ系は後で意味がわからなくなることが多いですが、これぐらい薄かったらアリだと思います。
- スキーマ駆動開発といいつつ、データベースはやっぱり別で書かないとだめだなー
  というのは仕方ないというか現時点ではそうあるべきだと思いました。（ページネーションとかの実装をみて）
- GraphQLの公式サイトにある [graphgql-go](https://github.com/graphql-go/graphql) も試したんですが、
  あちらはスキーマ駆動じゃないし、型が `interface{}` いっぱいですし、プルリク
  送ってもなかなか反応が無いので個人的にはあまりオススメしないです。
  （わかりやすいのはわかりやすかったです。）
