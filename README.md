
Docker "Deep Dive"
===

```

                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""\___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
         \    \        __/
          \____\______/

          |          |
       __ |  __   __ | _  __   _
      /  \| /  \ /   |/  / _\ |
      \__/| \__/ \__ |\_ \__  |
```

## Docker Life Cycle

```
Docker Hub <---> Pull/Push <---> Docker Engine (daemon)
                                 Docker Run
                                 Docker Stop
                                 Docker Rm
```

* imagens vs contêiners

    Fundamentalmente, um contêiner não passa de um processo em execução, com alguns recursos adicionais de encapsulamento aplicados a ele para mantê-lo isolado do seu S.O e de outros contêineres.

    Um dos aspectos mais importantes do isolamento de contêiner é que cada contêiner interage com seu próprio sistema de arquivos privado; esse sistema de arquivos é fornecido por uma imagem do Docker.

    Uma imagem inclui tudo o necessário para executar um aplicativo - o código ou binário, runtimes, dependências e outros objetos do sistema de arquivos necessários.

    * abra dois terminais: (terminal 1) e (terminal 2)

    * no terminal 1 digite (host):`docker images` e observe as imagens locais.

    * no terminal 1 digite (host): `docker pull debian`

    * no terminal 1 digite (host):`docker images` e observe a lista de imagens locais.

    * no terminal 1 digite (host): `docker run debian bash`

    * no terminal 2 digite (host): `docker ps` e observe a lista de contêiners.

    * no terminal 2 digite (host): `docker ps --all` e observe a lista de contêiners.

    * no terminal 1 digite (host): `docker run --tty --interactive debian bash`

    * no terminal 2 digite (host): `docker ps` e observe a lista de contêiners.

    * no terminal 1 digite (contêiner) `exit` ou `Control+D`

    * no terminal 2 digite (host): `docker ps` e observe a lista de contêiners.

    * no terminal 2 digite (host): `docker ps --all` e observe a lista de contêiners.

    * no terminal 2 digite (host): `docker rm $(docker ps --all --quiet)`

    * no terminal 2 digite (host): `docker ps --all` e observe a lista de contêiners.

* prática:

    1. executar um contêiner e adicionar um processo em execução em primeiro plano

    `docker pull debian`

    `docker run --name=baleia debian /bin/bash -c "while true; do echo baleia: \$(id); sleep 1; done"`

    2. pare o contêiner em execucação

    `docker ps`

    `docker stop baleia`

    3. remova o contêiner

    `docker ps --all`

    `docker rm baleia`

    4. execute novamente o container

    `docker run --name=baleia debian /bin/bash -c "while true; do echo baleia: \$(id); sleep 1; done"`

    5. pare o contêiner em execucação (usando docker kill)

    `docker ps`

    `docker kill baleia`

    6. remova o contêiner

    `docker ps --all`

    `docker rm baleia`

## Dig a Docker Image

* abra um terminal

    * digite (host): `docker pull hello-world`

    * digite (host): `docker save hello-world > hello-world.tar`

    * digite (host): `mkdir --parents hello-world`

    * digite (host): `tar --directory hello-world --extract --file hello-world.tar`

    * digite (host): `tree hello-world` ou `find hello-world`

    * digite (host): `jq < hello-world/manifest.json`

    * digite (host): `jq < hello-world/c8f25f0787b14b09638a294eab0b30b81c82295b8aa5e1df3fb36796fa6d0179/json`

    * digite (host): `tar --extract --file hello-world/c8f25f0787b14b09638a294eab0b30b81c82295b8aa5e1df3fb36796fa6d0179/layer.tar`

    * digite (host): `./helo`

*  prática:

    https://github.com/nginxinc/docker-nginx/blob/594ce7a8bc26c85af88495ac94d5cd0096b306f7/mainline/buster/Dockerfile

    1. exportar a imagem do nginx e verificar seu conteúdo

    `docker pull nginx:1.17.10`

    `docker save nginx:1.17.10 > nginx.tar`

    `mkdir --parents nginx`

    `tar --directory nginx --extract --file nginx.tar`

    2. observar as "layers"

    `tree nginx`

    `jq < nginx/manifest.json`

    `jq '.[0].Config' < nginx/manifest.json`

    `docker images --no-trunc nginx:1.17.10`

    3. descompatando algumas "layers"

    `jq '.[0].Layers' < nginx/manifest.json`

    `mkdir --parents layer1 layer2 layer3`

    `tar --directory layer1 --extract --file nginx/4b06198a66029c40c12e10d43cc47901e788295c906306fb0f44ea5e5b300dc9/layer.tar`

    `tar --directory layer2 --extract --file nginx/ad83e86a3b51398162fa7838af4f42c392036c53b3795267e9f36fd322f2588d/layer.tar`

    `tar --directory layer3 --extract --file nginx/e581864dfa52cf8881e598876fe4bcebb373db22a2a9b6f6d6fccce623239b89/layer.tar`

    5. navegando no sistema de arquivos das "layers"

    Layer1

    `du -hs layer1`

    `ls layer1`

    `ls layer1/etc`

    `cat layer1/etc/debian_version`

    `find layer1 -name nginx`

    Layer2

    `du -hs layer2`

    `ls layer2`

    `cat layer2/etc/debian_version`

    `find layer2 -name nginx`

    `ldd layer2/usr/sbin/nginx`

    `./layer2/usr/sbin/nginx`

    Layer3

    `du -hs layer3`

    `ls layer3`

    `ls layer3/usr/sbin/nginx`

    `find layer3`


## Docker Network

O subsistema de rede do Docker é conectável, usando drivers. Vários drivers existem por padrão e fornecem funcionalidades básicas de rede.

Abra um terminal e digite: `docker info` observe os drivers de rede e demais informações.

* **bridge [legacy]** é o driver de rede padrão. Se você não especificar um driver, este é o tipo de rede que você está criando.

    * abra três terminais: (terminal 1), (terminal 2) e (terminal 3)

    * no terminal 1 digite (host): `docker run --name=conteiner1 --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (contêiner 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (contêiner 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (host): `docker run --name=conteiner2 --link=conteiner1 --rm --tty --interactive debian /bin/bash`

    * no terminal 2 digite (contêiner 2): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (contêiner 2): `ip route show` observe o endereço do gateway padrão.

    * no terminal 3 digite (host): `docker network ls` observe as redes.

    * no terminal 3 digite (host): `brctl show` observe as interfaces.

    * no terminal 3 digite (host): `ip address show` e observe as interfaces /endereços de rede.

    * no terminal 3 digite (host): `ip address show docker0` e observe o endereço de rede.

    * no terminal 3 digite (host): `sudo tcpdump --interface docker0 -Nnnl`

    * no terminal 2 digite (contêiner 2): `ping [endereço do contêiner 1]`

    * no terminal 3 (host): observe o resultado.

    * no terminal 1 digite (contêiner 1): `cat /etc/hosts`

    * no terminal 1 digite (contêiner 1): `ping conteiner2`

    * no terminal 2 digite (contêiner 2): `cat /etc/hosts`

    * no terminal 2 digite (contêiner 2): `ping conteiner1`

    * no terminal 3 (host): observe o resultado.

* **bridge [user-defined]** fornecem resolução automática de DNS, compartilham variáveis de ambiente e melhor insolamento entre contêineres. Os contêineres também podem ser conectados e desconectados de redes em tempo de execução.

    * abra três terminais: (terminal 1), (terminal 2) e (terminal 3)

    * no terminal 3 digite (host): `docker network create neotech`

    * no terminal 3 digite (host): `docker network ls`

    * no terminal 3 digite (host): `docker network inspect neotech`

    * no terminal 3 digite (host): `brctl show` observe as interfaces.

    * no terminal 3 digite (host): `ip address show br-NNNN` e observe o endereço de rede (trocar br-NNNN pelo nome correto da interface).

    * no terminal 3 digite (host): `sudo tcpdump --interface br-NNNN -Nnnl` (trocar br-NNNN pelo nome correto da interface).

    * no terminal 1 digite (host): `docker run --name=conteiner1 --network=neotech --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (contêiner 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (contêiner 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (host): `docker run --name=conteiner2 --network=neotech --rm --tty --interactive debian /bin/bash`

    * no terminal 2 digite (contêiner 2): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (contêiner 2): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (contêiner 2): `ping [endereço do contêiner 1]`

    * no terminal 3 (host): observe o resultado.

    * no terminal 1 digite (contêiner 1): `cat /etc/hosts`

    * no terminal 1 digite (contêiner 1): `ping conteiner2`

    * no terminal 2 digite (contêiner 2): `cat /etc/hosts`

    * no terminal 2 digite (contêiner 2): `ping conteiner1`

    * no terminal 3 (host): observe o resultado.

* prática:

    1. criar dois contêiners (um servidor web e outro client) conectados em uma rede exclusiva (docker network).

    `docker network create tarrafa`

    `docker run --name=webserver --network=tarrafa --detach nginx:1.17.10`

    `docker run --name=webclient --network=tarrafa --rm --tty --interactive debian /bin/bash`

    `apt-get --yes update && apt-get --yes install curl`

    `curl http://webserver`

    2. em outro terminal desconectar o contêiner webclient da rede criada

    `docker network disconnect tarrafa webclient`

    3. retorno para o terminal onde o contêiner webclient está em execucação

    `curl http://webserver`

    `ip address show`

    4. em outro terminal conectar o contêiner webclient na rede criada

    `docker network connect tarrafa webclient`

    5. retorno para o terminal onde o contêiner webclient está em execucação

    `ip address show`

    `curl http://webserver`

    6. verifique os detalhes das redes dos contêiners

    `docker network inspect tarrafa  -f "{{json .Containers }}" | jq`

    `docker inspect webserver -f "{{json .NetworkSettings.Networks }}" | jq`

    `docker inspect webclient -f "{{json .NetworkSettings.Networks }}" | jq`

    7. remover os contêiners e a rede criada

    `docker ps`

    `docker stop webserver`

    `docker rm webserver`

    `docker network rm tarrafa`

*  **host** pode ser útil para otimizar o desempenho. Se você usar o modo de rede host para um contêiner, a stack de rede desse contêiner não será isolada do host. O driver de rede do host funciona apenas em hosts Linux.

    * abra dois terminais: (terminal 1) e (terminal 2)

    * no terminal 1 digite (host): `docker run --name=conteiner1 --network=host --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (contêiner 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (contêiner 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (host): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (host): `ip route show` observe o endereço do gateway padrão.

* prática:

    1. criar um contêiner (servidor web) utilizando a rede do host.

    `docker run --name=webserver --network=host --detach nginx:1.17.10`

    `docker exec --tty --interactive webserver /bin/bash`

    `apt-get --yes update && apt-get --yes install iproute2`

    `ip address show`

    2. abrir no navegador:

    `http://127.0.0.1`

    3. remover o contêiner

    `docker ps`

    `docker stop webserver`

    `docker rm webserver`

Por padrão, quando você cria um contêiner, ele não publica nenhuma de suas portas no "mundo externo". Para disponibilizar uma porta para serviços fora do Docker ou para contêineres que não estão conectados, use a opção `--publish` ou `-p`. Dessa forma o Docker cria uma regra de firewall que mapeia uma porta do contêiner para uma porta no host. Obs: Não é um "driver" de rede do Docker.

* abra três terminais: (terminal 1), (terminal 2) e (terminal 3)

    * no terminal 1 digite (host):`docker run --name=nginx --publish=8080:80 --rm nginx:1.17.10`

    * no terminal 2 digite (host): `docker port nginx` ou `docker ps`

    * no terminal 3 digite (host): `sudo tcpdump --interface docker0 -Nnnl`

    * no terminal 2 digite (host): `docker inspect -f "{{ .NetworkSettings.IPAddress }}" nginx` anote o endereço do contêiner.

    * no terminal 2 digite (host): `curl http://ip-conteiner:80` (trocar ip-conteiner pelo endereço do contêiner)

    * no terminal 3 (host): observe o resultado.

    * no terminal 2 digite (host): `sudo iptables --table nat --list DOCKER --numeric --verbose`

    * no terminal 2 digite (host): `ip address show $(ip route get 8.8.8.8 | awk '{print $5}')` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (host): `curl http://ip-host:8080` (trocar ip-host pelo endereço da interface)

    * no terminal 3 (host): observe o resultado.
