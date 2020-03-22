---
lang: en
title: 'How to secure your LoopBack 4 application with JWT authentication'
keywords: LoopBack 4.0, LoopBack 4, Authentication, Tutorial
sidebar: lb4_sidebar
permalink: /doc/en/lb4/Authentication-Tutorial.html
summary: A LoopBack 4 application that uses JWT authentication
---

## 概観

LoopBack 4には、カスタム認証ストラテジと認証デコレーター`@authenticate`を使用して、
アプリケーションのAPIエンドポイントを保護できる認証パッケージ`@loopback/authentication`があります。

このチュートリアルでは、 `JSON Web Token (JWT)`アプローチに基づいて、カスタム認証戦略を**作成**および**登録**することにより、`authentication`が、[loopback4-example-shopping](https://github.com/strongloop/loopback4-example-shopping)アプリケーションにどのように追加されたかを紹介します。

以下で、JSON Web Token (JWT)アプローチの簡単なサマリを示しています。

![JSON Web Token Authentication Overview](../../imgs/json_web_token_overview.png)

**JSON Web Token (JWT)** 認証方式で、Userが**正しい資格情報**を**ログイン**エンドポイントに提供する場合、サーバーは、JWTトークンを作成し応答を返します。
トークンは**文字列型**で、次の3つの部分：**ヘッダー**・**ペイロード**・**シグニチャー** で構成されます。各部分は**シークレット**を使用して暗号化され、各部分はピリオドで区切られます。

例:

```ts
// {encrypted-header}.{encrypted-payload}.{encrypted-signature}
eyJhbXVCJ9.eyJpZCI6Ij.I3wpRNCH4;
// actual parts have been reduced in size for viewing purposes
```

{% include note.html content=" 注：ペイロードには、開発者が望むものを何でも含めることができますが、少なくともUserIDは含まれることが必須です。また、Userパスワードを含めることはできません。
" %}

ログインしてこのトークンを取得した後、Userが保護されたエンドポイントにアクセスしようとするときは常に、トークンを**Authorization**ヘッダーで提供する必要があります。サーバーは、トークンが有効で期限切れでないことを確認し、保護されたエンドポイントへのアクセスを許可します。

詳細については、[JSON Web Token (JWT)](https://en.wikipedia.org/wiki/JSON_Web_Token)を参照してください。

完成したl`loopback4-example-shopping` アプリケーションを表示して実行するには、[Try it out](#try-it-out) セクションの指示に従ってください。

LoopBack 4アプリケーションにJWT認証を追加する方法の詳細を理解するには、[Adding JWT Authentication to a LoopBack 4 Application](#adding-jwt-authentication-to-a-loopback-4-application) セクションをお読みください。


## やってみましょう

このチュートリアルの最終結果をアプリケーション例として見たい場合は、次の手順に従ってください。

1. アプリケーションを開始します。
   ```sh
   git clone https://github.com/strongloop/loopback4-example-shopping.git
   cd loopback4-example-shopping
   npm install
   npm run docker:start
   npm start
   ```

   次の表示が出るまで待ちます:

   ```sh
   Recommendation server is running at http://127.0.0.1:3001.
   Server is running at http://[::1]:3000
   Try http://[::1]:3000/ping
   ```

1. ブラウザで [http://[::1]:3000](http://127.0.0.1:3000) または
   [http://127.0.0.1:3000](http://127.0.0.1:3000)を開き、`/explorer`をクリックして`API Explorer`を開きます。

2. `UserController`セクションでは,`POST /users`、さらに`'Try it out'`をクリックして, 以下を指定します:

   ```json
   {
     "email": "user1@example.com",
     "password": "thel0ngp@55w0rd",
     "firstName": "User",
     "lastName": "One"
   }
   ```
  `'Execute'` をクリックして、`'User One'`というUserを**追加**します。

4. `UserController`セクションで、`POST /users/login`、`'Try it out'`をクリックし、以下を指定します。 
In the `UserController` section, click on `POST /users/login`, click on
   `'Try it out'`, specify:

   ```json
   {
     "email": "user1@example.com",
     "password": "thel0ngp@55w0rd"
   }
   ```

  `'Execute'`をクリックし、`'User One'`として**log in**します.

   JWT トークンがレスポンスで返送されます。

   例:

   ```json
   {
     "token": "some.token.value"
   }
   ```

5. API Explorerの上部までスクロールすると、`Authorize`ボタンが表示されます。これは、JWTトークンを設定できる場所です。
　　Scroll to the top of the API Explorer, and you should see an `Authorize`
   button. This is the place where you can set the JWT token.

   ![](../../imgs/api_explorer_authorize_button.png)

6. `Authorize` ボタンをクリックすると、ダイアログが開きます。

   ![](../../imgs/api_explorer_auth_token_dialog1.png)

7. `bearerAuth`値フィールドで、先ほど取得したトークン文字列を入力して、`Authorize`ボタンをクリックします。これで、このJWTトークンは、次に通信する`/users/me` エンドポイントで使用できるようになりました 。`Close` ボタンを押してダイアログを閉じます。
　　In the `bearerAuth` value field, enter the token string you obtained earlier,
   and press the `Authorize` button. This JWT token is now available for the
   `/users/me` endpoint we will interact with next. Press the `Close` button to
   dismiss the dialog.

   ![](../../imgs/api_explorer_auth_token_dialog2.png)

   {% include note.html content="<b>Logout</b> ボタンで、必要に応じいつでも新しい値を入力できます。" %}

１. `UserController` セクションまでスクロールダウンし、 `GET /users/me`を開けます。

   ![](../../imgs/api_explorer_usercontroller_section1.png)

   なお、このエンドポイントには**lock**アイコンがありますが、同セクションの他のエンドポイントにはないことに注意してください。これは、このエンドポイントが、オペレーショナルレベルを`security requirement object` であるとOpenAPI仕様で指定したためです。（詳細については、[Specifying the Security Settings in the OpenAPI Specification](#specifying-the-security-settings-in-the-openapi-specification)をご参照ください。)

   Notice it has a **lock** icon and the other endpoints in this section do not.
   This is because this endpoint specified an operation-level
   `security requirement object` in the OpenAPI specification. (For details, see
   the
   [Specifying the Security Settings in the OpenAPI Specification](#specifying-the-security-settings-in-the-openapi-specification)
   section.)

2. `GET /users/me` セクションを展開し、`Try it out`をクリックします。指定するデータがないため、そのまま`Execute`します。先ほど指定したJWTトークン　　`Authorization` が、リクエストのヘッダーに自動的に配置されました。

  認証が成功すると、現在認証されているUserの[user profile](https://github.com/strongloop/loopback-next/blob/master/packages/security/src/types.ts)がレスポンスとしてで返されます。トークンの欠落/無効/期限切れが原因で認証が失敗した場合、 [HTTP 401 UnAuthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)が返されます。

  応答には、（データベースによって生成される）`id`フィールドと、フルネームのUser名が入っている`name` フィールドに、それぞれ一意の値が含まれています。
   ```json
   {"id": "5dd6acee242760334f6aef65", "name": "User One"}
   ```

## LoopBack4アプリケーションへのJWT認証の追加
このセクションでは 、[JSON Web Token (JWT)](https://en.wikipedia.org/wiki/JSON_Web_Token) ）アプローチを使用して、 どのように[loopback4-example-shopping](https://github.com/strongloop/loopback4-example-shopping)アプリケーションに`authentication` が追加されたのかを示します。

### @loopback/authentication のインストール

`loopback4-example-shopping` アプリケーションは **すでに** `@loopback/authentication`依存関係のセットアップを**package.json**内に持っています・

以下をのコマンドで、プロジェクトの依存関係としてインストールされました。

```sh
npm install --save @loopback/authentication
```

### AuthenticationComponentをアプリケーションに追加する

認証フレームワークの中核は、[AuthenticationComponent](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/authentication.component.ts)にあるため、`ShoppingApplication` クラス
[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)に、コンポーネントを追加することが重要です。

```ts
import {AuthenticationComponent} from '@loopback/authentication';

export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    // ...

    // Bind authentication component related elements
    this.component(AuthenticationComponent);

    // ...
  }
  // ...
}
```

### 認証デコレータでエンドポイントを保護する

アプリケーションのAPIエンドポイントをセキュリティで保護するには、[Authentication Decorator](../../decorators/Decorators_authenticate.md)で、コントローラー関数を修飾します。

デコレータの構文は次のとおりです。
```ts
@authenticate(strategyName: string, options?: object)
```

 `loopback4-example-shopping` アプリケーションでは、 保護されているエンドポイントが一つだけあります。
[loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)の`UserController` では、Userは、 `printCurrentUser()` よって処理される`/users/me`エンドポイントで`GET`リクエストすることにより、 自分のUserProfileをプリントできます。

```ts
  // ...

  @get('/users/me', {
    responses: {
      '200': {
        description: 'The current user profile',
        content: {
          'application/json': {
            schema: UserProfileSchema,
          },
        },
      },
    },
  })
  @authenticate('jwt')
  async printCurrentUser(
    @inject(SecurityBindings.USER)
    currentUserProfile: UserProfile,
  ): Promise<UserProfile> {
    currentUserProfile.id = currentUserProfile[securityId];
    delete currentUserProfile[securityId];
    return currentUserProfile;
  }

  // ...
```

{% include note.html content="このコントローラーメソッドは（コンストラクターインジェクション [constructor injection](../../Dependency-injection.md#constructor-injection)の代わりに、メソッドインジェクション [method injection](../../Dependency-injection.md#method-injection) を介してSecurityBindings.USERを取得しており、このメソッドは<b>@authenticate</b> デコレーターで装飾されているため、<b>@inject(SecurityBindings.USER, {optional:true})</b>を指定する必要はありません。詳細については、認証デコレータの使用 [Using the Authentication Decorator](../../Loopback-component-authentication.md#using-the-authentication-decorator) を参照してください。
" %}

 `/users/me` は、以下のように修飾されています。

```ts
@authenticate('jwt')
```

認証は有効なJWTトークンが`Authorization` リクエストのヘッダーで提供される場合にのみ、成功します 。

基本的に、カスタムシーケンス`MyAuthenticationSequence`(後述)の[AuthenticateFn](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/providers/auth-action.provider.ts)アクションは、`'jwt'` という名称で(これは`JWTAuthenticationStrategy` と呼ばれるものであり、後述されます)認証ストラテジを解決するために、
[AuthenticationStrategyProvider](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/providers/auth-strategy.provider.ts)を依頼します。
そして、認証をリクエストするために、`AuthenticateFn` は `JWTAuthenticationStrategy`の`authenticate(request)`をコールします。

提供されたJWTトークンが有効な場合、`JWTAuthenticationStrategy`の `authenticate(request)`関数は、UserProfileを返します。次に、`AuthenticateFn`は`SecurityBindings.USER` バインディングキーを使用して、リクエストコンテキストにUserProfileを配置します。

UserProfileは、`currentUserProfile: UserProfile` 変数内で、`printCurrentUser()`コントローラファンクションに対して使用できます。
その際、同じバインディングキー`SecurityBindings.USER`を介した、依存性注入を通じて行われます。
すると、UserProfileが以下のような応答で返されます。
`currentUserProfile: UserProfileSecurityBindings.USER`

JWTトークンが欠落/期限切れ/無効の場合、`JWTAuthenticationStrategy`の`authenticate(request)`機能は失敗し、[HTTP 401 UnAuthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401) がスローされます。

もし、`@authenticate`デコレータで **unknown** な認証ストラテジ名が指定されている場合は、
```ts
@authenticate('unknown')
```
となり、
[AuthenticationStrategyProvider](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/providers/auth-strategy.provider.ts)の`findAuthenticationStrategy(name: string)` ファンクションは、その登録された認証ストラテジ名では見つけられないとして、 [HTTP 401 UnAuthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)が返されます。

したがって、エンドポイントを`@authenticate`デコレーターで装飾するときは、正しい認証戦略名を必ず指定してください。


### カスタムシーケンスの作成と認証アクションの追加

REST APIエンドポイントを使用するLoopBack 4アプリケーションでは、各リクエストは[Sequence](../../Sequence.md)と呼ばれるアクションのステートレスグループを通過します。

しかし、認証はデフォルトのアクションシーケンスの一部では**ない**ため、カスタムシーケンスを作成して認証アクションを追加する必要があります。

[loopback4-example-shopping/packages/shopping/src/sequence.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/sequence.ts)のカスタムシーケンス`MyAuthenticationSequence` は、[SequenceHandler](https://github.com/strongloop/loopback-next/blob/master/packages/rest/src/sequence.ts)インターフェイスを実装しています。

```ts
export class MyAuthenticationSequence implements SequenceHandler {
  constructor(
    @inject(SequenceActions.FIND_ROUTE) protected findRoute: FindRoute,
    @inject(SequenceActions.PARSE_PARAMS)
    protected parseParams: ParseParams,
    @inject(SequenceActions.INVOKE_METHOD) protected invoke: InvokeMethod,
    @inject(SequenceActions.SEND) protected send: Send,
    @inject(SequenceActions.REJECT) protected reject: Reject,
    @inject(AuthenticationBindings.AUTH_ACTION)
    protected authenticateRequest: AuthenticateFn,
  ) {}

  async handle(context: RequestContext) {
    try {
      const {request, response} = context;
      const route = this.findRoute(request);

      //call authentication action
      await this.authenticateRequest(request);

      // Authentication successful, proceed to invoke controller
      const args = await this.parseParams(request, route);
      const result = await this.invoke(route, args);
      this.send(response, result);
    } catch (error) {
      //
      // The authentication action utilizes a strategy resolver to find
      // an authentication strategy by name, and then it calls
      // strategy.authenticate(request).
      //
      // The strategy resolver throws a non-http error if it cannot
      // resolve the strategy. When the strategy resolver obtains
      // a strategy, it calls strategy.authenticate(request) which
      // is expected to return a user profile. If the user profile
      // is undefined, then it throws a non-http error.
      //
      // It is necessary to catch these errors and add HTTP-specific status
      // code property.
      //
      // Errors thrown by the strategy implementations already come
      // with statusCode set.
      //
      // In the future, we want to improve `@loopback/rest` to provide
      // an extension point allowing `@loopback/authentication` to contribute
      // mappings from error codes to HTTP status codes, so that application
      // don't have to map codes themselves.
      if (
        error.code === AUTHENTICATION_STRATEGY_NOT_FOUND ||
        error.code === USER_PROFILE_NOT_FOUND
      ) {
        Object.assign(error, {statusCode: 401 /* Unauthorized */});
      }

      this.reject(context, error);
      return;
    }
  }
}
```
認証アクション/機能は、
`AuthenticationBindings.AUTH_ACTION`バインディングキーを介して注入され、
`authenticateRequest`の名前が指定され 
[AuthenticateFn](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/types.ts)を持ちます。

コール中
```ts
await this.authenticateRequest(request);
```

コール前
```ts
// ...
const result = await this.invoke(route, args);
this.send(response, result);
// ...
```
コントローラのエンドポイントに到達する前に、認証が成功したことを確認します。

 `MyAuthenticationSequence`アプリケーションにカスタムシーケンスを追加するには、loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)で次のコードをコーディングする必要があります。

```ts
export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    // ...

    // Set up the custom sequence
    this.sequence(MyAuthenticationSequence);

    // ...
  }
}
```


### カスタムJWT認証戦略の作成

カスタム認証戦略を作成する場合、[AuthenticationStrategy](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/types.ts)インターフェースを実装する必要があります。
[loopback4-example-shopping/packages/shopping/src/authentication-strategies/jwt-strategy.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/authentication-strategies/jwt-strategy.ts)のカスタムJWT認証ストラテジ`JWTAuthenticationStrategy` iは 、次のように実装されました。

```ts
import {inject} from '@loopback/context';
import {HttpErrors, Request} from '@loopback/rest';
import {AuthenticationStrategy, TokenService} from '@loopback/authentication';
import {UserProfile} from '@loopback/security';

import {TokenServiceBindings} from '../keys';

export class JWTAuthenticationStrategy implements AuthenticationStrategy {
  name: string = 'jwt';

  constructor(
    @inject(TokenServiceBindings.TOKEN_SERVICE)
    public tokenService: TokenService,
  ) {}

  async authenticate(request: Request): Promise<UserProfile | undefined> {
    const token: string = this.extractCredentials(request);
    const userProfile: UserProfile = await this.tokenService.verifyToken(token);
    return userProfile;
  }

  extractCredentials(request: Request): string {
    if (!request.headers.authorization) {
      throw new HttpErrors.Unauthorized(`Authorization header not found.`);
    }

    // for example: Bearer xxx.yyy.zzz
    const authHeaderValue = request.headers.authorization;

    if (!authHeaderValue.startsWith('Bearer')) {
      throw new HttpErrors.Unauthorized(
        `Authorization header is not of type 'Bearer'.`,
      );
    }

    //split the string into 2 parts: 'Bearer ' and the `xxx.yyy.zzz`
    const parts = authHeaderValue.split(' ');
    if (parts.length !== 2)
      throw new HttpErrors.Unauthorized(
        `Authorization header value has too many parts. It must follow the pattern: 'Bearer xx.yy.zz' where xx.yy.zz is a valid JWT token.`,
      );
    const token = parts[1];

    return token;
  }
}
```

**name**に`'jwt'` を持ち、`async authenticate(request: Request): Promise<UserProfile | undefined>` ファンクションを実装しています。

JWTトークンを抽出するための追加機能`extractCredentials(request: Request): string` が追加されました。この認証戦略では、すべての要求が`Authorization`ヘッダーで有効なJWTトークンを渡すことを想定しています。

`JWTAuthenticationStrategy` はまた、`TokenServiceBindings.TOKEN_SERVICE` バインディングキーを介して注入される `TokenService`タイプの `tokenService`を利用します。
これは、JWTトークンの有効性を検証し、UserProfileを返すために使用されます。
このトークンサービスについては、後のセクションで説明します。


### カスタムJWT認証戦略の登録
カスタム認証戦略`JWTAuthenticationStrategy`を、 `'jwt'`**名称**で認証フレームワークの一環として登録するには、[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts).で次のようにコーディングする必要があります。

```ts
import {registerAuthenticationStrategy} from '@loopback/authentication';

export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);
    // ...
    registerAuthenticationStrategy(this, JWTAuthenticationStrategy);
    // ...
  }
}
```

### トークンサービスの作成

[loopback4-example-shopping/packages/shopping/src/services/jwt-service.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/services/jwt-service.ts)のトークンサービス `JWTService` は、**オプション** のヘルパー[TokenService](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/services/token.service.ts)インターフェイスを実装します。

```ts
import {inject} from '@loopback/context';
import {HttpErrors} from '@loopback/rest';
import {promisify} from 'util';
import {TokenService} from '@loopback/authentication';
import {UserProfile} from '@loopback/security';
import {TokenServiceBindings} from '../keys';

const jwt = require('jsonwebtoken');
const signAsync = promisify(jwt.sign);
const verifyAsync = promisify(jwt.verify);

export class JWTService implements TokenService {
  constructor(
    @inject(TokenServiceBindings.TOKEN_SECRET)
    private jwtSecret: string,
    @inject(TokenServiceBindings.TOKEN_EXPIRES_IN)
    private jwtExpiresIn: string,
  ) {}

  async verifyToken(token: string): Promise<UserProfile> {
    if (!token) {
      throw new HttpErrors.Unauthorized(
        `Error verifying token: 'token' is null`,
      );
    }

    let userProfile: UserProfile;

    try {
      // decode user profile from token
      const decryptedToken = await verifyAsync(token, this.jwtSecret);
      // don't copy over  token field 'iat' and 'exp', nor 'email' to user profile
      userProfile = Object.assign(
        {id: '', name: ''},
        {id: decryptedToken.id, name: decryptedToken.name},
      );
    } catch (error) {
      throw new HttpErrors.Unauthorized(
        `Error verifying token: ${error.message}`,
      );
    }

    return userProfile;
  }

  async generateToken(userProfile: UserProfile): Promise<string> {
    if (!userProfile) {
      throw new HttpErrors.Unauthorized(
        'Error generating token: userProfile is null',
      );
    }

    // Generate a JSON Web Token
    let token: string;
    try {
      token = await signAsync(userProfile, this.jwtSecret, {
        expiresIn: Number(this.jwtExpiresIn),
      });
    } catch (error) {
      throw new HttpErrors.Unauthorized(`Error encoding token: ${error}`);
    }

    return token;
  }
}
```

`JWTService` は [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)の `sign` ・`verify`ファンクションを使用して、JWT tokens を生成・検証します。

It makes use of `jwtSecret` and `jwtExpiresIn` **string** values that are
injected via the `TokenServiceBindings.TOKEN_SECRET` and the
`TokenServiceBindings.TOKEN_EXPIRES_IN` binding keys respectively.

この `async generateToken(userProfile: UserProfile): Promise<string>` 関数は、
[UserProfile](https://github.com/strongloop/loopback-next/blob/master/packages/security/src/types.ts)型のUserProfileを 受け取り、ペイロードとしての **user profile** 、**jwtSecret**、そして **jwtExpiresIn**を使用して`string` 型のJWTトークンを生成します。

この `async verifyToken(token: string): Promise<UserProfile>` 関数は、`string`タイプのJWTトークンを受け取り、JWTトークンを検証し、`UserProfile`タイプのUserProfileであるトークンのペイロードを返します。

JWT `secret`、 `expires in` 値、および`JWTService`クラスをバインディングキーにバインドするには、[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)で次のようにコーディングする必要があります。

```ts
export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    // ...
    this.setUpBindings();
    // ...
  }

  setUpBindings(): void {
    // ...

    this.bind(TokenServiceBindings.TOKEN_SECRET).to(
      TokenServiceConstants.TOKEN_SECRET_VALUE,
    );

    this.bind(TokenServiceBindings.TOKEN_EXPIRES_IN).to(
      TokenServiceConstants.TOKEN_EXPIRES_IN_VALUE,
    );

    this.bind(TokenServiceBindings.TOKEN_SERVICE).toClass(JWTService);

    // ...
  }
}
```
上記のコードで`TOKEN_SECRET_VALUE` は、`'myjwts3cr3t'`の値を、`TOKEN_EXPIRES_IN_VALUE`は`'600'`の値を持っています。

`JWTService` は、アプリケーション内の2つの場所で使用されています：
- `JWTAuthenticationStrategy` [loopback4-example-shopping/packages/shopping/src/authentication-strategies/jwt-strategy.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/authentication-strategies/jwt-strategy.ts)
- `UserController` [loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)

### Userサービスの作成

[loopback4-example-shopping/packages/shopping/src/services/user-service.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/services/user-service.ts)のUserサービス`MyUserService`は、  **追加の**ヘルパー
[UserService](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/src/services/user.service.ts)インターフェイスを実装します。

```ts
export class MyUserService implements UserService<User, Credentials> {
  constructor(
    @repository(UserRepository) public userRepository: UserRepository,
    @inject(PasswordHasherBindings.PASSWORD_HASHER)
    public passwordHasher: PasswordHasher,
  ) {}

  async verifyCredentials(credentials: Credentials): Promise<User> {
    const foundUser = await this.userRepository.findOne({
      where: {email: credentials.email},
    });

    if (!foundUser) {
      throw new HttpErrors.NotFound(
        `User with email ${credentials.email} not found.`,
      );
    }
    const passwordMatched = await this.passwordHasher.comparePassword(
      credentials.password,
      foundUser.password,
    );

    if (!passwordMatched) {
      throw new HttpErrors.Unauthorized('The credentials are not correct.');
    }

    return foundUser;
  }

  convertToUserProfile(user: User): UserProfile {
    // since first name and lastName are optional, no error is thrown if not provided
    let userName = '';
    if (user.firstName) userName = `${user.firstName}`;
    if (user.lastName)
      userName = user.firstName
        ? `${userName} ${user.lastName}`
        : `${user.lastName}`;
    return {id: user.id, name: userName};
  }
}
```
 `async verifyCredentials(credentials: Credentials): Promise<User>` 関数は[Credentials](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/repositories/user.repository.ts)型の資格情報を取り込み、[User](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/models/user.model.ts)タイプの **user**を返します。
そして、`UserRepository`タイプの、インジェクトされたUserリポジトリを検索します。

この`convertToUserProfile(user: User): UserProfile` 関数は、  `User`タイプの **user**を取り込み、[UserProfile](https://github.com/strongloop/loopback-next/blob/master/packages/security/src/types.ts)タイプのuser profileを返します 。この場合、一つのuser profileは、認証済みUserを識別する、Userプロパティの最小セットです。

`MyUserService`は、 `UserController`に[loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)で使用されています。

MyUserService` クラスと、それが使用するパスワードハッシュユーティリティをバインドするには、
[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)で、下記のようにコーディングする必要があります。

```ts
export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    // ...

    this.setUpBindings();

    // ...
  }

  setUpBindings(): void {
    // ...

    // Bind bcrypt hash services - utilized by 'UserController' and 'MyUserService'
    this.bind(PasswordHasherBindings.ROUNDS).to(10);
    this.bind(PasswordHasherBindings.PASSWORD_HASHER).toClass(BcryptHasher);

    this.bind(UserServiceBindings.USER_SERVICE).toClass(MyUserService);

    // ...
  }
}
```

### Userを追加する

[loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)の`UserController` クラスでは、 `create()`関数で `/users` エンドポイントに`POST`リクエストを送ることで、userを追加できます。

パスワードなどのUser資格情報はメインUserProfileの外部に保存されるため、新しいモデル(
[Data Transfer Object](https://en.wikipedia.org/wiki/Data_transfer_object))を作成して、新しいUserの作成に必要なデータを記述する必要があります。クラスは `User`モデルから継承して、すべてのuser profileプロパティを含めます。また、クライアントがパスワードも指定できるようにするために、新しいプロパティ `password`も追加します。

```ts
@model()
export class NewUserRequest extends User {
  @property({
    type: 'string',
    required: true,
  })
  password: string;
}
```
したがって、コントローラーメソッド`UserController.create`は、`password` といった追加のプロパティをデータをリポジトリ（およびデータベース）に渡す前に、削除する必要があります。
The controller method `UserController.create` then has to remove additional
properties like `password` before passing the data to Repository (and database).

```ts
export class UserController {
  constructor(
    // ...
    @repository(UserRepository) public userRepository: UserRepository,
    @inject(PasswordHasherBindings.PASSWORD_HASHER)
    public passwordHasher: PasswordHasher,
    @inject(TokenServiceBindings.TOKEN_SERVICE)
    public jwtService: TokenService,
    @inject(UserServiceBindings.USER_SERVICE)
    public userService: UserService<User, Credentials>,
  ) {}

  // ...

  @post('/users')
  async create(
    @requestBody({
      content: {
        'application/json': {
          schema: getModelSchemaRef(NewUserRequest, {
            title: 'NewUser',
          }),
        },
      },
    })
    newUserRequest: NewUserRequest,
  ): Promise<User> {
    // ensure a valid email value and password value
    validateCredentials(_.pick(newUserRequest, ['email', 'password']));

    // encrypt the password
    const password = await this.passwordHasher.hashPassword(
      newUserRequest.password,
    );

    // create the new user
    const savedUser = await this.userRepository.create(
      _.omit(newUserRequest, 'password'),
    );

    // set the password
    await this.userRepository
      .userCredentials(savedUser.id)
      .create({password});

    return savedUser;
  }

  // ...
```

Userの電子メールとパスワードの値が許容可能な形式である場合、[User](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/models/user.model.ts)
タイプのuserは、Userリポジトリを介してデータベースに追加されます。


### ログイン成功時にJWTトークンを発行する

[loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)の `UserController` クラスで、`login()`関数で定義された `/users/login` エンドポイントに`POST` リクエストを行うことで、Userは `log in` できます。
 リクエストには、`email` と`password`を含んでいます。

```ts
export class UserController {
  constructor(
    // ...
    @repository(UserRepository) public userRepository: UserRepository,
    @inject(PasswordHasherBindings.PASSWORD_HASHER)
    public passwordHasher: PasswordHasher,
    @inject(TokenServiceBindings.TOKEN_SERVICE)
    public jwtService: TokenService,
    @inject(UserServiceBindings.USER_SERVICE)
    public userService: UserService<User, Credentials>,
  ) {}

  // ...

  @post('/users/login', {
    responses: {
      '200': {
        description: 'Token',
        content: {
          'application/json': {
            schema: {
              type: 'object',
              properties: {
                token: {
                  type: 'string',
                },
              },
            },
          },
        },
      },
    },
  })
  async login(
    @requestBody(CredentialsRequestBody) credentials: Credentials,
  ): Promise<{token: string}> {
    // ensure the user exists, and the password is correct
    const user = await this.userService.verifyCredentials(credentials);

    // convert a User object into a UserProfile object (reduced set of properties)
    const userProfile = this.userService.convertToUserProfile(user);

    // create a JSON Web Token based on the user profile
    const token = await this.jwtService.generateToken(userProfile);

    return {token};
  }
}
```

UserServiceは、電子メールとパスワードが有効であると確認されると、Userオブジェクトを返します。そうでない場合は、[HTTP 401 UnAuthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)をスローします。次に、UserServiceを呼び出して、Userオブジェクトから、よりスリムなUserProfileを作成します。次に、このUserProfileは、トークンサービスによって作成されたJWTトークンのペイロードとして使用されます。トークンは応答で返されます。


### OpenAPI仕様での、セキュリティ設定の指定

shopping cart アプリケーションでは、1つのエンドポイント、 `GET /users/me`のみがカスタムJWT認証戦略で保護されています。`API Explorer` のJWT tokenを、REST APIクライアントを使用する形でなく）`set`、`use` できるようにするためには、
- [security scheme object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#security-scheme-object)と
- [security requirement object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#securityRequirementObject)
の情報を、アプリケーションのOpenAPI仕様に指定する必要があります。

[loopback4-example-shopping/packages/shopping/src/utils/security-spec.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/utils/security-spec.ts)では、以下のように定義されています。

```ts
import {SecuritySchemeObject, ReferenceObject} from '@loopback/openapi-v3';

export const OPERATION_SECURITY_SPEC = [{bearerAuth: []}];
export type SecuritySchemeObjects = {
  [securityScheme: string]: SecuritySchemeObject | ReferenceObject;
};
export const SECURITY_SCHEME_SPEC: SecuritySchemeObjects = {
  bearerAuth: {
    type: 'http',
    scheme: 'bearer',
    bearerFormat: 'JWT',
  },
};
```

`SECURITY_SCHEME_SPEC` は、アプリケーションに対してグローバルに定義されている、セキュリティスキームオブジェクト定義のマップです。ここでは、[bearerAuth](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#jwt-bearer-sample)定義を含む、単一のセキュリティスキームオブジェクトのみが含まれています。

`OPERATION_SECURITY_SPEC`は、`bearerAuth` セキュリティスキームオブジェクト定義 を参照する、**operation-level**の[security requirement object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#securityRequirementObject)です。[loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)の`/users/me` エンドポイントで使用されます。

以下の行の

```
security: OPERATION_SECURITY_SPEC,
```

以下のコードに注意してください。

```ts
@get('/users/me', {
  security: OPERATION_SECURITY_SPEC,
  responses: {
    '200': {
      description: 'The current user profile',
      content: {
        'application/json': {
          schema: UserProfileSchema,
        },
      },
    },
  },
})
@authenticate('jwt')
async printCurrentUser(
  @inject(SecurityBindings.USER)
  currentUserProfile: UserProfile,
): Promise<UserProfile> {
  currentUserProfile.id = currentUserProfile[securityId];
  delete currentUserProfile[securityId];
  return currentUserProfile;
}
```

[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)は、下記のように、 `security scheme object` 定義をOpenAPIの仕様に寄与します。

```ts
import {SECURITY_SCHEME_SPEC} from './utils/security-spec';

export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    this.api({
      openapi: '3.0.0',
      info: {title: pkg.name, version: pkg.version},
      paths: {},
      components: {securitySchemes: SECURITY_SCHEME_SPEC},
      servers: [{url: '/'}],
    });
// ...
```

後に、アプリケーションの実行中に[http://[::1]:3000/openapi.json](http://[::1]:3000/openapi.json) にアクセスすると、 `bearerAuth`テキストを検索します。すると、次の2つのオカレンスが見つかります。

```
"components": {
    "securitySchemes": {
      "bearerAuth": {
        "type": "http",
        "scheme": "bearer",
        "bearerFormat": "JWT"
      }
    },
```

そして

```
"/users/me": {
      "get": {
        "x-controller-name": "UserController",
        "x-operation-name": "printCurrentUser",
        "tags": [
          "UserController"
        ],
        "security": [
          {
            "bearerAuth": []
          }
        ],
```

です。さらにその後、アプリケーションの実行中に [http://[::1]:3000/explorer/](http://[::1]:3000/explorer/)にアクセスすると、上部に `Authorize` ボタンが表示されます。

![](../../imgs/api_explorer_authorize_button.png)

さらに、`UserController` の `GET /users/me` エンドポイントに、**ロック**アイコンも現れます。

![](../../imgs/api_explorer_usercontroller_section1.png)

### すべてのエンドポイントに単一のOpenAPI仕様セキュリティ要件オブジェクトを指定する方法

現在の `loopback4-example-shopping`アプリケーションは未実装ではあるものの、同じOpenAPI仕様のセキュリティ要件オブジェクトを、アプリケーションのすべてのエンドポイントに指定する方法があります。

セキュリティスキームオブジェクトの定義はまだ`components`セクションで定義されていますが、`security`プロパティの値は、**operation**レベルではなく **top** レベルで 設定されます。

```
"components": {
    "securitySchemes": {
      "bearerAuth": {
        "type": "http",
        "scheme": "bearer",
        "bearerFormat": "JWT"
      }
    },
  // ...
},
"security": [
    {
      "bearerAuth": []
    }
  ]
```

これを実現するには、前述のコード例に若干の変更を加えるだけです。

[loopback4-example-shopping/packages/shopping/src/utils/security-spec.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/utils/security-spec.ts)で、 `OPERATION_SECURITY_SPEC` を `SECURITY_SPEC`にリネイムするだけです。

```ts
import {SecuritySchemeObject, ReferenceObject} from '@loopback/openapi-v3';

export const SECURITY_SPEC = [{bearerAuth: []}];
export type SecuritySchemeObjects = {
  [securityScheme: string]: SecuritySchemeObject | ReferenceObject;
};
export const SECURITY_SCHEME_SPEC: SecuritySchemeObjects = {
  bearerAuth: {
    type: 'http',
    scheme: 'bearer',
    bearerFormat: 'JWT',
  },
};
```

[loopback4-example-shopping/packages/shopping/src/controllers/user.controller.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/controllers/user.controller.ts)で、以下の行を削除します。

```
security: SECURITY_SPEC_OPERATION,
```

from the `/users/me` endpoint:

```ts
@get('/users/me', {
  responses: {
    '200': {
      description: 'The current user profile',
      content: {
        'application/json': {
          schema: UserProfileSchema,
        },
      },
    },
  },
})
@authenticate('jwt')
async printCurrentUser(
  @inject(SecurityBindings.USER)
  currentUserProfile: UserProfile,
): Promise<UserProfile> {
  currentUserProfile.id = currentUserProfile[securityId];
  delete currentUserProfile[securityId];
  return currentUserProfile;
}
```

[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)で、以下の行を追加します。

```
security: SECURITY_SPEC
```

 `this.api({...})`を呼び出すためです。これは基本的に、セキュリティ要件オブジェクト定義`bearerAuth`が、すべてのエンドポイントに適用されることを意味します。 
 
```ts
import {SECURITY_SCHEME_SPEC, SECURITY_SPEC} from './utils/security-spec';

export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    this.api({
      openapi: '3.0.0',
      info: {title: pkg.name, version: pkg.version},
      paths: {},
      components: {securitySchemes: SECURITY_SCHEME_SPEC},
      servers: [{url: '/'}],
      security: SECURITY_SPEC
    });
// ...
```

Visiting [http://[::1]:3000/explorer/](http://[::1]:3000/explorer/) while the
application is running, you should still see an `Authorize` button at the top as
before.
アプリケーションの実行中に [http://[::1]:3000/explorer/](http://[::1]:3000/explorer/) を開くと、まだ`Authorize` ボタンがあるはずです。

![](../../imgs/api_explorer_authorize_button.png)

しかし、今回は **全ての** エンドポイントにロックアイコンがあります。

![](../../imgs/api_explorer_all_sections_lock_icons1.png)

これは、`Authorize button/dialog`を介してJWTトークンを1回設定でき、対向するすべてのエンドポイントでトークンを使用できることを意味します。

なお、extensionPoint/extensions パターン ([Issue #3854](https://github.com/strongloop/loopback-next/issues/3854) )を介した、OpenAPI仕様への貢献を許可する計画があります。認証方式がセキュリティスキーム/要件オブジェクト情報に自動的に提供されるようにすることを含んでいます（[Issue #3669](https://github.com/strongloop/loopback-next/issues/3669) )。


### サマリ

ここまで、JWT `authentication` を`loopback4-example-shopping` アプリケーションに追加し、`security scheme/requirement object`設定を、OpenAPI仕様に追加するためにの一連の手順を実行してきました。

[loopback4-example-shopping/packages/shopping/src/application.ts](https://github.com/strongloop/loopback4-example-shopping/blob/master/packages/shopping/src/application.ts)の`ShoppingApplication` クラスの最終形は、以下のようになります。

```ts
import {BootMixin} from '@loopback/boot';
import {ApplicationConfig, BindingKey} from '@loopback/core';
import {RepositoryMixin} from '@loopback/repository';
import {RestApplication} from '@loopback/rest';
import {ServiceMixin} from '@loopback/service-proxy';
import {MyAuthenticationSequence} from './sequence';
import {
  RestExplorerBindings,
  RestExplorerComponent,
} from '@loopback/rest-explorer';
import {
  TokenServiceBindings,
  UserServiceBindings,
  TokenServiceConstants,
} from './keys';
import {JWTService} from './services/jwt-service';
import {MyUserService} from './services/user-service';

import path from 'path';
import {
  AuthenticationComponent,
  registerAuthenticationStrategy,
} from '@loopback/authentication';
import {PasswordHasherBindings} from './keys';
import {BcryptHasher} from './services/hash.password.bcryptjs';
import {JWTAuthenticationStrategy} from './authentication-strategies/jwt-strategy';
import {SECURITY_SCHEME_SPEC} from './utils/security-spec';

/**
 * Information from package.json
 */
export interface PackageInfo {
  name: string;
  version: string;
  description: string;
}
export const PackageKey = BindingKey.create<PackageInfo>('application.package');

const pkg: PackageInfo = require('../package.json');

export class ShoppingApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options?: ApplicationConfig) {
    super(options);

    this.api({
      openapi: '3.0.0',
      info: {title: pkg.name, version: pkg.version},
      paths: {},
      components: {securitySchemes: SECURITY_SCHEME_SPEC},
      servers: [{url: '/'}],
    });

    this.setUpBindings();

    // Bind authentication component related elements
    this.component(AuthenticationComponent);

    registerAuthenticationStrategy(this, JWTAuthenticationStrategy);

    // Set up the custom sequence
    this.sequence(MyAuthenticationSequence);

    // Set up default home page
    this.static('/', path.join(__dirname, '../public'));

    // Customize @loopback/rest-explorer configuration here
    this.bind(RestExplorerBindings.CONFIG).to({
      path: '/explorer',
    });
    this.component(RestExplorerComponent);

    this.projectRoot = __dirname;
    // Customize @loopback/boot Booter Conventions here
    this.bootOptions = {
      controllers: {
        // Customize ControllerBooter Conventions here
        dirs: ['controllers'],
        extensions: ['.controller.js'],
        nested: true,
      },
    };
  }

  setUpBindings(): void {
    // Bind package.json to the application context
    this.bind(PackageKey).to(pkg);

    this.bind(TokenServiceBindings.TOKEN_SECRET).to(
      TokenServiceConstants.TOKEN_SECRET_VALUE,
    );

    this.bind(TokenServiceBindings.TOKEN_EXPIRES_IN).to(
      TokenServiceConstants.TOKEN_EXPIRES_IN_VALUE,
    );

    this.bind(TokenServiceBindings.TOKEN_SERVICE).toClass(JWTService);

    // // Bind bcrypt hash services
    this.bind(PasswordHasherBindings.ROUNDS).to(10);
    this.bind(PasswordHasherBindings.PASSWORD_HASHER).toClass(BcryptHasher);

    this.bind(UserServiceBindings.USER_SERVICE).toClass(MyUserService);
  }
}
```

## 完成形のアプリケーションの実行

完成したアプリケーションを実行するには、[Try it out](#try-it-out)セクションの指示に従って ください。
詳細については、[Authentication Component](../../Loopback-component-authentication.md)をご覧ください 。

## バグ/フィードバック

[loopback4-example-shopping](https://github.com/strongloop/loopback4-example-shopping)でイシューをオープンしてください。こちらで確認します！

## 貢献する

- [Guidelines](https://github.com/strongloop/loopback-next/blob/master/docs/CONTRIBUTING.md)
- [Join the team](https://github.com/strongloop/loopback-next/issues/110)

## テスト

`npm test`フォルダから実行します。

## Contributors

[all contributors](https://github.com/strongloop/loopback-next/graphs/contributors)をご覧ください。

## ライセンス

MIT
