(Japanese)

# PG-REX9.5用 Ansible Playbook

このリポジトリは、PG-REX9.5 のインストール手順を Ansible で自動化した Playbook の例です。
[Pacemaker リポジトリパッケージ用 Ansible Playbook](https://github.com/kskmori/ansible-pacemaker)と組み合わせて使用します。

## 対象バージョン・手順

以下のページに記載されているバージョン・手順を Ansible Playbook にしたものです。

* 対象バージョン: [PG-REX9.5 1.1.0] (https://osdn.net/projects/pg-rex/releases/66221)

## 前提条件

この　playbook を使うには以下の設定をあらかじめ行っておいてください。

* PostgreSQL 本体および PG-REX 9.5 のツール類のパッケージは別途ダウンロードして以下のディレクトリへ配置しておくこと。
  * 配置ディレクトリ: roles/pgrex-install/files/
  * 配置するファイル一覧:
    * postgresql95-9.5.*.rhel7.x86_64.rpm
    * postgresql95-contrib-9.5.*.rhel7.x86_64.rpm
    * postgresql95-docs-9.5.*.rhel7.x86_64.rpm
    * postgresql95-libs-9.5.*.rhel7.x86_64.rpm
    * postgresql95-server-9.5.*.rhel7.x86_64.rpm
    * IO_Tty-1.11-1.el7.x86_64.rpm
    * Net_OpenSSH-0.62-1.el7.x86_64.rpm
    * pg-rex_operation_tools_script-1.7.2-1.el7.noarch.rpm

* [Pacemaker リポジトリパッケージ用 Ansible Playbook](https://github.com/kskmori/ansible-pacemaker)を使用し、Pacemaker のインストール(10-pacemaker-install.yml)を完了していること。
* STONITH機能の利用に必要なパッケージのインストールと設定を完了していること。
  * IPMI(物理環境の場合): ipmitools パッケージのインストール、ハードウェアのIPMIデバイスの設定
  * libvirt(仮想環境の場合): libvirt-client パッケージのインストール、ホストへのsshパスワード無しログイン設定
* OSメディアもしくはリポジトリが参照できるように /etc/yum.repo.d を設定しておくこと(OS標準の依存パッケージを自動的にインストールするため)


## 設定

以下のファイルを環境に合わせ設定します。詳細はファイルの中身を参照してください。

* hosts
  * インベントリファイル。hosts.sample を参考に構築対象ホスト名を修正して作成してください。
  * 一番目に記述したホストが初期構築時の Master ノードとなります。
* group_vars/pgrex-cluster
  * PG-REX環境定義パラメタ設定。pgrex-cluster-sample を参考に構築環境に合わせて設定してください。

## 実行例

* (1) PG-REX 環境の構築
  * PG-REX環境の構築とPacemaker設定ファイル(crm設定ファイル)の作成を行います。

  > $ ansible-playbook -i hosts 01-pgrex-install.yml

  * Pacemaker設定ファイル(crm設定ファイル)は Masterノードの root ユーザのホームディレクトリに作成されます。

* (2) PG-REX の起動
  * PG-REX の起動は PG-REX のマニュアルの手順に従います。詳細は PG-REX のマニュアルを参照してください。
  * Master ノードにログインし Master 側を起動します。初回起動時のみ、引数に以下のPacemaker設定ファイルを指定し、Pacemaker設定を反映します。

  >  # pg-rex_master_start PG-REX9.5_pm_crmgen_env.crm

  * Master ノードの起動完了後、Slave 側ノードにログインし Slave 側を起動します。

  >  # pg-rex_slave_start 

* (3) PG-REX の停止

  * Slave ノードにログインし、Slave 側を停止します。

  >  # pg-rex_stop 

  * Master ノードにログインし、Master 側を停止します。

  >  # pg-rex_stop

* (4) PG-REX のアンインストール
  * PG-REXを全てアンインストールします。確認のプロンプトが出ます。
  * デフォルトではデータベースの内容は削除しませんが、`-e DROP_DB=true` オプションを付与することでデータベースの内容も全て削除します。

  > $ ansible-playbook -i hosts 99-pgrex-uninstall.yml

  * データベースも全て削除する場合。

  > $ ansible-playbook -i hosts -e DROP_DB=true 99-pgrex-uninstall.yml


## 補足

* playbook のファイル名の数字は単にファイル名のソート順のために付与したもので深い意味はありません。
