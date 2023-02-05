## サーバーの展開

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

## 

**hosts.yml**のIPアドレスはよしなに設定。  

```bash
# タスク内容の確認
ansible-playbook -i hosts.yml --list-tasks install_nginx.yml

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

- todo
  - nginxの設定ファイルを配布 -> 検証 -> 再起動
