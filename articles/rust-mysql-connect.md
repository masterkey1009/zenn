---
title: "RustとMysqlを接続する"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Rust, Mysql]
published: false
---

# Rustの環境構築

## Rustのinstall
RustのinstallはHomebrewを使う。
```
$ brew install rust
```
```
brew install rust  
==> Downloading https://ghcr.io/v2/homebrew/core/rust/manifests/1.57.0_1
Already downloaded: /Users/masaki/Library/Caches/Homebrew/downloads/5e9055ce02c980a2da1e31faca4b8569c4b98c2bc943718c4fd47e21e3ee5847--rust-1.57.0_1.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/rust/blobs/sha256:2612f8b7a49c5e7dc1e53b03c7c1e5e77ad38f945859990900e969f17792bb37
Already downloaded: /Users/masaki/Library/Caches/Homebrew/downloads/c829e010ef4381a2bfdb24e9325f855c544ae2d498f4b52677134c878bf29274--rust--1.57.0_1.monterey.bottle.tar.gz
==> Pouring rust--1.57.0_1.monterey.bottle.tar.gz
==> Caveats
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/rust/1.57.0_1: 31,371 files, 794.9MB
==> Running `brew cleanup rust`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Warning: Calling bottle :unneeded is deprecated! There is no replacement.
Please report this issue to the nektos/tap tap (not Homebrew/brew or Homebrew/core):
  /usr/local/Homebrew/Library/Taps/nektos/homebrew-tap/Formula/act.rb:9
```
rustがinstallされているかの確認をするために、下記のコマンドを実行。
```
$rustc --version
```
```
rustc 1.57.0
```
バージョンが表示されたので、installは正常に出来た。

## Rustをuninstallする
```
$ brew uninstall rust
```

## Rustをupdateする
```
$ brew upgrade
```
Homebrewを使ってアップデートする。Homebrewを使うと、個別にアップデートすることは出来ず、全体をアップデーする形になる。

`rust`をinstallすると`cargo`も使用できるようになっているのでそれを使ってプロジェクトを作成する。

# プロジェクトを作成する
```
$ cargo new rust_mysql --bin
    Created binary (application) `rust_mysql` package
```
`cargo new rust_mysql --bin`を実行すると`Cargo.toml`と`src/main.rs`がプロジェクト内に作成される。
```
$ cd rust_mysql
$ tree
.
├── Cargo.toml
└── src
    └── main.rs

```
`Cargo.toml`の内容は下記の通り。
```toml:./Cargo.toml
[package]
name = "rust_mysql"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```
`./src/main.rs`の内容は下記の通り。

```rust:./src/main.rs
fn main() {
    println!("Hello, world!");
}
```
## buildしてみる
```
$ cargo build
```
を実行するとビルドできる。

```
$ cargo build
   Compiling rust_mysql v0.1.0
    Finished dev [unoptimized + debuginfo] target(s) in 1.56s
```
実行可能ファイルは
`./target/debug/`以下に配置されるので、
```
$ ./target/debug/file_name
```
上記を叩くと実行できる。
```
$ ./target/debug/rust_mysql 
Hello, world!
```

## コードのコンパイルと実行を同時に行うコマンド
```
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/rust_mysql`
Hello, world!
```

## コンパイルチェックを行うコマンド
```
$ cargo check
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
```

## リリース用ビルドを行うコマンド
```
$ cargo build --release
    Finished release [optimized] target(s) in 0.00s
```

# IntelliJ IDEAにプロジェクトをimportする
intelliJ IDEAでプロジェクトを開くだけで、インポートされるはず。
ウェルカム画面を開いた時に、`shift` + `command` + `A` を押して、`project from existing sources`と入力すると手動でインポートできる画面が開けるので、そこでポチポチしていっても良い。

設定がきちんと出来ていれば、`fn main()`が記述してある行に`▶`ボタンが表示されるので、そこから`cargo run`が実行できる。`build`は画面右上にあるトンカチ？みたいなビルドボタンを押せば実行できる。

# Rustのフレームワーク`axum`を導入する
github: https://github.com/tokio-rs/axum

`Cargo.toml`の`dependencies`に`axum`を追加する。

```toml:./Cargo.toml
[dependencies]
axum = { version = "0.4.4", features = ["multipart"] }
tokio = { version = "1", features = ["full"] }
```

```rust:./src/main.rs
use std::net::SocketAddr;
use axum::{
    routing::get,
    Server,
    Router
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(root));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    Server::bind(&addr)
        .server(app.into_make_service())
        .await
        .unwrap()
}

async fn root() -> &'static str {
    "Hello, Rust"
}
```
`cargo run`を実行して起動する。
```
$ cargo run
Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/rust_mysql`
```
`curl`でレスポンス確認。
```
$ curl http://localhost:3000
Hello, Rust%
```

## routingを追加してみる
```diff rust:./src/main.rs
use std::net::SocketAddr;
use axum::{
    routing::get,
    Server,
    Router
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(root))
+       .route("/api/users", get(find_all).post(create))
+       .route("/api/users/:user_id", get(find_one).patch(update).delete(delete));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

+async fn root() -> &'static str {
+    "Hello, Rust"
+}

+async fn find_all() -> &'static str {
+    "run find all"
+}

+async fn find_one() -> &'static str {
+    "run find one"
+}

+async fn create() -> &'static str {
+    "run create"
+}

+async fn update() -> &'static str {
+    "run update"
+}

+async fn delete() -> &'static str {
+    "run delete"
+}
```

routingを追加したので動作確認してみる。

```
$ curl http://localhost:3000/api/users
run find all%

$ curl http://localhost:3000/api/users/1
run find one%

$ curl -X POST  http://localhost:3000/api/users
run create%

$ curl -X PATCH  http://localhost:3000/api/users/1
run update%

$ curl -X DELETE  http://localhost:3000/api/users/1
run delete%
```
## routingを別ファイルに定義する
このまま書いてしまうと、`./src/main.rs`の中身が肥大していってしまいそうなので、ルーティングを分割するようにして、更にパスパラメータを受け取れるように改良する。

```
$ mkdir -p src/routes
$ touch src/routes/user.rs
```
上記で作成した`routes`ディレクトリと同じ名前のファイルを作成する。
```
$ touch src/routes.rs
```

```rust:./src/routes.rs
pub mod users;
```
`./src/main.rs`内で記述していたルーティングの設定をこちらに移す。
```rust:./src/routes/user.rs
use axum::{
    extract::Path,
    routing::get,
    Router
};

pub fn users_api() -> Router {
    Router::new()
        .route("/", get(find_all).post(create))
        .route("/:user_id", get(find_one).patch(update).delete(delete));
}

async fn find_all() -> &'static str {
    "find all users"
}

async fn create(Path(user_id): Path<u64>) -> String {
    format!("create user user_id: {}", user_id)
}

async fn find_one(Path(user_id): Path<u64>) -> String {
    format!("find_one user user_id: {}", user_id)
}

async fn update(Path(user_id): Path<u64>) -> String {
    format!("update user user_id: {}", user_id)
}

async fn delete(Path(user_id): Path<u64>) -> String {
    format!("delete user user_id: {}", user_id)
}

async fn create(Path(user_id): Path<u64>) -> String {
    format!("create user user_id: {}", user_id)
}
```

```diff rust:./src/main.rs
use std::net::SocketAddr;
- use axum::{
-    routing::get,
-    Server,
-    Router
-};
+use axum::{Router}

+mod routes;

#[tokio::main]
async fn main() {

+   let users_api = routes::users::users_api;

    let app = Router::new()
-       .route("/", get(root))
+       .nest("/api/users", users_api);
-       .route("/api/users", get(find_all).post(create))
-       .route("/api/users/:user_id", get(find_one).patch(update).delete(delete));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

-async fn root() -> &'static str {
-    "Hello, Rust"
-}

-async fn find_all() -> &'static str {
-    "run find all"
-}

-async fn find_one() -> &'static str {
-    "run find one"
-}

-async fn create() -> &'static str {
-    "run create"
-}

-async fn update() -> &'static str {
-    "run update"
-}

-async fn delete() -> &'static str {
-    "run delete"
-}
```