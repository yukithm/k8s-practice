# k8s-practice

Vagrantとkubeadmを使った勉強用のKubernetesクラスター構築セット。

## 出来上がる環境

- control plane: 1 node
    - `192.168.56.161`
- worker: 2 nodes
    - `192.168.56.171`
    - `192.168.56.172`
- network: Calico
- service-cidr: `10.96.0.0/12`
- pod-network-cidr: `10.224.0.0/16`
- ユーティリティ
  - `kubectl`設定済み
  - `kubectl`のcompletion設定済み
  - `alias k=kubectl`設定済み（completionも設定済み）
  - `jq`, `yq`インストール済み

IPアドレスは決め打ちで設定済み。変更したい場合は`Vagrantfile`と`ansible/*`の該当箇所を探して変更する。

## 必要なもの

- CPU: 4コアぐらい
- メモリ: 8GBぐらい
- VirtualBox
- Vagrant
- Ansible
  - VagrantのAnsible Provisionerでセットアップする

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
