# 仮想化の概念と理論


## 仮想化支援機能について

intelでは vmx

amdでは svm

## エミュレーションについて

準仮想化においてハードウェアエミュレーションは不要

# Xen

## 準仮想化について

デバイスドライバは準仮想化に対応したものを使用する


## xlコマンドのサブコマンド

コマンド体系は省略なし。-でつなぐ

起動はcreate

終了はshutdown,haltではない

強制終了はdestroy

vcpu-xxxxはsはいらない

起動時メッセージはdmesg

マイグレーションはmigrate,オプションは不要

* console
* dmesg
* info
* list
* migrate
* restore
* save
* top
* create
* destroy
* pause
* reboot
* shutdown
* unpause
* vcpu-list
* vcpu-set
* block-attach
* block-detach
* block-list
* cd-eject
* cd-insert

マシン起動はcreate,openなどはない。-cで起動後にコンソール接続

## xl.cfg

主な設定項目

kernelでカーネルイメージを指定

vifでネットワークの設定ifではない

vcpusでCPU数、xlのコマンドvcpu-setと混同しないように

builderオプションを指定しないと準仮想化になる
## 仮想化に対応しているかの確認方法

xl dmesgにおいてHVMという単語が確認できるのでそれで確認(intel,amdともに)

## xenのツールスタック、daemon,CLUコマンド

XAPI,xxapi,xe

xl,xenstoredとxenconsoled,xl

## XAPIとは

Xen管理ツールスタック

# KVM

完全仮想化、QEMUによるハードウェアエミュレーション

メモリ管理はホスト側を使う

## 必要なカーネルモジュール

* kvm_intel
* kvm_amd

CPU_Flagと混同しないように

## qemu-kvmのオプション

デフォルトでは、-net tap,-net userが指定される

-のみ

メモリの指定は-m -memoryではない

カーネルイメージの指定は-kernel

コマンドラインパラメータの指定は-append

初期RAMディスクの指定は-initrd

CDROMを入れて起動する場合は-cdrom

-net nicでVLANに接続
## qemu-imgのオプション

* create
* convert
* resize
* info
* snapshot
### デフォルト値

-net nic

-net user

-boot order=c

## 追加のコマンドラインユーティリティ

Debian系 kvm
RedHat系 qemy-kvm

## QEMUモニタコマンド

info サブコマンドで情報取得

info cpus sをつける

change メディア入れ替え xlと違ってcd-はつけない

スナップショット関係のコマンド

* savevm
* loadvm
* delvm

# その他の仮想化ソリューション

## LXCの非特権、特権コンテナ

特権コンテナはコンテナrootプロセスはホストでもroot

セキュリティを担保するには強制アクセス制御、ケーパビリティの削除を行う

## vzctlコマンドの主なサブコマンド

* suspend 一時停止
* set 各種設定
* enter コンテナに入る
* console コンテナのコンソールに接続
* status コンテナの情報表示
* destroy | delete コンテナ破棄

情報表示はstatus,infoなどではない

コンテナ削除にrmはない
## vagrantコマンドの主なサブコマンド

* box イメージ管理
* destroy 仮想マシン停止、関連リソース削除
* suspend 仮想マシンの状態保存

## LXCの主なコマンド

* lxc-destroy コンテナ削除
* lxc-start 起動
* lxc-info 情報表示
* lxc-ls 一覧

情報表示はinfo,vzctlはstatusなので混同しないようにする

# Libvirtと管理ツール

## dominfoで得られる情報

* 使用メモリ
* 割当メモリ
* 割当CPU数
* CPU時間

空きメモリ、CPU使用率は表示されない

## virshの主なサブコマンド

* destroy 強制終了
* net-destroy ネットワーク強制停止
* change-media 仮想CD変更
* suspend 一時停止
* resume 再開
* dominfo ドメイン情報を表示
* setmem メモリサイズ指定
* vcpuinfo cp情報を表示
* vcpupin 仮想CPUアフィニティの制御
* setvcpus 仮想マシンのCPU数を設定

仮想CPU関連はvcpu*
suspend,resumeはxlのpause,unpauseと混同しないこと
## libvirtツール

* virt-viewer ゲストOSGUI表示

viewではない

## virt-managerでの仮想マシン作成時の注意

メモリ、CPUは仮想マシン作成時に決定する必要がある

# クラウド管理ツール

* CloudStack マネジメントサーバと管理されるリソースで構成される
* OpenStack NASAが開発。ストレージサービス AWSEC2,S3互換性を念頭に開発されている
* Eucalyptus カリフォルニア大学開発、AWS同等環境
* OpenNebula 仮想データセンター向け

## CloudStackのコンポーネント

* マネージドサーバ
* データベースサーバ 状態保存用
* コンピューティングノード
  * Region
  * Zone
  * Pod
  * Cluster
* プライマリストレージ
* セカンダリストレージ テンプレート、イメージ、スナップショット保存先

## OpenStackのコンポーネント

* Nova 仮想マシン管理
* Cinder ブロックストレージ管理
* Swift オブジェクトストレージ管理
* Neutron 仮想ネットワーク管理
* Glance 仮想マシンイメージ管理
* Keystone 認証基盤
* Horizon WebGUI