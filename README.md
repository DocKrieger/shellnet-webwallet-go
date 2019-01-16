
## Setup on Ubuntu 16.04+

#### Install the required packages

```
apt-get update
sudo add-apt-repository ppa:certbot/certbot -y
apt-get update
sudo apt install -y git nginx python-certbot-nginx postgresql postgresql-contrib redis-server	
```
#### Install PostgreSQL

Set a password for postgresql
```
sudo passwd postgres
```

Login and set password with postgres 
```
su - postgres
psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'newpassword';"
```


#### Install Go

```
wget https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.11.4.linux-amd64.tar.gz
rm go1.11.4.linux-amd64.tar.gz
export GOPATH=$HOME/work
export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
go get github.com/gomodule/redigo/redis \
	github.com/julienschmidt/httprouter \
	github.com/lib/pq \
	github.com/opencoff/go-srp
	
```

#### Install Azur Deamon
```
wget https://getazur.org/linux-cli.tar.gz
tar -xvf linux-cli.tar.gz
rm linux-cli.tar.gz
```

#### Setup Azur Deamon

Create Wallet

```
cd linux-cli
./zedwallet
```


Run this once.
```
./azur-service --container-file <container name> -p <password> -g
```

Point turtle-service at an existing daemon like this
```
./azur-service --rpc-password <rpc password> --container-file <container name> -p <container password> --daemon-address node-1.getazur.org --daemon-port 15251
```


#### Postgres Setup

```
gitclone --recursive https://github.com/getazur/shellnet-webwallet-go.git
cat user_db.sql | psql -U postgres -h 127.0.0.1
cat transaction_db.sql | psql -U postgres -h 127.0.0.1
```

#### Setup Web Wallet

Edit these files:
* services/main/run.sh  
```bash
#!/usr/bin/env bash
HOST_URI='https://shellnet.pw' \ # Web wallet address
HOST_PORT=':8080' \ # Internal server port
USER_URI='http://localhost:8081' \ # Internal requests to user api
WALLET_URI='http://localhost:8082' \ # Internal requests to wallet api
go run main.go utils.go
```
* services/wallet/run.sh  
```bash
#!/usr/bin/env bash
DB_USER=<postgres username> \ # Postgres DB username, NOT system account username
DB_PWD=<postgres password> \ # Postgres DB password, NOT system account password
HOST_URI='http://localhost' \ # Internal wallet api
HOST_PORT=':8082' \ # Internal wallet api port
RPC_PWD=<turtle-service RPC password>  \ # Your turtle-service RPC password
RPC_PORT=':8070' \ # Your turtle-service RPC port
go run wallet.go utils.go
```
* services/user/run.sh  
```bash
#!/usr/bin/env bash
DB_USER=<postgres username> \ # Postgres DB username, NOT system account username
DB_PWD=<postgres password> \ # Postgres DB password, NOT system account password
HOST_URI='http://localhost' \ # Internal user api
HOST_PORT=':8081' \ # Internal user api port
WALLET_URI='http://localhost:8082' \ # Internal wallet api
go run users.go utils.go
```

`~$ cd services/main ; ./run.sh & disown`  
`~$ cd services/wallet ; ./run.sh & disown`  
`~$ cd services/user ; ./run.sh & disown` 
