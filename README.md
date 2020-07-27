# Rails 6 capistrano setup for staging environment

## specification
* Ruby 2.6.5
* Rails 6.0.0
* capistrano
* ubuntu 18.04
* nginx / passenger

## documents
* https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-18-04
* https://gorails.com/deploy/ubuntu/18.04

## employed environment variables
```
# .env

PEM_DIRECTORY=
REPO_URL=
APP_NAME=
STAGING_DEPLOY_DIRECTORY=
STAGING_SERVER_IP=
STAGING_SERVER_USER=

# Double check requirement of these for variables below in the staging environment .env file

STAGING_DB_DATABASE=
STAGING_DB_USERNAME=
STAGING_DB_PASSWORD=
STAGING_SECRET_KEY_BASE=
# not sure if the last STAGING_SECRET_KEY_BASE can be dealt by rails credentials rather than here.
```
## potential issues/setbacks
* rails6 secret issues (https://medium.com/@kirill_shevch/encrypted-secrets-credentials-in-rails-6-rails-5-1-5-2-f470accd62fc)
* rails secret_key_base setup example (https://stackoverflow.com/a/34350507)
```
$ irb
>> require 'securerandom'
=> true
>> SecureRandom.hex(64)
=> "3fe397575565365108556c3e5549f139e8078a8ec8fd2675a83de96289b30550a266ac04488d7086322efbe573738e7b3ae005b2e3d9afd718aa337fa5e329cf"
>> exit
```
* nginx setup
```
# /etc/nginx/sites-enabled/myapp
server {
  listen 80;
  listen [::]:80;

  server_name _;
  root /home/deploy/myapp/current/public;

  passenger_enabled on;
  passenger_app_env production;

  location /cable {
    passenger_app_group_name myapp_websocket;
    passenger_force_max_concurrent_requests_per_process 0;
  }

  # Allow uploads up to 100MB in size
  client_max_body_size 100m;

  location ~ ^/(assets|packs) {
    expires max;
    gzip_static on;
  }
}

# double check to match  passenger_app_env to staging or your specific environment
```
* webpacker staging environment specification(https://itnext.io/deploy-staging-and-production-applications-to-single-server-using-capistrano-rails-1d5ab558d44f)
```
ii. Define staging environment for Webpacker — Rails 6 also requires setting environment for webpacker.
You need to add staging section in config/webpacker.yml file.
You can simply copy the production block from the same file and edit for staging.
Also you need to create config/webpack/staging.js file.
You can copy the contents from config/webpack/production.js into staging.js file.
```
* employing .env variables with the right syntax
```
## pay attention to the syntax
# config/database.yml
staging:
  adapter: mysql2
  database: <%= ENV["STAGING_DB_DATABASE"] %>
  username: <%= ENV["STAGING_DB_USERNAME"] %>
  password: <%= ENV["STAGING_DB_PASSWORD"] %>
  encoding: utf8
  host: localhost
  timeout: 5000
  encoding: utf8mb4
  collation: utf8mb4_unicode_ci
```
* adding '.env' into linked file in capistrano setup for staging environment
```
# config/deploy.rb
append :linked_files, ".env"

## this creates sym link in the server under myapp/shared/
```
