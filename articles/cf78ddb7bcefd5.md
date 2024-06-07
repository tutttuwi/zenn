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

## Docker周辺知識メモ

Dockerの歴史

2008年
Solomon Hykes 言語中立なPaaS提供を掲げた dotCloudが設立

dotCloud→Docker Incに名前を変えた
2014年6月にDocker1.0
安定性と信頼という点で大きな飛躍を遂げた
SpotifyやBaiduなどで利用されることになった

2014年10月 MicrosoftがWindowsをサポートする宣言

2014年12月
DockerConEUでは
Dockerクラスタマネージャ：Docker Swarm
Dockerホストプロビジョニング用CLIツール：Docker Machine

### インストール

- ここからインストールスクリプトが取得できる<https://get.docker.com>
- WEBに記載されたマニュアル通りインストールも可能

- SELinuxがpermissiveモードになっているかどうかチェックする
SELinux (Security-Enhanced Linux) は、システムにアクセスできるユーザーを管理者がより詳細に制御できるようにする Linux® システム 用のセキュリティ・アーキテクチャ

`sestaus`コマンドで確認できる

enforcingになっていたらenforcingモードになっており、ルールが強制される
permissiveモード有効
SELinuxには、disableモードを含め3つのモードがあります。
disableモード：SELinuxを完全に無効にする。
enforcingモード：ポリシーに違反するアクセスに対し、ログに書き出して拒否する。
permissiveモード：ポリシーに違反するアクセスに対し、ログに書き出して許可する。

`sudo setenforce 0`を実行するだけでpermissiveモードにできる

- sudoを使わない実行

ユーザーをdockerグループに追加すれば避けられる









## 1章　Kubernetes入門

- Kubenetesはオープンソースのオーケストレータ
- 当初はGoogleが開発

- コンテナやKubenetesのようなコンテナAPIを使う理由
  - ベロシティ(開発者の効率を強化する機能)
    - イミュータブルなインフラ
    - 宣言的設定
    - 自己回復
  - （ソフトウェアとチームの両方の意味で）スケールすること
    - 分離するメリット
      - LBを用いて、サービス(APIなど)を構成するプログラムをスケールするのが簡単
      - 開発チームがマイクロサービスに集中できるので、開発チームのスケールも簡単／コミュニケーションオーバーヘッドも小さく保てる
    - 抽象化層の提供（Pod/Service/Namespace/Ingressなど）：小さいチームを保てる
    - アプリケーション開発者はSLAがどのように実現されているか気にせずに、コンテナオーケストレーションAPIによって提供されるSLAに依存できる
  - インフラの抽象化
    - OSSストレージソリューションを利用することなどで、特定のマシンに依存せず、ポータブルな移行が可能
    - Serviceでクラウドでも、ローカルでもロードバランサを作れる
  - 効率性
    - 抽象化がもたらす経済的利益
      - アプリケーションはなんの影響も受けずに1台のマシンに同居できる
      - 複数ユーザからのタスクも少数のマシンにきっちり詰め込める

### 1.1　ベロシティ

- ベロシティ(開発者の効率を強化する機能)
  - 可用性を保ちながらもすばやく進化していくのに必要な機能を提供
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
  - ミュータブル：コンテナにログインし、新しいソフトウェアをダウンロードするコマンドを実行し、ソフトウェアをインストールし、サーバプロセスを再起動する。
  - イミュータブル：新しいコンテナイメージを構築し、そのイメージをコンテナレジストリにプッシュし、既存のコンテナを停止し、新しいコンテナを起動する。

#### 1.1.2　宣言的設定

- 命令的設定
  - 一連の命令の実行によって状態が定義される（Aを起動、Bを起動、Cを起動）
- 宣言的設定
  - 望ましい状態を宣言する（レプリカ数は3に等しい）
  - バージョン管理システムに保存された宣言的状態と、望ましい状態とを最終的に一致させるKubenetesの能力によって、変更のロールバックが簡単になる

#### 1.1.3　自己回復するシステム

- 望ましい状態が宣言されたら、その状態に一致するように継続して動く

### 1.2　サービスとチームのスケール

#### 1.2.1　分離

- 分離アーキテクチャ
  - 定義済みのAPIとサービスロードバランサによって他のコンポーネントから分けられています。
  - APIとロードバランサは、システムの各部分を隔離します。
  - APIは、APIを実装するコンポーネント（implementer）と、
  - APIを使用するコンポーネント（consumer）の間のバッファになり、ロードバランサは各サービスを実行しているインスタンス間のバッファになっています。

- 分離するメリット
  - LBを用いて、サービス(APIなど)を構成するプログラムをスケールするのが簡単
  - 開発チームがマイクロサービスに集中できるので、開発チームのスケールも簡単／コミュニケーションオーバーヘッドも小さく保てる

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
- コンテナオーケストレーションAPIが、
  - クラスタオーケストレーションのオペレータの責任(SLAを守る)と、
  - アプリケーションオペレータの責任(アプリケーションを開発する)とを切り離すためのルールとして存在
- アプリケーション開発者はSLAがどのように実現されているか期にせずに、コンテナオーケストレーションAPIによって提供されるSLAに依存できる

### 1.3　インフラの抽象化

- Kubenetesのようなアプリ指向のコンテナAPIへ移行する利点
  - 特定のマシンから開発者を分離

- OSSストレージソリューションを利用することで、ポータブルになる

### 1.4　効率性

- 抽象化がもたらす経済的利益
  - アプリケーションはなんの影響も受けずに1台のマシンに同居できる
  - 複数ユーザからのタスクも少数のマシンにきっちり詰め込める

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

- コンテナ内にパスワードを入れないようにしましょう
- Vaultなどを使ってパスワードを外だしする

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

Kubernetesクラスタを構成する多くのコンポーネントが、Kubernetes自体を使ってデプロイされる

KubernetesにおけるNamespaceとは、リソースを整理する陸生のこと
ファイルシステムにおけるフォルダと考えれば良い

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

複数のコンテナを1つのアトミックな単位であるPodにまとめることができる

WEBサーバとGit同期サーバを１つのコンテナにまとめてしまうと、それぞれに求められるSLAや要件が異なるため不都合が生じる

Git動機サーバにメモリリークのバグが有ると・・？WEBサーバも落ちる

Podは鯨の群れのことで、鯨に関連する語を使うDockerの慣習

### 5.1　KubernetesにおけるPod

Podとは同じ実行環境上で動くアプリケーションコンテナとストレージボリュームの集まり

Kubernetesコンテナ上ではPodが最小のデプロイ単位

### 5.2　Pod単位で考える

PodにWordpressのコンテナとMySQLデータベースのコンテナを共存させるのはよくあるアンチパターン

理由：Wordpressを拡張するタイミングとデータベースを拡張すべきタイミングは異なるため

Podを作成する際に、このコンテナはそれぞれ違うマシン二配置されても正常に動作するかどうかを考えるべき

動作しないならコンテナをまとめる単位としてPodが正しい選択

### 5.3　Podマニフェスト

Podの設定はPodマニフェストに記述

Kubernetes APIサーバはPodマニフェストを受け入れ、その後永続化ストレージ（etcd）に保存

スケジューラは、ノードに割り当てられていないPodを見つけるためにKubernetes APIを使用

未割当のPodを見つけたら、Podマニフェストに書かれたリソースなどの制約を満たしたノードに、そのPodを割り当て

ノードにリソースが十分にあれば、1つのノードに複数の同じPodが割り当てられることもあるが、

同じアプリケーションの複数のPodを同一ノードに割り当てるのは、障害の影響を受ける範囲が一緒になってしまうので、信頼性の点ではよくないためため、

ノードの障害などに対する信頼性の観点から、Kubernetesスケジューラは同じアプリケーションのPodを別々のノードに分散しようとする

Podがノードに割り当てられると、明示的に削除したり割り当て直したりしない限り、そのPodは同じノード上で動き続ける

同じPodを複数動かす場合はReplicaSetを使うことを考える（Podを１つしか動かさない場合もReplicaSetのほうが好ましい）

#### 5.3.1　Podの作成

- Podを作成するには`kubectl run`コマンドを実行するのがシンプル
  - `kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:1`

この方法でPodを作成すると、`Deployment`と`ReplicaSet`オブジェクトを使ってPodが作られる

#### 5.3.2　Podマニフェストの作成


```bash
$ docker run -d --name kuard \
  --publish 8080:8080 \
  gcr.io/kuar-demo/kuard-amd64:1
```

上記と同じ定義のマニフェストが以下

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

### 5.4　Podを動かす

以下のコマンドでマニフェストを適用する

`kubectl apply -f kuard-pod.yaml`

#### 5.4.1　Podの一覧表示

`kubeectl get pods`

#### 5.4.2　Podの詳細情報

`kubectl describe pods kuard`

#### 5.4.3　Podの削除

`kubectl delete pods/kuard`

`kubectl delete -f kuard-pod.yaml`

削除コマンドを実行したとき、直ぐに削除されない

Terminatingという状態に移行した後、30秒の削除の猶予時間が経過すると削除される

コンテナに保存されたデータはすべて削除されてしまうので、永続化したいなら、`PersistentVolume`を使用する必要がある

### 5.5　Podへのアクセス

#### 5.5.1　ポートフォワードの使用

`kubectl port-forward kuard 8080:8080`

#### 5.5.2　ログからの詳細情報の取得

`kubectl logs kuard`
`kubectl logs kuard -f`: 流れるログを連続的に終える

#### 5.5.3　execを使用したコンテナ内でコマンド実行

`kubectl exec kuard date`
`kubectl exec -it kuard ash`

#### 5.5.4　コンテナとローカル間でのファイル転送

```bash
kubectl cp <Pod名>:/captures/capture3.txt ./capture3.txt
kubectl cp $HOME/config.txt <Pod名>:/config.txt
```

### 5.6　ヘルスチェック

Kubernetes上でアプリをコンテナとして動かくときには、プロセスヘルスチェックの機能で常に動いている状態が維持されるが、

シンプルなプロセス監視では十分でないケースもあるため、`liveness(起動しているかどうか)`に対するヘルスチェックが可能

#### 5.6.1　Liveness probe

- kuard-pod-health.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

```bash
kubectl apply -f kuard-pod-health.yaml
kubectl port-forward kuard 8080:8080
```

ブラウザでhttp://localhost:8080を開き、"Liveness Probe"タブをクリック

kuardのこのインスタンスが受信したすべての監視の一覧を確認、ページ内の"fail"リンクをクリックすると、kuardのヘルスチェックが失敗し始める。

しばらく待つと、Kubernetesがコンテナを再起動します。

この時点で、表示がリセットされます。再起動の詳細情報は、kubectl describe pods kuardで確認できます。"Event"セクションは次と同じようになっているはずです。

#### 5.6.2　Readiness probe

Kubernetesは、liveness（起動しているかどうか）とreadiness（応答できるかどうか）を区別しています。

- Liveness probeは、アプリケーションが正常に動作しているかどうかを判断します。
- readinessとは、コンテナがユーザからのリクエストを処理できることを表します。

#### 5.6.3　ヘルスチェックの種類

`tcpSocket`: TCPソケットを開いて接続が成功したら監視成功

`exec`監視も可能：コンテナ内でスクリプトなどを実行して返り値が0なら監視成功

### 5.7　リソース管理

リソース要求：アプリケーションを動かすのに最低限必要なリソースを指定

リソース制限：アプリケーションが使用する可能性のある最大リソース量を指定

#### 5.7.1　リソース要求 : 必要最低限のリソース

Kubernetesでは、コンテナを動かすために必要なリソースをPodが要求する

Kubernetesは、Podが要求したリソースが使用可能なことを保証

要求されることが多いリソースとしてはCPUやメモリがあるが、KubernetesはGPUなど他のリソースタイプもサポートしています。

例としてkuardコンテナは、マシン上の1 CPUの半分が空いていて128MBのメモリを割り当てられるマシンに配置される必要があるとしましょう。その場合、Podの定義は例5-3のようになります。

- kuard-pod-resreq.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

- リソースの要求はPodごとではなくコンテナ毎に行う
- Podによって要求される総リソース量は、Pod内の全コンテナが要求するリソースの総和になる

#### 5.7.2　limitsを使ったリソース使用量の制限

- kuard-pod-reslim.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

### 5.8　Volumeを使ったデータの永続化

#### 5.8.1　VolumeとPodの組み合わせ

- kuard-pod-vol.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes: # Podマニフェスト内のコンテナからアクセスされる可能性のあるすべてのVolumeの一覧の配列
    - name: "kuard-data"
      hostPath:
        path: "/var/lib/kuard"
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      volumeMounts: #特定のコンテナどのパスにボリュームをマウントするかを指定
        - mountPath: "/data"
          name: "kuard-data"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP

```

#### 5.8.2　VolumeとPodを組み合わせる別の方法

- コミュニケーション、同期
  - emptyDir
    - このVolumeはPodが停止されるまでしか使えないが、
    - 2台のコンテナで共有でき、Gitの同期コンテナとWebサーバコンテナの間のコミュニケーション基盤になる
- キャッシュ
  - キャッシュ用のボリュームもemptyDirが有効
- 永続化データ
  - 永続化データを保存するストレージも利用可能
  - KubernetesではNFSやiSCSIのようなプロトコル、AWS EBS, Azure Files and Disk Storage, GoogleのPersistentDiskなどもサポート
- ホストのファイルシステムのマウント
  - hostPath
    - システム上のデバイスに対してブロックレベルでアクセスするため、`/dev`ファイルシステムにアクセスしたいケースなどで利用

#### 5.8.3　リモートディスクを使った永続化データ

NFSでネットワークのボリュームをマウントできる例

```yaml
...
# Pod定義はここでは省略
volumes:
  - name: "kuard-data"
    nfs:
      server: my.nfs.server.local
      path: "/exports"
```

### 5.9　すべてまとめて実行する

- kuard-pod-full.yaml

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
    - name: "kuard-data"
      nfs:
        server: my.nfs.server.local
        path: "/exports"
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      volumeMounts:
        - mountPath: "/data"
          name: "kuard-data"
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 30
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3

```

## 5.10　まとめ

- PodマニフェストをAPIサーバに送信
- KubernetesスケジューラはPodが動作できるマシンを探し、マシンにPodを割り当て
- 割当が済むとマシン上の`kubelet`デーモンがPodに対応したコンテナを作成しPodマニフェストに定義されたヘルスチェックを実行

## 6章　LabelとAnnotation

- Label：PodやReplicaSetといったKubernetesのオブジェクトに付与できるキーと値のペア
  - 任意で付けることができ、Kubernetesのオブジェクトを特定するための情報を付与できる
  - Labelはオブジェクトをグループ化するための基盤となる機能
- Annotation：Labelと似たストレージの仕組み
  - キーと値のペアだが、ツールやライブラリを便利に使用するために必要になる、オブジェクトを特定しない情報を入れられる

### 6.1　Label

- オブジェクトのメタデータを特定するためのもの
- オブジェクトをグループ化したり、一覧表示したり、操作したりする際の基本的機能

#### 6.1.1　Labelの適用

#### 6.1.2　Labelの変更

`kubectl label deployments alpaca-test "canary=true"`

#### 6.1.3　Labelセレクタ

`kubectl get pods --show-labels`: ラベル一覧を表示

`kubectl get pods --selector="app=bandicoot,ver=2"`: セレクタで絞り込み表示

#### 6.1.4　APIオブジェクト内のLabelセレクタ

### 6.2　Annotation

Labelとほぼ同じ用途で利用可能

迷ったらAnnotationを使って、Selectorとして検索したくなったらLabelに昇格させるのがいい

- Annotationの用途
  - オブジェクトの変更理由の記録
  - 特別なスケジューラへの特別なスケジューリングポリシーの伝達
  - リソースを更新したツールと、どのように更新したかの情報の付加（他のツールから更新を検知したり、うまくマージしたりするため）
  - Labelでは表現しづらいビルド、リリース、イメージ情報の保存（Gitのハッシュやタイムスタンプ、プルリクエスト番号など）
  - Deploymentオブジェクト（12章）によるロールアウトのための、ReplicaSetの追跡情報の保存
  - UIの見た目の品質や使いやすさを高める情報の保存。例えば、オブジェクトに対応するアイコンへのリンクの保存など（base64エンコードされた画像自体を入れることも可能）
  - Kubernetesでのプロトタイプ機能の提供（専用のAPIフィールドを作る代わりに、その機能のパラメータを入れるのにAnnotationを使う）

#### 6.2.1　Annotationの定義

- Annotationは、次の例のようにKubernetesオブジェクトのmetadataセクション内で定義

```yaml
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

### 6.3　後片付け

`kubectl delete deployments --all`

### 6.4　まとめ

## 7章　サービスディスカバリ

### 7.1　サービスディスカバリとは

ディスカバリサービス：何かを見つけるという問題とその解決策を一般的にこう呼ぶ

DNSはインターネット上の伝統的なサービスディスカバリシステムだが、Kubernetesのダイナミックな正解には不十分

### 7.2　Serviceオブジェクト

Kubernetesにおける本当のサービスディスカバリは`Sevice`オブジェクトから始まる

`Service`オブジェクトは名前のついたLabelセレクタを作る仕組み

`kubectl expose`コマンドでServiceを作成可能

- 作成例）

```bash

$ kubectl run alpaca-prod \
  --image=gcr.io/kuar-demo/kuard-amd64:1 \
  --replicas=3 \
  --port=8080 \
  --labels="ver=1,app=alpaca,env=prod"
$ kubectl expose deployment alpaca-prod
$ kubectl run bandicoot-prod \
  --image=gcr.io/kuar-demo/kuard-amd64:2 \
  --replicas=2 \
  --port=8080 \
  --labels="ver=2,app=bandicoot,env=prod"
$ kubectl expose deployment bandicoot-prod
$ kubectl get services -o wide
NAME            CLUSTER-IP    ... PORT(S)  ... SELECTOR
alpaca-prod     10.115.245.13 ... 8080/TCP ... app=alpaca,env=prod,ver=1
bandicoot-prod  10.115.242.3  ... 8080/TCP ... app=bandicoot,env=prod,ver=2
kubernetes      10.115.240.1  ... 443/TCP  ... <none>

```

これらのコマンドを実行した後には、3つのServiceが確認できる

kubernetesというServiceは、アプリケーションからKubernetesのAPIに接続し通信するために、自動的に作成されたもの

ServiceはクラスタIPと呼ばれる新しいタイプの仮想IPアドレスを割り当てる

これは、同じセレクタの付けられたすべてのPodにロードバランスするために使用する、特別なIPアドレス

Serviceと通信するために、alpacaの中のPodにポートフォワードの設定をしましょう。ターミナル内で次のコマンドを実行して下さい。コマンドを終了しない限りポートフォワードが有効で、

http://localhost:48858でalpacaにアクセスが可能

```bash
$ ALPACA_POD=$(kubectl get pods -l app=alpaca \
  -o jsonpath='{.items[0].metadata.name}')
$ kubectl port-forward $ALPACA_POD 48858:8080
```

#### 7.2.1　Service DNS

#### 7.2.2　Readiness probe

Podがリクエストを受け付けられる状態かどうか監視する機能であるReadiness probe

### 7.3　クラスタの外に目を向ける

外部からのアクセスを受け入れる際、Serviceを拡張するNodePortと呼ばれる機能を使うのが最もポータブル

- `spec.type`フィールドを`NodePort`に変更するか
- `kubectl expose`コマンドに`--type=NodePort`を指定することでService作成時にも設定可能

### 7.4　クラウドとの統合

クラスタを動かしているクラウドでサポートされていて、

かつクラスタが使用できるよう設定されていれば、LoadBalancerタイプを使用可能

この機能で、クラウド上に新しいロードバランサが作成され、そのロードバランサ配下にクラスタ内のノードが入れられることで、NodePortを使って通信できるようになる

`kubectl edit service alpaca-prod`を実行し、Service alpaca-prodを編集して`spec.type`を`LoadBalancer`に変更する

編集直後に`kubectl get services`を実行すると、

alpaca-prodのEXTERNAL-IP列が<pending>と表示される

しばらく待つと、クラウドがパブリックアドレスを割り当てられる。

### 7.5　より高度な詳細

#### 7.5.1　Endpoints

- （略）

#### 7.5.2　手動でのサービスディスカバリ

`kubectl`を使用することで、各PodにどのIPアドレスが割り当てられているかを確認できる

手動では限界があるので`Service`オブジェクトができた

#### 7.5.3　kube-proxyとクラスタIP

- （略）

#### 7.5.4　クラスタIP関連の環境変数

### 7.6　後片付け

`kubectl delete services,deployments -l app`

### 7.7　まとめ

## 8章　ReplicaSet

- 冗長性
  - インスタンスを複数動かすと、障害を許容できる
- スケール
  - インスタンスを複数動かすと、多くのリクエストを受け付けられます
- シャーディング
  - 異なるコンテナのレプリカを動かすと、異なる種類の処理を同時に受け付けられる

それぞれ違うPodマニフェストを使ってPodのコピーを手動で作るのも可能だが、退屈だし間違いを起こしやすくなる

Podのレプリカを作るなら、そのレプリカ郡は１つのまとまりとして考えて管理するのが普通→これこそがReplicaSetの考え方

ReplicaSetはクラスタ全体のPodマネージャ

### 8.1　調整ループ

調整ループのベースには、望ましい状態（desired state）の概念がある

望ましい状態とは、こうあって欲しいという状態

ReplicaSetで言えば、レプリカの数や、複製するPodの定義

例）kuardサーバを動かすPodのレプリカが3つ動いているようにしたい

一方で現在の状態（observed stateまたはcurrent state）とは、システムのその時の状態

例）kuardというPodが2つだけ動いている

調整ループは、連続して動き続け、システムの現在の状態を観察し、システムの現在の状態が望ましい状態に一致するようにアクションを起す。

前の例で言えば、現在の状態がレプリカ3つを持つという望ましい状態に一致するように、kuardのPodを1つ作成することに当たる

状態を管理する方法としての調整ループには、たくさんの利点がある。

調整ループは、本質的にゴール駆動型で、自己回復システムであり、しかもほとんどの場合数行のコードで表現可能

この利点の具体例として、ReplicaSetの調整ループは、1つのループでReplicaSetのスケールアップもスケールダウンも、さらにはノード障害やノードの復活も扱える点がある

### 8.2　PodとReplicaSetの関連付け

ReplicaSetとPodの関係も疎結合になっている

ReplicaSetはPodを作成して管理するが、ReplicaSetがPodを所有しているわけではない。

ReplicaSetは、管理すべきPodの集まりをLabelクエリによって識別

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

Job=1回限りの短い時間しか動かさない処理

### 10.1　Jobオブジェクト

Jobオブジェクトは、Jobの設定に書かれたテンプレートで定義されたPodの作成や管理を実施、処理が成功するまで動き続ける。複数のPodを並列に動かすための調整も行う

処理が完了する前に失敗した場合、JobコントローラはJobの設定内のPodテンプレートを元に、新しいPodを作成

Podは必ずどこかのノードに割り当てられる必要があるので、必要なリソースをスケジューラが見つけられない場合、すぐにJobが実行されない可能性もある

分散システムの性質上、障害の発生時には、同じタスクを実行するPodが複数作られることある

### 10.2　Jobのパターン

| パターン                   | 使用例                                              | 動作                                         | completions | parallelism |
| -------------------------- | --------------------------------------------------- | -------------------------------------------- | ----------- |
| 1回限り                    | データベースマイグレーション                        | 1つのPodが処理                               | 1           | 1           |
| 一定数成功するまで並列実行 | 複数のPodでタスクの集まりを並列処理                 | 指定回数成功するまで1つ以上のPodが複数回処理 | 1以上       | 1以上       |
| 並列実行キュー             | 集約されたキューに入れられたタスクを複数のPodで処理 | 1回成功するまで1つ以上のPodが処理            | 1           | 2以上       |

#### 10.2.1　1回限り

```bash
kubectl run -i oneshot \
  --image=gcr.io/kuar-demo/kuard-amd64:1 \
  --restart=OnFailure \
  -- --keygen-enable \
     --keygen-exit-on-complete \
     --keygen-num-to-gen 10
```

- job-oneshot.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: oneshot
  labels:
    chapter: jobs
spec:
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:1
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

- ジョブ作成：`kubectl apply -f job-oneshot.yaml`

- Podの障害
- 複数のPodでエラーが発生
  - restartPolicy: Neverを設定すると、kubeletはJob失敗時にPodを再起動せず、代わりにそのPodを失敗と宣言
  - Jobオブジェクトがこれを検知すると、代わりのPodを作成
  - このため、注意しておかないとクラスタ内にゴミが溜まってしまう
  - Job実行に失敗したPodが再実行されるように、restartPolicy: OnFailureにしておくことを推奨

#### 10.2.2　一定数成功するまで並列実行

completionsとparallelismの両パラメータを組み合わせることで、並列実行する

クラスタを処理能力いっぱいまで使ってしまわないように、同時に5つまでしかPodを動かさないようにする

これを実現するには、completionを10に、parallelismを5二設定

- job-parallel.yaml

```yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: parallel
  labels:
    chapter: jobs
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:1
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

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

- ConfigMap：環境毎の差異情報を管理
- Secret：キー情報、パスワードなどを管理

## 12章　Deployment

`Deployment`オブジェクト：新しいバージョンのリリースを管理する仕組み

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