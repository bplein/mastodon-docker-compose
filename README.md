# mastodon-docker-compose

The goal of this repo is to provide a simple way to configure and deploy an instance of Mastodon using a released image (vs. building your own).

The official Mastodon Github repository at https://github.com/mastodon/mastodon has a `docker-compose.yml` file but I found that it could be optimized more for my liking. My goals were to move all of the editable configuration into `.env` and leave the `docker-compose.yml` file as generic as possible across all use cases.

## Prequisites:
- Docker-compose v2 or newer
- A containerized nginx-proxy, we recommend https://github.com/nginx-proxy/nginx-proxy along with https://github.com/nginx-proxy/acme-companion for LetsEncrypt automation. 

We will publish a simple working example of those with docker-compose to go along with this repo.

If you are using an external nginx that expects to reach the web and streaming servers via a port, there will be a second docker-compose.yml published later with the appropriate edits.

## Prepare your environment

Clone this repository and then rename the directory to whatever you like, probably representing the name of the Mastodon instance (especially if you want to run several different instances)

CD to the directory and then run these commands to prep the filesystem:

Prepare directory for elastic https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html

```
sudo mkdir -p ./elasticsearch/data
sudo mkdir -p ./elasticsearch/logs
sudo chmod -R g+rwx ./elasticsearch
sudo chgrp -R 0 ./elasticsearch
```

Prepare public directory for proper permissions
```
sudo mkdir -p ./public/system
sudo chown -R 991:991 ./public
sudo chmod -R 755 ./public
```

Prepare host for elastic. https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html
Increase vm.max_map_count to at least 262144 (prevents crash in elastic)
```
sudo sysctl -w vm.max_map_count=262144 
sudo echo "vm.max_map_count = 262144" > /etc/sysctl.d/11.mastodon.conf
```

Run once and manually add to .env, without clobbering anything in there
```
docker-compose run --rm web bundle exec rake mastodon:webpush:generate_vapid_key
```

Run twice for secrets, manually update into your .env
```
docker-compose run --rm web bundle exec rake secret
```
## Initialize your system
Bring up everything
```
docker-compose up -d
```
Finalize the database creation
```
docker-compose run --rm web rails db:migrate
```
Restart everything
```
docker-compose down
docker-compose up -d
```

## Access your site

You should now be able to go to your site and create your userid. This will create a normal (not admin) user.

After creating the user (and validating email), run `tootctl` to elevate your user to `Owner` (v4.0.0 and later) or `Admin` (pre 4.0)

```
docker-compose run --rm web bin/tootctl accounts modify <username> --role Owner
```