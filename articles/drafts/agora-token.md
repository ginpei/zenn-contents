---
title: "Agoraのトークンを（Firebase Functionsで）生成する"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["agora", "webrtc", "javascript", "firebase"]
published: false
---

## Agora?

https://zenn.dev/ginpei/articles/agora-voice-chat

## トークンの作り方

- [Generate a Token](https://docs.agora.io/en/Voice/token_server?platform=All%20Platforms)
  - [API reference](https://docs.agora.io/en/Voice/token_server?platform=All%20Platforms#api-reference)

```js
import { RtcTokenBuilder, RtcRole } from "agora-access-token";

const token = RtcTokenBuilder.buildTokenWithUid(
  appID,
  appCertificate,
  channelName,
  uid,
  role,
  privilegeExpiredTs // Unix time (sec)
);
```

## Firebase Cloud Functions

秘密情報を設定。

```bash
$ npx firebase functions:config:set agora.app_id="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
$ npx firebase functions:config:set agora.app_certificate="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

> Error: Invalid config name X, cannot use upper case.

### エミュレーターで使う

`.runtimeconfig.json` を設置する。

とりあえずサービスに設定されている情報を取得。

```bash
$ npx firebase functions:config:get > .runtimeconfig.json
```
