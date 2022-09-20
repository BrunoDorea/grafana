# Grafana

## Preparação do ambiente

- CMDer
- Vagrant
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