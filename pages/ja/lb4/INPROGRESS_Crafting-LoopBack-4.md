---
lang: jp
title: 'LoopBack 4を構築する'
keywords: LoopBack 4.0, LoopBack 4
sidebar: lb4_sidebar
permalink: /doc/en/lb4/Crafting-LoopBack-4.html
---

##  バックグラウンド

[LoopBack](http://loopback.io)は、 API開発者向けに構築された、オープンソースの[Node.js](https://nodejs.org)のフレームワークです。主な目的は、既存のサービス/データベースからマイクロサービスとしてAPIを作成し、Web、モバイル、IoTなどのクライアントアプリケーションのエンドポイントとして公開することです。
LoopBackは、APIリクエストの受け入れと、バックエンドリソースとのやり取りの間をつなぎます。
Loopbackと他のフレームワークの違いは、開発者がすぐに使用できる統合機能を使用してAPIロジックを実装できる点です。これにより、LoopBackは[Express](https://expressjs.com)、[Hapi](https://hapijs.com)、 [Sails](http://sailsjs.com)などとは異なる、優れたAPI構成レイヤーとなっています。


![loopback-composition](./imgs/loopback-composition.png)

LoopBackは、バージョン3.xまで、[Express framework](https://expressjs.com)に構築されました。
結果的には、LoopBackをExpressに基づかせることは正しい決定でした。
Expressの力を借りることで、LoopBackは、「車輪の再発明」のごとく既存システムの修正に手間取られることなく、API作成エクスペリエンスの価値を高めることに集中できました。LoopBackは、npmのすぐに使用できるミドルウェアモジュールなど、Expressのエコシステムやコミュニティによる知識とサポートの恩恵が受けられるのも魅力です。

LoopBackを使用することで、開発者はまるでレシピに従うように簡単にAPIを作成-公開できます。LoopBackは、API実装の重要な側面をまとめた一連のコアコンセプトを導入します。そして開発者は、既存のデータベースやサービスからAPIを作成するために、LoopBackアプリケーションを足場とし、必要なJSON宣言とNode.jsコードを追加して、APIを数分で起動して実行できます。

LoopBackは、認証、承認、ルーティングなどのAPIユースケースの要求/応答パイプラインへの接続として、Expressのルーティングとミドルウェアを使用します。そして、受信HTTP処理機能を超えて、モデル、データソース、コネクタなどの統合機能を提供し、APIロジックがデータベース、REST API、SOAP Webサービス、gRPCマイクロサービスなどの様々なバックエンドシステムと対話できるようにします。また、インバウンド通信とアウトバウンド統合を結び付ける機能により、LoopBackはAPI開発者にとって非常に強力なフレームワークになります。
下の図は、LoopBackが典型的なエンドツーエンドAPI処理フローにどのように適合するかを示しています。

![loopback-overview](./imgs/loopback-overview.png)

LoopBackは長年の開発歴とリリースにより、機能-ユーザー数ともに大幅に成長しました。 LoopBackは、開発者コミュニティにも広く受け入れられています。コミュニティによる[様々な拡張機能]（https://github.com/pasindud/awesome-loopback）がその一例です。 
コアチームもまた、コミュニティによるフィードバックから多くのことを学びながら、相互的にLoopbackを発展させています。

## Why LoopBack 4?

世の中の多くのプロジェクトと同様に、LoopBackにもまた次のような課題が明らかになってきています。

1. より多くのモジュールとより多くの機能を備えたコードベースは、時間とともにより複雑になります。私たちはより多くのメンテナーとコントリビューターを助けたい一方、学習曲線は急勾配になっています。原因の1つは、JavaScriptそのものです。これは、型指定が弱く、コード間のコントラクトを明示的に定義するインターフェイス等の構造がありません。初心者には、かなりの背景知識が必要となります。

2. 技術的な負債も累積しています。例えば、モジュール間で一貫性のないデザインや、異なる動作の機能フラグがあります。以下に例を示します。

-各モジュールが、それぞれ異なるレジストリを使用して個々のアーティファクトを管理している。アーティファクトの例は、リモーティングメタデータ、モデル、データソース、ミドルウェア等。
-それぞれ異なるフレーバーを使用して、各レイヤーでカスタムロジックが要求/応答をインターセプトしている。レイヤーの例は、ミドルウェア、リモートフック、CRUD操作フック、コネクターフック等。
-ユーザーが新しい動作にオプトインできるようにする一方で、下位互換性を維持するために、機能フラグの追加が増えている。

3. 一部の領域が現在の設計の限界に達し始めているため、新しい機能を追加したり、バグを修正したりすることがより困難になっています。

-このloopback-datasource-jugglerモジュールは、タイピング、データモデリング、検証、集約、永続性、サービス統合など、まるであらゆることを全て請け負っているキッチンシンクのようなものです。
-モデルには、データ表現、永続性、RESTへのマッピングなど、複数の役割があります。モデルはデータソースに関連付けられており、異なるデータソースに対して同じモデル定義を再利用するのは簡単ではありません。

4. コアチームへのLoopBackモジュールのコード変更リクエストなしにフレームワークを拡張するのは、簡単ではありません。LoopBackの現在のバージョンには、様々なレイヤーでアドホックな拡張性があります。そのため、拡張ポイントは一貫して定義されていません。例えば、

-Expressを使用してミドルウェアを登録します。
-リモートフックを傍受するには、リモートフックを使用します。
-CRUDフックを使用して、CRUD操作に関するロジックを追加します。

5. より多くのプロジェクトが、基になるプラットフォームとしてLoopBackを使用し始めています。このような使用例では、LoopBackを活用してメタデータ駆動型アプローチを使用してアーティファクトを管理および構成するために、LoopBack内部の詳細な知識と柔軟性と動的性が必要です。良い例を次に示します。

-テナント間のアーティファクトの分離を必要とする、マルチテナント
-モデル定義とデータソースを管理/アクティブ化する、メタデータAPI
-イベントやメッセージングなど、コネクタの新しい相互作用パターン
-モデル定義の追加のメタデータ

3.xのリリース以来、チームはLoopBackを維持し、前進させる方法についてブレーンストーミングを行ってきました。多くの課題解決を行い、既存のGitHubの問題をトリアージし、コミュニティメンバーとダウンストリーム製品に向き合い、関連するフレームワークとテクノロジーを評価して、次の質問に答えてきました。

-LoopBackの対象読者は誰か。LoopBackに興味があるのはなぜか。彼らは何のためにLoopBackを使用し、どのようにそれを使用するのか。
-重大な問題点は何か。新しい基盤を再構築せずに、段階的に対処できるか。
-最もリクエストの多い機能は何か。現在のデザインに、そういった機能を追加することは可能か。
-世界中で最新かつ最高の技術は何か。それらを採用し始めたら、それらはどのような価値をもたらすのか。
-LoopBackの開発と保守をどのようにスケーリングするか。LoopBackを使用してAPIを作成する上で、大規模な開発チームが協力できるようにするにはどうすればよいか。
-コミュニティをさらに成長させ、そのエコシステムを拡大するにはどうしたらよいか。LoopBackにより多くのユーザーと貢献者をもたらすために何ができるか。

LoopBackは、Node.jsアプリケーション開発者以外も、以下のように様々なユーザーの間で注目を集めています。

-**API開発者 ** -LoopBackを使用して、Node.jsでAPIを作成できます。
-**LoopBackのメンテナーとコントリビューター** -LoopBackプロジェクトによるモジュールのビルドとメンテナンスができます。
-**拡張機能の開発者** -LoopBackに拡張機能を提供して、フレームワークを強化できます。
-**プラットフォーム開発者**  -付加価値製品を構築するベースとして、LoopBackを活用できます。

![loopback-ecosystem](./imgs/loopback-ecosystem.png)

そこでコアチームは、上記のすべてのグループのニーズを満たすために、大胆にLoopBackを再構築することを決定しました。この決定により、新世代のAPI作成プラットフォームであるLoopBack 4が始まりました。詳細については、ブログ投稿[Loopbackを簡単に拡張できるようにするための次のステップ、Loopbackを発表](https://strongloop.com/strongblog/announcing-loopback-next)をお読みください。

目的
LoopBack 4の目標は次のとおりです。

1. 最新かつ最高のテクノロジーの進歩に追いつく。
-メンテナンスの容易さと生産性のために、 [ES2016/2017](http://exploringjs.com/es2016-es2017/index.html) と [TypeScript](https://www.typescriptlang.org/) を採用します。
- [OpenAPI Spec](https://www.openapis.org/)や [GraphQL](http://graphql.org/) などの、新しい標準を採用します。

2. エコシステムを成長させるための拡張性を促進します。
-最小限のコアを構築し、拡張機能を介して他のすべてを実装できるようにします。
-より多くの [拡張ポイント](https://github.com/strongloop/loopback-next/issues/512)と、拡張機能のドアを開きます。

3. マイクロサービス向けのクラウドネイティブエクスペリエンスと連携します。
- [Cloud Native Computing Foundation](https://www.cncf.io/)などのイニシアチブを採用して、クラウドネイティブマイクロサービスを採用し ます。
-LoopBackをマイクロサービスエコシステムの第一級市民にします。

4. モジュール間の複雑さと矛盾を取り除きます。
-一貫したレジストリとAPIを使用して、アーティファクトとその依存関係を管理します。
-複雑なモジュールをリファクタリングすることにより、技術的な負債を払い戻します。

5. 構成可能性を高めるための個別の懸念。
-コントローラーやリポジトリーなどの新しい概念を導入して、様々な役割を担わせます。
-ランタイムを一連のサービスとして分類し、拡張ポイント/拡張パターンを使用して、登録、解決、および構成を管理します。

## 設計原則
LoopBack 4を構築するために「ビッグバン」アプローチを採用しないことにしました。代わりに、より小さなステップで段階的に複数の段階で構築を行っています。このアプローチにより、最初からLoopbackコミュニティとの関わりを高めることができます。アーキテクチャのシンプルさと拡張性を追求するために、以下の原則に従っています。

1. **最初に命令型を、後に宣言型を**

すべては `APIs`を介してコードで実行できます。LoopBackチームまたはコミュニティの貢献者は、そのようなAPIを使用して様々なユーザーエクスペリエンスを作成できます。たとえば、モデルを定義するAPIを使用すると、アプリケーションでJSONまたはYAMLファイルでモデルを宣言して、モデルを検出およびロードできるようになります。拡張機能は、JSONスキーマ、デコレーターを含むES6クラス、OpenAPI仕様のスキーマ、さらにはXMLスキーマなど、他の形式のモデル定義をLoopBackモデル定義に解析できます。

また、[デコレータ](https://www.typescriptlang.org/docs/handbook/decorators.html)などのプログラミング構造を活用して 、開発者がコードでメタデータを提供できるようにすることも可能です。さらに、LoopBackアーティファクトはJSONまたはYAMLファイルで宣言できます。これは、ユーザーが手動またはツールで生成および操作するのに便利です。

2. **最小限の機能を構築し、必要に応じて後で追加**
YAGNI (You Aint’t Gonna Need It）の原則を適用します。将来必要になると思われるものではなく、現在必要なものを設計および構築します。APIの作成には様々な観点があり、人々は多くの機能を求めています。MVPから始めることで、ノイズに悩まされることなく問題の根本に到達し、コアビルディングブロックとして不可欠な機能を構築できます。

3. **開発者の最初の経験**
LoopBackは、開発者によって開発者向けに構築されていることに常に留意してください。私たちの最優先事項は、API開発者の仕事を楽にすることです。APIやCLIやGUIなどのユーザーインターフェイスを設計するときは、それらが思考プロセスにとって直感的で自然なものであることを確認したいと考えています。


## 実装段階
以下に示すように、LoopBack 4の最終バージョンに向かって進んでいる段階を以下に示します。

1. コアのリベースとリライト
-TypeScriptを活用して、コードの品質と生産性を向上させます。
    -JavaScriptのオプションの型システムを提供します。
    -将来のJavaScriptエディションから現在のJavaScriptエンジンに予定されている機能を提供します。
-非同期プログラミングモデル/スタイルを統一します。
    - 100％約束ベースのAPI。
    - 一流の非同期プログラミングスタイルとしての非同期/待機。
- 可視性と拡張性を高めるためにIoCコンテナーを実装する
    - さまざまなモジュールにわたるユニバーサルレジストリ
    - 依存関係を管理するパターンとしての依存性注入
- 拡張機能のパッケージングモデルとしてコンポーネントを導入する
    - コンポーネントは、npmモジュールまたはローカルディレクトリにすることができます
    - コンポーネントは拡張機能のリスト全体をカプセル化します

2. REST / HTTP呼び出しチェーンを実装して、コア設計を検証します
- OpenAPI仕様で始まるトップダウンREST API作成を追加します。
- インバウンドHTTP処理のアクションのビルドシーケンス
    - アクションの構成としてシーケンスを導入する
    - 最も重要なアクションを実装して、REST APIのルーティングと呼び出しを実現します
- API関連のビジネスロジックのエントリポイントとしてコントローラーを導入します。
モデルは、現在のLoopBackアプリケーションの中心的存在です。。彼らは複数の責任を負います：
    - データモデリング
    - API関連のビジネスロジックのアンカー
    - 永続性またはサービス呼び出し
    - REST HTTP / JSONエンドポイントへのマッピング
- コンポーネントとしての認証
以下を含むコンポーネントとして認証のコア機能を実装します。

認証要件を示すデコレーター
authenticate 認証を処理するアクション
さまざまな認証戦略の拡張ポイント
統合および構成機能を再構築する

CRUDやキー/値ストアなどのデータアクセスパターンを表すリポジトリを導入する
レガシージャグラーとコネクタを使用して、CRUDおよびKVのリポジトリインターフェースのリファレンス実装を提供します。
ジャグラーを個別のモジュールにリファクタリング/リライトします
入力システム
モデルと関係の定義
検証
クエリおよび突然変異言語
情報元
リポジトリインターフェイスとデータアクセスの実装
サービス呼び出しのサービスインターフェイスと実装
コネクタのインターフェイスとメタデータを定義する
書き換えコネクタ
宣言的なメタデータとブートストラップ

LoopBackは、モデル、関係、データソース、コネクター、ACL、コントローラー、リポジトリー、アクション、シーケンス、コンポーネント、ユーティリティー関数、OpenAPI仕様などの一連の成果物を管理します。これらのアーティファクトをコード（apiおよびデコレータ）で記述するプログラム的なアプローチに加えて、JSON / YAMLファイルで宣言できるように宣言的なサポートを追加したいと思います。

JSON / YAML形式の新しいドメイン固有言語（DSL）と対応するテンプレートを定義します。
プロジェクトレイアウトを定義して、プロジェクトの成果物を整理します。
IoCコンテキストを活用して、拡張ポイント/拡張パターンに従ってそのようなアーティファクトのメタデータ/インスタンスを管理します。
各タイプの成果物のライフサイクルおよびシリアル化/逆シリアル化の要件を定義します。
ブートコンポーネントを追加して、アーティファクトを検出/ロード/解決/アクティブ化します。ブートプロセスは、ツールとランタイムの両方に合わせて調整できます。
ツーリング（CLIおよびUI）

CLIおよびUIツールを以下に追加します。
Scaffold LoopBack 4アプリケーション
シーケンス、アクション、コントローラー、リポジトリ、サービス、データソース、モデルなどの成果物を管理します
クラウドネイティブエクスペリエンスを有効にする

コントローラがgRPCサービスとして公開されることを許可する
他のgRPCサービスとの相互作用を許可する
DockerやKubernetesなどのマイクロサービス展開インフラストラクチャとの統合
サービスメッシュとの統合
次の図は、LoopBack 4の高レベルの構成要素を示しています。



The core team decided to make a bold move and rebuild LoopBack to meet the needs
of all the above groups. The decision led to the inception of LoopBack 4, a new
generation of API creation platform. For more information, read the blog post
[Announcing LoopBack.next, the Next Step to Make LoopBack Effortlessly Extensible](https://strongloop.com/strongblog/announcing-loopback-next).

## Objectives

LoopBack 4's goals are:

1.  Catch up with latest and greatest technology advances.

    - Adopt [ES2016/2017](http://exploringjs.com/es2016-es2017/index.html) and
      [TypeScript](https://www.typescriptlang.org/) for ease of maintenance and
      productivity.
    - Embrace new standards such as [OpenAPI Spec](https://www.openapis.org/)
      and [GraphQL](http://graphql.org/).

2.  Promote extensibility to grow the ecosystem.

    - Build a minimal core and enable everything else to be implemented via
      extensions.
    - Open the door for more
      [extension points and extensions](https://github.com/strongloop/loopback-next/issues/512).

3.  Align with cloud native experience for microservices.

    - Adopt cloud native microservices by adopting initiatives such as
      [Cloud Native Computing Foundation](https://www.cncf.io/).
    - Make LoopBack a first-class citizen of the microservices ecosystem.

4.  Remove the complexity and inconsistency across modules.

    - Use a consistent registry and APIs to manage artifacts and their
      dependencies.
    - Pay down technical debts by refactoring complex modules.

5.  Separate concerns for better composability.
    - Introduce new concepts such as controllers and repositories to represent
      different responsibilities.
    - Break down the runtime as a set of services and utilize the extension
      points/extensions pattern to manage the registration, resolution, and
      composition.


## Implementation stages

Here are the stages we are marching through toward the final version of LoopBack
4 as illustrated below.

1.  **Rebase and rewrite the core**

    - Leverage TypeScript for better code quality and productivity.

      - Provide optional type system for JavaScript.
      - Provide planned features from future JavaScript editions to current
        JavaScript engines.

    - Unify the asynchronous programming model/style.

      - 100% promise-based APIs.
      - Async/Await as first-class async programming style.

    - Implement an IoC Container for better visibility and extensibility

      - Universal registry across different modules
      - Dependency injection as a pattern to manage dependencies

    - Introduce Component as packaging model for extensions
      - Component can be a npm module or a local directory
      - Component encapsulates a list of extensions as a whole

2.  **Validate the core design by implementing an REST/HTTP invocation chain**

    - Add top-down REST API creation which starts with OpenAPI specs.

    - Build sequence of actions for inbound http processing

      - Introduce sequence as the composition of actions
      - Implement the most critical actions to fulfill the REST API routing and
        invocation

    - Introduce controllers as entry points for API-related business logic.

      Models are the centerpieces of the current LoopBack applications. . They
      take multiple responsibilities:

      - Data modeling
      - Anchor for API related business logic
      - Persistence or service invocation
      - Mapping to REST HTTP/JSON endpoints

    - Authentication as a component

      Implement the core functionality of authentication as a component, which
      includes:

      - Decorators to denote authentication requirement
      - `authenticate` action to handle authentication
      - Extension points for various authentication strategies

3.  **Rebuild our integration and composition capabilities**

    - Introduce repositories to represent data access patterns such as CRUD or
      Key/Value stores
    - Provide a reference implementation of CRUD and KV flavors of repository
      interfaces using the legacy juggler and connectors
    - Refactor/rewrite the juggler into separate modules
      - Typing system
      - Model and relation definition
      - Validation
      - Query and mutation language
      - DataSource
      - Repository interfaces and implementations for data access
      - Service interfaces and implementations for service invocations
    - Define interfaces and metadata for connectors
    - Rewrite connectors

4.  **Declarative metadata and bootstrapping**

    LoopBack manages a set of artifacts, such as models, relations, datasources,
    connectors, ACLs, controllers, repositories, actions, sequences, components,
    utility functions, and OpenAPI specs. In addition to the programmatic
    approach to describe these artifacts by code (apis and decorators), we would
    like to add declarative support so that they can be declared in JSON/YAML
    files.

    - Define a new domain-specific language (DSL) in JSON/YAML format and
      corresponding templates.
    - Define the project layout to organize project artifacts.
    - Leverage the IoC Context to manage metadata/instances of such artifacts
      following the extension point/extension pattern.
    - Define the lifecycle and serialization/de-serialization requirements for
      each type of artifact.
    - Add a boot component to discover/load/resolve/activate the artifacts. The
      boot process can be tailored for both tooling and runtime.

5.  **Tooling (CLI & UI)**

    - Add CLI and UI tools to:
      - Scaffold LoopBack 4 applications
      - Manage artifacts such as sequences, actions, controllers, repositories,
        services, datasources and models

6.  **Enable cloud native experience**

    - Allow controllers to be exposed as gRPC services
    - Allow interaction with other gRPC services
    - Integration with microservices deployment infrastructure such as Docker
      and Kubernetes
    - Integration with service mesh

The following diagram illustrates the high-level building blocks of LoopBack 4:

![loopback-stack](./imgs/loopback-stack.png)

Please note there is a common layer below the different functional areas in the
stack. Let's examine the need to build a new core foundation for LoopBack 4.

## A new core foundation

### The core responsibility

LoopBack itself is already modular. For example, a typical LoopBack 3.x
application's dependency graph will have the following npm modules:

- loopback
- strong-remoting
- loopback-datasource-juggler
- loopback-connector-\*
- loopback-component-explorer

LoopBack manages various artifacts across different modules. The following are a
list of built-in types of artifacts that LoopBack 3.x supports out of box:

- Model definitions/relations: describes data models and their relations
- Validation: validates model instances and properties
- Model configurations: configures models and attaches them to data sources
- Datasources: configures connectivity to backend systems
- Connectors: implements interactions with the underlying backend system
- Components: wraps a module that be bootstrapped with LoopBack
- Remoting: maps JavaScript methods to REST API operations
- ACLs: controls access to protected resources
- Built-in models: provides set of prebuilt models such as User, AccessToken,
  and Role
- Hooks/interceptors
  - Express middleware
  - remote hooks
  - CRUD operation hooks
  - connector hooks
- Security integration
  - Identity and token management
  - Authentication schemes
    - Passport component for various authentication strategies
- Storage component for various local/cloud object storage systems
  - Local file system
  - Amazon S3
  - Rackspace
  - Azure
  - Google Cloud
  - OpenStack
  - IBM Cloud Object Storage
- Push component for mobile push notifications
  - iOS
  - Android
- Different API styles
  - REST
  - gRPC
  - GraphQL

Metadata for these artifacts form the knowledge base for LoopBack to glue all
the pieces together and build capabilities to handle common API use cases.

How to represent the metadata and their relations is the key responsibility of
the LoopBack core foundation. It needs to provide a consistent way to contribute
and consume such building blocks.

### Key ingredients for the core

The core foundation for LoopBack 4 is responsible for managing various artifacts
independent of the nature of such artifacts.

- A consistent registry to provide visibility and addressability for all
  artifacts.

  - Visibility: Each artifact has a unique address and can be accessed via a URI
    or key. Artifacts can also be visible at different scopes.

  - Extensibility: LoopBack artifacts can be managed by types. New artifact
    types can be introduced. Instances for a given type can be added, removed,
    or replaced. Organizing artifacts in a hierarchy of extension
    points/extensions decouples providers and consumers.

- Ability to compose with dependency resolution.

  - Composability: It's common that one artifact to have dependencies on other
    artifacts. With dependency injection or service locator patterns, the core
    will greatly simplify how multiple artifacts work together.

- A packaging model for extensions.

  - Pluggability: Extensions can be organized and contributed as a whole. We
    need to have a packaging model so that extension developers can create their
    own modules as bundles and plug into a LoopBack application.

### Why Express behind the scene?

#### Background

LoopBack had always been built on Express so we can leverage the vast community
and middleware in the Express ecosystem **BUT** it presented some challenges for
LoopBack. With LoopBack 4 we considered moving away from Express (and even built
the framework without Express) but eventually circled back to Express because of
its vast ecosystem.

Some of the gaps between what Express offers and LoopBack's needs are:

- Lack of extensibility
  > Express is only extensibile via middleware. It neither exposes a registry
  > nor provides APIs to manage artifacts such as middleware or routers.
- Lack of composability
  > Express is not composable. For example, `app.use()` is the only way to
  > register a middleware. The order of middleware is determined by the order of
  > `app.use`.
- Lack of declarative support
  > In Express, everything is done by JavaScript ... In contrast, LoopBack is
  > designed to facilitate API creation and composition by conventions and
  > patterns as best practices.

#### Circling back to Express with a twist

The main purpose of LoopBack is to make API creation easy, interacting with
databases, services, etc., not middleware for CORS, static file serving, etc. We
didn't want to reinvent the wheel by writing new middleware for LoopBack 4.

The team explored leveraging
[Express or Koa](https://github.com/strongloop/loopback-next/pull/1082) (but
only for their middleware support). The final decision was to use Express in a
way that bridges the gap by addressing the gaps identified above as follows:

- LoopBack provides its own
  [Controller / OpenAPI metadata optimized routing engine](Routes.md)
- Express is used exclusively for allowing us to consume Express middleware
  (CORS, Static File Serving)
- LoopBack uses a [Sequence of Actions](Sequence.md) to craft the response in a
  composable manner and leverages `@loopback/context` as a registry.

You can learn more details in our blog post on
[improving inbound http processing](https://strongloop.com/strongblog/loopback4-improves-inbound-http-processing).

## Deep dive into LoopBack 4 extensibility

There are several key pillars to make extensibility a reality for LoopBack 4.

- [Context](Context.md), the IoC container to manage services
- [Dependency injection](Dependency-injection.md) to facilitate composition
- [Decorators](Decorators.md) to supply metadata using annotations
- [Component](Using-components.md) as the packaging model to bundle extensions

Please check out [Extending LoopBack 4](Extending-LoopBack-4.md).

## Rebuilding LoopBack experience on top of the new core

With the extensible foundation in place, we start to rebuild the LoopBack REST
API experience by "eating your own dog food" with the following artifacts:

- [Sequence and actions](Sequence.md): A sequence of actions to handle HTTP
  requests/responses.
- [Controllers](Controllers.md): A class with methods to implement API
  operations behind REST endpoints.
- [Model](Model.md): Definition of data models.
- [Repositories](Repositories.md): Interfaces of access patterns for data
  sources.

The features are provided by the following modules:

- [@loopback/rest](https://github.com/strongloop/loopback-next/tree/master/packages/rest/)
- [@loopback/repository](https://github.com/strongloop/loopback-next/tree/master/packages/repository/)

## Example for application developers

Before we go further, let's try to build a 'hello world' application with
LoopBack 4.

### Basic Hello-World

[@loopback/example-hello-world](https://github.com/strongloop/loopback-next/tree/master/examples/hello-world)

### Intermediate example

[@loopback/example-todo](https://github.com/strongloop/loopback-next/tree/master/examples/todo)

## Example for extension developers

### Learn from existing ones

- [@loopback/example-log-extension](https://github.com/strongloop/loopback-next/tree/master/examples/log-extension)
- [@loopback/authentication](https://github.com/strongloop/loopback-next/tree/master/packages/authentication)

## References

- <https://strongloop.com/strongblog/announcing-loopback-next/>
- <https://www.infoq.com/articles/driving-architectural-simplicity>
- <https://strongloop.com/strongblog/creating-a-multi-tenant-connector-microservice-using-loopback/>
- <https://strongloop.com/strongblog/loopback-as-an-event-publisher/>
- <https://strongloop.com/strongblog/loopback-as-a-service-using-openwhisk/>
