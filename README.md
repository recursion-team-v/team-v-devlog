# Recursion 上級者チーム開発 12/16 ~ 01/28 - Team-v 議事録

## 成果物

- [ボムボムパニック](https://bombompanic.vercel.app/)

<img src="uploads/play_movie.gif" style="width: 500px" />

- [Github](https://github.com/recursion-team-v/bomb)
- [Discord](https://discord.com/channels/684232065423900721/1051945817285599306)

---

## 概要

今回の要件はボンバーマンのようなオンライン対戦型ゲームを開発する事であり、ゲームの基本的な概要は、

- 爆風や敵キャラにタッチした場合死亡する
- 初期段階では、一回における爆弾は１つ、爆風は上下左右に１マス
- ブロックを爆風で破壊して以下のアイテムを取得できる、
  - ボムアップ：１回に置ける爆弾が１つ増える
  - ファイヤーアップ：爆風が１マス増える
  - スピードアップ：移動速度が早くなる

実装内容は各チームによって異なり、主に以下の三つのレベルを参考に取り組みます。

1. 初心者レベル

- オンライン対戦は実装せず、ボンマーマンゲームの基本的な概要のみの機能実装を目標とする

2. 中級者レベル

- オンライン対戦は実装せず、CPU のアルゴリズム強化や、ユーザ登録の機能実装を目標とする
- また、クリアするごとにステージが広くなったり敵キャラが強くなったりする機能の追加

3. 上級者レベル

- オンラインでのリアルタイム対戦の機能実装を目標とする
- ロビーを作成して、そのロビーに参加するためのコード発生機能また公開ロビー検索などの実装

---

## 要件定義（Requirements）

- **目的（Purpose)**:
  - ブラウザ上で動作するオンラインでのリアルタイム対戦の実装を最優先として、敵AIなども追加していく事を目標とする

- **機能要件（Functional Requirements)**:
  - ゲーム
    - 最大四人まで一緒に遊べる
    - プレイヤーの名前を入力できる
    - プレイヤーは入力キーで四方向に移動できる（斜め移動も可）
    - 爆弾を設置可能な位置のみに配置できる
    - 爆風は上下左右、プレイヤーに当たった場合ダメージを受ける
    - ボムアップ、ファイヤーアップ、スピードアップなどのアイテムを取得して効果を得られる
    - 敵AIはゲームの状態を把握していて、アイテム・ブロック・爆弾の位置を考慮して最適な行動を取る
    - 勝利条件は全ての対戦相手を倒した場合
    - 敗北条件はプレイヤーの残機数が尽きた場合
    - 引き分け条件はタイマーが止まるまでに勝利条件を満たしているプレイヤーがいない場合
    - ゲーム終了後上記の条件によって結果が表示される
  - ロビー
    - 部屋を作成できる
    - 部屋に参加できる
    - 部屋に参加した場合、現在参加している人数と各プレイヤーの名前が表示される
    - ゲーム終了後ロビーに戻れる

- **非機能要件（Non-Functional Requirements)**:
  - リアルタイム対戦ではレイテンシを最小限に抑え、遅延の高い接続地域にも対応し、滑らかで反応性が高いゲームプレイである事を優先する
  - ゲームには遊び続けたくなるようなグラフィックや効果音・BGMを追加する
  - Compatability
    - OS・ブラウザ関係なく動作する

---

## 議事録

### 12/16 - 開発環境の構築

**やったこと**

オンライン対戦の実装にサーバとクライアントが必要な為、モノレポで frontend と backend を管理し、frontend は React + Vite と TypeScript を使用。backend はコンパイル言語である Go を推奨していたが、リアルタイムゲームサーバのフレームワークが少ない事とクライアントとの相性が悪い理由として Node.js に変更。また、最低限の CI（github actions）として eslint + prettier を main へ対するプルリク時に実行させる様にした（関連 Issue: [最低限の ci を用意する #4](https://github.com/recursion-team-v/bomb/issues/4)）。

**直面した問題**

1. CI で prettier が変更内容を自動的に commit してくれず（おそらくレポへ対する権限の問題）、 CI を管理するボットを contributor として追加し、そのアカウント情報を commit 時に設定する事で解決

```yml
prettier:
  runs-on: ubuntu-latest
  ....
    - name: commit and push
      uses: EndBug/add-and-commit@v9.1.1
      with:
        message: "refactor: format code"
        committer_name: "github-actions[bot]" # 追加したボットアカウントの情報
        committer_email: "41898282+github-actions[bot]@users.noreply.github.com"
        push: true
```

**課題**

- ゲームエンジン（衝突判定・動作などの機能）の選定
- リアルタイムゲームサーバのフレームワークの選定
- ホスティング環境の選定

<br />

### 12/21 - 技術スタックの候補

**やったこと**

ホスティング環境は frontend と backend で以下を使用することにした。

- frontend:
  - vercel
- backend:
  - リアルタイムゲームサーバとしてレイテンシ（遅延）を最小限に抑える事が必須であったため、各サービスのレイテンシを [Cloud Ping Test](https://cloudpingtest.com/) を使って比較し、結論無料枠で使える GCP Cloud Run を使用（関連 Issue: [hosting 環境を決める #5](https://github.com/recursion-team-v/bomb/issues/5)）。

ゲーム機能の実装に必要なゲームエンジンの候補として、

- [PixiJS](https://pixijs.com/)
- [Phaser](https://phaser.io/phaser3)

リアルタイムゲームサーバのフレームワークの候補として、

- [geckos.io](https://geckos.io/)
  - WebRTC ベースで早いが、UDP をフルオープンにする都合上 VM が必要（マネージドサービスとの相性が悪い）
- [colyseus.io](https://www.colyseus.io/)
  - Node.js と websocket ベースのリアルタイムゲーム通信フレームワーク

**課題**

- 上記の候補を検証する

<br />

### 12/23 - 技術スタックの選定

**やったこと**

以下の様にゲームエンジンの比較を行い、結論 Phaser を使用。

- PixiJS
  - ドキュメントが少ないし、使っている記事も古いのが多い
  - 描画のみ行えるフレームワーク
  - rendering がそもそも大変で、キャラを移動させる方法もよくわからなかった
- Phaser
  - ドキュメントや事例が多い
  - 今回のゲームに必要である衝突判定・動作などが組み込まれている

一般的なゲームのサーバ・クライアントアーキテクチャを参考とし（下図の右）、

1. サーバでゲームをホストして、ゲーム内のロジック（衝突判定、プレイヤー動作）、状態（時間、爆弾の位置、アイテムの位置など）を管理する。その際、ゲームの物理エンジンをサーバでも持つ必要があるため、[Matter.js](https://brm.io/matter-js/) という物理エンジンを Phaser と共に使用（関連 Issue: [example: phaserjs #27](https://github.com/recursion-team-v/bomb/issues/27)）。
2. クライアントはプレイヤーのキーの入力や情報をサーバに送り、サーバがそれを検証してその結果をクライアントに返す

<img src="uploads/game_architecture.png" style="width: 500px" />

また、このアーキテクチャは Authoritative Server とも呼ばれ、クライアント側を信頼せずに全てサーバ側で管理するという意味ではチート防止などにも役立つ。

**課題**

- 一旦、通信は後回しにしてフロントエンドで全部動かす。できてから、バックエンドや共通ライブラリに切り分ける。
  - キャラを動かせる
  - ステージがある
  - 壁がある
  - 爆弾が置ける
  - 爆弾が爆発する
  - アイテムが取れる

<br />

### 12/31 - サーバに Colyseus と Matter.js を追加

**やったこと**

<img src="uploads/server_player_movement.gif" style="width: 300px" />

- プレイヤーと壁・爆風の衝突判定の実装
  - Matter.js の ```collisionStart``` イベントを使って以下の考えられる衝突の組み合わせを実装
```
    player - item
    player - bomb
    player - explosion
    player - wall
    player - block(破壊できる)
    player - player
    bomb - item
    bomb - wall
    bomb - block
    bomb - bomb
    bomb - explosion
    item - wall ?
    item - block ?
    item - explosion
```
- サーバ側でのプレイヤー動作の実装
- サーバ・クライアントで秒間 60 回の更新（60FPS）

**直面した問題**
1. Matter.js の衝突判定の callback に渡される引数の型が不明である（関連 Issue: [Game: 衝突判定 #50](https://github.com/recursion-team-v/bomb/issues/50)）
2. サーバでプレイヤー動作を管理しているため、クライアントでのプレイヤーの移動がスムーズじゃない
  - クライアント側で線形補間を使用してプレイヤーの位置を予測することで、プレイヤーの移動をスムーズにし、より連続的に描画できる様にした
  
```ts
this.x = Math.ceil(Phaser.Math.Linear(this.x, this.serverX, 0.35));
this.y = Math.ceil(Phaser.Math.Linear(this.y, this.serverY, 0.35));
```

**課題**

- サーバ側での爆弾・爆風の実装
- サーバ側でのアイテムの実装

<br />

### 01/07 - サーバ側でのゲームロジックの管理

**やったこと**

<img src="uploads/game1.gif" style="width: 300px" />

今まで一旦クライアントで実装していた以下をサーバ側で管理する様にした。
- ゲーム進行時のタイマー（関連 pull request: [タイマーをサーバ側に寄せた #98
](https://github.com/recursion-team-v/bomb/pull/98)）
- マップの生成（ブロックの配置）（関連 pull request: [#75 feat: generate map on server #85
](https://github.com/recursion-team-v/bomb/pull/85)）
- ボムの設置（関連 pull request: [サーバ側で爆発を追加 #133
](https://github.com/recursion-team-v/bomb/pull/133)）
- アイテムの配置（関連 pull request: [サーバー側にアイテムを設置 #130
](https://github.com/recursion-team-v/bomb/pull/130)）

**直面した問題**
1. 爆弾の同期
  - 爆弾を設置する場合の通信フローは以下のようになっている。

  <img src="uploads/bomb_sync_architecture1.png" style="width: 500px" />
  
  - ローカルAから爆弾設置リクエストが飛んでくるとサーバは全てのクライアントに通知する。
  - この時、各クライアントとサーバ間は異なる latency で動作しているため、クライアントがレスポンスを受け取ったタイミングで爆弾を設置すると、クライアント間で異なる状況になってしまう可能性がある。
  - そのため、各クライアントとサーバ間で同じ時刻を共有した上で、以下の図のように 「いつ爆発させるのか？」を含めた情報を同時に送り、クライアントの画面で爆発するタイミングを同期させる様にした。

  <img src="uploads/bomb_sync_architecture2.png" style="width: 500px" />

**課題**

- ロビーを作る
  - 部屋を作れる
  - 他のプレイヤーが存在している部屋に参加できる
  - ゲーム開始後部屋に参加出来ない様にする
- ゲームの流れ
  - タイトル -> ロビー -> ゲーム -> リザルト

<br />

### 01/13 - ゲーム全体の流れ

**やったこと**

<img src="uploads/game2.gif" style="width: 500px" />

- 一通りゲーム全体の流れを実装
- ロビーでは各プレイヤーが以下のゲーム状態を保持し、
```ts
export const GAME_STATE = {
  WAITING: 1, // ゲーム開始前
  READY: 2, // ゲーム開始準備完了
  PLAYING: 3, // ゲーム中
  FINISHED: 4, // ゲーム終了
} as const;
```
1. 部屋（colyseus room）の作成後 ```WAITING``` 状態
2. Ready? ボタンでゲーム開始の準備ができ、いつでも始めて OK という ```READY``` 状態
3. 入室している全 client が ```READY``` 状態になった場合サーバでゲームを開始して ```PLAYING``` 状態
4. ゲーム終了時に ```FINISHED``` 状態

**直面した問題**
1. 誘爆の同期処理
- 誘爆の場合下の図のように爆弾が爆発する直前に、クライアントから爆弾設置のリクエストが送られてくると、（爆風の範囲内であれば）まだクライアントに同期していない爆弾も爆発してしまう。

<img width="600" alt="20230217005058" src="https://user-images.githubusercontent.com/45121253/221391847-d0065521-a5c8-4fdc-aec6-38dcc7e24df6.png">

- そこで各クライアントとサーバ間で「いつ爆発させるのか？」だけではなく 「いつ爆弾を設置するのか？」 を含めた情報を同時に送り、クライアントの画面に出現するタイミングと爆発するタイミングを両方同期するようにした。

<img width="600" alt="20230217004053" src="https://user-images.githubusercontent.com/45121253/221391890-9b8ffd5c-c119-4064-9d9a-e936904412e1.png">

**課題**

↑優先
- ゲームスタートしてからタイマースタートしてほしい
- ゲームのアセットを入れ替える
- リザルトのバグ修正
- CPU 組み込み

<br />

### 01/26 - タッチアップ

**やったこと**

![215282387-64b98417-3c72-4069-bb61-288eab4fa98d_AdobeExpress](https://user-images.githubusercontent.com/45121253/221392134-7c79c996-910b-43a1-a1f9-a2be696220e2.gif)

 - ゲームアセットの差し替え
 - ロビー周りの修正
   - 部屋に参加している人数を表示
   - 各プレイヤーが Ready かどうかをロビーで表示
 - 敵AIの導入

**直面した問題**
1. Google Cloud Run での CPU 使用率
- 敵AIを追加した事で、一人で対戦する場合敵AIは三人追加され、GCPのメトリックスを検証すると CPU 使用率がかなり高くなっている

<img width="300" alt="image" src="https://user-images.githubusercontent.com/45121253/221392299-6aa49f07-72d0-4cf6-8e16-e4631f30efa5.png">

- また、複数部屋を開いた場合動作しない時も発生
- シングルスレッドで複数の部屋と敵AIを管理しているため、かなりCPUに負荷がかかっている
- 考えられる解決策として、
  - 敵AIの計算量を下げる
  - Node.js の Worker Threads モジュールを活用してマルチスレッドにする（ただし、現状シングルスレッドであることを前提に作っているので、マルチスレッドで正しく動くかは不明であり、作業量もかなり多い）
  - redis などの外部ストレージに状態を持たせて部屋ごとに cloud run を分ける
