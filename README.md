# Monitoring Server Setup

This repo is meant to make setting up a full `tick` stack with SSL protection and proper authentication (Google OAuth) easy to get up and running quickly from a clean VM. For most setups (<10 monitored hosts) 1 CPU and 2-4 GB RAM works just fine. 

There are 4 parts to this system:
- [Chronograf](https://github.com/influxdata/chronograf) - GUI
- [InfluxDB](https://github.com/influxdata/influxdb) - Datastore
- [Kapacitor](https://github.com/influxdata/kapacitor) - Stream Processing / Alerting
- [Telegraf](https://github.com/influxdata/telegraf) - Collection

This setup exposes two auth enabled services at:

```bash
# A Chronograf Instance to manage your monitoring installation
https://mon.example.com

# The InfluxDB instance backing the installation
https://mondb.example.com
```

### `mon` && `mondb`

The SSL and load balancing is provided by this really nice repo: [evertramos/docker-compose-letsencrypt-nginx-proxy-companion](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion)
To setup the repo in docker mon, for a production `tick` installation, first set two DNS A records (`mon.example.com` and `mondb.example.com`) to the IP of you host, then run the following commands:

```shell
# First copy the files from the docker-mon folder to the server:
scp ./docker-mon/ <user>@<host>:$HOME/docker-mon

# Install apt dependencies
sudo apt-get update
sudo apt-get install -y curl git

# Install docker
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
sudo sudo usermod -aG docker johnzampolin

# Install docker compose
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install nginx-proxy setup with SSL magic ✨
git clone https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion.git $HOME/nginx-proxy
cd $HOME/nginx-proxy
mkdir -p data
cp .env.sample .env

# Edit the .env file to point to your newly created data directory for nginx
# And uncomment the logging limits at the bottom
nano .env

# Then start the proxy
./start.sh

# Then install the tick stack
cd $HOME/docker-mon
mkdir data
chmod +x ./ops
./ops init-mon $(pwd)/data
cp .env.sample .env

# The following env need changing:
# LETSENCRYPT_EMAIL
# CHRONOGRAF_DATA
# CHRONOGRAF_TOKEN_SECRET
# CHRONOGRAF_GOOGLE_CLIENT_ID
# CHRONOGRAF_GOOGLE_CLIENT_SECRET
# CHRONOGRAF_PUBLIC_URL
# CHRONOGRAF_GOOGLE_DOMAINS
# CHRONOGRAF_VIRTUALHOST
# KAPACITOR_DATA
# KAPACITOR_CONFIG
# INFLUXDB_DATA
# INFLUXDB_CONFIG
# INFLUXDB_VIRTUALHOST
# TELEGRAF_CONFIG
nano .env

# Then spin up the services
docker-compose up -d

# Once the Influx server is running you will need to enable admin on influxdb
# https://docs.influxdata.com/influxdb/v1.4/query_language/authentication_and_authorization/#set-up-authentication
# This will require an instance restart (docker-compose down && docker-compose up -d)

# Change the kapacitor and telegraf configs to use the newly user created
# This will require an instance restart (docker-compose down && docker-compose up -d)

# At this point you should be able to log into the Chronograf web interface located @ mon.example.com
# Register the influxdb and kapacitor instances in the Chronograf dashboard and your setup is ready to be written to!
```
