# Monitoring Server Setup

There are 4 parts to this system:
- [Chronograf](https://github.com/influxdata/chronograf) - GUI
- [InfluxDB](https://github.com/influxdata/influxdb) - Datastore
- [Kapacitor](https://github.com/influxdata/kapacitor) - Stream Processing / Alerting
- [Telegraf](https://github.com/influxdata/telegraf) - Collection


### `mon1` && `mon1db`

The SSL and load balancing is provided by this really nice repo: https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion
To setup the repo in docker mon, for a production `tick` installation, first set two DNS A records to the IP of you host, then run the following commands:

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
sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
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
./ops init-mon $(pwd)/data
cp .env.sample .env

# The following env need changing:
# CHRONOGRAF_DATA
# KAPACITOR_DATA
# INFLUXDB_DATA
# CHRONOGRAF_GOOGLE_CLIENT_ID
# CHRONOGRAF_GOOGLE_CLIENT_SECRET
# CHRONOGRAF_PUBLIC_URL
# CHRONOGRAF_VIRTUALHOST
# INFLUXDB_VIRTUALHOST
nano .env

# Then spin up the services
docker-compose up -d

# You will also need to setup an admin user on the influxdb instance
# https://docs.influxdata.com/influxdb/v1.4/query_language/authentication_and_authorization/#set-up-authentication

# Change the kapacitor and telegraf config to use the new user created

# Register the influxdb and kapacitor instances in the Chronograf dashboard
```
