# README

This README would normally document whatever steps are necessary to get the
application up and running.

Things you may want to cover:

- Deployment

- Database

# Deployment

### Setup EC2

- Setup ubuntu

  ```sh
    ssh -i /path/to/private-key.pem ubuntu@your-ec2-instance-ip

    sudo apt update && sudo apt upgrade -y

    sudo apt install -y curl git-core build-essential zlib1g-dev libssl-dev \
    libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev \
    libxslt1-dev libcurl4-openssl-dev software-properties-common \
    libffi-dev nodejs yarn nginx \

  ```

- Setup Ruby & Bundler

  ```sh
    git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
    echo 'eval "$(rbenv init -)"' >> ~/.bashrc
    source ~/.bashrc

    git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

    rbenv install 3.1.0
    rbenv global 3.1.0

    gem install bundler
    rbenv rehash

    gem update --system
  ```

- Setup PostgreSQL

  ```sh
    sudo apt install -y postgresql postgresql-contrib libpq-dev

    sudo -u postgres createuser -s ${user}
    sudo -u postgres createdb ${database_name}

    sudo -u postgres psql
    ALTER USER ${user} WITH PASSWORD ${password};
  ```

- Setup Capistrano

  ```sh
    mkdir -p /home/ubuntu/my_app/shared/config
    mkdir -p /home/ubuntu/my_app/shared/config/credentials
  ```

  - Sync to Ec2: .env, credentials, puma.rb .......

### Setup Nginx && Puma

- Setup Nginx

  ```sh
    sudo nano /etc/nginx/sites-available/my_app

    # add to file
    server {
    listen 80;

    root /home/ubuntu/my_app/current/public;
    server_name ${YOUR_PUBLIC_IP};

    location / {
        proxy_pass http://unix:/home/ubuntu/my_app/shared/tmp/sockets/puma.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
    }
  ```

  ```sh
    sudo ln -s /etc/nginx/sites-available/my_app /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl reload nginx
  ```

- Setup Puma

  ```sh
    sudo nano /etc/systemd/system/puma.service

    # add to file
    [Unit]
    Description=Puma application server
    After=network.target

    [Service]
    Type=simple
    User=ubuntu
    WorkingDirectory=/home/ubuntu/my_app/current
    Environment="RAILS_ENV=production"
    ExecStart=/home/ubuntu/.rbenv/shims/bundle exec puma -C /home/ubuntu/my_app/shared/config/puma.rb
    Restart=always

    [Install]
    WantedBy=multi-user.target
  ```

- Change own file socket

  ```sh
    sudo chown ubuntu:www-data /home/ubuntu/my_app/shared/tmp/sockets/puma.sock
    sudo chmod 660 /home/ubuntu/my_app/shared/tmp/sockets/puma.sock
    sudo usermod -aG ubuntu www-data
  ```

  ```sh
    sudo systemctl restart puma
  ```

# Run

  ```sh
    cap production deploy
  ```
