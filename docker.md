# Docker - Máquina "server"

## 1. Adicionar docker na Vm server
Instale o docker na máquina em que os containers irão rodar:

```bash
sudo apt update
```
```bash
sudo apt install docker.io
```
```bash
sudo systemctl start docker
```
```bash
sudo systemctl enable docker
```

Instale o docker compose:
```bash 
sudo apt install docker-compose 
```

## 2. Criar o container dos servidores 
crie os arquivos para cada máquina server:
```bash
mkdir server1 server2 server3
```

Adicione os arquivos html nos servidores
```bash
echo '<html><body><h1>server 1</h1></body></html>' > server1/index.html
echo '<html><body><h1>server 2</h1></body></html>' > server2/index.html
echo '<html><body><h1>server 3</h1></body></html>' > server3/index.html

```

crie o arquivo ```docker-compose.yml``` e adicione:

```bash
version: '3'

services:
  server1:
    image: nginx
    container_name: server1
    ports:
      - "8080:80"
    volumes:
      - ./server1:/usr/share/nginx/html

  server2:
    image: nginx
    container_name: server2
    ports:
      - "8081:80"
    volumes:
      - ./server2:/usr/share/nginx/html

  server3:
    image: nginx
    container_name: server3
    ports:
      - "8082:80"
    volumes:
      - ./server3:/usr/share/nginx/html

```

Estrutura de como ficaram os arquivos:

```bash
ls
```
output: 
```docker-compose.yml  server1/  server2/  server3/```


## 3. Executando os containers: 
Os containers precisam estar rodando para que a aplicação funcione.
- para rodar os containers
```bash 
docker-compose up -d
```

- verifique o funcionamento individual dos servidores:
```bash
  curl http://localhost:8080  # Deve retornar "Servidor 1"
  curl http://localhost:8081  # Deve retornar "Servidor 2"
  curl http://localhost:8082  # Deve retornar "Servidor 3"
```

# Na máquina nginx:
## 1. Alteração no load balancer:
altere a configuração de balanceamento em conf.d:

```bash
upstream servidorgustavo {
    server <IP-VM-SERVER>:8080;
    server <IP-VM-SERVER>:8081;
    server <IP-VM-SERVER>:8082;
}

server {
    listen 8083;
    server_name load;

    location / {
        proxy_pass http://servidorgustavo; # proxy reverso
    }

    error_page 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

}
```
reinicie o nginx:

```bash
sudo nginx -s reload
```

### (extra) 
## Libere as portas necessárias no grupo de segurança:
Para garantir que o tráfego seja permitido nas portas ```8080```, ```8081```, ```8082``` e ```8083```, adicione regras de entrada no AWS Security Group associado à sua instância EC2:

1) No console da AWS, vá para a página da sua instância EC2.
2) Encontre o Security Group associado à instância.
3) Clique em Editar regras de entrada.
4) Adicione regras para permitir tráfego HTTP ```(8080, 8081,8082, 8083)```, como no exemplo abaixo:
- <strong>Tipo:</strong> HTTP
- <strong>Protocolo:</strong> TCP
- <strong>Porta:</strong> 80
- <strong>Fonte/origem:</strong>  0.0.0.0/0 (para permitir de qualquer IP)