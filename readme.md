# Grafana

## Preparação do ambiente

- CMDer
- Vagrant
- Telegraf
- VirtualBox

## Configurações

### Criando a maquina virtual

```script
vagrant up
```

### Acessando

```script
vagrant ssh
```
 
### Atualizando pacotes

```script
sudo apt-get update && sudo apt-get upgrade -y
```

### Instalando o Docker

```script
curl -fsLS https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh
```

### Adicionar usuario ao vagrant

```script
sudo usermod -aG docker vagrant
```

### Configurando e Instalando o grafana

```script
mkdir -p volumes/grafana

docker network create grafana-net

ID=$(id -u)

docker run -d --user $ID -v /home/vagrant/volumes/grafana/:/var/lib/grafana -p 3000:3000 --name=grafana --network=grafana-net grafana/grafana
```

### Configurando e Instalando o InfluxDB

```script
mkdir -p $PWD/volumes/influxdb

docker run -d -v "$PWD/volumes/influxdb:/var/lib/influxdb" -p 8083:8083 -p 8086:8086 -p 25826:25826/udp --name=influxdb --network=grafana-net influxdb:1.0

wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -

/etc/lsb-release

"deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt-get update && sudo apt-get install telegraf

sudo service telegraf start
```

### Conectando no InfluxDB e acessando as métricas

```script
docker exec -ti influxdb bash

influx

use telegraf

show measurements

exit
exit
```

### Ferramenta para estressar o servidor

```script
sudo apt-get install stress-ng

stress-ng -c 0 -l 95
```

### Estressando a Memória do servidor

```script
$ stress-ng --vm-bytes $(awk '/MemAvailable/{printf "%d\n", $2 * 0.9;}' < /proc/meminfo)k --vm-keep -m 1
```
