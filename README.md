# Docker

## Conhecendo Docker

### Como containers funcionam

- por que são mais leves?
- como garantem o isolamento
- como funcionam sem “instalar um SO”
- como fica a divisão de recursos do sistema

Os containers funcionam como um processo, enquanto maquinas virtuais tem toda a parte de virtualização

![Untitled](Docker%20f2181766190c4e2aadc0e4d5cc7d9b8f/Untitled.png)

sobre o isolamento, quando os containers estiverem em execução, eles usam o conceito de namespace, que garante o isolamento em determinados niveis

![Untitled](Docker%20f2181766190c4e2aadc0e4d5cc7d9b8f/Untitled%201.png)

graças ao namespace uts, cada container vai ter “um pedaço” do kernel isolado

conceito de Cgroups garante que consigamos definir automaticamente e manualmente como o consumo sera feito para cada um dos containers

## Os primeiros comandos

### Conhecendo o docker hub

```bash
docker run hello-world # baixa e executa imagem
docker pull ubuntu # baixa a imagem
```

o docker procura pelos repositorios das imagens dentro do docker hub

o docker também possui imagens de sistemas operacionais. ex: ubuntu

ao rodar “docker run ubuntu” não aconteceu nada.

Perguntas pra proxima aula:

- pq o container não rodou?
- o que o docker run fez por baixo dos panos?

### Fluxo da criação de containers

```bash
docker ps # mostra quais containers estão em execução
docker container ls # igual ao docker ps
docker ps -a # mostra todos os containers, mesmo que não estejam em execução
```

a partir do momento que um container é executado, ele executa um comando e finaliza (visivel no docker ps -a que a imagem ubuntu executa o comando bash)

para que o container fique em execução, deve ter pelo menos 1 processo dentro dele.

```bash
docker run --help # docker [OPTIONS] run [COMMAND]
docker run ubuntu sleep 1d # agora o container vai ficar em execução
docker run -it ubuntu bash # cria um container interativo que fica em execução. No momento em que eu sair do terminal, ele mata o container

```

quando executamos uma imagem, o docker mostra um “digest”, que é a validação em hash de que a imagem baixada é a mesma que procuramos

### Outros comandos importantes

- `docker stop <container>`: para a execução de um container
- docker start <container>: executa um container que ja foi criado
- `docker rm <container>`: remove um container # —force para remover um container em execução
- `docker images`: lista as imagens disponíveis no host
- `docker rmi <image>`: remove uma imagem do host
- `docker exec -it <container> <command>`: executa um comando dentro do container em execução
- `docker logs <container>`: mostra os logs do container em execução
- `docker inspect <container>`: retorna informações detalhadas sobre o container

docker exec -it <ubuntu_container_id> bash: -i é para ser interativo. -t para entrar no terminal padrão do container. foi usado um contianer linux com o comando sleep 1d já rodando

docker exec serve para executar um comando em um container já ativo

### Mapeando portas

vamos utilizar uma aplicação web para poder ver uma saída no navegador

Dockersamples - organização que disponibila exemplos

Imagens não oficiais tem o nome de <usuario>/<imagem>

```bash
docker run -d dockersamples/static-site # vai executar o container e deixar em background. -d de dettached
docker ps # mostra portas do container
docker run -d -P dockersamples/static-site # flag -P faz um mapeamento automatico de portas, visivel no docker ps
docker port <container> # mostra o mapeamento de portas container -> host
docker run -d -p 8080:80 dockersamples/static-site # flag -p faz o mapeamento de portas host:container
```

## Criando e compreendendo imagens

### Entendendo imagens

Conjunto de camadas que juntas formam uma imagem. cada camada é independente e possui seu ID.

```bash
docker images # mostra as imagens que estão no meu pc
docker inspect <image_id> # mostra infos da imagem
docker history <image_id> # mostra as camadas que compoem a imagem
```

num docker pull, o docker baixa as camadas, que podem ser utilizadas em outras imagens, assim evitando duplicidade

a imagem é read only

quando escrevemos dentro de um container, é porque o container pega a imagem e adiciona uma nova camada de read/write. Por isso a informação é perdida quando o container é apagado

![Untitled](Docker%20f2181766190c4e2aadc0e4d5cc7d9b8f/Untitled%202.png)

### Criando a primeira imagem

O primeiro passo é criar um Dockerfile

No exemplo usamos um exemplo de app node do link [https://github.com/danielartine/alura-docker/blob/aula-3/app-exemplo.zip?raw=true](https://github.com/danielartine/alura-docker/blob/aula-3/app-exemplo.zip?raw=true)

```docker
# utilizando a imagem do node na versão 14
FROM node:14

# diretorio de trabalho padrão
WORKDIR /app-node

# copiar o diretorio atual (onde está o dockerfile) para o workdir do container
COPY . .

# executar o comando npm install na criação da imagem
RUN npm install

# quando o container for executado, rodar o npm start
ENTRYPOINT npm start
```

```bash
docker build -t arnold/app-node:1.0 .
# cria uma imagem a partir do dockerfile. -t é para colocar o nome da imagem. : a versão. o ponto referencia o diretorio atual

# necessário ser na porta 3000 por causa do servidor node express
docker run -it -p 8081:3000 arnold/app-node:1.0
```

Agora podemos acessar o navegador [localhost:3000](http://localhost:3000) e veremos a aplicação online

### Incrementando a imagem

ao criar a imagem, podemos **deixar exposto a porta que o container estará ouvindo 1.1**

```docker
# utilizando a imagem do node na versão 14
FROM node:14

# diretorio de trabalho padrão
WORKDIR /app-node

# a aplicação vai estar exposta na porta 3000
EXPOSE 3000

# copiar o diretorio atual (onde está o dockerfile) para o workdir do container
COPY . .

# executar o comando npm install na criação da imagem
RUN npm install

# quando o container for executado, rodar o npm start
ENTRYPOINT npm start
```

```bash
docker build -t arnold/app-node:1.1 .
```

Agora, ao rodar um container com essa aplicação sem o mapeamento de portas, o docker ps mostra que a porta 3000 estará exposta, que há algo aberto naquela porta

********************passando a porta como parâmetro 1.2********************

```docker
# utilizando a imagem do node na versão 14
FROM node:14

# diretorio de trabalho padrão
WORKDIR /app-node

# vamos pegar PORT como uma variável no parâmetro
ARG PORT_BUILD=6000

ENV PORT_BUILD=$6000

# a aplicação vai estar exposta na porta 3000
EXPOSE $PORT_BUILD

# copiar o diretorio atual (onde está o dockerfile) para o workdir do container
COPY . .

# executar o comando npm install na criação da imagem
RUN npm install

# quando o container for executado, rodar o npm start
ENTRYPOINT npm start
```

PROBLEMA: ARG só funciona no contexto de criação, ENV é a variável ambiente que será passada para dentro do container

## Persistindo dados

### Problema de persistir dados

```bash
docker ps -s # mostra o size do container (virtual é o tamanho da imagem + os dados dentro do container)
```

toda vez que deletamos um container, os dados são apagados

rodar a mesma imagem não significa que os dados estarão lá.

a solução é usar Volumes:

- bind mount: ponte entre o file system do host e do container
- volume
- tmpfs: temporário

### Utilizando bind mounts

```bash
docker run -it -v <caminho_host>:<container_dir> ubuntu
docker run -it --mount type=bind, source=<source_path>,target=/app ubuntu bash
```

a flag -v faz o mapeamento de um diretório do host e o container. O dir do container será criado

agora todo container criado com esses caminhos terão os dados pesistidos

obs: o docker recomenda a utilização da flag —mount ao invés de —v

### Utilizando volumes

mais recomendado em ambientes produtivos pq a area dos volumes dentro do host é gerenciada pelo docker (mais segurança)

```bash
docker volume create <name> # cria um volume com o driver local do host
docker volume ls # lista os volumes
docker run -it -v <volume_name>:/app ubuntu bash
docker run -it --mount source=<volume_name>,target=/app ubuntu bash # caso o volume não exista, ele é criado automaticamente
```

onde está o volume?

os volumes nomeados ficam dentro de /var/lib/docker/volumes sendo necessário acessar com o usuario root “sudo su”

You can find WSL2 volumes **under a hidden network share**. Open Windows Explorer, and type \\wsl$ into the location bar. Hit enter, and it should display your WSL volumes, including the ones for Docker for Windows

\\wsl.localhost\docker-desktop-data\data\docker\volumes

nesses caminhos, é o docker que gerencia

### Utilizando tmpfs

Obs: só funciona no host linux

```bash
docker run -it --tmpfs=/app ubuntu bash
docker run -it --mount type=tmpfs, destination=/app ubuntu bash
```

isso signifca que a pasta é temporária e é criado na memória do host. Quando criamos um novo container, não há como pegar os dados do tmpfs do anterior

a utilidade é que os dados não são escritos na camada Read/Write do container, e sim diretamente na memória do host

## Comunicação através de redes

### Conhecendo a rede bridge

```bash
docker inspect <container>
```

o comando retorna varias configurações do container, a que interessa aqui é a “Networks”

rede “bridge” é automaticamente configurada pelo docker, padrão quando criamos um container sem especificar network

```bash
docker network ls # retorna as redes
# o docker possui três redes padrão: none, host e bridge
```

caso os containers estejam na mesma rede, eu devo conseguir fazer a comunicação via IP entre eles. O comando ping com o ipv4 pode fazer o teste.

Problema: Os containers podem ser recriados, reiniciados, isso não garante que o container tenha sempre o mesmo IP. é mais interessante uma conexão via DNS ou hostname

### Criando uma rede bridge

Ao dar docker ps, o nome do container é uma informação mais estável do que o IP

```bash
docker run -it --name ubuntu1 ubuntu bash
```

Para que seja possível a comunicação entre containers via hostname (nome do container), é necessário criar uma nova rede

```bash
docker network create --driver bridge minha-bridge
docker run -it --name ubuntu1 --network minha-bridge ubuntu bash
docker run -it --name ubuntu2 --network minha-bridge ubuntu sleep 1d
# docker inspect agora vai mostrar "minha-bridge" na parte de networks
```

agora quando criarmos dois containers dentro da mesma network bridge, eles poderão se comunicar via hostname

entrar no terminal de ubuntu1 e rodar ping ubuntu2

### Redes none e host

- none: o container não terá nenhuma operação de rede.
- host: utiliza a mesma interface do host. Lembrando o exemplo do node, não precisaria ter o mapeamento de portas, pq seria a porta diretamente do host

### Comunicando aplicação e mongo

## Coordenando containers

### Conhecendo o docker compose

windows ja tem.

linux necessário instalar 

### Definindo os serviços

arquivo docker-compose.yaml

```yaml
version: "3.9"
services:
  mongodb:
    image: mongo:4.4.6
    container_name: meu-mongo
    networks: 
      - compose-bridge
  
  alurabooks:
    image: aluradocker/alura-books:1.0
    container_name: alurabooks
    networks:
      - compose-bridge
    ports:
      - 3000:3000

networks:
  compose-bridge:
    driver: bridge
```

```bash
docker-compose up # rodar no mesmo diretorio do arquivo docker-compose.yaml
docker-compose down # finaliza e remove os containers e networks criados
```

### Complementando o Compose

podemos definir algumas instruções no compose, por exemplo, dependencia entre serviços. Essa dependencia espera o container estar pronto, e não a aplicação necessariamente on

```yaml
version: "3.9"
services:
  mongodb:
    image: mongo:4.4.6
    container_name: meu-mongo
    networks: 
      - compose-bridge
  
  alurabooks:
    image: aluradocker/alura-books:1.0
    container_name: alurabooks
    networks:
      - compose-bridge
    ports:
      - 3000:3000
		depends_on:
			- mongodb

networks:
  compose-bridge:
    driver: bridge
```