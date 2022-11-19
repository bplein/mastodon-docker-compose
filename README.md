# mastodon-docker-compose

The goal of this repo is to provide a simple way to configure and deploy an instance of Mastodon using a released image (vs. building your own).

## Prepare your environment

Clone this repository and then rename the directory to whatever you like, probably representing the name of the Mastodon instance (especially if you want to run several different instances)

Then run these commands to prep the filesystem:

# prepare directory for elastic https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html

mkdir -p ./elasticsearch/data
mkdir -p ./elasticsearch/logs
chmod -R g+rwx ./elasticsearch
chgrp -R 0 ./elasticsearch

#prepare public directory for proper permissions
mkdir -p ./public/system
chmod -R 644 ./public
chown -R 991 ./public

# prepare host for elastic. https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html
# increase vm.max_map_count to at least 262144 (prevents crash in elastic)
# sysctl -w vm.max_map_count=262144 
# echo "vm.max_map_count = 262144" > /etc/sysctl.d/11.mastodon.conf

# run once and add to .env, without clobbering anything in there
docker-compose run --rm web bundle exec rake mastodon:webpush:generate_vapid_key
# to-do: add code to update .env

# twice for secrets, need to get this automated to edit the .env file and not to clobber
docker-compose run --rm web bundle exec rake secret
# to-do: add code to update .env

# bring up everything
docker-compose up -d

# finalize any database migration
docker-compose run --rm web rails db:migrate

# restart everything
docker-compose down
docker-compose up -d


