# Privoxy on AWS Lightsail
- DockerでPrivoxy(Adblock)を実行
- zabbix監視可能とするためagentインストール
- 不要サービス停止

## Lightsail側

### Lightsail 起動スクリプト

Lightsail作成時に下記起動スクリプトを実行させる
- 公開鍵はPC環境に応じてあらかじめ変更
- rootで実行されるため、sudo不要だが後程手動でできるようにここではsudoを含めて記載
- SSHポートはデフォルトの22だけではなく別ポート(例:2222)も有効にし、ファイアウォールで有効・無効にする
```
## Make swapfile
sudo dd if=/dev/zero of=/swapfile bs=1M count=512
sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
grep "^/swapfile " /etc/fstab || echo "/swapfile   swap        swap    defaults        0   0" | sudo tee -a /etc/fstab

## Append ssh port
echo "Port 22" | sudo tee -a /etc/ssh/sshd_config
echo "Port 2222" | sudo tee -a /etc/ssh/sshd_config

## Append authorized_keys
cat << EOL >> /home/ec2-user/.ssh/authorized_keys
  ### SSHログイン元となるPCの公開鍵を記載 ###
EOL

sudo yum update -y
sudo yum install -y git python-pip
sudo pip install ansible
sudo reboot
```

Lightsailマシンが完成したら次に進む

## 管理PC側(Ansible実行)

### Ansible Playbookを準備・実行
インベントリファイルを変更し、設定先のLightsailサーバへ変更する
- privoxy.lightsail.example.net → 要変更
```
cp -p inventory.demo.conf inventory.conf
vi inventory.conf
```

playbookを実行し、各種設定を行う
```
ansible-playbook-3 -i inventory.conf playbook.yml
```

