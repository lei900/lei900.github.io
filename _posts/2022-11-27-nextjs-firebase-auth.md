---
title: Next.js編 - Rails + Next.js + Firebase V9 Authentication で認証付きのCRUDアプリを作る
category: "Rails"
tags: [React, Next.js, Rails, API, Firebase, TypeScript]
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
// .env.local
NEXT_PUBLIC_FIREBASE_API_KEY=<YOUR_API_KEY>;
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=<YOUR_DOMAIN>;
NEXT_PUBLIC_FIREBASE_PROJECT_ID=<YOUR_PROJECT_ID>;
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=<YOUR_STORAGE_BUCKET>;
NEXT_PUBLIC_FIREBASE_MESSEGING_SENDER_ID=<YOUR_MESSEGING_SENDER_ID>;
NEXT_PUBLIC_FIREBASE_APP_ID=<YOUR_APP_ID>;
```

※ この変数はブラウザ 側で処理するので、変数名には`NEXT_PUBLIC`を追加する必要  
 [#exposing-environment-variables-to-the-browser](https://nextjs.org/docs/basic-features/environment-variables#exposing-environment-variables-to-the-browser)

そして、Firebase SDK インストール `npm install --save firebase`

## Firebase 初期化と Firebase App オブジェクトを作成する

```Typescript
// lib/initFirebase.ts
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

## ログイン関数作成

Firebase が提供する`GoogleAuthProvider`と`signInWithPopup`を利用する

```JavaScript
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

  ```JavaScript
  displayName: string | null; // ユーザー表示名
  email: string | null; // ユーザーメール
  phoneNumber: string | null; // ユーザー電話番号
  photoURL: string | null; // Googleプロフィール写真URL
  uid: string; // Firebaseが生成するユニークID
  ```

ちなみに、Google ログインページへの遷移を Redirect にしたいなら、`signInWithRedirect`を利用する

```JavaScript
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

```JavaScript
// Adds an observer for changes to the user's sign-in state.
// @param auth — The Auth instance.
// @param nextOrObserver — callback triggered on change.
onAuthStateChanged(auth: Auth, nextOrObserver: NextOrObserver<User>): Unsubscribe
```

`useFirebaseAuth()`関数を作成する

```JavaScript
// hooks/useFirebaseAuth.ts
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

```JavaScript
const clear = () => {
  setCurrentUser(null);
  setLoading(false);
};

const logout = () => signOut(auth).then(clear);
```

`useFirebaseAuth()`の全体像

```JavaScript
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

## user context を作成する

ユーザー情報を app 内で共有するため、`AuthContext` を作成する

```JavaScript
// context/AuthContext.tsx
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

context を全 app 範囲内で適用できるようにする。

```JavaScript
// pages/_app.tsx
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

## ログイン必要なページ内で、ユーザーログイン状況を確認する

```JavaScript
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

```JavaScript
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

```JavaScript
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

Rails 編はこちら -> [Rails 側の実装](https://lei900.github.io/22/11/rails-firebase-auth)
