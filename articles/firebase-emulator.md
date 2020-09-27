---
title: "Firestoreのエミュレーターを使ってローカルで開発する"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "firestore", "開発環境"]
published: false
---

もしかしたら皆さん知ってたんでしょうか。おれは最近知りました。（普通にドキュメントに載ってる。）

### 先にまとめ

- エミュレーターがあればだいたいローカルで開発できるぞ
- `npx firebase init` でインストール
- `npx firebase emulators:start` で起動
- `app.firestore().settings({ host: "localhost:8080" })` でアプリから接続
- 公式：[Firebase Local Emulator Suite の概要](https://firebase.google.com/docs/emulator-suite)

Firestore 以外の Firebase の機能もあるんだけど、本稿では Firestore に絞ってお話しします。

※↓こんな感じです

![ウェブ UI のスクリーンショット。Firebase ターミナルの Firestore 画面に似ている](https://storage.googleapis.com/zenn-user-upload/ek4trbhecxuz6jt4mi38y2hmd974)

### エミュレーターでできること

- 複数人で DB を使いまわさなくて済む
- Security rules をデプロイする前に試せる
- スナップショットでやり直せる
- DB を利用する試験を書ける
  - CI でもいける
- 試験は完全ローカルでできるし CI でも動く
- Firebase コンソールみたいなウェブ UI もある

### エミュレーターでできないこと

- Firebase 全体のエミュレーターでは**ない**ので、完全ローカルにはならない
- ログイン (Authentication) のエミュレーターがない
  - 試験では、ライブラリーで擬似ログインできる

## インストール

Firebase の諸機能と同様 `firebase init` からインストールできます。

ちなみに既に Firebase の設定が終わってる場合でも気にせず init し直して大丈夫。（エミュレーターに限らず。）　またエミュレーターだけ入れたい場合は `firebase init emulator` と最初から指定するとちょっと工程が短縮されるようです。

### 事前準備

`firebase-tools` をインストールしていない場合、準備してください。まあ Firebase の話してるのでもう入ってると思うけど。

ここでは開発用の依存パッケージ `-D` としてインストールしてるけどお好みでグローバル `-g` でも構いません。

```console
$ npm install -D firebase-tools
$ npx firebase --version
```

エラーなくバージョンが表示されれば成功。

### エミュレーターのインストール

```console
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
   - 他にも必要なものあれば
6. "Which port do you want to use for the firestore emulator?"
7. "Would you like to enable the Emulator UI?"
8. "Which port do you want to use for the Emulator UI"
9. "Would you like to download the emulators now?"
   - `y`
   - ダウンロードが始まります
   - 間違えて `N` にしてしまったら気にせず `init` し直してください。ダウンロードだけなら `npx firebase setup:emulators:firestore` でもできます
10. "Firebase initialization complete!"

おつかれさまでした。

## 起動

起動したままターミナルを占有するので別タブとか用意しといてください。（Windows なら cmd.exe じゃなくて MS 公式の [Windows Terminal](https://www.microsoft.com/ja-jp/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab) 使いましょう。）

```console
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

## アプリをエミュレーターへ接続

JavaScript の場合はこう。[公式ドキュメント](https://firebase.google.com/docs/emulator-suite/connect_firestore#android_ios_and_web_sdks)には Android (Java) と iOS (Swift) の例も載ってます。

```javascript
const isEmulating = location.hostname === "localhost";
if (isEmulating) {
  firebase.firestore().settings({
    host: "localhost:8080",
    ssl: false,
  });
}
```

`isEmulating` は何でも良くて、自分の場合は環境変数 `process.env.REACT_APP_FB_EMU` を見るようにしました。（[CRA](https://create-react-app.dev/) では `REACT_APP_*` はクライアント側へ露出します。）

## ウェブ UI

ブラウザーで http://localhost:4000 を開くと色々出てきて、内容を閲覧したり編集したりできます。（ポート番号は変わるかもしれません。ターミナル出力をご確認ください。）

![ウェブ UI のスクリーンショット。Firebase ターミナルの Firestore 画面に似ている](https://storage.googleapis.com/zenn-user-upload/ek4trbhecxuz6jt4mi38y2hmd974)

アプリの動作に合わせて更新されたりウェブ UI での入力がアプリへ反映されたりするのを確認しましょう。すげー。

これで Google のサーバーにあるデータを気にせずローカルであれこれ試せるようになりました。

## データの保存と読み込み（インポート、エクスポート）

基本的にはデータはエミュレーター終了と共に破棄されますが、保存して同じ内容を読み込みなおしたりもできます。

### 保存（エクスポート）

エミュレーターが起動している状態で、こう。

```console
$ npx firebase emulators:export path/to/export
```

指定のディレクトリー配下にファイルが出力されます。JSONかと思ったらバイナリです。

### 読み込み（インポート）

エミュレーターを起動する際に `--import` オプションを加え、エクスポートしたディレクトリーを指定します。

```console
$ npx firebase emulators:start --import=path/to/export
```

### 自動保存

エミュレーター終了時に自動で保存というのもできます。アプリを動かしながら試行錯誤するときに便利。

`--export-on-exit` オプションを与えます。保存先ディレクトリーを指定できるけど、省略して `--import` と併用すると自動的に同じ場所へ書き込むのでその方が使い勝手が良いことでしょう。

```console
$ npx firebase emulators:start --import=path/to/export --export-on-exit
```

## Firestore へ接続するコードや Security Rules を試験する

JavaScript からは `@firebase/rules-unit-testing` という npm パッケージを利用するとうまいこと試験できます。近々記事にしますね。

ちなみに本稿執筆時点でドキュメントの日本語版では `@firebase/testing` が紹介されてるんですが、これは 2020 年 8 月に deprecated になったようです。知らなかったのでこの古い方で例を用意していました。つら。

### Security rules

Firestore やってると `firestore.rules` を編集して Security rules をエンヤコラする必要があります。

エミュレーターは変更を検知して再起動なしで反映してくれるので、ローカルでルールの挙動を確認することができます。代わりに本番 Firestore の方にあるやつはエミュレーターに搭載されてないです。

※↓本番 Firestore の方にあるやつ（コンソール > Firestore > Rules > Rules Playground）

![ルール確認に便利そうな設定画面のスクリーンショット](https://storage.googleapis.com/zenn-user-upload/tjqawl03zm98u36n0yapk6qkfjeh)

## その他

### ポート使用済みエラー

たまにこういうのが出ます。

```
Error: Could not start Firestore Emulator, port taken.
```

読んでそのまま、`firebase.json` で指定のポート（通常 `8080`）が他のアプリに取られてるって話なので、それを終了させるか他のポートを指定してください。

ただなんかときどきエミュレーターが終了失敗してポートを占拠し続けることが……ある……ような……。

### エミュレーターがスッと落ちて起動しない

スッ……

```console
$ npx firebase emulators:start --import=emu-export
i  emulators: Starting emulators: firestore
i  emulators: Shutting down emulators.

Error: An unexpected error has occurred.
```

パス間違えたとかでインポート先が用意されてないときにこうなりました。エラー出してよ。

インポート先をちゃんと用意するか、 `--import` オプションを外して起動してください。

### ウェブ UI が出てこない

理由はよくわからないんですが、default project を設定していないと Firestore エミュレーターは出るけど UI が出てきません。

```
┌───────────┬────────────────┐
│ Emulator  │ Host:Port      │
├───────────┼────────────────┤
│ Firestore │ localhost:8080 │
└───────────┴────────────────┘
```

適当に `.firebaserc` を用意してやれば出てきます。空文字列でなければ何でも良さそう。

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

なんで？

### Java が必要

エミュレーターの実行に Java が必要らしいです。

うちの環境には入ってたので未確認だけどなんか動かなかったらそこら辺確認してみてください。

### Security rules が反映されない

再起動なしで反映、と前述しましたが、なんだか Jest と組み合わせがどうにか悪いのか試験として実行するときはエミュレーターの再起動が必要でした。謎。アプリからは即時反映される。

## おしまい

WIP

### 参照

- [アプリを Cloud Firestore エミュレータに接続する  |  Firebase](https://firebase.google.com/docs/emulator-suite/connect_firestore)
- [Cloud Firestore セキュリティ ルールをテストする  |  Firebase](https://firebase.google.com/docs/firestore/security/test-rules-emulator)
