---
title: Rails + Next.js + Firebase V9 Authentication で認証付きのCRUDアプリを作る
category: "Rails"
tags: [React, Next.js, Rails, API, Firebase, Typescript]
---

ソースコード：[Backend: Rails API](https://github.com/lei900/rails-fairebase-auth)、[Frontend: Next.js](https://github.com/lei900/rails-fairebase-auth)

デモページ：[next-firebase-auth-sample-app](https://next-firebase-auth-sample-seven.vercel.app/)

### 利用技術

フロントエンド

- Next.js
- TypeScript
- Tailwind CSS

バックエンド

- Rails 7.0.4(API モード)

認証部分

- Firebase Authentication(V9) - Google ログインのみ

# 背景

最近、個人 PF で使う認証方式を検討している。最初は Auth0 を使おうと思っていたけど、実装で躓いて、代替案を探していたとき、Firebase Authentication が目に留まった。

電話認証以外は基本的に無料で使えるし、利用できる認証方法も非常に豊富で、特に Auth0 が対応していない匿名認証もできる点に惹かれた。

> 特徴的なのは匿名認証で、一時的にユニークな ID を付与したユーザーとして扱いますが、その後他の認証方式に昇格することができます。

参照：[ちょっとでもセキュリティに自信がないなら、 Firebase Authentication を検討しよう](https://mizchi.dev/202008172159-firebase-authentication)

<details> 
<summary>実装でハマった話</summary>

Firebase Authentication を実装したところ、Auth0 の時と同じ、実装の仕組みやコードの意図が理解できず、一時的ハマった。

ちょっと焦っていたところ、この動画チュートリアルに救われた。

[NextJS Firebase Auth Tutorial • How to Authenticate Users for Your App](https://www.youtube.com/watch?v=BQrE98bP6m4)

今ままで参考にした記事はちょっと古いので、公式の記述と一致しないコードもあり、この動画で使用している Firebase SDK は最新の V9 バージョンで助かった。

また、JWT や認証の流れもわかりやすく説明されており、おかげでなんとなく仕組みがわかるようになった。

今回も、Auth0 の時も、コードが理解できなかったのは、そもそも JWT や認証の仕組みが全然わかっていないからだとわかった。(React Context API についての理解不足も原因の一つ..)

その後、検索キーワードを変えて、他の記事も見つけて、ようやく Rails 側のコードも理解できるようになった。

コードの全体流れが理解できたので、参照したソースコードを自分なりに少し書き換えてみた。自分の理解はメモとして整理したいと思う。

</details>

# 認証機能全体の流れ

1.  画面上のログインボタン押して、Google ログイン画面に遷移する。遷移形式は popup か redirect。
2.  Firebase 側は Google から送られてきた ID Token を検証する。成功すれば、Firebase 側で user_id を生成し、ID Token を発行する。
3.  Next.js 側は Firebase が発行する ID Token を取得して、Rails 側に送る。
4.  Rails 側で Firebase ID Token を検証する。Firebase が発行した user_id を利用して、ユーザーを新規登録か既存ユーザーを特定する。

- ユーザー情報の利用について

  Firebase 側で自動登録するユーザー個人情報はデフォルトで、email, displayName, photoUrl のみ

  追加情報必要なら、下記のいずれの処理が必要

  - Google アカウント関連の情報なら Google Access Token を使って取得する
  - Firebase Admin SDK 使って独自の情報コラムを追加する。
  - ユーザー個人情報を自前の DB で保存するなら、Rails 側で独自処理する

# Firebase Authentication の初期設定

Firebase のアカウント取得やプロジェクト作成などの初期設定は他の記事を参照できるので、ここで省略。

今回はユーザー利便性を考え、メール・パスワード形式を使わず、ソーシャルログインと匿名認証のみを実装予定で、まずは Google ログインを実装する。

参照：[Firebase の初期設定](https://reffect.co.jp/react/react18-firebase9-auth)

# Next.js 側の実装

まずは Firebase が発行した各種 key を環境変数に設定する

```ini
NEXT_PUBLIC_FIREBASE_API_KEY=<YOUR_API_KEY>;
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=<YOUR_DOMAIN>;
NEXT_PUBLIC_FIREBASE_PROJECT_ID=<YOUR_PROJECT_ID>;
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=<YOUR_STORAGE_BUCKET>;
NEXT_PUBLIC_FIREBASE_MESSEGING_SENDER_ID=<YOUR_MESSEGING_SENDER_ID>;
NEXT_PUBLIC_FIREBASE_APP_ID=<YOUR_APP_ID>;
```

{: file=".env.local" }

※ この変数はブラウザ 側で処理するので、変数名には`NEXT_PUBLIC`を追加する必要  
 [#exposing-environment-variables-to-the-browser](https://nextjs.org/docs/basic-features/environment-variables#exposing-environment-variables-to-the-browser)

そして、Firebase SDK インストール `npm install --save firebase`

## Firebase 初期化と Firebase App オブジェクトを作成する

```Typescript
import { initializeApp, getApps, getApp } from "firebase/app";
import { getAuth } from "firebase/auth";

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSEGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

// Firebase初期化し、FirebaseAppオブジェクトを作成
// appが既に存在する場合、そのappを取得する
const getFirebaseApp = () => {
  if (getApps().length === 0) {
    return initializeApp(firebaseConfig);
  } else {
    return getApp();
  }
};

const app = getFirebaseApp();

// FirebaseAppに関連付けられたAuthインスタンスを取得
export const auth = getAuth(app);
```

{: file="lib/initFirebase.ts" }

## ログイン関数作成

Firebase が提供する`GoogleAuthProvider`と`signInWithPopup`を利用する

```Typescript
import { signInWithPopup, GoogleAuthProvider } from "firebase/auth";
import { useRouter } from "next/router";

import { auth } from "../lib/initFirebase";

const loginWithGoogle = async () => {
  const provider = new GoogleAuthProvider();
  const result = await signInWithPopup(auth, provider);

  if (result) {
    // ログインしたユーザー情報を取得する
    const user = result.user;
    // Google APIを直接利用したいなら、Google Access Tokenを取得できる
    const credential = GoogleAuthProvider.credentialFromResult(result);
    const token = credential?.accessToken;
    // ログイン成功後、ホームページにリダイレクト
    router.push("/");
    return user;
  }
};
```

- `user`から取得できる情報

  ```Typescript
  displayName: string | null; // ユーザー表示名
  email: string | null; // ユーザーメール
  phoneNumber: string | null; // ユーザー電話番号
  photoURL: string | null; // Googleプロフィール写真URL
  uid: string; // Firebaseが生成するユニークID
  ```

ちなみに、Google ログインページへの遷移を Redirect にしたいなら、`signInWithRedirect`を利用する

```Typescript
import { signInWithRedirect, GoogleAuthProvider } from "firebase/auth";

const loginWithGoogle = async () => {
  const provider = new GoogleAuthProvider();
  await signInWithRedirect(auth, provider);

  const result = await getRedirectResult(auth);
  if (result) {
    const user = result.user;
    ...
  }
}
```

## ユーザーログイン状態の変化を監視する

Firebase が提供する`onAuthStateChanged`関数を使って、ユーザーログイン状態を監視することができる。

`onAuthStateChanged`関数の説明

```typescript
// Adds an observer for changes to the user's sign-in state.
// @param auth — The Auth instance.
// @param nextOrObserver — callback triggered on change.
onAuthStateChanged(auth: Auth, nextOrObserver: NextOrObserver<User>): Unsubscribe
```

`useFirebaseAuth()`関数を作成する

```Typescript
import { useState, useEffect } from "react";
import { User, onAuthStateChanged } from "firebase/auth";
import { useRouter } from "next/router";

import { auth } from "lib/initFirebase";

export default function useFirebaseAuth() {
  const [currentUser, setCurrentUser] = (useState < User) | (null > null);
  const [loading, setLoading] = useState(true);

  // listen for Firebase state change
  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, nextOrObserver);
    return unsubscribe;
  }, []);

  const nextOrObserver = async (user: User | null) => {
    if (!user) {
      setLoading(false);
      return;
    }

    setLoading(true);
    setCurrentUser(user);
    setLoading(false);
  };
}
```

## ログアウト関数追加

```Typescript
const clear = () => {
  setCurrentUser(null);
  setLoading(false);
};

const logout = () => signOut(auth).then(clear);
```

`useFirebaseAuth()`の全体像

```Typescript
import { useState, useEffect } from "react";
import {
  User,
  onAuthStateChanged,
  signOut,
  signInWithPopup,
  GoogleAuthProvider,
} from "firebase/auth";
import { useRouter } from "next/router";

import { auth } from "lib/initFirebase";

export default function useFirebaseAuth() {
  const [currentUser, setCurrentUser] = (useState < User) | (null > null);
  const [loading, setLoading] = useState(true);
  const router = useRouter();

  const loginWithGoogle = async () => {
    const provider = new GoogleAuthProvider();
    const result = await signInWithPopup(auth, provider);

    if (result) {
      const user = result.user;

      router.push("/");
      return user;
    }
  };

  const clear = () => {
    setCurrentUser(null);
    setLoading(false);
  };

  const logout = () => signOut(auth).then(clear);

  const nextOrObserver = async (user: User | null) => {
    if (!user) {
      setLoading(false);
      return;
    }

    setLoading(true);
    setCurrentUser(user);
    setLoading(false);
  };

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, nextOrObserver);
    return unsubscribe;
  }, []);

  return {
    currentUser,
    loading,
    loginWithGoogle,
    logout,
  };
}
```

{: file="hooks/useFirebaseAuth.ts" }

## user context を作成する

ユーザー情報を app 内で共有するため、`AuthContext` を作成する

```Typescript
import { createContext, useContext } from "react";
import useFirebaseAuth from "hooks/useFirebaseAuth";
import { User } from "firebase/auth";

interface AuthContext {
  currentUser: User | null;
  loading: boolean;
  loginWithGoogle: () => Promise<User | undefined>;
  logout: () => Promise<void>;
}

type AuthProviderProps = {
  children: React.ReactNode;
};

const AuthCtx = createContext({} as AuthContext);

const { currentUser, loading, loginWithGoogle, logout } = useFirebaseAuth();

const AuthContext: AuthContext = {
  currentUser: currentUser,
  loading: true,
  loginWithGoogle: loginWithGoogle,
  logout: logout,
};

export function AuthContextProvider({ children }: AuthProviderProps) {
  return <AuthCtx.Provider value={AuthContext}>{children}</AuthCtx.Provider>;
}
// custom hook to use the userContext and access currentUser and loading
export const useAuthContext = () => useContext(AuthCtx);
```

{: file="context/AuthContext.tsx" }

context を全 app 範囲内で適用できるようにする。

```Typescript
import type { AppProps } from "next/app";

import "../styles/globals.css";
import { AuthContextProvider } from "../context/AuthContext";

export default function App({ Component, pageProps }: AppProps) {
  return (
    <AuthContextProvider>
      <Component {...pageProps} />
    </AuthContextProvider>
  );
}
```

{: file="pages/\_app.tsx" }

## ログイン必要なページ内で、ユーザーログイン状況を確認する

```Typescript
import { useEffect } from "react";
import { useRouter } from "next/router";
import { useAuthContext } from "../context/AuthContext";

export default function ProtectedPage() {
  const { currentUser, loading } = useAuthContext();
  const router = useRouter();

  // ログインしていないユーザーであれば、ログインページへ飛ばす
  useEffect(() => {
    if (!loading && !currentUser) {
      router.push("/login");
    }
  }, [currentUser, loading]);

  return <h1>This page only for logged in users.</h1>;
}
```

## ログイン成功後、ID token 取得して Rails 側へ送る

user オブジェクトは`getIdToken`メソッドを使って、Firebase が発行した idToken を取得できる。

```Typescript
/**
  * Returns a JSON Web Token (JWT) used to identify the user to a Firebase service.
  *
  * @remarks
  * Returns the current token if it has not expired or if it will not expire in the next five
  * minutes. Otherwise, this will refresh the token and return a new one.
  *
  * @param forceRefresh - Force refresh regardless of token expiration.
  */
getIdToken(forceRefresh?: boolean): Promise<string>;
```

axios の使用方法についてはここで省略。

```Typescript
import axios from "axios";

import useFirebaseAuth from "hooks/useFirebaseAuth";

export default function LoginPage() {
  const { loginWithGoogle } = useFirebaseAuth();

  const handleGoogleLogin = () => {
    const verifyIdToken = async () => {
      const user = await loginWithGoogle();
      const token = await user?.getIdToken();

      const config = {
        headers: { authorization: `Bearer ${token}` },
      };

      try {
        axios.post("/auth", null, config);
      } catch (err) {
        let message;
        if (axios.isAxiosError(err) && err.response) {
          console.error(err.response.data.message);
        } else {
          message = String(err);
          console.error(message);
        }
      }
    };
    verifyIdToken();
  };

  return (
    <div>
      <button onClick={handleGoogleLogin}>
        <span>Sign in with Google</span>
      </button>
    </div>
  );
}
```

これでフロントエンド側の実装は完了した。

大変参考になったリソース

- コメント内で最新 Firebase v9 バージョン適用したコードを PR されたので、すごく参考になった。
  [Implementing authentication in Next.js with Firebase](https://blog.logrocket.com/implementing-authentication-in-next-js-with-firebase/)

- Firebase v8 バージョン使っているけど、参考になった。

  [Next.js × TypeScript × Firebase Authentication で Google 認証を実装する](https://qiita.com/y-shida1997/items/f5e52c7288813a8184ff)

- `onAuthStateChanged`を利用した hooks ライブラリもある。自前で context 実装必要ないので、便利らしい

  [react-firebase-hooks](https://github.com/csfrequency/react-firebase-hooks/tree/593f47965709d4abf23029c1160bb834c9f825bc/auth#useidtoken)

# Rails API 側の実装

Rails でやることは Next.js から送られてきた ID Token を検証すること。

本来 Firebase が提供する Admin SDK を使えば簡単になるけど、SDK がサポートしているバックエンド言語は Node.js, Java, Python, Go と C# のみで、残念ながら Ruby がサポートされていない。

公式説明の通り、第三者の JWT ライブラリを使って token を検証する必要がある。

JWT ライブラリは、ruby では [ruby-jwt](https://github.com/jwt/ruby-jwt)があるので、それを使って、token を検証することができる。

※ 検証ロジックは自前で実装以外に、gem を使うなど他の選択肢もある。関連記事もあるので、詳細はここで省略。

<details> 
<summary>検証に利用可能なGem</summary>

1. Gem 'firebase-admin-sdk-ruby'

こちらは ボランティアたちが作った Ruby 用 Firebase-admin-sdk。認証以外のユーザー管理機能などもあるので、公式 SDK の代替案として良いと思うけど、まだ alpha 版の段階で、プロダクションでの使用はまだ推奨されていない。

> This gem is currently in alpha and not recommended for production use (yet).

自前で実装しても、ソースコードが大変参考になると思う。

[firebase-admin-sdk-ruby](https://github.com/cheddar-me/firebase-admin-sdk-ruby)

2. Gem 'firebase_id_token'

Firebase ID token を検証する用の gem で、直近でも更新があり、良さそうな感じ。

検証に必要な Google 公開鍵証明書を Redis で保存するので、Redis の利用が必須になる。

[firebase_id_token](https://github.com/fschuindt/firebase_id_token)

3. Gem 'firebase-auth-rails'を使う

こちらの gem は firebase_id_token をベースにしたもの。利用方法はさらに簡単になる。

Redis の利用も必須。

※ まだベタ版で、最近は更新されていないよう。

[firebase-auth-rails](https://github.com/penguinwokrs/firebase-auth-rails)

</details>

## 検証の流れについて

公式説明の通り、[Firebase: Verify ID tokens using a third-party JWT library](https://firebase.google.com/docs/auth/admin/verify-id-tokens#verify_id_tokens_using_a_third-party_jwt_library)

検証内容は三つある。 トークン のヘッダー、ペイロードと署名。

<details> 
<summary>検証内容一覧</summary>

- ID Token Header

  - alg(Algorithm): 署名作成のアルゴリズムは "RS256"であること
  - kid(Key ID ): Key ID は [Google 公開鍵証明書リスト](https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com)の key の一つと一致すること

- ID Token Payload

  - exp(Expiration time ): token の有効期限は過ぎていないこと。
  - iat(Issued-at time ): token の発行日時は過去であること。
  - aud(Audience): token の想定利用者識別子は project_ID と一致すること。
  - iss(Issuer): token の発行者識別子は"https://securetoken.google.com/<project_id>"と一致すること
  - sub(Subject): uid となるユニークな値は空でない文字列であること。

- ID Token Signature
  - 最後に、[Google 公開鍵証明書](https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com)サイトから、`kid`と関連する証明書を取得し、公開鍵を生成して、署名の有効性を検証する

</details>

ここで特に注意必要なのは、token を**２回 decode する必要**があること。

token 署名を検証するための公開鍵を特定するには、token ヘッダー内の`kid`(Key ID)を使う必要がある。

そのため、token を検証する前に、まず検証なしで token を decode し、`kid`属性を取得する。

そして、取得した公開鍵を使って、再度 `JWT.decode`メソッド で token を検証する。

## 詳細やり方

検証ロジックのコードは`/app/lib/firebase_auth.rb`ファイルに置いている。

全体コードはこちら：

※. rails デフォルトの`/lib`を使わず、`/app`下に別途`/lib`を作るのは、`/app`配下のファイルは自動的にロードされるから。

### 基本設定

1. `rails new my-app —api`
2. gem を追加する。
   `jwt`、[`rack-cors`](https://github.com/cyu/rack-cors)、[`dotenv-rails`](<(https://github.com/bkeepers/dotenv)>)
3. `config/initializers/cors.rb`設定

```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins "http://localhost:3000"

    resource "*",
             headers: :any,
             methods: %i[get post put patch delete options head]
  end
end
```

4. `config/puma.rb`で rails のサーバポートを 3001 に設定。今回は Next.js 側は 3000 にしたので。

```ruby
port ENV.fetch("PORT") { 3001 }
```

5. project id を環境変数に設定する
   project id は`aud`と`iss`の検証に使うので、まず環境変数に設定する。

- `.env`ファイルを作成し、`FIREBASE_PROJECT_ID="XXXXXXXX"`を追加する。  
  ※. `.env`ファイルを`.gitignore`に追加するのを忘れなく。
- `FIREBASE_PROJECT_ID = ENV["FIREBASE_PROJECT_ID"]`で project id を使う。

ついでに、いくつかの定数を定義する。

```ruby
ALGORITHM = "RS256".freeze

# "iss"は "https://securetoken.google.com/<FIREBASE_PROJECT_ID>"
ISSUER_PREFIX = "https://securetoken.google.com/".freeze
FIREBASE_PROJECT_ID = ENV["FIREBASE_PROJECT_ID"]

# 下記のURLからGoogle公開鍵証明書リストを取得する
CERT_URI =
  "https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com".freeze
```

### 検証メソッドラッパー

このラッパーメソッドのロジックは：

- token を decode して、中身を取得する
- 取得した header を使って、公開鍵を取得する
- 公開鍵を使って、token を検証する
- token 検証失敗したら、error 情報を返す
- token 検証成功したら、ユーザー uid を返す

```ruby
def verify_id_token(id_token)
  payload, header = decode_unverified(id_token)
  public_key = get_public_key(header)

  error = verify(id_token, public_key)

  if errors.empty?
    return { uid: payload["user_id"] }
  else
    return { errors: errors.join(" / ") }
  end
end
```

最後に返す payload 中身はこんな感じ

```ruby
{
  name: "<username>",
  picture: "<user_profile_picture>",
  iss: "https://securetoken.google.com/<FIREBASE_PROJECT_ID>",
  aud: "<firebase_project_id>",
  auth_time: 1_668_430_866,
  user_id: "<user_id>(same as sub)",
  sub: "<subject>",
  iat: 1_668_488_296,
  exp: 1_668_491_896,
  email: "<user email>",
  email_verified: true,
  firebase: {
    identities: {
      "google.com": ["<google_user_id>"],
      email: ["<user_gmail>"],
    },
    sign_in_provider: "google.com",
  },
}
```

### Step 1: 検証なしで token を decode する

まず、token を検証なしで decode する。

```ruby
def decode_unverified(token)
  decode_token(
    token: token,
    key: nil,
    verify: false,
    options: {
      algorithm: ALGORITHM,
    },
  )
end

# Returns:
#    Array: decoded data of ID token =>
#     [
#      {"data"=>"data"}, # payload
#      {"typ"=>"JWT", "alg"=>"alg", "kid"=>"kid"} # header
#     ]
def decode_token(token:, key:, verify:, options:)
  JWT.decode(token, key, verify, options)
end
```

`ruby-jwt`の使い方は github から参照できる。  
参照： [ruby-jwt](https://github.com/jwt/ruby-jwt)

`decode`メソッド引数の中身はこんな感じ。  
`JWT.decode(token, key=nil, verify=false, option={algorithm: ALGORITHM})`

[`ruby-jwt`の保守管理者からの返答](https://github.com/jwt/ruby-jwt/issues/216#issuecomment-319010415)によると、
ここの`verify`を`false`にすることで、`JWT.decode`は token データの抽出のみを行い、検証プロセスを飛ばすので、処理速度が速く、全体のパフォーマンスに影響がないはず。

ここで decode したデータの形式は

```ruby
[
  {
    aud: "<firebase_project_id>",
    auth_time: 1_668_430_866,
    user_id: "<user_id>(same as sub)",
    sub: "<subject>",
    ...略
  }, # payload部分
  { alg: "RS256", kid: "XXXXXXX", typ: "JWT" } # header部分
]
```

となるので、
`payload, header = decode_unverified(id_token)`でそれぞれを取得する。

### Step 2: 公開鍵を取得する

続いて公開鍵を取得する。

```ruby
# 公開鍵取得のラッパーメソッド
def get_public_key(header)
  certificate = find_certificate(header["kid"])
  public_key = OpenSSL::X509::Certificate.new(certificate).public_key
rescue OpenSSL::X509::CertificateError => e
  raise "Invalid certificate. #{e.message}"

  return public_key
end
```

[Google 公開鍵証明書リスト](https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com)にアクセスするとわかると思うけど、

ここに載せている証明書は`{key: value}`のハッシュ形式で、ペアが二つあり、うち一つの `key` は今回の`kid`と一致する。

```ruby
{ key_1: "CERTIFICATE_1中身", key_2: "CERTIFICATE_2中身" }
```

そして、`kid`を使って、今回に使う公開鍵証明書を特定する。

```ruby
def find_certificate(kid)
  certificates = fetch_certificates
  unless certificates.keys.include?(kid)
    raise "Invalid 'kid', do not correspond to one of valid public keys."
  end

  valid_certificate = certificates[kid]
  return valid_certificate
end

# CERT_URLから証明書リストを取得する
def fetch_certificates
  uri = URI.parse(CERT_URI)
  https = Net::HTTP.new(uri.host, uri.port)
  https.use_ssl = true

  req = Net::HTTP::Get.new(uri.path)
  res = https.request(req)
  unless res.code == "200"
    raise "Error: can't obtain valid public key certificates from Google."
  end

  certificates = JSON.parse(res.body)
  return certificates
end
```

### Step 3: token の有効性を検証する

`sub`と`alg`は`JWT.decode`で自動検証できないため、追加検証必要。

```ruby
def verify(token, key)
  errors = []

  begin
    decoded_token =
      decode_token(
        token: token,
        key: key,
        verify: true,
        options: decode_options,
      )
  rescue JWT::ExpiredSignature
    errors << "Firebase ID token has expired. Get a fresh token from your app and try again."
  rescue JWT::InvalidIatError
    errors << "Invalid ID token. 'Issued-at time' (iat) must be in the past."
  rescue JWT::InvalidIssuerError
    errors << "Invalid ID token. 'Issuer' (iss) Must be 'https://securetoken.google.com/<firebase_project_id>'."
  rescue JWT::InvalidAudError
    errors << "Invalid ID token. 'Audience' (aud) must be your Firebase project ID."
  rescue JWT::VerificationError => e
    errors << "Firebase ID token has invalid signature. #{e.message}"
  rescue JWT::DecodeError => e
    errors << "Invalid ID token. #{e.message}"
  end

  sub = decoded_token[0]["sub"]
  alg = decoded_token[1]["alg"]

  unless sub.is_a?(String) && !sub.empty?
    errors << "Invalid ID token. 'Subject' (sub) must be a non-empty string."
  end

  unless alg == ALGORITHM
    errors << "Invalid ID token. 'alg' must be '#{ALGORITHM}', but got #{alg}."
  end

  return errors
end

def decode_options
  {
    iss: ISSUER_PREFIX + FIREBASE_PROJECT_ID,
    aud: FIREBASE_PROJECT_ID,
    algorithm: ALGORITHM,
    verify_iat: true,
    verify_iss: true,
    verify_aud: true,
  }
end
```

これで、token 検証が完了した。

取得した payload 内のユーザー情報を使って、新規ユーザーの登録などができる。

### 今後の課題

- Google 公開鍵証明書を cache する
- 匿名認証
- 他のソーシャルログイン(Twitter, line など)
- フロントエンド側のコードリファクタ、パフォーマンス改善

### 参考になったリソース

Rails 部分の検証で大変参考になった記事やソースコードは下記となる。

- [Ruby で Firebase の id トークンを認証に使ってみる](https://qiita.com/otakky/items/b7582202f5cde8f2dd21)

  説明が詳しくて大変参考になった。

- [Rails + Next.js + Firebase Authentication で認証付きアプリを作成する](https://qiita.com/kmgk0215/items/a174c6eef0a940ad03ea#%E3%83%90%E3%83%83%E3%82%AF%E3%82%A8%E3%83%B3%E3%83%89rails%E3%81%AE%E5%AE%9F%E8%A3%85)

  ソースコードはすごく勉強になった。とりわけ参考になった

- [Proper way to verify Firebase id tokens](https://github.com/jwt/ruby-jwt/issues/216)

  認証全体の流れはパッとわかるようになったのは、こちらの issue 文の説明のおかげだ。

- [How to validate Firebase ID token in Ruby](https://medium.com/@igorkhomenko/how-to-validate-firebase-id-token-in-ruby-23f4f54c89ab)

  同作者による詳細説明記事とソースコードも大変参考になった。

- [firebase-admin-sdk-ruby](https://github.com/cheddar-me/firebase-admin-sdk-ruby)

  gem を使っていないけど、ソースコード自体が大変参考になった。

- [firebase-admin-python](https://github.com/firebase/firebase-admin-python/blob/b9e95e8248eb1473ca5a13bf64e8a33b79dc9db3/firebase_admin/_token_gen.py#L292)

  こちらは Firebase 公式の python 版 admin-sdk ソースコード。
  コード内のコメント説明も大変参考になった。

- [How to Sign and Validate JSON Web Tokens – JWT Tutorial](https://www.freecodecamp.org/news/how-to-sign-and-validate-json-web-tokens/)

  JSON Web Token について勉強になった。
