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