---
title: "Agoraで君だけの最強のClubhouseを作ろう"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["agora", "webrtc", "javascript"]
published: true
---

**Clubhouse に招待されないので自分で作ってみた**、みたいな感じです。まあ別に招待してくれなくていいんですけどね、大して興味ないしそのうちオープンになるだろうしそれにほらどうせ Android 使ってるしあのぶどうは酸っぱいし^[[すっぱい葡萄 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%81%99%E3%81%A3%E3%81%B1%E3%81%84%E8%91%A1%E8%90%84)]。

初挑戦で頑張って調べて書いてる感じなので何かあれば PR とかお願いします。

## 先にまとめ

- Clubhouse は Agora を使っているらしい
- Agora - Real-Time Voice and Video Engagement
   -  https://www.agora.io/en/
- 無料枠は音声 10,000 分/月
  - （誤って『10,000 時間/月』と記載してました。分です。すみませんでした。）
- 公式チュートリアル：[Start a Voice Call](https://docs.agora.io/en/Voice/start_call_audio_web_ng?platform=Web#5-leave-the-channel)
- 今回作ったデモ：[ginpei/try-agora](https://github.com/ginpei/try-agora)

![](https://storage.googleapis.com/zenn-user-upload/jyl8xt5kufepxa47jn7vlt5y858e)
*毎月 10,000 分無料とのこと。[価格のページより](https://www.agora.io/en/pricing/)*

![](https://storage.googleapis.com/zenn-user-upload/ssyjh3kx1rbq4i7bjdsczgpw162w)
*デモのスクリーンショット。開始ボタンや参加者の ID 一覧など*

## Agora?

- https://www.agora.io/en/

音声や映像のストリーミングを行う API を提供するサービス。従量課金制だけど[音声 10,000 分/月の無料枠](https://www.agora.io/en/pricing/)があり、クレジットカード登録等なしで気軽に試せました。ちなみに 10,000 分 ≒ 7日です。豪勢だ。

ウェブ版の SDK は CDN か `npm install agora-rtc-sdk-ng` でインストール。TypeScript で使える型情報もきっちり提供されていて嬉しい。他にも iOS、Android はもちろん Unity や Flutter、React Native等々幅広く提供されてます。

そして噂の Clubhouse はこれを使って運営されているらしい。

https://zenn.dev/voluntas/scraps/9403b803320d6f

> - 利用しているリアルタイム配信サービスは Agora.io
>   - DNS 見ただけ
>   - ap-japan.agora.io が見えた

## デモを試す

これから作り方を紹介するんだけど、できあがりは GitHub で公開しています。（ライブデモはありませんすみません。）

- [ginpei/try-agora](https://github.com/ginpei/try-agora)
  - 実装は `public/main.js` の `main()` から
  - 本質的でない UI 操作とかの実装は `ui.js`

試すには `git clone` の後 `public/secrets.example.js` を `public/secrets.js` へ複製し、設定をしてください。Agora のアカウント等が必要になります。

友達と会話を試す場合、[ngrok](https://ngrok.com/) を使って localhost を外部へ繋げるのを `npm run serve:proxy` で提供しています。

![デモのスクリーンショット。開始ボタンや参加者の ID 一覧など](https://storage.googleapis.com/zenn-user-upload/ssyjh3kx1rbq4i7bjdsczgpw162w)

## 作ってみる

ウェブ版（クライアント側 JavaScript）です。

### アカウント作成

まずは Agora のアカウントと、デモアプリ用のプロジェクトを作成してください。無料です。

- [Agora.io Signup](https://sso.agora.io/en/v2/signup)

後で必要な情報を取りに戻ってきます。

:::details もうちょっと
初回はなかった気がするんだけども、もしプロジェクト作成のところで Authentication Mechanism の選択が出てきたら Secure mode を選択してください。token なしはちょっとこわいでしょ。

![](https://storage.googleapis.com/zenn-user-upload/swkzssyf0cy642smnms17e58pmir)
*新規プロジェクト作成画面。Testing mode なら token 不要とある。*

Signup からはチュートリアルみたいな感じでどんどん進むので特につまづくこともないと思うけど、こちらの記事に詳しく書いてあるのを見かけましたので良ければ併せてご覧ください。

- [Clubhouseも使っているらしいAgoraを使って簡単にビデオ通話](https://zenn.dev/arahabica/articles/0f54f2cdb1a29d)
:::

### アプリの骨組み作成

適当な HTML を用意してください。画面をブラウザーで表示するところまでできたものとします。

### SDK 読み込み

今回のデモでは素の JavaScript で書きました。サーバーも [Browsersync](https://www.browsersync.io/) なのでごく単純な構成です。ライブラリーは CDN で配信されていますので、差し当たりこいつを使いましょう。

```html
<script src="https://download.agora.io/sdk/release/AgoraRTC_N-4.3.0.js"></script>
```

お好みで React なりへ混ぜこんでくださっても結構。その場合は npm パッケージの方をどうぞ。

```bash
$ npm install agora-rtc-sdk-ng
```

```js
import AgoraRTC from "agora-rtc-sdk-ng"
```

### クライアントの用意

ここから実装です。

最初に Agora のサーバーへ接続するクライアントのインスタンスを生成します。

```js
const client = AgoraRTC.createClient({ mode: "rtc", codec: "vp8" });
```

- [createClient(config: ClientConfig): IAgoraRTCClient](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartc.html#createclient)

この先この `client` オブジェクトを使いまわすので持っておいてください。

### インタラクション

利用者の操作 4 種と Agora からの通知 4 種を処理します。

利用者操作は以下のボタンを HTML にご用意ください。そんで `button.onclick` とか。

- join - チャンネル参加
- leave - チャンネル離脱
- publish - 音声送信開始（アンミュート）
- unpublish - 音声送信終了（ミュート）

Agora からの通知は `client.on()` で監視できます。以下のイベントを利用します。

- `user-joined` - 誰か join した
- `user-left` - 誰か leave した
- `user-published` - 誰か publish した
- `user-unpublished` - 誰か unpublish した

というわけで、こんな感じです。コールバックの関数はこれから作っていきます。

```js
document.querySelector("#join").onclick = onJoinClick;
document.querySelector("#leave").onclick = onLeaveClick;
document.querySelector("#publish").onclick = onPublishClick;
document.querySelector("#unpublish").onclick = onUnpublishClick;

client.on("user-joined", onAgoraUserJoined);
client.on("user-left", onAgoraUserLeft);
client.on("user-published", onAgoraUserPublished);
client.on("user-unpublished", onAgoraUserUnpublished);
```

### チャンネル参加

```js
async function onJoinClick() {
  const uid = await client.join(appId, channel, token, null);
}
```

- [join(appid: string, channel: string, token: string | null, uid?: UID | null): Promise<UID>](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#join)

さてここで 4 つの値が必要です。Agora のコンソール > [Project Management](https://console.agora.io/projects)から探してきてください。

:::details 値の探し方
- `appId` - "App ID" のやつ
- `channel` - これはお好きに。空白や記号も使える様子（`join()` API の仕様を参照）
- `token` - `channel` などを元に生成されるもの
  - [Project Management](https://console.agora.io/projects) > Edit > Features > Generate temp token で生成
  - 通常はプロジェクト作成時に Secure mode を選択するので実質必須
- `uid` - 自動生成させるので `null`

トークン生成の際は利用するチャンネル名 `demo_channel_name` を正確に入力してください。これが食い違うと CAN_NOT_GET_GATEWAY_SERVER エラーになります。

![](https://storage.googleapis.com/zenn-user-upload/482yx81mhogijn2so7drxray8tmm)
*Generate temp token の様子。チャンネル名を入力する。*

なお `token` は、デモではコンソールから生成できたけど本番では自力で生成します。秘密の情報 App certificate を用いる必要があるためです。コードが露出するクライアント側では行えません。サーバー側かサーバーレスなやつでやろう。

- [Generate a Token](https://docs.agora.io/en/Voice/token_server?platform=All%20Platforms)
  - いきなり C++ のコードが出てくるけど爽やかにスルーして Node.js を探してください
- 公式サンプル：[Tools/RtcTokenBuilderSample.js at master · AgoraIO/Tools](https://github.com/AgoraIO/Tools/blob/master/DynamicKey/AgoraDynamicKey/nodejs/sample/RtcTokenBuilderSample.js)
- App certificate と App Secret は同じものを指す様子
:::

### チャンネル離脱

`client.localAudioTrack` は配列で、後の publish 時に増えます。

```js
async function onLeaveClick() {
  // 音声のトラックを閉じる（必須）
  client.localTracks.forEach((v) => v.close());

  await client.leave();
}
```

- [localTracks: ILocalTrack[]](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#localtracks)
- [close(): void](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/imicrophoneaudiotrack.html#close)
- [leave(): Promise<void>](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#leave)

音声のトラックは次の `onPublishClick()` で作成します。

[離脱時にこの音声のトラックを閉じるのは必須](https://docs.agora.io/en/Voice/start_call_audio_web_ng?platform=Web#5-leave-the-channel)とのこと。（でも代わりに `client.unpublish()` でも良さそう？）（しなかったらどうなるんだろう？）（API の説明に記載がない？）

> Destroying the local media tracks is mandatory. You can follow your own implementation preferences.

これで参加と離脱ができるようになりました。聞き専ならこれで終わり。

### 音声送信開始

喋り始めます。

```js
async function onPublishClick() {
  const localAudioTrack = await AgoraRTC.createMicrophoneAudioTrack();
  await client.publish([localAudioTrack]);
}
```

- [createMicrophoneAudioTrack(config?: MicrophoneAudioTrackInitConfig): Promise<IMicrophoneAudioTrack>](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartc.html#createmicrophoneaudiotrack)
- [publish(tracks: ILocalTrack | ILocalTrack[]): Promise<void>](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#publish)

### 音声送信終了

自身の音声をミュートします。

```js
async function onUnpublishClick() {
  await client.unpublish();
}
```

- [unpublish(tracks?: ILocalTrack | ILocalTrack[]): Promise<void>](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#unpublish)

ちなみに音声トラックは `publish()` で複数登録できて、`unpublish()` の引数で指定して任意のものを閉じることができるようです。また省略すると全て閉じるとのこと。`leave()` との関係はどんな感じなんだろ？

まあともかくこれで自身の操作はできるようになりました。ここからは他の人の音声を取得していきます。

### 誰か join した

参加者一覧に表示するとかしてください。（実際デモのコードではここで更新を行っています。）

```js
// client.on("user-joined", onAgoraUserJoined);

async function onAgoraUserJoined(user) {
}
```

- [user-joined(user: IAgoraRTCRemoteUser): void](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#event_user_joined)
- [IAgoraRTCRemoteUser](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcremoteuser.html)

[`user: IAgoraRTCRemoteUser`](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcremoteuser.html) は以下のプロパティを持ちます。

- `audioTrack`
- `hasAudio`
- `hasVideo`
- `uid`
- `videoTrack`

人の名前やアイコン画像なんかは Agora の責務の範囲外なので自前で名寄せしてください。ここの `user.uid` はその人の `client.join()` の戻り値と合致するので、参加時に Redis へ置くとか。

### 誰か leave した

```js
// client.on("user-left", onAgoraUserLeft);

async function onAgoraUserLeft(user, reason) {
}
```

- [user-left(user: IAgoraRTCRemoteUser, reason: string): void](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#event_user_left)

`reason` は以下の 3 種。

- `"Quit"` - 離脱した
- `"ServerTimeOut"` - オフラインになった
- `"BecomeAudience"` - *role* が audience になった

[*role*](https://docs.agora.io/en/Voice/API%20Reference/web_ng/globals.html#clientrole) は本稿では扱っていません。

### 誰か publish した

話し始めたので傾聴します。

```js
// client.on("user-published", onAgoraUserPublished);

async function onAgoraUserPublished(user, mediaType) {
  const track = await client.subscribe(user, mediaType);
  track.play();
}
```

- [user-published(user: IAgoraRTCRemoteUser, mediaType: "audio" | "video"): void](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#event_user_published)
- [subscribe(user: IAgoraRTCRemoteUser, mediaType: "audio"): Promise<IRemoteAudioTrack>](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#subscribe)
- [RemoteAudioTrack](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iremoteaudiotrack.html)
- [play(): void](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iremoteaudiotrack.html#play)

### 誰か unpublish した

```js
// client.on("user-unpublished", onAgoraUserUnpublished);

async function onAgoraUserUnpublished(user, mediaType) {
}
```

- [user-unpublished(user: IAgoraRTCRemoteUser, mediaType: "audio" | "video"): void](https://docs.agora.io/en/Voice/API%20Reference/web_ng/interfaces/iagorartcclient.html#event_user_unpublished)

特に `close()` など音声トラックを操作する必要はないみたいです。

ちなみにここ、チュートリアルだと謎に `<div>` を削除しようとしてるんだけどそんなもの作ってないし確認もしていないのでエラーになります。なんだろ、古い API ではそういう作業があったのかな？

ともかくこれで人の声も聞こえるようになり、音声チャットが完成しました。やったね。

## トラブルシューティング

### 画面読み込み時のエラー

> AgoraRTCError NOT_SUPPORTED: enumerateDevices() not supported.

> AgoraRTCError WEB_SECURITY_RESTRICT: Your context is limited by web security, please try using https protocol or localhost.

→ `localhost` あるいは `https://` の URL で開いてください。`http://` でこのエラーになります。

### チャンネル参加時のエラー

> AgoraRTCException
> AgoraRTCError CAN_NOT_GET_GATEWAY_SERVER: dynamic use static key

→ `token` を `null` にせず、文字列を与える。ドキュメントには `string | null` とあるが実質必須。

> AgoraRTCError INVALID_PARAMS: Invalid token: . If you don not use token, set it to null

→ `secrets.js` で `token` を設定する。空文字列のままになっている。

> Choose server https://webrtc2-ap-web-1.agora.io/api/v1 failed, message: AgoraRTCError CAN_NOT_GET_GATEWAY_SERVER: dynamic key expired, retry: false

→ `token` を再生成する。期限切れ。24 時間程度？

> AgoraRTCError CAN_NOT_GET_GATEWAY_SERVER: invalid token, authorized failed

→ `token` が正しいチャンネル名で生成されたか確認する。

### `token` を取得するには

こちら：

- [Project Management](https://console.agora.io/projects) > Edit > Features > Generate temp token

あるいはこちらのドキュメントに従ってください： [Generate a Token](https://docs.agora.io/en/Voice/token_server?platform=All%20Platforms)

### 音声送信開始時のエラー

> AgoraRTCError INVALID_OPERATION: Can't publish stream, haven't joined yet!

→ 先に `client.join()` を呼ぶ。

### 誰かが leave したときのエラー

> TypeError: Cannot read property 'remove' of null

→ 要素が存在することを確認する。

ドキュメントでは `document.getElementById(user.uid)` の結果の要素を削除しろと言うんだけど、ドキュメント探してもそんなものは作られていないんだよなあ。

## おしまい

これで Clubhouse クローンが作れるようになりました。あとはログイン制御とかして、良い感じに UI を整えたら良さそうです。Agora 簡単だし無料枠もなかなかあるので、別のアプリに組み込んでもおもしろいかも。映像の方も。

君だけの最強の Clubhouse を作ろう！

### 参考

- [Agora Developer Center](https://docs.agora.io/en/Voice/landing-page?platform=Web)
- [Start a Voice Call](https://docs.agora.io/en/Voice/start_call_audio_web_ng?platform=Web) - 公式チュートリアル
- [Clubhouse リアルタイム配信の仕組みについて](https://zenn.dev/voluntas/scraps/9403b803320d6f)
- [Clubhouseも使っているらしいAgoraを使って簡単にビデオ通話](https://zenn.dev/arahabica/articles/0f54f2cdb1a29d)

[^1]: https://ja.wikipedia.org/wiki/%E3%81%99%E3%81%A3%E3%81%B1%E3%81%84%E8%91%A1%E8%90%84
