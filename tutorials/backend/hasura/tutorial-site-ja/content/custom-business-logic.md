---
title: "カスタムビジネスロジック"
metaTitle: "カスタムビジネスロジック | Hasura GraphQLチュートリアル"
metaDescription: "カスタムビジネスロジックは、Hasuraを使用して2つの方法で処理できます。1つは、カスタムGraphQLリゾルバーを書いて、リモートスキーマとして追加する方法です。もう1つは、ミューテーション後に非同期的にウェブフックをトリガーする方法です。"
---

Hasuraは、承認とアクセス制御があるCRUD + リアルタイムGraphQL APIを提供します。ただし、アプリにカスタム/ビジネスロジックを追加したい場合があります。例えば、構築中のtodoアプリで、公開フィードにtodoを挿入する前に、テキストに不適切な表現が含まれていないかを検証したい場合などです。

Hasuraではカスタムビジネスロジックは、いくつかの柔軟な方法で処理できます。

アクション（推奨）{#actions}
---------------------

[アクション](https://hasura.io/docs/latest/graphql/core/actions/index.html)は、カスタムクエリとミューテーションを使用して、カスタムビジネスロジックでHasuraのスキーマを拡張する方法です。アクションは、データ検証、外部ソースからのデータ強化、その他の複雑なビジネスロジックなどの様々なユースケースを処理するためにHasuraに追加できます。

![アクションアーキテクチャ](https://hasura.io/docs/latest/_images/actions-arch1.png)

アクションは、クエリまたはミューテーションのいずれかになります。

- `Query Action`- 外部APIからいくつかのデータをクエリする場合、またはデータを返す前にいくつかのバリデーション/変換を行っている場合、クエリアクションを使用できます。
- `Mutation Action` - データベースを操作する前にデータ検証またはカスタムロジックを実行する場合は、ミューテーションアクションを使用できます。

リモートスキーマ {#remote-schemas}
--------------

Hasuraには、リモートGraphQLスキーマを統合して、統一されたGraphQL APIを提供する機能があります。自動化されたスキーマステッチのようなものだと考えてください。これにより、カスタムGraphQLリゾルバーを書いて、リモートスキーマとして追加できます。

![リモートスキーマアーキテクチャ](https://hasura.io/docs/latest/_images/remote-schema-arch1.png)

アクションかリモートスキーマかを選択する際、注意すべき点があります。

- GraphQLサーバーがある場合、または自分で構築したい場合、リモートスキーマを使用します。
- REST APIを呼び出す必要がある場合、アクションを使用します

イベントトリガー {#event-triggers}
--------------

Hasuraを使用すると、[Postgresデータベース内のテーブルにイベントトリガーを作成できます](https://hasura.io/learn/database/postgresql/triggers/)。イベントトリガーが、指定されたテーブルでイベントを確実にキャプチャして、カスタムロジックを実行するためにウェブフックを呼び出します。ミューテーション操作後、トリガーは非同期的にウェブフックを呼び出すことができます。

todoアプリのユースケース {#use-case-todo-app}
-------------------------

構築したtodoアプリのバックエンドには、追加したい特定のカスタム機能があります。

Auth0からプロファイル情報を取得したい場合は、トークンでAuth0へのAPI呼び出しを行う必要があります。Auth0は、GraphQLではなく、REST APIのみ公開します。このAPIは、GraphQLクライアントに公開する必要があります。

APIを拡張するためにHasuraにアクションを追加します。また、リモートスキーマとしてカスタムGraphQLサーバーが追加されたことで、同じことをどのように行えるかについても説明します。

- 新しいユーザーがアプリに登録するたびに、電子メールで通知を受けます。これは、イベントトリガーウェブフック経由で呼び出すことができる非同期操作です。

これら2つのユースケースをHasuraでどのように処理できるかを紹介します。
