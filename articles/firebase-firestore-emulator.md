---
title: "Firebase Firestoreのエミュレーターを使ってローカルで開発する"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "firestore", "開発環境"]
published: true
---

もしかしたら皆さん知ってたんでしょうか。おれは最近知りました。（普通にドキュメントに載ってる。）

### 先にまとめ

- エミュレーターがあればだいたいローカルで開発できるぞ
- Firebase コンソールみたいなウェブ UI で閲覧、編集もできる
- `npx firebase init` でインストール
- `npx firebase emulators:start` で起動
- `app.firestore().useEmulator("localhost", 8080);` でアプリから接続
- 公式：[Firebase Local Emulator Suite の概要](https://firebase.google.com/docs/emulator-suite)
- 例：[ginpei/firebase-emu-example](https://github.com/ginpei/firebase-emu-example)

Firestore 以外の Firebase の機能もあるんだけど、本稿では主に Firestore についてお話しします。

※↓こんな感じです

![ウェブ UI のスクリーンショット。Firebase ターミナルの Firestore 画面に似ている](https://storage.googleapis.com/zenn-user-upload/ek4trbhecxuz6jt4mi38y2hmd974)

### エミュレーターでできること

- 本物 Firestore のデータを気にしなくて済む
- 複数人で DB を使いまわすこともなくなる
- Security rules をデプロイする前に試せる
- 動作確認用にスナップショットから再開できる
- Security rules 等の試験を書ける
- 試験は完全ローカルでできるし CI でも動く

### エミュレーターでできないこと

- Firebase 全体のエミュレーターでは**ない**ので、完全ローカルにはならない
  - 例えば Storage のエミュレーターがない
  - そのうち増えるかも？（この記事の初版では Auth はなかった）
- 実際の Firebase の代わりとして利用してアプリ公開は無理

## インストール

Firebase の諸機能と同様 `firebase init` からインストールできます。

ちなみに既に Firebase の設定が終わってる場合でも気にせず init し直して大丈夫。（エミュレーターに限らず。）　またエミュレーターだけ入れたい場合は `firebase init emulator` と最初から指定するとちょっと工程が短縮されるようです。

### 事前準備

`firebase-tools` をインストールしていない場合、準備してください。まあ Firebase の話してるのでもう入ってると思うけど。

```bash
$ npm install -D firebase-tools
```

ここでは開発用の依存パッケージ `-D` としてインストールしてるけどお好みでグローバル `-g` でも構いません。

エラーなくバージョンが表示されれば成功。

```bash
$ npx firebase --version
```

### エミュレーターのインストール

```bash
$ npx firebase init
```

対話型のインストーラーが起動します。だいたいは初期値のままで進めて大丈夫。

1. "Which Firebase CLI features do you want to set up for this folder?"
   - **Firestore** と **Emulators** を選択
   - 他に必要なものがあればそれも
2. "Please select an option"
   - 既存のものか新規を選択
   - ここで "Don't set up a default project" にするとウェブ UI が出てこない（なんで）
3. "What file should be used for Firestore Rules?"
4. "What file should be used for Firestore indexes?"
5. "Which Firebase emulators do you want to set up?"
   - **Firestore** を選択
   - 他にも Authentication など必要なものあれば
6. "Which port do you want to use for the firestore emulator?"
7. "Would you like to enable the Emulator UI?"
8. "Which port do you want to use for the Emulator UI"
9. "Would you like to download the emulators now?"
   - `y` でダウンロード開始
   - `N` にしてしまっても気にせず続けて、次に `init` し直してください。ダウンロードだけなら `npx firebase setup:emulators:firestore` でもできます
10. "Firebase initialization complete!"

おつかれさまでした。

## 起動

起動したままターミナルを占有するので別タブとか用意しといてください。（Windows なら cmd.exe じゃなくて MS 公式の [Windows Terminal](https://www.microsoft.com/ja-jp/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab) 使いましょう。）

```bash
$ npm firebase emulators:start

…

┌───────────┬────────────────┬─────────────────────────────────┐
│ Emulator  │ Host:Port      │ View in Emulator UI             │
├───────────┼────────────────┼─────────────────────────────────┤
│ Firestore │ localhost:8080 │ http://localhost:4000/firestore │
└───────────┴────────────────┴─────────────────────────────────┘
  Other reserved ports: 4400, 4500

Issues? Report them at https://github.com/firebase/firebase-tools/issues and attach the *-debug.log files.
```

書いてある通り localhost:4000 がウェブ UI で、Firestore のデータを入力したりできます。localhost:8080 はアプリが接続する方。

### 終了

`Ctrl+C` で終了します。

### コマンド実行

`exec` にすると指定のコマンドをエミュレーターと共に実行し、終了し次第エミュレーターも終わってくれます。

```bash
$ npx firebase emulators:exec "npm start"
```

この場合はウェブ UI は起動しないみたいです。

あとなんかときどき変な死に方をしてポートを握ったまま沈黙することがあって、あんまり信頼してません。

## アプリをエミュレーターへ接続

JavaScript で Firestore をエミュレーターへ繋ぐには、こう。

```javascript
firebase.firestore().useEmulator("localhost", 8080);
```

ちょっと前までは `firestore().settings()` で設定してたんだけど、今は `useEmulator()` 使うのが良いみたいです。

### 具体例

- [ginpei/firebase-emu-example](https://github.com/ginpei/firebase-emu-example)
  - [public/scripts.js](https://github.com/ginpei/firebase-emu-example/blob/f53c414825f83ade710fc2223ee02f5a94ee299f/public/scripts.js#L18-L32)

```js
function initializeFirebase() {
  firebase.initializeApp({
    apiKey: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    appId: "x-local-emu",
    authDomain: "x-local-emu.firebaseapp.com",
    projectId: "x-local-emu",
  });

  const isEmulating = window.location.hostname === "localhost";
  if (isEmulating) {
    firebase.auth().useEmulator("http://localhost:9099");
    firebase.functions().useEmulator("localhost", 5001);
    firebase.firestore().useEmulator("localhost", 8080);
  }
}
```

## ウェブ UI

ブラウザーで http://localhost:4000 を開くと色々出てきて、内容を閲覧したり編集したりできます。（ポート番号は変わるかもしれません。ターミナル出力をご確認ください。）

![ウェブ UI トップページのスクリーンショット。Firestore 等のエミュレーターが並んでいる](https://storage.googleapis.com/zenn-user-upload/sxv7781thztt0fvtvfkho8phmtkx)

これで Google のサーバーにあるデータを気にせずローカルであれこれ試せるようになりました。

アプリの動作に合わせて更新されたりウェブ UI での入力がアプリへ反映されたりするのを確認しましょう。すげー。

![ウェブ UI のスクリーンショット。Firebase ターミナルの Firestore 画面に似ている](https://storage.googleapis.com/zenn-user-upload/ek4trbhecxuz6jt4mi38y2hmd974)

## データの保存と読み込み（インポート、エクスポート）

基本的にはデータはエミュレーター終了と共に破棄されますが、保存して同じ内容を読み込みなおしたりもできます。

手動でアプリの挙動を試しまくるときとか E2E の試験とかで役立つと思います。

### 保存（エクスポート）

エミュレーターが起動している状態で、別のターミナルからこう↓。

```bash
$ npx firebase emulators:export path/to/export
```

指定のディレクトリー配下にファイルが出力されます。JSON かと思ったらバイナリです。それはそうか。

### 読み込み（インポート）

エミュレーターを起動する際に `--import` オプションを加え、エクスポートしたディレクトリーを指定します。

```bash
$ npx firebase emulators:start --import=path/to/export
```

ディレクトリーがなかったりすると詳しいエラー情報もなくスッと落ちます。

### 自動保存

`--export-on-exit` オプションを与えるとエミュレーター終了時に自動で保存というのもできます。アプリを動かしながら試行錯誤するときに便利。

保存先ディレクトリーを指定できるけど、それを省略して `--import` と併用すると自動的に同じ場所へ書き込みます。この方が使い勝手が良いことでしょう。

```bash
$ npx firebase emulators:start --import=path/to/export --export-on-exit
```

## Firestore へ接続するコードや Security Rules を試験する

JavaScript からは [`@firebase/rules-unit-testing` という npm パッケージ](https://www.npmjs.com/package/@firebase/rules-unit-testing)を利用するとうまいこと試験できます。近々記事にします。

### Security rules

これも詳しくは別記事で。

Firestore やってると `firestore.rules` を編集して Security rules をエンヤコラする必要があります。

エミュレーターを使えばデプロイすることなくローカルでルールの挙動を確認することができます。代わりに本番 Firestore の方にあるやつはエミュレーターに搭載されてないです。

※↓本番 Firestore の方にあるやつ（コンソール > Firestore > Rules > Rules Playground）

![ルール確認に便利そうな設定画面のスクリーンショット](https://storage.googleapis.com/zenn-user-upload/tjqawl03zm98u36n0yapk6qkfjeh)

## Firestore 以外のエミュレーター

以下がその一覧です。（firebase v8.2.1 現在）

- Authentication
- Cloud Functions
- Firestore
- Realtime Database
- Hosting
- Cloud Pub/Sub

Hosting は開発サーバーとしても使える。自動再読み込みみたいなのはないけど。

:::details 全部利用する設定にすると `firebase.json` はこんな感じ。

```json
{
  "database": {
    "rules": "database.rules.json"
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  },
  "storage": {
    "rules": "storage.rules"
  },
  "emulators": {
    "auth": {
      "port": 9099
    },
    "functions": {
      "port": 5001
    },
    "firestore": {
      "port": 8080
    },
    "database": {
      "port": 9000
    },
    "hosting": {
      "port": 5000
    },
    "pubsub": {
      "port": 8085
    },
    "ui": {
      "enabled": true
    }
  },
  "remoteconfig": {
    "template": "remoteconfig.template.json"
  }
}
```
:::

エミュレーターへ接続するコードはこうです。Authentication だけインターフェイス違う。

```js
firebase.auth().useEmulator("http://localhost:9099");
firebase.functions().useEmulator("localhost", 5001);
firebase.firestore().useEmulator("localhost", 8080);
firebase.database().useEmulator("localhost", 9000);
```

Functions の URL はこうなってました。

```
http://localhost:5001/PROJECT_ID/us-central1/FUNCTION_NAME
```

### Authentication エミュレーター

メールとパスワードのもののみ設定可能みたい。本物は Google アカウントでログインしたりできる。

名前や画像 URL などは設定可能。

## その他

### ログファイル

いくつかのログファイルを出力するので、`.gitignore` に追加してリポジトリーから除外しといてください。

```
# Firebase
database-debug.log
firebase-debug.log
firestore-debug.log
pubsub-debug.log
ui-debug.log
```

`*-debug.log` でまとめちゃっても良いかな。

### ポート使用済みエラー

たまにこういうのが出ます。

```
Error: Could not start Firestore Emulator, port taken.
```

読んでそのまま、`firebase.json` で指定のポート（通常 `8080`）が他のアプリに取られてるって話なので、それを終了させるか他のポートを指定してください。

ただなんかときどきエミュレーターが終了失敗してポートを占拠し続けることが……ある……ような……。

ときどき「見つからなかったから省略するよ」ってメッセージが出ることもあるんだけど謎。

### エミュレーターがスッと落ちて起動しない

スッ……

```bash
$ npx firebase emulators:start --import=path/to/export
i  emulators: Starting emulators: firestore
i  emulators: Shutting down emulators.

Error: An unexpected error has occurred.
```

パス間違えたとかでインポート先が用意されてないときにこうなりました。もうちょっと詳しくお願いします。

インポート先をちゃんと用意するか、 `--import` オプションを外して起動してください。

### ~~ウェブ UI が出てこない~~

:::details 現在は修正済み。以下古い内容。
理由はよくわからないんですが、default project を設定していないと Firestore エミュレーターは出るけど UI が出てきません。

```
┌───────────┬────────────────┐
│ Emulator  │ Host:Port      │
├───────────┼────────────────┤
│ Firestore │ localhost:8080 │
└───────────┴────────────────┘
```

適当に `.firebaserc` で用意してやれば出てきます。空文字列でなければ何でも良さそう。

```json
{
  "projects": {
    "default": "xxxxxxxx"
  }
}
```

```
┌───────────┬────────────────┬─────────────────────────────────┐
│ Emulator  │ Host:Port      │ View in Emulator UI             │
├───────────┼────────────────┼─────────────────────────────────┤
│ Firestore │ localhost:8080 │ http://localhost:4000/firestore │
└───────────┴────────────────┴─────────────────────────────────┘
```

なんで出してくれないんだろ。たしかにウェブ UI から本物のコンソールへのリンクが付いてたりはするけれど。
:::

### Java が必要

エミュレーターの実行に Java が必要です。インストールされていないと起動時にこけます。

```
⚠  firestore: Fatal error occurred:
   Firestore Emulator has exited because java is not installed, you can install it from https://openjdk.java.net/install/,
   stopping all running emulators
```

手元の Ubuntu (WSL) の場合。

```bash
$ sudo apt install default-jdk-headless -y
```

### 制限

- 最大同時接続数は 100
- 容量とかは [Spark プラン](https://firebase.google.com/pricing/)に従う

だそうです。他にもいくつかあるので[ドキュメント](https://firebase.google.com/docs/rules/unit-tests#differences_between_the_emulator_and_production)参照。

### `--help`

抜粋します。情報が少ない……。

```bash
$ npx firebase --help
…
Commands:
…
  emulators:exec [options] <script>                         start the local Firebase emulators, run a test script, then shut down the emulators
  emulators:export [options] <path>                         export data from running emulators
  emulators:start [options]                                 start the local Firebase emulators
…
  setup:emulators:database                                  downloads the database emulator
  setup:emulators:firestore                                 downloads the firestore emulator
  setup:emulators:pubsub                                    downloads the pubsub emulator
```

```bash
$ npx firebase emulators
Error: emulators is not a Firebase command

Did you mean emulators:exec?
```

### Android/iOS 端末の試験

エミュレーターじゃないんですが、モバイルアプリを Google が保有する実機とかで動かしてくれる機能があるそうです。

- [Firebase Test Lab](https://firebase.google.com/docs/test-lab)

[Pricing](https://firebase.google.com/pricing/) によると無料の Spark プランでも 1 日あたり仮想デバイス 10 件、物理デバイス 5 件まで利用できるとのこと。

自分は使ったことないです。

## おしまい

エミュレーターがあると中途半端なデータ構造とかでも動かして試せるようになるので開発が捗って良いですね。さらに試験（特に security rules）を書けるようになってさらに捗ります。素晴らしい。

### 参考

- [Firebase Local Emulator Suite の概要](https://firebase.google.com/docs/emulator-suite)
- [アプリを Cloud Firestore エミュレータに接続する  |  Firebase](https://firebase.google.com/docs/emulator-suite/connect_firestore)
- [Cloud Firestore セキュリティ ルールをテストする  |  Firebase](https://firebase.google.com/docs/firestore/security/test-rules-emulator)
