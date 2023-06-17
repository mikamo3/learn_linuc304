# DRBD / cLVM

## DRBD

Distributed Replicated Block Device 分散複製ブロックデバイス

ファイルシステムより下位レベルで動作

なのでファイルシステムは自由

カーネルはdrbd.ko

/proc/drbd でステータスが見られる

/etc/debd.conf、サブディレクトリなし
TCP/IPを使用

### クラスタ構成

以下の点に配慮する

* レプリケーション動作
* プライマリ、セカンダリの制約事項
* サービス起動タイミング
* 帯域
* ファイルシステム

ブロックデバイスとして利用する場合

* 利用するサービス起動絵にマウントする
* ノードのコントロールをDRBDのロールとともに遷移するようにする

### drbdadm

オプション

* -d --dry-run 標準出力に書き出す(-sではない)
* -m drbdmetaのパス
* -s drbdsetupのパス


## cLVM

LVMの拡張

clvmdがデーモン

共有ストレージ上にクラスタ内のすべてのノードで同時に利用可能な論理ボリュームを作成する

### vgs

ボリューム情報確認

Attr

* w writable
* z resizeable
* x exported
* p partial
* contiguous,cling,normal,anyware 通常はノーマル
* c clustered, s shared

### vgchange

オプション

* -c y|n クラスタ利用の有無
* -l 論理ボリュームの最大数変更
* -p 物理ボリュームの最大数変更
* -s 物理ボリュームのエクステントサイズの変更


# クラスタファイルシステム

クラスタ環境に最適化されたファイルシステム

排他制御、同期管理、冗長性、透過的レプリケーション、耐障害性（ジャーナリング）

## 一覧

* GFS2 RedHatGlobalFileSystem それぞれのノードが個別のジャーナルを持つ
* OCFS2 OracleClusterFileSystem
* GlusterFS FUSE実装
* CephFS RADOSという分散オブジェクトストアがある
* AFS カーネギーメロン大学産、セキュリティ、キャッシュ機能に特徴がある 改良版にCoda,Arla
## OCFS2

フロントエンドツール

* mounted.ocfs2 クラスタ上のすべてのボリュームを表示
* o2image メタデータのコピー、復元

## O2CB

主なコンポーネント

* o2nm ノードマネージャ
* o2hb ハートビート
* o2net 通信エージェント
* o2dlm 分散ロックマネージャ
* dlmfs インメモリファイルシステム

設定ファイルは

/etc/ocfs2/cluster.conf
/etc/sysconfig/o2cb

### o2cb設定項目

* O2CB_ENABLED 起動時有効にするか
* O2CB_BOOTCLUSTER 起動クラスタ名
