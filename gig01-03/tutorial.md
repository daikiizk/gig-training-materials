# G.I.G ハンズオン #3

## Google Cloud Platform（GCP）プロジェクトの選択

ハンズオンを行う GCP プロジェクトを作成し、 GCP プロジェクトを選択して **Start/開始** をクリックしてください。

**今回のハンズオンは Firebase, Firestore を使って行うため、既存のプロジェクト (特にすでに使っているなど) だと不都合が生じる恐れがありますので新しいプロジェクトを作成してください。**

<walkthrough-project-setup>
</walkthrough-project-setup>

## ハンズオンの内容

### 目的

Firestore と Firebase を使って実装が複雑になりがちな認証、クライアントとのリアルタイム同期がスムーズに行えることを体験します。

### シナリオ

1. Firebase Hosting を使って静的ファイルをホスティング
2. Firestore のデータを更新してリアルタイムにデータが更新されることを確認
3. Firebase Authentication にて認証
4. Firestore Security Rules を用いてログイン済みユーザのみが閲覧可能なデータを作成

### 対象プロダクト

以下が今回学ぶ対象のプロタクトの一覧です。

- Firebase Hosting
- Firebase Authentication
- Firestore
- Firestore Security Rules

### 下記の内容をハンズオン形式で学習します。

- 環境準備: 10 分
  - GCPプロジェクト作成
  - gcloud コマンドラインツール設定
  - Firebase CLI のインストール
  - Firestore API 有効化
  - Firebase プロジェクト作成
  - Firebase で使用するリージョンの設定

- Firebase を用いた Web アプリケーション作成: 25分
  - Firebase CLI の初期化
  - Firebase プロジェクトの初期化

  - Firestore Database 初期設定
  - アプリケーションのデプロイ(Firebase Hosting)
  - Firestore をクライアント側と同期
  - アプリケーションの認証
  - Firestore Security Rules を用いたセキュアなデータ管理

- クリーンアップ: 10 分
  - プロジェクトごと削除
  - (オプション) 個別リソースの削除
    - Firebase プロジェクトの削除
      - 可能なのか
        - yes => まるごと削除
        - no => Firestore の削除, Firebase のアプリケーションの登録解除, Firestore Security Rulesの削除

## 環境準備

<walkthrough-tutorial-duration duration=10></walkthrough-tutorial-duration>

最初に、ハンズオンを進めるための環境準備を行います。

下記の設定を進めていきます。

- gcloud コマンドラインツール設定
- Firebase CLI のインストール
- Firestore API 有効化
  - Firestore 初期設定
- Firebase プロジェクト作成

## gcloud コマンドラインツール設定

前回と同様の内容なので、設定完了の方はスキップしてください。

### GCP プロジェクト ID を環境変数に設定

`GOOGLE_CLOUD_PROJECT` に GCP プロジェクト ID を設定します。

```bash
export GOOGLE_CLOUD_PROJECT="{{project-id}}"
```

### gcloud コマンドラインツールから利用するデフォルトプロジェクトを設定

プロジェクトを設定します。

```bash
gcloud config set project ${GOOGLE_CLOUD_PROJECT}
```

以下のコマンドで、現在の設定を確認できます。

```bash
gcloud config list
```

## Firebase CLI のインストール

```bash
npm install -g firebase-tools
```

## Firestore API 有効化

今回のハンズオンでは Firestore ネイティブモードを使用します。

1. [Datastore](https://console.cloud.google.com/datastore/entities/query/kind?project={{project-id}})に移動します
2. ![switch to native mode](https://storage.googleapis.com/gig-03/static/screenshot/firestore-select-mode.png) の画面よりネイティブモードを選択します
3. ![select to nam5](https://storage.googleapis.com/gig-03/static/screenshot/firestore-select-region.png) の画面より nam5 リージョンを選択します
4. リージョンを選択後、 *データベースを作成* ボタンをクリックします
5. `データベースを作成しています。` というメッセージが表示され、Firestore のデータベースを表示するページにリダイレクトされます

## Firebase プロジェクト作成

1. [Firebase Console](https://console.firebase.google.com/)に移動します
2. ![create a new firebase project](https://storage.googleapis.com/gig-03/static/screenshot/firebase-create-project.png) *プロジェクトを追加* をクリックします
3. ![name firebase project id](https://storage.googleapis.com/gig-03/static/screenshot/firebase-name-project-id.png) プロジェクト名にGCP プロジェクト ID を入力して *続行* をクリックします
4. ![firebase billing plan](https://storage.googleapis.com/gig-03/static/screenshot/firebase-billing-plan.png)Blaze プラン (従量課金制) になっていることを確認し、 *プランを確認* をクリックします
5. **続行** をクリックします
6. ![firebase disalbe ga](https://storage.googleapis.com/gig-03/static/screenshot/firebase-disable-ga.png)今回はGoogle Analyticsを使用しないので、こちらは一旦 *有効にする* のボタンをOFFにして *Firebaseを追加* をクリックします
7. **新しいプロジェクトの準備ができました** と表示されたらプロジェクトの作成は完了です

## Firebase で使用するリージョンの設定

1. [Firebase Console 内の Setting ページ](https://console.firebase.google.com/project/{{project-id}}/settings/general) に移動します
2. ![Firebase default location](https://storage.googleapis.com/gig-03/static/screenshot/firebase-default-location.png) *デフォルトの GCP リソースロケーション* を `nam5` に設定します

これにて環境準備は完了です。

## Firebase を用いた Web アプリケーション作成

<walkthrough-tutorial-duration duration=25></walkthrough-tutorial-duration>

Firebase プロジェクト上に Web アプリケーションを作成し、

- Firebase Hosting
- Firebase Authentication
- Firestore Security Rules

を連携していきます。

## Firebase CLI の初期化

Firebase CLI が使用できるように初期化を行います。

```bash
firebase login --no-localhost
```

1. 上記コマンドの結果、表示されたURLをブラウザにて開きます
2. アカウントの選択画面が表示されるので Google Account を指定します
3. ![allow Firebase CLI](https://storage.googleapis.com/gig-03/static/screenshot/allow-firebase-cli.png) *許可* をクリックします
4. ![firebase authorization code](https://storage.googleapis.com/gig-03/static/screenshot/firebase-authorization-code.png) 認可コードが表示されるので、コピーし、CLIの `Paste authorization code here:` の項目にペーストします
5. `Success! Logged in as 選択した Google Account` が表示されれば完了です

```bash
firebase projects:list
```

こちらのコマンドで権限のある Firebase Project の一覧を確認できます。

## Firebase プロジェクトの初期化

```bash
firebase init
```

対話式のコマンドが立ち上がります。

### 1. 使用する Firebase CLI の選択

```
? Which Firebase CLI features do you want to set up for this folder? Press Space to select features, then Enter to confirm your choices.
 ◯ Database: Configure Firebase Realtime Database and deploy rules
 ◉ Firestore: Deploy rules and create indexes for Firestore
 ◯ Functions: Configure and deploy Cloud Functions
❯◉ Hosting: Configure and deploy Firebase Hosting sites
 ◯ Storage: Deploy Cloud Storage security rules
 ◯ Emulators: Set up local emulators for Firebase features
 ◯ Remote Config: Get, deploy, and rollback configurations for Remote Config
```

矢印キーまたは j k で上下に移動し、 `Firestore` , `Hosting` を space キーで選択肢、 Enter キーで決定します。

### 2. Firebase プロジェクト名の入力

```
? Please select an option: (Use arrow keys)
❯ Use an existing project
  Create a new project
  Add Firebase to an existing Google Cloud Platform project
  Don't set up a default project
```

`Use an existing project` を選択肢、決定します。

```
? Please input the project ID you would like to use:
```

先程作成した Firebase プロジェクト名 ({{project-id}}) を入力し決定します。

### 3. Firestore Security Rules の設定ファイルの名前の設定

```
? What file should be used for Firestore Rules? (firestore.rules)
```

デフォルトのまま変更せずに決定します。

### 4. Firestore Indexes の設定ファイルの名前の設定

```
? What file should be used for Firestore indexes? (firestore.indexes.json)
```

デフォルトのまま変更せずに決定します。

### 5. デプロイの際の公開用ディレクトリの名前の設定

```
? What do you want to use as your public directory? (public)
```

デフォルトのまま変更せずに決定します。

### 6. single-page app の設定

```
? Configure as a single-page app (rewrite all urls to /index.html)? (y/N)
```

N (No) を入力して決定します。

### 7. Github 連携の設定

```
? Set up automatic builds and deploys with GitHub? (y/N)
```

N (No) を入力して決定します。

### 8. 404.html の上書き設定

```
? File public/404.html already exists. Overwrite? (y/N)
```

今回は 404.html を別途用意してあるので N (No) を入力して決定します。

### 9. index.html の上書き設定

```
? File public/index.html already exists. Overwrite? (y/N)
```

今回は index.html を別途用意してあるので N (No) を入力して決定します。

```
i  Writing configuration info to firebase.json...
i  Writing project information to .firebaserc...
i  Writing gitignore file to .gitignore...

✔  Firebase initialization complete!
```

が表示されれば完了です。


## TBD

- firebase hostingを使ってHTMLをpublishしてUIを作成する
- 未ログイン状態でUIに何も表示させてないことを確認
  - ここ、reactにするか何にするか決める
- publicなデータをfirestoreに突っ込む
  1. firestoreのテーブル作成
  2. firestoreに/publicにデータを入れる
  3. firestoreに/privateにデータを入れる
- publicなデータに対して、security ruleでpublic属性をつける
- privateなデータに対して、security ruleでprivate属性をつける
  - ログイン済みユーザのみが見れる状態にする
- UIに戻ってpublicなデータが見れるか確認する
- ログインの画面を出す
- ログインする
- ユーザが登録されているかをconsole上もしくはCLIで確認する
  - ここ、何で確認できるか確認する
- UIに戻ってprivateなデータが見れるか確認する

## TO-DO

- projectの準備