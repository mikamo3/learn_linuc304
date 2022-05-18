# 1-1 仮想化の概念と理論
## 仮想化とは？
## 仮想化のメリットとデメリット

### メリット

物理的スペースの削減
物理的サーバの運用コストの削減

### デメリット

性能劣化

障害原因の特定の難化

学習コスト

特有のセキュリティ問題

アプリの動作検証

## 仮想化方式の種類
### ホストOS型

物理亭なハードウェア上のOSにインストールしてその上で動かす

* VMware
* VirtualPC
* VirtualBox

### ハイパーバイザー型

ホストOS不要

ハイパーバイザーというソフトを物理ハードウェア上で動かしてその上で仮想マシンを動かす

ホストOS型よりパフォーマンスが高い

* VMwreESXi
* Xen
* Hyper-V

### コンテナ型

ホストOS型の一種として分類

コンテナにはゲストOSや仮想ハードウェアは含まれない

ホストOSにインストールされたLinuxカーネルが同じであれば別のディストリビューションを動かせる

ホストと異なるOSは利用できない

高パフォーマンス、低リソース、構成管理が容易

* OpenVZ
* LXC
* Docker

### 仮想マシンの移行(マイグレーション)

P2V 物理サーバから仮想サーバへ移行
V2V 仮想サーバから仮想サーバ

### 準仮想化と完全仮想化

ハイパーバイザー型において

準仮想化 ゲストOSに修正が必要、ゲストOSのシステムコールをハイパーバイザーが処理する（ハイパーバイザーコール）。ハイパーバイザーコールに置き換えるための修正

nonプロプライエタリなOS向け

デバイスドライバはゲストOSにインストールされる

完全仮想化 ゲストOSの修正不要、プロプライエタリOS向け。パフォーマンスは落ちる。仮想ハードウェアによるハードウェアエミュレーションが行われる

### CPUモードと特権モード

CPUモード 特権モードとユーザーモードに分類,デバイスドライバやOS

特権モード　CPUが備える機能をすべて利用可能,アプリケーション

ユーザモード　制限あり

ゲストOSはアプリケーションのため特権CPU命令を実行できないため解決策として

* バイナリトランスレーション
* CPU仮想化支援機能

がある。

### CPUの仮想化支援機能とCPUフラグ

CPUの仮想化支援機能

ハードウェアレベルの仮想化支援技術

* Intel VT
* AMD-V

CPUフラグ

CPUのサポートしている機能を表すフラグ

```
/proc/cpuinfo
```
に

vmx (intel)

svm(amd)

があれば仮想化支援機能が使える

### クラウドコンピューティング

インターネットを通じて物理リソースやソフトを含めたコンピューティング全体を利用、提供するアーキテクチャ

### クラウドコンピューティングのサービスモデル

* SaaS
* PaaS
* IaaS

# 1-2 Xen

## Xenとは？

ハイパバイザー型

## Xenの基本的なアーキテクチャ

ドメイン 仮想OSの実行単位

Domain-0 (dom0) 管理用OSを実行するドメイン
Domain-U (domU) ゲストOSを実行するドメイン

ゲストOSからのハイパーバイザーコールはXenがうけとり、Dom0の仮想デバイスとデバイスドライバを介して物理ハードを制御する

Xenハイパーバイザーにはデバイスドライバは含まれない

domUではストレージドライバが不要、アクセス可能なのはdom0がマウントしたファイルシステムのイメージファイル、iSCSI,NBD

ゲストは準仮想化、完全仮想化をサポート

完全仮想化ゲストをHVMドメインと呼ぶ

## Xenの設定

domUの設定ファイルは

```
/etc/xen/ドメイン名
```

完全仮想化度名動かす場合

```
builder='hvm'
```

と設定する

自動起動は

```
/etc/xen/auto
```

にシンボリックリンクをはってxendomainsサービスを自動起動するようにする

メモリ容量はブートローダの設定に

```
dom0_mem=512M
```

などとする

domUのネットワークには仮想NICが作成される。dom0の仮想NICに紐付けされる。

ブリッジ、ルーティング、NATのモードがサポートされる

ブリッジ　ドメインを物理ホストと同じ物理ネットワークに接続したい場合に使う

閉じたネットワークはNAT,ルーティングモードを使う

## Xenのユーティリティとツールスタック

LibXenlightが共通ライブラリ

XL,XAPI,XE,Libvirt,virshがフロントエンドツール

xentop,xenstore-lsなど

完全、準どちらも同じツールを使う

XENDは削除

### xlコマンド

Xen管理のデフォルトのツール,4.1以降,3のxmコマンドの拡張

主なコマンド

* xl shutdown
* xl destroy
* xl block-list あタッチされているブロックデバイスの表示
* xl create -c domain1 設定ファイルを指定してコンソールを現在のコマンドラインで開く
* xl list

基本　xxx-動詞

xl listのstateの意味

* b blocked
* c crashed
* d dying
* p paused
* r running
* s shutdown

設定ファイルの位置は

```
/etc/xen/xl.comf
```

### XAPI/XE

デフォルトの管理ツールとインターフェースを提供

通常のツールスタックに追加して色々機能追加

XAPIはxeコマンドを使用して操作

### xentop

リソース情報をリアルタイム表示

### xenstore

Xenドメインに関する実行時情報を階層的に格納するデータベース

dom0のxenstoredによって管理

情報はXenのドメイン間で共有される

情報はxenstore-lsで取得できる

## KVM

Kernal-based Virtual Machine

完全仮想化、windowsなども動かせる、CPU仮想化支援を使うこと前提

ハードウェアデバイスエミュレータが必要、QEMUをつかう

1仮想マシン1QEMUプロセスが対応
```
/dev/kvm
```

というKVデバイスファイルを介してやり取りする

KVMはカーネルの一部のためカーネルのデバイスドライバをそのまま使える

SMPサポート

設定ファイルは

```
/etc/kvm
```

ディレクトリに配置

### KVMのセットアップ

kvm.ko カーネルモジュールが必要

### KVMのユーティリティ

基本はlibvirtが提供するvirshを使う

### qemu-img

仮想ディスクの作成や変換機能を提供

### KVMのスナップショット

スナップショットを表示するには

```
qemu-img snapshot -l イメージファイル名 
```

スナップショットはqcow形式で保存される

qcow2は複数の仮想マシンスナップショットをサポート

### QEMUモニタ

QEMUが提供する仮想コンソール

スナップショットから復元するにはコンソールで

```
loadvm
```

### qemu-kvmコマンド

KVMを管理、操作

起動パラメータの指定は -bootオプションを使用

-boot order=nが指定されると関連付けされたすべてのネットワークインターフェースが検索されて起動するOSが決まる

新しいドライブを割り当てるには-driveオプションで

QEMUではダイレクトLinuxブーティング（ブートローダを迂回してLinuxと初期RAMをロード）ができる。

-appendオプションでrootファイルシステムの物理パーティションを指定して起動

### KVMのネットワーキング

デフォルトホストにvirbr0という仮想ブリッジとTAPデバイスvnet~が作成される

複数のゲストが物理NICを共有して外部ネットワークに接続可能

### brctlコマンド

ブリッジを管理

サブコマンドは

* addbr
* addif
* show

### tunctlコマンド

RUN/TAPデバイスを管理

こっちはbrctlと違ってサブコマンドでなくオプションでオペレーションする

# その他の仮想化ソリューション

## OpenVZ

コンテナ型仮想化

OSテンプレートはEZテンプレートという

```
vzpkg install template
```

でテンプレートをインストール

## OpenVZのユーティリティ

vzctl

サブコマンドのexec,exec2で任意のコマンドを実行


## LXC

コンテナ型

Linuxのカーネル機能を使用して実現している

高性能

テンプレートを使用する

```
/usr/share/lxc/templates
```

に配置

### lxcのユーティリティ

* lxc-console
* lxc-create
* lxc-destroy
* lxc-start
* lxc-stop

## その他の仮想化テクノロジ

### Virtualbox

Oracle

マルチプラットフォーム

CPUの仮想化支援機能対応

パフォーマンス向上のためのデバイスドライバを提供

VBoxManage(CUI),VirtualBoxManager(GUI)

### Vagrant

仮想マシン作成、管理ツール

同一環境をかんたんに構築できる

プロビジョング機能

Vagrantfileに環境構築用の設定を記載する

### Packer

Vanrantで作成した仮想マシンイメージのパッケージやクラウド環境に対応したイメージを作成できる

### Docker

コンテナ型

# Libvirtおよび関連ツール

## libvirtとは？

仮想環境を操作、管理するための共通のインタフェース

特定の環境に依存しない

## libvirtの基本アーキテクチャ

libvirtdというデーモンが仮想環境を操作する

設定ファイルは

```
/etc/libvirt/libvirtd.conf
```

### libvirtを使用した仮想マシンの作成

ツール群は以下

* virt-clone
* virt-image
* virt-install
* virt-manager
* virt-viewer

### libvirtのネットワーキング

起動時にデフォルトでdefaultとうNATブリッジが作成される

### libvirtのストレージ管理

Volume,Poolのいうコアコンセプトに基づいて設計されている

ボリューム　ゲストに割り当てられる単一のストレージボリューム
プール　ボリュームを保存する際に使用可能なホスト上のストレージ

## virshによるlibvirtの操作

virshはlibvirtが提供する標準のコマンドラインツール

```
virsh サブコマンド 引数
```

### コマンド例

ゲストの一覧表示

```
virsh list --all
```

ゲストの起動

```
virsh start domain1
```

コンソール接続

```
virsh console domain1
```

ライブマイグレーション

```
virsh migrate --live guest1 qemu+ssh://ds/sys
```

仮想ネットワークの一覧表示

```
virsh net-list
```

デフォルトネットワークの起動

```
virsh net-start default
```

自動起動設定

```
virsh net-autostart default
```

設定変更

```
virsh net-edit mynetwork
```

仮想CPU情報

```
virsh vcpuinfo domain1
```

CPUアフィニティ(物理CPUと仮想CPUの割当)の設定


仮想CPU1と物理CPU4の関連付け
```
virsh vcpupin domain 1 4
```

イメージファイルの一覧

```
virsh vol-list pool1
```

仮想マシンの復元

```
virsh restore vmfile
```

## oVirt

KVMのハイパーバイザー上に構築された管理ツール

webインターフェース
