# Laradock-project-standard
軽量OS「bargeOS」に標準的なLaradock環境を構築するvagrantです。
php7 + Laravel5 + nginx + postgresql がインストールされます。


## 使用方法
TortoiseGit等でWindows上にGitクローンしたらコンソールより下記を実行します。  
※実行前にVagrantfileの「APP_NAME = "sample_app"」でアプリ名を適宜変更して下さい。  
※環境にもよりますが１時間位掛かります。  
```
vagrant up && vagrant ssh
```

## 必要なもの
* Vagrant  
https://www.vagrantup.com/downloads.html
* VirtualBox  
https://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html?ssSourceSiteId=otnjp
