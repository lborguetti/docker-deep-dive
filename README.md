
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

    * no terminal 1 digite:`docker images` e observe as imagens locais.

    * no terminal 1 digite: `docker pull debian`

    * no terminal 1 digite:`docker images` e observe a lista de imagens locais.

    * no terminal 1 digite: `docker run debian bash`

    * no terminal 2 digite: `docker ps` e observe a lista de contêiners.

    * no terminal 2 digite: `docker ps --all` e observe a lista de contêiners.

    * no terminal 1 digite: `docker run --tty --interactive debian bash`

    * no terminal 2 digite: `docker ps` e observe a lista de contêiners.

    * no terminal 1 digite `exit` ou `Control+D`

    * no terminal 2 digite: `docker ps` e observe a lista de contêiners.

    * no terminal 2 digite: `docker ps --all` e observe a lista de contêiners.

    * no terminal 2 digite: `docker rm $(docker ps --all --quiet)`

    * no terminal 2 digite: `docker ps --all` e observe a lista de contêiners.

* prática:

    1. executar um contêiner e adicionar um processo em execução em primeiro plano

    `docker pull debian`

    `docker run --name baleia debian /bin/bash -c "while true; do echo baleia: \$(id); sleep 1; done"`

    2. pare o contêiner em execucação

    `docker ps`

    `docker stop`

    6. remova o contêiner

    `docker ps --all`

    `docker rm`

## Dig a Docker Image

* abra um terminal

    * digite: `docker pull hello-world`

    * digite: `docker save hello-world > hello-world.tar`

    * digite: `mkdir -p hello-world`

    * digite: `tar --directory hello-world --extract --file hello-world.tar`

    * digite: `tree hello-world` ou `find hello-world`

    * digite: `jq < hello-world/manifest.json`

    * digite: `jq < hello-world/c8f25f0787b14b09638a294eab0b30b81c82295b8aa5e1df3fb36796fa6d0179/json`

    * digite: `tar --extract --file hello-world/c8f25f0787b14b09638a294eab0b30b81c82295b8aa5e1df3fb36796fa6d0179/layer.tar`

    * digite: `./helo`

*  prática:

    1. exportar a imagem do nginx e verificar seu conteúdo

    `docker pull nginx`

    `docker save nginx > nginx.tar`

    `mkdir -p nginx`

    `tar --directory nginx --extract --file nginx.tar`

    2. observar as "layers"

    `tree nginx`

    `jq < nginx/manifest.json`

## Docker Network

O subsistema de rede do Docker é conectável, usando drivers. Vários drivers existem por padrão e fornecem funcionalidades básicas de rede.

Abra um terminal e digite: `docker info` observe os drivers de rede e demais informações.

* **bridge [legacy]** é o driver de rede padrão. Se você não especificar um driver, este é o tipo de rede que você está criando.

    * abra três terminais: (terminal 1), (terminal 2) e (terminal 3)

    * no terminal 1 digite (contêiner 1): `docker run --name conteiner1 --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (contêiner 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (contêiner 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (contêiner 2): `docker run --name conteiner2 --link conteiner1 --rm --tty --interactive debian /bin/bash`

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

    * no terminal 1 digite (contêiner 1): `docker run --name conteiner1 --network neotech --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (contêiner 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (contêiner 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (contêiner 2): `docker run --name conteiner2 --network neotech --rm --tty --interactive debian /bin/bash`

    * no terminal 2 digite (contêiner 2): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (contêiner 2): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (contêiner 2): `ping [endereço do contêiner 1]`

    * no terminal 3 (host): observe o resultado.

    * no terminal 1 digite (contêiner 1): `cat /etc/hosts`

    * no terminal 1 digite (contêiner 1): `ping conteiner2`

    * no terminal 2 digite (contêiner 2): `cat /etc/hosts`

    * no terminal 2 digite (contêiner 2): `ping conteiner1`

    * no terminal 3 (host): observe o resultado.

*  **host** se você usar o modo de rede host para um contêiner, a stack de rede desse contêiner não será isolada do host Docker. A rede em modo host pode ser útil para otimizar o desempenho. O driver de rede do host funciona apenas em hosts Linux.

    * abra dois terminais: (terminal 1) e (terminal 2)

    * no terminal 1 digite (contêiner 1): `docker run --name conteiner1 --rm --tty --interactive --net=host debian /bin/bash`

    * no terminal 1 digite (contêiner 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (contêiner 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (host): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (host): `ip route show` observe o endereço do gateway padrão.

* **--publish**: por padrão, quando você cria um contêiner, ele não publica nenhuma de suas portas no "mundo externo". Para disponibilizar uma porta para serviços fora do Docker ou para contêineres que não estão conectados, use a opção `--publish` ou `-p`. Dessa forma o Docker cria uma regra de firewall que mapeia uma porta do contêiner para uma porta no host.

    * abra três terminais: (terminal 1), (terminal 2) e (terminal 3)

    * no terminal 1 digite (host):`docker run --rm --name nginx --publish 8080:80 nginx`

    * no terminal 2 digite (host): `docker port nginx` ou `docker ps`

    * no terminal 3 digite (host): `sudo tcpdump --interface docker0 -Nnnl`

    * no terminal 2 digite (host): `docker inspect -f "{{ .NetworkSettings.IPAddress }}" nginx` anote o endereço do contêiner.

    * no terminal 2 digite (host): `curl http://ip-conteiner:80` (trocar ip-conteiner pelo endereço do contêiner)

    * no terminal 3 (host): observe o resultado.

    * no terminal 2 digite (host): `sudo iptables --table nat --list DOCKER --numeric --verbose`

    * no terminal 2 digite (host): `ip address show $(ip route get 8.8.8.8 | awk '{print $5}')` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (host): `curl http://ip-host:8080` (trocar ip-host pelo endereço da interface)

    * no terminal 3 (host): observe o resultado.
