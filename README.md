# k8s-practice

Vagrantとkubeadmを使った（主にCKAの）勉強用のKubernetesクラスター構築セット。

CKAの勉強用なので、なるべくプレーンな感じの設定になっている。
KrewとかHelmとか便利ツール類は自分で入れること。

ただし、毎回設定するのがだるいので、`k`エイリアスとcompletionの設定だけは入れてある。
試験時に自分で設定する方法がわからなくても、[kubectlチートシート](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/)を見ればよい。

## 出来上がる環境

- Kubernetes: v1.22.5
- control plane: 1 node
    - `192.168.56.161`
    - taintは解除していない
- worker: 2 nodes
    - `192.168.56.171`
    - `192.168.56.172`
    - `Vagrantfile`の`NUM_OF_WORKERS`でノード数を変更可能
- 各ノードの`/etc/hosts`は何も設定追加していない
  - `worker1`とかで参照できないので、IPアドレスで参照するか自分で追加する
- network: Calico
- service-cidr: `10.96.0.0/12`
- pod-network-cidr: `10.224.0.0/16`
- NFS
  - control-planeの`/var/nfs`がエクスポートされる
  - worker側は`/etc/fstab`で`/mnt/nfs`にマウントする設定を書いてあるが、`noauto`で自動マウントしないようにしてある
    - マウント不要だし、トラブルの元なので、必要があるときだけ手動マウントする
- ユーティリティ
  - `kubectl`設定済み
  - `kubectl`のcompletion設定済み
  - `alias k=kubectl`設定済み（completionも設定済み）
  - `jq`, `yq`インストール済み
  - `./shared/kube_config`にkubectl用の設定ファイルが出力される

IPアドレスは決め打ちで設定済み。変更したい場合は`Vagrantfile`と`ansible/*`の該当箇所を探して変更する。

## 必要なもの

- CPU: 4コアぐらい
- メモリ: 8GBぐらい
- VirtualBox
- Vagrant
- Ansible
  - VagrantのAnsible Provisionerでセットアップする
  - あまり丁寧に定義していないので、再実行しても綺麗にならないかもしれない

## 使い方

環境の起動:

```bash
vagrant up
```

VMへのssh:

```bash
vagrant ssh control-plane
vagrant ssh worker1
vagrant ssh worker2
```

### VMを終了する時

`vagrant halt`すると`control-plane`がなかなか終了しなくて、forcing shutdownしてしまう場合がある。

おそらく、`worker`が落ちて、追いやられたPodを`control-plane`が引き受けて、落ちるに落ちられなくなってるのだと思われる。

気にせず終了してもいいが、環境を再開する予定があるなら`vagrant suspend`で休止状態にするほうが良いかもしれない。

## Tips

### テキストエディタの設定

デフォルト環境だとコマンドによって`nano`が立ち上がったり`vim`が立ち上がったり様々なので、`EDITOR`環境変数で設定しておくとよい。

```bash
# vimにする
export EDITOR=vim

# nanoにする
export EDITOR=nano
```

また、`KUBE_EDITOR`環境変数に設定すれば、`kubectl edit`の時のエディタだけ変えることも出来る（設定されていなければ`EDITOR`が参照される）。

### Vimの設定

デフォルト状態だとマニフェストのYAMLを編集するのに向いてないので、以下のような設定を作ると多少マシになる。
（試験環境でもソラで作れそうなミニマルな設定内容）

`~/.vimrc`

```vim
source $VIMRUNTIME/defaults.vim
set number
set ts=2 sts=2 sw=2 et
```

あるいは

```vim
filetype plugin indent on
syntax on
set number
set ts=2 sts=2 sw=2 et
```

### tmuxの使い方

デフォルト設定のままのtmuxの使い方。

#### マウスを有効にする

tmuxでマウスを使うには、ターミナルソフトもマウス入力に対応している必要がある。

**注意：試験環境のWebターミナルでマウスが有効化できるかどうかはわからない。**

1. `C-b :`でコマンド入力モードにする
2. vimみたいに画面下でコマンドを入力できるモードになるので`set mouse`を実行する

マウスを有効にすると何ができるのか:

- 画面分割したペインをマウスクリックでフォーカス切り替え
- 画面分割したペインの中でホイールスクロール
- 画面分割の線をドラッグしてペインをリサイズ

#### 最低限バージョン

```
プレフィックスキーはC-b

C-b ?               キーマッピング一覧を表示
                    ただし見てもよくわからないものが多い

ウィンドウ
タブみたいなもの
今自分がどのウィンドウにいるかは、画面下のタブ名に*がついてるかどうかでわかる
C-b c               新規ウィンドウ作成
C-b n               次のウィンドウに切り替え
C-b p               前のウィンドウに切り替え
C-b w               ウィンドウ一覧から選択して切り替え
                    左右キーでツリーを展開してペインの切り替えもできる
C-b "               ウィンドウを水平に分割
C-b %               ウィンドウを垂直に分割

ペイン関係
ウィンドウの中にある個々の領域（画面分割していないときも1つだけのペイン）
C-b o               次のペインに切り替え
C-b ;               直前のペインに切り替え
C-b カーソルキー      ペインの切り替え
```

#### これだけ知ってたら十分バージョン

```
プレフィックスキーはC-b

C-b ?               キーマッピング一覧を表示
                    ただし見てもよくわからないものが多い
C-b d               クライアントのデタッチ
                    tmux attachで再アタッチできる
C-b :               tmuxのコマンド入力モード
                    ここでselect-windowとかコマンド使ってもいい
C-b t               時計を表示

ウィンドウ
タブみたいなもの
今自分がどのウィンドウにいるかは、画面下のタブ名に*がついてるかどうかでわかる
C-b c               新規ウィンドウ作成
C-b n               次のウィンドウに切り替え
C-b p               前のウィンドウに切り替え
C-b 0..9            指定の番号のウィンドウに切り替え
C-b w               ウィンドウ一覧から選択して切り替え
                    左右キーでツリーを展開してペインの切り替えもできる
C-b "               ウィンドウを水平に分割
C-b %               ウィンドウを垂直に分割

ペイン関係
ウィンドウの中にある個々の領域（画面分割していないときも1つだけのペイン）
C-b o               次のペインに切り替え
C-b ;               直前のペインに切り替え
C-b カーソルキー      ペインの切り替え
C-b z               現在のペインを最大化（もう一度実行すると元に戻る）
C-b {               ペインの入れ替え
C-b }               ペインの入れ替え
C-b Space           ペインのレイアウトを順番に切り替える（5種類ぐらいある）
C-b Alt-1           均等な水平分割レイアウトに切り替え
C-b Alt-2           均等な垂直分割レイアウトに切り替え
C-b Alt-3           メインとサブの水平分割レイアウトに切り替え
C-b Alt-4           メインとサブの垂直分割レイアウトに切り替え
C-b Alt-5           タイル型のレイアウトに切り替え
C-b C-カーソルキー    ペインのリサイズ
C-b Alt-カーソルキー  ペインのリサイズ（5文字幅ずつ）
```

### tmuxで画面分割するとはかどる

1. `C-b "`で水平分割する
2. 上ペインで`watch kubectl get deploy,po,svc -o wide --show-labels` などを実行しっぱなしにする
3. 状況がリアルタイムに把握できる👍

### kubectl小ネタ

#### コマンドのヘルプ

`-h`をつければ大抵のサブコマンドはヘルプが出てくる。

```bash
kubectl get -h
kubeclt create -h
kubeclt create deploy -h
```

#### ヘルプの中のExampleだけさっと見たい

```bash
# Examples:の行から下20行を表示
# 20の部分は適当に変えてもよい
kubectl get -h | grep Examples -A 20
```

#### マニフェストの項目を調べる

`kubectl explain`で調べられる。

```bash
kubectl explain pod
kubectl explain pod.spec.containers
```

#### `--all-namespaces`って打つのだるい

`-A`でもよい。

```bash
kubectl get pods --all-namespaces
kubectl get po -A
```

#### `po`とか`svc`みたいな省略形を知りたい

`kubectl api-resources`で一覧表示できる。

APIのバージョンも表示されるので、微妙にバージョンが違って、`v1beta1`って指定したら`DEPRECATED`と怒られた、などという場合に確認できる。
というか、そういう場合は`kubectl explain`でマニフェストの項目を確認したほうがよい（バージョンも確認できる）。
