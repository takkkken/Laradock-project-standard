# -*- mode: ruby -*-
# vi: set ft=ruby :

# ---------------------------------------------------------
# config
# ---------------------------------------------------------
HOST_DIR = "./export"
GUEST_DIR = "/home/bargee/common"
HOST_APP_DIR = "./app"
GUEST_APP_DIR = "/opt/app"
APP_NAME = "sample_app"
# ---------------------------------------------------------

GUEST_APP_DIR2 = GUEST_APP_DIR.gsub(/\//,'\\/')

Vagrant.configure(2) do |config|
  config.vm.box = "ailispaw/barge"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "forwarded_port", guest: 80,   host: 10080
  config.vm.synced_folder HOST_DIR, GUEST_DIR, mount_options: ['dmode=777', 'fmode=777']
  config.vm.synced_folder HOST_APP_DIR, GUEST_APP_DIR, mount_options: ['dmode=777', 'fmode=777']

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048 # MB
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
  end

#  #vagrant reloadで実行してテスト用
#  config.vm.provision :shell, run: "always", :inline => <<-EOT
#    sudo su -
#    cd #{GUEST_APP_DIR}/laradock
#    pwd
#    docker-compose up -d nginx postgres-postgis
#    docker-compose ps
#
#  EOT


  #初回起動時のみ実行（この中は\は\でエスケープすること）
  config.vm.provision :shell, :inline => <<-EOT

    sudo su -

    # ReInstall Docker latest version
    /etc/init.d/docker restart latest

    wget -qL https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m`
    chmod +x docker-compose-`uname -s`-`uname -m`
    mv docker-compose-`uname -s`-`uname -m` /opt/bin/docker-compose
    chown root:root /opt/bin/docker-compose

    # JP LocaleSetting
    cd /etc
    unlink localtime
    ln -s ../usr/share/zoneinfo/Etc/GMT-9 localtime

    # StartUp Shell Install
    cp #{GUEST_DIR}/start.sh /etc/init.d/start.sh
    chmod +x /etc/init.d/start.sh
    /etc/init.d/start.sh

    # Laradock Install
    mkdir #{GUEST_APP_DIR}
    cd #{GUEST_APP_DIR}
    git config --global http.sslVerify false
    git clone https://github.com/Laradock/laradock.git
    cd laradock

    # Laravelのアプリパスを修正
    #sudo cp env-example .env
    sed -e "s/APP_CODE_PATH_HOST=\\.\\.\\//APP_CODE_PATH_HOST=#{GUEST_APP_DIR2}/" env-example > .env

    # Image & Container build 超絶長い2時間位
    docker-compose up -d nginx postgres-postgis
    docker-compose exec -T workspace sh -c "composer create-project --prefer-dist laravel/laravel #{APP_NAME}"

    # Laravelのアプリパスを再度修正（※注意　最初にアプリパスをAPP_NAMEに設定すると、APP_NAME/APP_NAMEのディレクトリ構成になるのであえて２度編集を加える）
    cat .env > .env.tmp
    sed -e "s/APP_CODE_PATH_HOST=#{GUEST_APP_DIR2}/APP_CODE_PATH_HOST=#{GUEST_APP_DIR2}\\/#{APP_NAME}/" .env.tmp > .env
    rm .env.tmp
    docker-compose up -d nginx postgres-postgis

    # アプリ側に移動
    cd ../#{APP_NAME}

    # ストレージのパーミッションを設定
    chmod 777 -R storage

    # PostgreSQLの接続設定（デフォはMySQLの為MySQLの設定は削除）
    cat .env > .env.tmp
    sed -e "/^DB_/d" .env.tmp > .env
    #rm .env.tmp
    cat <<EOF >> .env

DB_CONNECTION=pgsql
DB_HOST=postgres-postgis
DB_PORT=5432
DB_DATABASE=default
DB_USERNAME=default
DB_PASSWORD=secret
EOF

    # laravel-Admin Install
    docker-compose exec -T workspace sh -c "composer require encore/laravel-admin"
    docker-compose exec -T workspace sh -c 'php artisan vendor:publish --provider="Encore\\Admin\\AdminServiceProvider"'
    docker-compose exec -T workspace sh -c "php artisan admin:install"


  EOT
end



