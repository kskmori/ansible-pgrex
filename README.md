(Japanese)

# PG-REX Ansible Playbook

PG-REX は、Pacemaker リポジトリパッケージと PostgreSQL レプリケーション機能を組み合わせた、データベースの高可用ソリューションです。

* PG-REX プロジェクトホームページ
  * https://osdn.net/projects/pg-rex/

このリポジトリは、PG-REX の付属ドキュメントに記載されたインストール手順を Ansible で自動化した Playbook の例です。
[Pacemaker リポジトリパッケージ用 Ansible Playbook](https://github.com/kskmori/ansible-pacemaker)と組み合わせて使用します。

## 対象バージョン・手順

以下のページに記載されているバージョン・手順を Ansible Playbook にしたものです。

* 対象バージョン: [PG-REX9.6 1.0.4] (https://ja.osdn.net/projects/pg-rex/releases/69268)

以前のバージョンを利用する場合は、対応するブランチを checkout して使ってください。

 * PG-REX9.5: ブランチ [branch-pg-rex95](https://github.com/kskmori/ansible-pgrex/tree/branch-pg-rex95)

## 前提条件

この　playbook を使うには以下の準備をあらかじめ行っておいてください。

* PostgreSQL 本体および PG-REX ツール類のパッケージを別途ダウンロードしておくこと。
  * 必要なパッケージは以下のコマンドでダウンロードできます。

  > $ ansible-playbook 00-download.yml

  * もしくは以下のファイルを手動でダウンロードし、所定のディレクトリへ配置してください。
    * 配置ディレクトリ: roles/pgrex-install/files/
    * 配置するファイル一覧:
      * [PostgreSQL公式ダウンロードサイト](https://www.postgresql.org/download/)から入手するもの
        * postgresql96-9.6.*.rhel7.x86_64.rpm
        * postgresql96-contrib-9.6.*.rhel7.x86_64.rpm
        * postgresql96-docs-9.6.*.rhel7.x86_64.rpm
        * postgresql96-libs-9.6.*.rhel7.x86_64.rpm
        * postgresql96-server-9.6.*.rhel7.x86_64.rpm
      * [PG-REXプロジェクト](https://osdn.net/projects/pg-rex/)から入手するもの
        * IO_Tty-1.11-1.el7.x86_64.rpm
        * Net_OpenSSH-0.62-1.el7.x86_64.rpm
        * pg-rex_operation_tools_script-1.8.3-1.el7.noarch.rpm

* [Pacemaker リポジトリパッケージ用 Ansible Playbook](https://github.com/kskmori/ansible-pacemaker)を実行するために必要な準備・ファイルのダウンロードを完了しておくこと。
* STONITH機能の利用に必要なパッケージのインストールと設定を完了していること。
  * IPMI(物理環境の場合): ipmitools パッケージのインストール、ハードウェアのIPMIデバイスの設定
  * libvirt(仮想環境の場合): libvirt-client パッケージのインストール、ホストへのsshパスワード無しログイン設定
* OSメディアもしくはリポジトリが参照できるように /etc/yum.repo.d を設定しておくこと(OS標準の依存パッケージを自動的にインストールするため)

## Pacemaker リポジトリパッケージのインストール

最初に、[Pacemaker リポジトリパッケージ用 Ansible Playbook](https://github.com/kskmori/ansible-pacemaker)を使用し、「Linux-HA Japan 追加パッケージ利用例」の手順に従って Pacemaker のインストールを行ってください。

* 留意点
  * PG-REX では pm_logconv-cs の利用を前提としているため、「Linux-HA Japan 追加パッケージ利用例」の手順(10-pacemaker-install.yml, 11-pacemaker-tools-enable.yml の二つ)を実施してください。
  * 設定は group_vars/hacluster.pg-rex のサンプルを参考にしてください。
    * "PM_LOGCONV_CONFIG" 設定は不要です。本 playbook の中で設定します。

## PG-REX playbook 設定

以下のファイルを環境に合わせ設定します。詳細はファイルの中身を参照してください。

* hosts
  * インベントリファイル。hosts.sample を参考に構築対象ホスト名を修正して作成してください。
  * 一番目に記述したホストが初期構築時の Master ノードとなります。
* group_vars/pgrex-cluster
  * PG-REX環境定義パラメタ設定。pgrex-cluster-sample を参考に構築環境に合わせて設定してください。

## PG-REX playbook 実行例

* (1) PG-REX 環境の構築
  * PG-REX環境の構築とPacemaker設定ファイル(crm設定ファイル)の作成を行います。

  > $ ansible-playbook -u root -i hosts 10-pgrex-install.yml

  * Pacemaker設定ファイル(crm設定ファイル)は Masterノードの root ユーザのホームディレクトリに作成されます。

* (2) PG-REX の起動
  * PG-REX の起動は PG-REX のマニュアルの手順に従います。詳細は PG-REX のマニュアルを参照してください。
  * Master ノードにログインし Master 側を起動します。初回起動時のみ、引数に以下のPacemaker設定ファイルを指定し、Pacemaker設定を反映します。

  >  \# pg-rex_master_start PG-REX9.6_pm_crmgen_env.crm

  * Master ノードの起動完了後、Slave 側ノードにログインし Slave 側を起動します。

  >  \# pg-rex_slave_start

* (3) PG-REX の停止

  * Slave ノードにログインし、Slave 側を停止します。

  >  \# pg-rex_stop

  * Master ノードにログインし、Master 側を停止します。

  >  \# pg-rex_stop

* (4) PG-REX のアンインストール
  * PG-REXを全てアンインストールします。確認のプロンプトが出ます。
  * デフォルトではデータベースの内容は削除しませんが、`-e DROP_DB=true` オプションを付与することでデータベースの内容も全て削除します。

  > $ ansible-playbook -u root -i hosts 99-pgrex-uninstall.yml

  * データベースも全て削除する場合。

  > $ ansible-playbook -u root -i hosts -e DROP_DB=true 99-pgrex-uninstall.yml


## 補足

* 本 playbook は、Pacemaker 本体の設定と独立して実行やバージョンアップができるように playbook をあえて分離しています。実システムへの適用時は、それぞれを適宜組み合わせて playbook を作成するなどの部品として活用していただければと思います。
* playbook のファイル名の数字は単にファイル名のソート順のために付与したもので深い意味はありません。
* 19-pgrex-crm-config.yml は リソース設定の変更(Pacemaker設定ファイルの再作成と /etc/pm_logconv.conf 再設定)のみを行いたい場合に利用できます。通常は 10-pgrex-install.yml を利用するだけで充分です。
