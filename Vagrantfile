# frozen_string_literal: true

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config| # rubocop:disable Metrics/BlockLength
  config.vm.box = "ubuntu/trusty64"
  config.vm.provider :virtualbox do |box|
    box.memory = "2048"
  end

  # rubocop:disable Style/NumericLiterals
  config.vm.network :forwarded_port, guest: 3000, host: 3036
  # rubocop:enable Style/NumericLiterals

  # let anyone connect to the vm db without a pw
  pg_hba_conf = <<-CONF
    # Database administrative login by Unix domain socket
    local   all             postgres                                peer

    # TYPE  DATABASE        USER            ADDRESS                 METHOD

    # "local" is for Unix domain socket connections only
    local   all             all                                     trust
    # IPv4 connections:
    host    all             all             0.0.0.0/0               trust
    # IPv6 local connections:
    host    all             all             ::1/128                 trust
  CONF

  ruby_version = if File.exist?(".ruby-version")
                   File.read(".ruby-version").strip
                 else
                   "2.6.3"
                 end

  config.vm.provision :shell, inline: <<~BASHBABY
    cd /etc/apt/sources.list.d/
    sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" > pgdg.list'
    wget -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -

    apt-get update
    apt-get upgrade

    apt-get install -y libssl-dev libreadline-dev
    apt-get install -y libpq-dev postgresql-11 postgresql-contrib-11 sqlite3 libsqlite3-dev mysql-server
    apt-get install -y nodejs git zip ntp qt5-default libqt5webkit5-dev qtbase5-dev xvfb
    apt-get install -y libgmp-dev tree
    apt-get install -y imagemagick
    apt-get install -y redis-server

    sudo -u postgres PGPASSWORD=postgres psql -U postgres -c "CREATE USER root WITH PASSWORD 'password';"
    sudo -u postgres PGPASSWORD=postgres psql -U postgres -c "ALTER USER root CREATEDB; ALTER ROLE root SUPERUSER;"
    sudo -u postgres PGPASSWORD=postgres psql -U postgres -c "CREATE USER vagrant WITH PASSWORD 'password';"
    sudo -u postgres PGPASSWORD=postgres psql -U postgres -c "ALTER USER vagrant CREATEDB; ALTER ROLE vagrant SUPERUSER;"

    mv /etc/postgresql/11/main/pg_hba.conf /etc/postgresql/11/main/pg_hba.conf.bak
    echo "#{pg_hba_conf}" > /etc/postgresql/11/main/pg_hba.conf
    echo "listen_addresses = '*'" >> /etc/postgresql/11/main/postgresql.conf
    chown postgres:postgres /etc/postgresql/11/main/pg_hba.conf
    service postgresql restart

    service redis-server restart
  BASHBABY

  config.vm.provision :shell, privileged: false, inline: <<~BASHBABY
    gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3

    git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

    echo 'cd /vagrant/' >> ~/.bash_profile

    echo '# tune ruby GC especially for rspec' >> ~/.bash_profile
    echo 'export RUBY_GC_MALLOC_LIMIT=100000000' >> ~/.bash_profile
    echo 'export RUBY_GC_HEAP_INIT_SLOTS=1000000' >> ~/.bash_profile
    echo 'export RUBY_HEAP_FREE_MIN=10000' >> ~/.bash_profile

    . ~/.bash_profile

    echo "echo ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~" >> ~/.bash_profile
    echo "echo 'helpful commands:'" >> ~/.bash_profile
    echo "echo 'start the webserver: bin/rails s -b 0.0.0.0'" >> ~/.bash_profile
    echo "echo 'start a console to type commands: bin/rails c'" >> ~/.bash_profile
    echo "echo 'migrate the db when migrations pending: bin/rails db:migrate'" >> ~/.bash_profile
    echo "echo ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~" >> ~/.bash_profile

    rbenv install #{ruby_version}
    rbenv global #{ruby_version}

    gem install bundler
    gem install rails
    rbenv rehash

    echo ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    echo Installation successful!
    echo 'try running `vagrant ssh`'
    echo ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  BASHBABY
end
