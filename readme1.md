Pessoal trabalhei um pouco nos arquivos pra gerar os testes. Vi que gerar os testes de deploy soh nos comandos estava ficando dificil e comecei a montar um compose pra tentar facilitar.
Para isso, iniciei alterando apenas duas linhas do docker-compose da aplicação adicionando eles numa rede (nomeada "backend"), e adicionando uma variável ${LOCAL_PORT} pra sser passada pelo terminal pra setar o valor da porta do host. Se deixar o valor da porta do host estático como está da conflito em gerar mais de uma vez a app (pois usaria o mesmo valor do port=8000). 
Adicionei entao um arquivo além do compose original da aplicação chamado docker-compose.override.yml, na mesma pasta, para adicionar os serviços da nossa solução. Quando estão na mesma pasta e rodamos um docker-compose up -d, o docker  faz tipo um merge substituindo configurações do docker-composer.yml pelo docker-compose.override, além disso podemos tb iniciar novos serviços no docker-compose.override.yml . no override entao coloquei nossos serviços nos namespaces de rede dos serv. da aplicação
Defini esta rede no docker-compose.override.yml colocando um subnet tipo 172.20.0.0/16, pq isso? pq não temos como descobrir o endereço do Coletor para poder endereçar o fprobe, se colocarmos numa rede "default" o docker da um endereço qualquer pra ele que fica difícil descobrir, e daí definindo um subnet sabemos que todos os containers (aplicaçao e solução) estarão na mesma subrede.  Assim posso definir o valor do subnet e o endereço de IP do coletor pela linha de comando passando argumentos da seguinte forma:

```
LOCAL_PORT=8080 SUBNET=192.20.0.0/16  COLLECTOR_IP=192.20.0.2 docker-compose up -d
```
O problema é que no modo docker-compose aparece um erro na criação dos conteiners q tem o fprobe.  na parte da passagem das ENV variables. Dai não incia o serviço de forma adequada. Tentei de várias formas solucionar isso primeiro definindo as variáveis de ambiente no compose:

```
 services:
   fprobea: imagem: aqualtune/netflow_data_export:latest
   depends_on:      
      - collector          
   environment:       
     - INTERFACE=eth0       
     - FLOW_COLLECTOR= "${COLLECTOR_IP}"      
     - COLLECTOR_PORT=2055    
      networks:          
        network_mode: service:wordpress
```

não funcionou, tb mudei a sintaxe de "- INTERFACE=" para " INTERFACE: eth0", etc mas tb não funcionou...o mais perto que cheguei foi salvando as variáveis num .env file e lendo elas na criação do serviço: 

```
services:
  fprobea:  imagem: aqualtune/netflow_data_export:latest 
  depends_on: 
     - collector     
  env_file 
     - fprobe.env     
  networks:
    network_mode: service:wordpres 
``` 

esse método funciona quando rodo um "docker run"
```
docker run -itd --name probea --env-file=fprobe.env aqualtune/netflow_data_export:latest
```
mas nao funciona no docker compose. Não descobri o porque desse erro

Enfim  podem baixar os arquivos do git e tentar soluções possíveis. quando acharmos a solução pra esse problema podemos lançar varios serviços alterando o endereço de IP do coletor a subrede o host port e tb mudando o nome do projeto tipo:
```
LOCAL_PORT=8000 COLLECTOR_IP=198.16.238.10  SUBNET=198.16.238.0/24  docker-compose  --project-name=t1 up -d
```
e o script que chama isso pode ser por exemplo:
```
teste.sh:#!/bin/sh

echo "starting Application and Solution deploy time test $1x ... "
i=1
while [ $i -le $1 ]do 
echo -e "--------------------------------------------------\n"	 
echo -e  "\tStart deploy number:$i...\n " 
echo -e "--------------------------------------------------\n"       
/usr/bin/time -o $2 -a  -f    "%E,%e"  LOCAL_PORT=8000 COLLECTOR_IP=198.16.238.10  SUBNET=198.16.238.0/24  docker-compose --project-name=t1 up -d
```
depois de ver como variar o LOCAL_PORT o  COLLECTOR_IP e o SUBNET, podemos lançar o teste n vezes:

```
bash teste.sh 8 nomedoarquivo.csv
```