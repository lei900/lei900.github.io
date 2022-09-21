---
title: Readable Codeメモ：コードの見た目を美しくする
category: "Software Design"
tags: ["Better Code", Refactor, "Reading Notes"]
---

**読みやすさの基本定理とは**

>コードは他の人が最短時間で理解できるように書かなければいけない。

## 第4章：美しさ（レイアウトの改善）

**三つの原則**
- 読み手が慣れているパターンと一貫性のあるレイアウトを使う
- 似ているコードは似ているように見せる
- 関連するコードをまとめてブロックにする

**技法：**

- 複数のコードブロックで同じようなことをしていたら、シルエットも同じようなものにする
- コードの「列」を整列すれば、概要が把握しやすくなる。
- ある場所でA, B, Cのように並んでいたものを、他の場所でB, C, Aのように並べてはいけない。意味のある順番を選んで、常にその順番を守る
- 空行を使って大きなブロックを論理的な「段落」に分ける

**1. 一貫性のある簡潔な改行位置**

適切な改行を入れることにより、「似ているコードは似ているように見える」という原則を守る。

整形前：
```java
public class PerformanceTester {
    public static final TcpConnectionSimulator wifi = new TcpConnectionSimulator (
        500, /* Kbps */
        80, /* millisecs latency */
        200, /* jitter */
        1 /* packet loss % */);
```
整形後：
```java
public class PerformanceTester {
    public static final TcpConnectionSimulator tb_fiber = 
        new TcpConnectionSimulator (
            500,   /* Kbps */
            80,    /* millisecs latency */
            200,   /* jitter */
            1      /* packet loss % */);
```
更に簡潔にする多め、コメントを上位に書いて下位の引数を並べるようにする。
```java
public class PerformanceTester {
   // TcpConnectionSimulator (throughtput, latency, jitter, packet_loss)
   //                            [Kbps]     [ms]     [ms]   [percent]
   public static final TcpConnectionSimulator wifi = 
       new TcpConnectionSimulator (500,   80,  200, 1);
   public static final TcpConnectionSimulator t3_fiber =
       new TcpConnectionSimulator (45000, 10,  0,   0);
   public static final TcpConnectionSimulator cell = 
       new TcpConnectionSimulator (100,   400, 250, 5);
}
```

**2. メソッドを使った整列**

見た目が美しくないコードや、同じことを何度も書いているコードは、ヘルパーメソッドを使って改善する。

整形前
```c++
DatabaseConnection database_connnection;
string error;
assert(ExpandFullName(database_connection, "Doug Adams", &error) == "Mr. Douglas Adams");
assert(error == "");
assert(ExpandFullName(database_connection, "Jake Brown", &error) == "Mr. Jacob Brown Ⅲ");
assert(error == "");
assert(ExpandFullName(database_connection, "No Such Guy", &error) == "");
assert(error == "no match found");
```

整形後
```c++
CheckFullName("Doug Adams", "Mr. Douglas Adams", "");
CheckFullName("Jake Brown,", "Mr. Jake Bron III", "");
CheckFullName("No Such Guy", "", "no match found");
CheckFullName("John", "", "more than one result");

void CheckFullName(...) {
  ...
}
```

**3. 縦の線をまっすぐにする(列を整列する)**

縦の線をまっすぐにし、列を「整列」させれば、コードが読みやすくなることがある。

前述の例をさらに整形すると
```c++
CheckFullName("Doug Adams"  , "Mr. Douglas Adams", "");
CheckFullName(" Jake Brown,", "Mr. Jake Bron III", "");
CheckFullName("No Such Guy" , ""                 , "no match found");
CheckFullName("John"        , ""                 , "more than one result");
```
他の例：
```c++
commands[] = {
    ...
    { "timeout",    NULL,              cmd_spec_timeout },
    { "timestamp",  &opt.timestamping, cmd_boolean },
    { "tries",      &opt.ntry,         cmd_number_inf },
    { "useproxy",   &opt.use_proxy,    cmd_boolean },
    { "useragent",  NULL,              cmd_spec_useragent },
    ...
};
```

**4. 一貫性と意味のある並び**

例：  
- 対応するHTMLフォームの`<input>`フィールドと同じ並び順にする
- 「最重要」なものから重要度順に並べる
- アルファベット順に並べる

**5. コードを「段落」に分割する**

整形前
```python
# ユーザのメール帳をインポートして、システムのユーザと照合する。
# そして、まだ友達になっていないユーザの一覧を表示する。
def suggest_new_friends (user, email_password):
   friends = user.friends()
   friend_emals = set(f.email for f in friends)
   contacts = import_contacts(user.email, email_password);
   contact_emals = set(c.email for c in contacts)
   non_friend_emails = contact_emals - friend_emails
   suggested_friends = USer.objects.select(email__in=non_friend_emails)
   display['user'] = user
   display['friends'] = friends
   display['suggested_friends'] = suggested_friends
   return render("suggested_friends.html", display)
```

段落に分けると、
```python
# ユーザのメール帳をインポートして、システムのユーザと照合する。
# そして、まだ友達になっていないユーザの一覧を表示する。
def suggest_new_friends (user, email_password):
   # ユーザの友達のメールアドレスを取得する。
   friends = user.friends()
   friend_emals = set(f.email for f in friends)
   
   # ユーザのメールアカウントからすべてのメールアドレスをインポートする。
   contacts = import_contacts(user.email, email_password);
   contact_emals = set(c.email for c in contacts)
   
   # まだ友達になっていないユーザを探す。
   non_friend_emails = contact_emals - friend_emails
   suggested_friends = USer.objects.select(email__in=non_friend_emails)

   # それをページに表示する
   display['user'] = user
   display['friends'] = friends
   display['suggested_friends'] = suggested_friends
   return render("suggested_friends.html", display)
```

---
参照  
[リーダブルコード ―より良いコードを書くためのシンプルで実践的なテクニック](https://www.oreilly.co.jp/books/9784873115658/)