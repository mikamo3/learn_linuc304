# 高可用性の概念と理論

## 稼働率

(MTBF)/(MTTR+MTBF)

## ディザスタリカバリ

DR 災害からシステムを復旧すること

RPO (Recovery Point Objective) どの地点まで復旧するか
RTO (Recovery Time Objective) どのくらいの時間で復旧するか

RPOの要件を満たすために遠隔レプリケーション、遠隔へのフェイルオーバーの整備で担保できる

# ロードバランスクラスタ 

keepalived サーバの死活監視

LVS( linux virtual server)ロードバランサ、死活監視にkeepalivedなどをしよう

IPVS(IP Virtual Server) LVSの中核を担うコンポーネント

ipvsadm IPVSコマンドラインユーティリティ

VRRP(Virtual Router Redundncy Protocol)ロードバランサの冗長化しSPOFをなくす

HAProxy ロードバランサー。TCP,HTTP通信の負荷分散を行う。LVSに比べて高機能,ACLに基づいた分散も可能

ldirectord クラスタソフトウェアHeartbeatの一部、死活監視、レイヤ7ロードバランサとして動作する

## LVS
クラスタシステム。IPVS(Ip Virtual Server)というロードバランシング機能を持ったカーネルモジュールを使用して負荷分散する

## IPVS

負荷分散を行う

(IP Virtual Server) LVSの中核を担うコンポーネント

ip_vs.koを読み込む

レイヤー4のみ対応

リアルサーバの監視にはnannyというプロセスを使う

### パケット転送方式

* NAT(マスカレーディング) IPをリアルサーバのIPに変換する方式、リアルサーバ側の設定変更は不要
* ダイレクトルーティング MACアドレスのみリアルサーバのものに変換する
* トンネリング ロードバランサ側のパケットをリアルサーバのIPでカプセル化する

## ipvsadm

IPVSコマンドラインユーティリティ

ip_vs.koをロードする必要あり

### コマンド

ipvsadm サブコマンド パラメータ

サービスは大文字、サーバは小文字

-L -l --list 仮想サービスの一覧表示(3こあるので注意)

-t --tcp-service tcpサービスを指定,serviceのつけ忘れに注意

--start-daemon {master|backup} 同期デーモンの起動
### スケジューリングアルゴリズム

デフォルトはwlc

## keepalibed


リアルサーバの死活監視を行う

LVSでは複雑な死活監視をできないのでそれを補完する

VRRPで冗長化できる
### 設定項目

/etc/keepalived/keepalived.conf

* vrrp_instance 冗長化に対する設定
* virtual_server
  * digest URLコンテンツのMD5ハッシュ
  * lvs_sched スケジューリングアルゴリズムの設定
  * lb_algo スケジューリングアルゴリズムの設定
  * lvs_method パケット転送方式の設定
  * lb_kind パケット転送方式の設定
  * real_server リアルサーバの定義

### 死活チェックモジュール

* HTTP_GET
* SSL_GET
* TCP_CHECK
* SMTP_CHECK
* DNS_CHECK
* MISC_CHECK

## HAProxy

### 設定項目

/etc/haproxy/haproxy.cfg


* global
* proxy
  * defaults
  * frontend
    * default_backend デフォルトのバックエンドサーバ
  * backend
    * server バックエンドサーバの設定
    * balance ロードバランスアルゴリズムの設定
  * listen フロント、バック両方の設定を行える

## ldirectord

### 設定項目

/etc/ha.d/ldirectord.cf

# フェイルオーバークラスタ

* Pacemaker リソース制御のみ。Heartbeatがクラスタ制御を行う。（他の選択肢もあり,Corosync推奨？）
* Corosync Redhat支援。クラスタ制御のみ。OpenAISがリソース制御を行う(他の選択肢もあり)

Corosync,Heartbeat,OpenAIS(前身は)がメッセージ通信レイヤ機能を持つ

## Pacemaker

高可用クラスタソフト,リソース制御機能のみ提供

### コンポーネント

* CRMd(CrusterResourceManagementDaemon) 自ノード、ほかノードとメッセージ送受信を行う
* CIB(Cruster Information Base) クラスタの現在情報をcib.xmlに記録するデータベース
* PEngine(Policy Engine) CIBに基づいてDCが出す命令を計算
* LRMd(Local Resource Management Daemon) 自ノードの管理、リリースエージェント(RA)を直接操作
* STONITHd(ShootTheOtherNodeInTheHead) ノード異常時に電源制御を行う

他のノードとやり取りするのはCRMd

クラスタ構成と現在の情報を記録しているのはCIB

### Pacemakerで扱えるリソース

* primitive 基本的なリソース Active/Standby構成
* clone primitiveをもとに作成 Active/Active構成にできる。
* ms multi-state cloneリソース、かつ親子関係のあるリソース Master/Slave構成
* group 複数のprimitiveをグループ化したリソース

### リソースエージェント

リソースクラス（記述形式の分類）

* LSB Linux起動スクリプト
* OCF (Open Cluster Framework) スクリプト、パラメータ受取可
* Systemd
* Upstart jobsでサービスを提供する
* Service Systemd,Upstart,LSBの組み合わせ
* Fencing
* Nagios Plugins

### cibadnimコマンド

cib.xmlを表示、変更するコマンド

基本サブコマンドは大文字

### CRMdツール

* crm_verify CIBの整合性チェック

### CIBの構造

* cib.xml
  * crm
    * configuration
      * crm_config
      * nodes
      * resources
      * constraints
    * status

### crm-cliコマンド

CRMdを設定、管理するコマンド

crm サブコマンド

* cib
* cluster
* configure
* node
* ra
* resource リソース定義はconfigureで行う
* status

crm,crmshは対話型で起動できる
### crm configureで設定できる制約

crm configure 制約 制約名 スコア値: リソースA リソースB

* location
* order
* colocation

### pcsコマンド

corosyncも対応

pcs サブコマンド

* constraint リソース制約の設定
* property
* resource
* cluster
* config
* status
* stonith

制約を追加するとき

crm configure ...

pcs consraint ...
### pcs propertyの主なプロパティ

* no-quorum-policy quorumを持たないときの動作
* stonith-enabled 故障発生時にノードを隔離するかどうか
* stonith-action
* maintainance-mode

## corosync
### corosyncの設定

/etc/corosync/corosync.conf

### コマンドラインツール

* corosync-cfgtool パラメータ表示と設定
* corosync-cmapctl オブジェクトDBへのアクセスツール
* corosync-quorumtool
* corosync-keygen
* stonith_admin フェンスデバイスに関する情報表示、設定

# エンタープライズLinuxディストリビューションの高可用性

## Red Hat High Availability Add-On

### RHEL 7

* Colosync
* DLM
* Pacemaker
* pcs
* pcsd

Pacemaker,colosyncの組み合わせ、ロック機構はDLM,あとはツールにpcsを使う

### RHEL 6以前

* CMAN
* DLM
* RGManager
* Conga
* ccs Luci
* Ricci

## 追加コンポーネント

* GFS2
* cLVM
* LVS

## Suse Linux

* Corosync,AopenAIS
* Pacemaker
* HAProxy
* SBD(フェンシング)
* cLVM2
* OCFS2,GFS2 (クラスタ向けファイルシステム)
* DRBD
* ReaR
* HAWK (WebUI)