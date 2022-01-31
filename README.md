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

## 試験の情報

試験環境のKubernetesバージョンは定期的に更新されるので、受験要項やカリキュラムのページで確認すること。

- [Certified Kubernetes Administrator \(CKA\) \| Cloud Native Computing Foundation](https://www.cncf.io/certification/cka/)
- [cncf/curriculum: 📚Open Source Curriculum for CNCF Certification Courses](https://github.com/cncf/curriculum)
- [Training and Certification \- T&C DOCS \(Candidate Facing Resources\)](https://docs.linuxfoundation.org/tc-docs/)
  - [Linux Foundation Certification Exam: Candidate Handbook \(using PSI ExamsLocal Proctoring platform\) \- T&C DOCS \(Candidate Facing Resources\)](https://docs.linuxfoundation.org/tc-docs/certification/lf-candidate-handbook)
  - [Important Instructions: CKA and CKAD \- T&C DOCS \(Candidate Facing Resources\)](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad)

試験におけるKubernetesとetcdのバージョンは[Important Instructions: CKA and CKAD \- T&C DOCS \(Candidate Facing Resources\)](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad)で確認できる。

## Tips

→ [Tips](tips.md)
