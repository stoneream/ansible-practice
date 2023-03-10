# 標準的なWebアプリケーションのサーバー構築をAnsibleでやりたい

## サーバー構成

| ホスト名 | 説明         |
| :------- | :----------- |
| web01    | Webサーバー（nginx）  |
| web02    | Webサーバー（nginx）  |
| mon01    | 監視サーバー（Prometheus） |
| db01 | DBサーバー（マスター） |
| db02 | DBサーバー（スレーブ） |

- それぞれのサーバーにはPrometheusのexporterが動いている
- **そのうちやる**
  - アプリケーションサーバーを立てる
    - 動かすアプリケーションは決まってない
    - Webサーバーをリバースプロキシにする
    - デプロイ & 切り戻し をできるようにする
  - バックアップとかその他の保守系の作業がAnsibleでできるか検証
  - 秘匿情報をうまく扱う仕組みがあるっぽいので検証
  - インストール系のplaybookを一つにまとめたい

## サーバーの展開

とりあえず検証したいだけなのでvagrantで立ている。  

```bash
cd vagrant
vagrant up
```

コントロールノードがSSHで初めて接続するケース（厳密にはfingerprintが~/.ssh/known_hostsに保存されていないケース）で

```
fatal: [web01]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host."}
fatal: [web02]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host."}
```

みたいなエラーが発生するため、サーバーが立ったら一度手動でSSHを接続しておくのが良い。  
検証を無視するオプションも存在する（StrictHostKeyCheckingをnoにするとか）が、あまりやりたくない。  

## テンプレートの記述

デバッグで使い勝手が良かったツール

- [ansible.builtin.debug module – Print statements during execution — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)
  - 変数の中身を確認したりするときに便利
- [ansible.builtin.template lookup – retrieve contents of file after templating with Jinja2 — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_lookup.html)
  - 試しに書いたテンプレートファイルが思った通りに出力されるかチェックするときに便利

## マジック変数について

- [マジック変数｜Ansible の使い方](https://zenn.dev/y_mrok/books/ansible-no-tsukaikata/viewer/chapter11)

## hostsファイルを配る

```
ansible-playbook -i hosts.yml roles/common/tasks/configure-hosts-file.yml
```

## nginxのインストール

**hosts.yml**のIPアドレスはよしなに設定。  

```bash
# タスク内容の確認
ansible-playbook -i hosts.yml --list-tasks roles/web/tasks/install_nginx.yml

# 実行
ansible-playbook -i hosts.yml roles/web/tasks/install_nginx.yml
```

```bash
PLAY [install nginx & update cache] ****************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [web01]
ok: [web02]

TASK [cache updated & nginx installed] *************************************************************************************************************************************
changed: [web01]
changed: [web02]

PLAY RECAP *****************************************************************************************************************************************************************
web01                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web02                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

[ansible-playbook — Ansible Documentation](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)

## インストールしたnginxの情報を確認する（アドホックコマンド）

```bash
ansible web -i hosts.yml -m ansible.builtin.command -a "nginx -V"

# web01 | CHANGED | rc=0 >>
# nginx version: nginx/1.18.0 (Ubuntu)
# built with OpenSSL 3.0.2 15 Mar 2022
# TLS SNI support enabled
# configure arguments: --with-cc-opt='-g -O2 -ffile-prefix-map=/build/nginx-d8gVax/nginx-1.18.0=. -flto=auto -ffat-lto-objects -flto=auto -ffat-lto-objects -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -flto=auto -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --add-dynamic-module=/build/nginx-d8gVax/nginx-1.18.0/debian/modules/http-geoip2 --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_sub_module
# 
# web02 | CHANGED | rc=0 >>
# nginx version: nginx/1.18.0 (Ubuntu)
# built with OpenSSL 3.0.2 15 Mar 2022
# TLS SNI support enabled
# configure arguments: --with-cc-opt='-g -O2 -ffile-prefix-map=/build/nginx-d8gVax/nginx-1.18.0=. -flto=auto -ffat-lto-objects -flto=auto -ffat-lto-objects -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -flto=auto -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --add-dynamic-module=/build/nginx-d8gVax/nginx-1.18.0/debian/modules/http-geoip2 --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_sub_module
```

[アドホックコマンドの概要 — Ansible Documentation](https://docs.ansible.com/ansible/2.9_ja/user_guide/intro_adhoc.html)

## 監視サーバーの構築

nginx, prometheusをインストールし、設定ファイルを投入

```bash
ansible-playbook -i hosts.yml roles/monitoring/tasks/install_nginx.yml

ansible-playbook -i hosts.yml roles/monitoring/tasks/install_prometheus.yml

ansible-playbook -i hosts.yml roles/monitoring/tasks/configure-prometheus.yml
```

監視対象のサーバーにprometheus-node-exporterをインストール

```bash
ansible-playbook -i hosts.yml roles/common/tasks/install_prometheus_node_exporter.yml
```

### 設定ファイルの配布 -> 検証 -> 再起動 の流れについて

- [Recording rules | Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/#syntax-checking-rules)
  - Syntax-checking rulesを参照
- [ansible.builtin.command module – Execute commands on targets — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html)

設定ファイルをコピーしたあとにシンタックスチェックを行う。  
エラーだった場合は当該タスク以降の実行を注意する。（今回の場合はPrometheusの再起動）  
（ansible.builtin.commandモジュールは実行したコマンドの戻り値が0以外の場合はエラーと見なされる。）  

## DBサーバーの構築

MariaDBをインストールする。

```bash
ansible-playbook -i hosts.yml roles/db/tasks/install_mariadb.yml
```

todo マスター・スレーブ構成を構築する

## メモ

- [nginx documentation](https://nginx.org/en/docs/)
