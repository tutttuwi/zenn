---
title: "Kubenetes入門"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## はじめに

### Kubenetesの目的

- 分散システムを構築し、デプロイし、メンテナンスするタスクを根本的にシンプルにすること

### サンプルリポジトリ

<https://github.com/kubernetes-up-and-running/examples>

## 1章　Kubernetes入門

- Kubenetesはオープンソースのオーケストレータ
- 当初はGoogleが開発

### 1.1　ベロシティ

- ベロシティ(開発者の効率を強化する機能)

可用性を保ちながらもすばやく進化していくのに必要なツールを、コンテナとKubernetesは提供できます。

- これを可能にする核となるコンセプトが、
  - イミュータブルであること（immutability）
  - 宣言的設定（declarative configuration）
  - オンラインで自己回復するシステム（online self-healing system）

- これらの考え方がすべて関連し合うことで、信頼性を保ちながらソフトウェアをデプロイするスピードを劇的に速くします。

#### 1.1.1　イミュータブルであることの価値

- ミュータブルなインフラ
  - `apt-get update`ツールを使ったシステムのアップデートはミュータブルなシステムに更新を加える良い例
- イミュータブルなインフラ
  - 積み重ねられてきた一連の更新や変更の代わりに、全く新しいイメージが作られ、
  - 1回のオペレーションでイメージ全体をその新しいイメージで置き換えてしまいます。
  - 変更が積み重ねられていくことはありません。これは、設定管理に関する従来の考え方から、大きく異なります。

- それぞれの例
  - コンテナにログインし、新しいソフトウェアをダウンロードするコマンドを実行し、ソフトウェアをインストールし、サーバプロセスを再起動する。
  - 新しいコンテナイメージを構築し、そのイメージをコンテナレジストリにプッシュし、既存のコンテナを停止し、新しいコンテナを起動する。

#### 1.1.2　宣言的設定

- 命令的設定
  - 一連の命令の実行によって状態が定義される
- 宣言的設定
  - 望ましい状態を宣言する

#### 1.1.3　自己回復するシステム

- 望ましい状態が宣言されたら、その状態に一致するように継続して動く

### 1.2　サービスとチームのスケール

#### 1.2.1　分離

- 分離アーキテクチャ
  - 定義済みのAPIとサービスロードバランサによって他のコンポーネントから分けられています。
  - APIとロードバランサは、システムの各部分を隔離します。
  - APIは、APIを実装するコンポーネント（implementer）と、
  - APIを使用するコンポーネント（consumer）の間のバッファになり、ロードバランサは各サービスを実行しているインスタンス間のバッファになっています。

#### 1.2.2　アプリケーションとクラスタの簡単なスケール

#### 1.2.3　マイクロサービスによる開発チームのスケール

- 理想的なチームのサイズは「ピザ2枚分のチーム」：6~8人が理想

- Kubenetesは分離されたマイクロサービスアーキテクチャの構築を簡単にする抽象化層やAPIを提供
  - Pod：コンテナグループ
  - Service：マイクロサービス間の分離のためにロードバランシングやネーミング、ディスカバリの機能を提供
  - Namespace：マイクロサービス同士の連携範囲を制御できるように、分離とアクセス制御
  - Ingress：複数のマイクロサービスをまとめつつ、単一の外部からアクセス可能なAPIを提供できる、簡単なフロントエンドを提供

#### 1.2.4　一貫性とスケールのための依存関係の切り離し

- `not my monkey, not my circus`「自分には関係ない」
- コンテナオーケストレーションAPIが、クラスタオーケストレーションのオペレータの責任と、アプリケーションオペレータの責任とを切り離すためのルールとして存在
- アプリケーション開発者はSLAがどのように実現されているか期にせずに、コンテナオーケストレーションAPIによって提供されるSLAに依存できる

### 1.3　インフラの抽象化

- Kubenetesのようなアプリ指向のコンテナAPIへ移行する利点
  - 特定のマシンから開発者を分離

- OSSストレージソリューションを利用することで、ポータブルになる

### 1.4　効率性

### 1.5　まとめ

## 2章　コンテナの作成と起動

### 2.1　コンテナイメージ

- コンテナのカテゴリ
  - システムコンテナ：仮想マシンとよく似た動きをし、完全なブートプロセスを実行
  - アプリケーションコンテナ：アプリケーションを１つだけ動かす

#### 2.1.1　Dockerイメージフォーマット

## コンテナのレイヤ

### 2.2　Dockerでのアプリケーションイメージの作成

アプリケーションコンテナを中心に見ていく

#### 2.2.1　Dockerfile

#### 2.2.2　イメージのセキュリティ
#### 2.2.3　イメージサイズの最適化
### 2.3　リモートレジストリへのイメージの保存
### 2.4　Dockerコンテナランタイム
#### 2.4.1　Dockerでコンテナを動かす
#### 2.4.2　kuardアプリケーションへのアクセス
#### 2.4.3　リソース使用量の制限
### 2.5　後片付け

- docker-gcが紹介されていたが、今はpublic archives<https://github.com/spotify/docker-gc>

### 2.6　まとめ

アプリケーションコンテナを使うと、アプリケーションをきれいに抽象化できます。Dockerイメージフォーマットでパッケージすれば、
ビルドもデプロイも配布も簡単になります。コンテナは同じマシン上で動くアプリケーション同士を分離する機能も提供し、依存性が衝突するのを防ぎます。

## 3章　Kubernetesクラスタのデプロイ
### 3.1　パブリッククラウドへのKubernetesのインストール
#### 3.1.1　Google Kubernetes EngineへのKubernetesのインストール
#### 3.1.2　Azure Container ServiceへのKubernetesのインストール
#### 3.1.3　Amazon Web ServiceへのKubernetesのインストール
### 3.2　minikubeを使ったローカルへのKubernetesのインストール
### 3.3　Raspberry PiでKubernetesを動かす
### 3.4　Kubernetesクライアント
#### 3.4.1　クラスタのステータス

- バージョン確認：`kubectl version`
- クラスタが正常かどうか確認：`kubectl get componentstatuses`
- クラスタ上のすべてのノードを表示：`kubectl get nodes`
- ノードの詳細確認：`kubectl describe nodes node-1`

#### 3.4.2　Kubernetesのワーカノードの表示

### 3.5　クラスタのコンポーネント

#### 3.5.1　Kubernetes proxy

#### 3.5.2　Kubernetes DNS

#### 3.5.3　KubernetesのUI

- `kubectl proxy`

### 3.6　まとめ

## 4章　よく使うkubectlコマンド

Kubenetes APIとやり取りするのに、`kubectl`コマンドを使う

### 4.1　Namespace

クラスタ内のオブジェクトを構造化するために、Namespaceを使う
Namespaceはオブジェクトの集まりをいれるフォルダ

kubectlコマンドはデフォルトでは`default`というNamespaceとやり取りする

`mystuff`というNamespace内のオブジェクトを参照する場合は、`kubectl --namespace=mystuff`

### 4.2　Context

Context: デフォルトのNamespaceを恒久的に変えたい場合に使用する

`kubectl config set-context my-context --namespace=mystuff`

この設定はコンテキストを作るだけで使用はされないため、使用する場合は以下を実行

`kubectl config use-context my-context`

### 4.3　Kubernetes APIオブジェクトの参照

KubenetesオブジェクトはすべてRESTfulリソースで表すことができる
例）<https://your-k8s.com/api/v1/namespaces/default/pods/my-pod>

特定のリソース情報がほしいときは`kubectl get <リソース名> <オブジェクト名>`

特定のオブジェクトの詳細が知りたい場合は`kubectl describe <リソース名> <オブジェクト名>`

### 4.4　Kubernetesオブジェクトの作成、更新、削除

Kubenetes APIのオブジェクトは、JSONあるいはYAMLファイルとしても表現可能

このファイルを使ってKubenetesサーバ上のオブジェクトの作成、更新、削除が可能

`kubectl apply -f obj.yml`: obj.ymlに定義されたオブジェクトを作成できる

オブジェクトに変更を加えたときも、同じコマンドで更新可能

対話的に設定を編集したいときは`kubectl edit <リソース名> <オブジェクト名>`

オブジェクトを削除する際は`kubectl delete -f obj.yml`

リソースの型とオブジェクト名を指定しても削除可能`kubectl delete <リソース名> <オブジェクト名>`

### 4.5　オブジェクトのLabelとAnnotation

`label`コマンドや`annotate`コマンドを使うとkubenetesオブジェクトのlabelやAnnotation情報を更新可能

`kubectl label pods bar color=red` : barという名前のpodにcolor=redというラベル付与

ラベルを削除する場合は`kubectl label pods bar color-`

### 4.6　デバッグ用コマンド

`kubectl logs <Pod名>`

Pod内にコンテナが複数ある場合は、`-c`を使うとコンテナを指定可能

継続的にターミナル煮出したい場合は`-f`を使う

`kubectl exec -it <Pod名> -- bash`: 実行中のPodでコマンド実行

cpコマンドでファイルコピーも可能
`kubectl cp <Pod名>:/path/to/remote/file /path/to/local/file`

このあたりはdokcerコマンドとほぼ一緒

### 4.7　まとめ

ヘルプを見ることも可能

`kubectl help`

`kubectl help コマンド名`

## 5章　Pod
### 5.1　KubernetesにおけるPod
### 5.2　Pod単位で考える
### 5.3　Podマニフェスト
#### 5.3.1　Podの作成
#### 5.3.2　Podマニフェストの作成
### 5.4　Podを動かす
#### 5.4.1　Podの一覧表示
#### 5.4.2　Podの詳細情報
#### 5.4.3　Podの削除
### 5.5　Podへのアクセス
#### 5.5.1　ポートフォワードの使用
#### 5.5.2　ログからの詳細情報の取得
#### 5.5.3　execを使用したコンテナ内でコマンド実行
#### 5.5.4　コンテナとローカル間でのファイル転送
### 5.6　ヘルスチェック
#### 5.6.1　Liveness probe
#### 5.6.2　Readiness probe
#### 5.6.3　ヘルスチェックの種類
### 5.7　リソース管理
#### 5.7.1　リソース要求 : 必要最低限のリソース
#### 5.7.2　limitsを使ったリソース使用量の制限
### 5.8　Volumeを使ったデータの永続化
#### 5.8.1　VolumeとPodの組み合わせ
#### 5.8.2　VolumeとPodを組み合わせる別の方法
#### 5.8.3　リモートディスクを使った永続化データ
### 5.9　すべてまとめて実行する
## 5.10　まとめ
## 6章　LabelとAnnotation
### 6.1　Label
#### 6.1.1　Labelの適用
#### 6.1.2　Labelの変更
#### 6.1.3　Labelセレクタ
#### 6.1.4　APIオブジェクト内のLabelセレクタ
### 6.2　Annotation
#### 6.2.1　Annotationの定義
### 6.3　後片付け
### 6.4　まとめ
## 7章　サービスディスカバリ
### 7.1　サービスディスカバリとは
### 7.2　Serviceオブジェクト
#### 7.2.1　Service DNS
#### 7.2.2　Readiness probe
### 7.3　クラスタの外に目を向ける
### 7.4　クラウドとの統合
### 7.5　より高度な詳細
#### 7.5.1　Endpoints
#### 7.5.2　手動でのサービスディスカバリ
#### 7.5.3　kube-proxyとクラスタIP
#### 7.5.4　クラスタIP関連の環境変数
### 7.6　後片付け
### 7.7　まとめ
## 8章　ReplicaSet
### 8.1　調整ループ
### 8.2　PodとReplicaSetの関連付け
#### 8.2.1　既存のコンテナを養子に入れる
#### 8.2.2　コンテナの検疫
### 8.3　ReplicaSetを使ったデザイン
### 8.4　ReplicaSetの定義
#### 8.4.1　Podテンプレート
#### 8.4.2　Label
### 8.5　ReplicaSetの作成
### 8.6　ReplicaSetの調査
#### 8.6.1　PodからのReplicaSetの特定
#### 8.6.2　ReplicaSetに対応するPodの集合の特定
### 8.7　ReplicaSetのスケール
#### 8.7.1　kubectl scaleを使った命令的スケール
#### 8.7.2　kubectl applyを使った宣言的スケール
#### 8.7.3　ReplicaSetのオートスケール
### 8.8　ReplicaSetの削除
### 8.9　まとめ
## 9章　DaemonSet
### 9.1　DaemonSetスケジューラ
### 9.2　DaemonSetの作成
### 9.3　特定ノードに対するDaemonSetの割り当ての制限
#### 9.3.1　ノードへのLabelの追加
#### 9.3.2　ノードセレクタ
### 9.4　DaemonSetの更新
#### 9.4.1　個別のPodの削除によるDaemonSetの更新
#### 9.4.2　DaemonSetのローリングアップデート
### 9.5　DaemonSetの削除
### 9.6　まとめ
## 10章　Job
### 10.1　Jobオブジェクト
### 10.2　Jobのパターン
#### 10.2.1　1回限り
#### 10.2.2　一定数成功するまで並列実行
#### 10.2.3　並列実行キュー
### 10.3　まとめ
## 11章　ConfigMapとSecret
### 11.1　ConfigMap
#### 11.1.1　ConfigMapの作成
#### 11.1.2　ConfigMapの使用
### 11.2　Secret
#### 11.2.1　Secretの作成
#### 11.2.2　Secretの使用
#### 11.2.3　プライベートDockerレジストリ
### 11.3　命名規則
### 11.4　ConfigMapとSecretの管理
#### 11.4.1　一覧表示
#### 11.4.2　作成
#### 11.4.3　更新
### 11.5　まとめ
## 12章　Deployment
### 12.1　最初のDeployment
#### 12.1.1　Deploymentの仕組み
### 12.2　Deploymentの作成
### 12.3　Deploymentの管理
### 12.4　Deploymentの更新
#### 12.4.1　Deploymentのスケール
#### 12.4.2　コンテナイメージの更新
#### 12.4.3　ロールアウト履歴
### 12.5　Deployment戦略
#### 12.5.1　Recreate戦略
#### 12.5.2　RollingUpdate戦略
#### 12.5.3　サービスの正常性を確保するゆっくりしたロールアウト
### 12.6　Deploymentの削除
### 12.7　まとめ
## 13章　ストレージソリューションとKubernetesの統合
### 13.1　外部サービスのインポート
#### 13.1.1　セレクタのないService
#### 13.1.2　外部サービスの制限 : ヘルスチェック
### 13.2　信頼性のある単一Podの実行
#### 13.2.1　MySQLの単一Podでの実行
#### 13.2.2　動的ボリューム割り当て
### 13.3　StatefulSetを使ったKubernetesネイティブなストレージ
#### 13.3.1　StatefulSetの特徴
#### 13.3.2　StatefulSetを使ったMongoDBの手動レプリケーション設定
#### 13.3.3　MongoDBクラスタ構築の自動化
#### 13.3.4　PersistentVolumeとStatefulSet
#### 13.3.5　最後のポイント : Readiness probe
### 13.4　まとめ
## 14章　実用的なアプリケーションのデプロイ
### 14.1　Parse
#### 14.1.1　前提条件
#### 14.1.2　parse-serverの構築
#### 14.1.3　parse-serverのデプロイ
#### 14.1.4　Parseのテスト
### 14.2　Ghost
#### 14.2.1　Ghostの設定
### 14.3　Redis
#### 14.3.1　Redisの設定
#### 14.3.2　RedisのServiceの作成
#### 14.3.3　Redisのデプロイ
#### 14.3.4　Redis Clusterを触ってみる
### 14.4　まとめ
## 付録A　Raspberry Piを使ったKubernetesクラスタ構築
## A.1　パーツ一覧
## A.2　イメージの書き込み
## A.3　マスタの起動
## A.3.1　ネットワークのセットアップ
## A.3.2　Kubernetesのインストール
## A.3.3　クラスタのセットアップ
## A.4　まとめ
## 訳者あとがき
## 著者紹介
## 奥付
## 