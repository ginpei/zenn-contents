---
title: Firebase Firestoreでタイムライン作るならfan-outする
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "firestore"]
published: false
---

- `src/index.ts`

```ts
export * from "./follow";
```

- `src/follow.ts`

```ts
import * as functions from "firebase-functions";
import {db} from "./firebase";

export const follow = functions.https.onCall(async (data, context) => {
  const userId = context.auth?.uid;
  if (!userId) {
    throw new Error("User must have logged in");
  }

  const {userId: targetId} = data;
  if (typeof targetId !== "string") {
    throw new Error(`User ID must be string but got ${typeof targetId}`);
  }

  if (!targetId) {
    throw new Error("User ID must be set");
  }

  // ここから本編

  const collUsers = db.collection("users");
  const docUser = collUsers.doc(userId);
  const docTarget = collUsers.doc(targetId);

  // 一括処理
  db.runTransaction(async (transaction) => {
    // 自分が相手をフォロー
    const pAddFollowings = transaction.get(docTarget).then((ssTarget) => {
      transaction.set(
          docUser.collection("followings").doc(targetId),
          ssTarget.data()
      );
    });

    // 自分を相手のフォロワーへ追加
    const pAddFollower = transaction.get(docUser).then((ssUser) => {
      transaction.set(
          docTarget.collection("followers").doc(userId),
          ssUser.data()
      );
    });

    // 両方待つ
    await Promise.all([pAddFollowings, pAddFollower]);
  });
});
```
