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

## xl.cfg

主な設定項目

kernelでカーネルイメージを指定

vcpusでCPU数、xlのコマンドvcpu-setと混同しないように

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

-のみ

メモリの指定は-m

カーネルイメージの指定は-kernel

コマンドラインパラメータの指定は-append

初期RAMディスクの指定は-initrd

CDROMを入れて起動する場合は-cdrom

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


スナップショット関係のコマンド

* savevm
* loadvm
* delvm