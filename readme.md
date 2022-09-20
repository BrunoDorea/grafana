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
stress-ng --vm-bytes $(awk '/MemAvailable/{printf "%d\n", $2 * 0.9;}' < /proc/meminfo)k --vm-keep -m 1
```

### Alterando as permissões do socket do Docker

```script
sudo usermod -aG docker telegraf

cat telegraf.conf | grep -A 45 inputs.docker | egrep -v "^#"

[[inputs.docker]]
   endpoint = "unix:///var/run/docker.sock"
   container_names = []
   container_name_include = []
   container_name_exclude = []
   timeout = "5s"
   total = false

sudo service telegraf status
```

### Instalando o Apache

```script
sudo apt-get install -y apache2
```

### Alterando permissões para leitura de log do Apache

```script
sudo usermod -a -G adm telegraf

cat telegraf.conf | grep -A 45 "\[\[inputs.logparser\]\]" | egrep -v "^#"
 [[inputs.logparser]]
   files = ["/var/log/apache2/access.log"]
   from_beginning = true
   [inputs.logparser.grok]
     patterns = ["%{COMBINED_LOG_FORMAT}"]
     measurement = "apache_access_log"

sudo service telegraf restart
```

### Criando painéis de monitoramento

```script
SELECT count("request")
FROM "apache_access_log"
WHERE "host" =  'ubuntu-bionic'
  AND "resp_code" = '404'
  AND $timeFilter 
  AND "agent" != 'Go-http-client/1.1'
  AND agent != 'worldping-api'
```

```script
SELECT  "request"
FROM "apache_access_log"
WHERE "host" =~ /^$server$/ 
  AND "resp_code" =~ /^$code$/ 
  AND $timeFilter  

SELECT  "client_ip"
FROM "apache_access_log"
WHERE "host" =~ /^$server$/ 
  AND "resp_code" =~ /^$code$/ 
  AND $timeFilter  
```

### Estressando a aplicação

```script
for i in {1..501}; do curl http://localhost/  > /dev/null 2>&1;done

for i in {1..501}; do curl http://localhost/alura  > /dev/null 2>&1;done
```


```script

```

```script

```
