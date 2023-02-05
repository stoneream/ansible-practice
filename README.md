# ansible-practice

Ansibleでサーバー構築する練習のメモ  
いろいろ雑多においてあります。  

- [標準的なWebアプリケーションのサーバー構築をAnsibleでやりたい](web-app/index.md)

## Ansibleのインストール方法

```bash
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
ansible --version

# ansible [core 2.14.2]
#   config file = /etc/ansible/ansible.cfg
#   configured module search path = ['/home/user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
#   ansible python module location = /usr/lib/python3/dist-packages/ansible
#   ansible collection location = /home/user/.ansible/collections:/usr/share/ansible/collections
#   executable location = /usr/bin/ansible
#   python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0] (/usr/bin/python3)
#   jinja version = 3.0.3
#   libyaml = True
```

- [Installing Ansible on specific operating systems — Ansible Documentation](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu)

## ディレクトリ構造について

- [Sample Ansible setup — Ansible Documentation](https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html)

### playbookのディレクトリを掘るショートカット

```bash
mkdir playbook && cd playbook
mkdir {group_vars,host_vars,roles}
```

## インベントリの書き方とか

inventory = 対象サーバー（たち）  
グループ化しておいたほうが扱いやすい  

iniで書く場合は暗黙的にallというグループが定義され、全ホストが所属しているっぽい

### 参考

- [Ansibleのinventory入門 | DevelopersIO](https://dev.classmethod.jp/articles/inventory/)
- [How to build your inventory — Ansible Documentation](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups)
- [インベントリーの構築方法 — Ansible Documentation](https://docs.ansible.com/ansible/2.9_ja/user_guide/intro_inventory.html#inventoryformat)

## 学習リソース

- [Ansible tips and tricks — Ansible Documentation](https://docs.ansible.com/ansible/latest/tips_tricks/index.html)
  - 公式ドキュメント
- [Ansible の使い方](https://zenn.dev/y_mrok/books/ansible-no-tsukaikata)
  - わかりやすかった
