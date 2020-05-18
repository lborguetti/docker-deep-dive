
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

* images vs containers

    Fundamentalmente, um contêiner não passa de um processo em execução, com alguns recursos adicionais de encapsulamento aplicados a ele para mantê-lo isolado do seu S.O e de outros contêineres.

    Um dos aspectos mais importantes do isolamento de contêiner é que cada contêiner interage com seu próprio sistema de arquivos privado; esse sistema de arquivos é fornecido por uma imagem do Docker.

    Uma imagem inclui tudo o necessário para executar um aplicativo - o código ou binário, runtimes, dependências e outros objetos do sistema de arquivos necessários.

    * abra dois terminais: (terminal 1) e (terminal 2)

    * no terminal 1 digite:`docker images` e observe as imagens locais.

    * no terminal 1 digite: `docker pull debian`

    * no terminal 1 digite:`docker images` e observe a lista de imagens locais.

    * no terminal 1 digite: `docker run debian bash`

    * no terminal 2 digite: `docker ps` e observe a lista de containers.

    * no terminal 2 digite: `docker ps --all` e observe a lista de containers.

    * no terminal 1 digite: `docker run --tty --interactive debian bash`

    * no terminal 2 digite: `docker ps` e observe a lista de containers.

    * no terminal 1 digite `exit` ou `Control+D`

    * no terminal 2 digite: `docker ps` e observe a lista de containers.

    * no terminal 2 digite: `docker ps --all` e observe a lista de containers.

    * no terminal 2 digite: `docker rm $(docker ps --all --quiet)`

    * no terminal 2 digite: `docker ps --all` e observe a lista de containers.

* prática:

    1. executar um container e adicionar um processo em execução em primeiro plano

    `docker pull debian`

    `docker run --name baleia debian /bin/bash -c "while true; do echo baleia: \$(id); sleep 1; done"`

    2. pare o container em execucação

    `docker ps`

    `docker stop`

    6. remova o container

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

    * no terminal 1 digite (container 1): `docker run --name container1 --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (container 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (container 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (container 2): `docker run --name container2 --link container1 --rm --tty --interactive debian /bin/bash`

    * no terminal 2 digite (container 2): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (container 2): `ip route show` observe o endereço do gateway padrão.

    * no terminal 3 digite (host): `docker network ls` observe as redes.

    * no terminal 3 digite (host): `brctl show` observe as interfaces.

    * no terminal 3 digite (host): `ip address show` e observe as interfaces /endereços de rede.

    * no terminal 3 digite (host): `ip address show docker0` e observe o endereço de rede.

    * no terminal 3 digite (host): `sudo tcpdump -i docker0 -Nnnl`

    * no terminal 2 digite (container 2): `ping [endereço do container 1]`

    * no terminal 3 (host): observe o resultado.

    * no terminal 1 digite (container 1): `cat /etc/hosts`

    * no terminal 1 digite (container 1): `ping container2`

    * no terminal 2 digite (container 2): `cat /etc/hosts`

    * no terminal 2 digite (container 2): `ping container1`

    * no terminal 3 (host): observe o resultado.

* **bridge User-defined** fornecem resolução automática de DNS, compartilham variáveis de ambiente e melhor insolamento entre contêineres. Os contêineres também podem ser conectados e desconectados de redes em tempo de execução.

    * abra três terminais: (terminal 1), (terminal 2) e (terminal 3)

    * no terminal 3 digite (host): `docker network create neotech`

    * no terminal 3 digite (host): `docker network ls`

    * no terminal 3 digite (host): `docker network inspect neotech`

    * no terminal 3 digite (host): `brctl show` observe as interfaces.

    * no terminal 3 digite (host): `ip address show br-NNNN` e observe o endereço de rede (trocar br-NNNN pelo nome correto da interface).

    * no terminal 3 digite (host): `sudo tcpdump -i br-NNNN -Nnnl` (trocar br-NNNN pelo nome correto da interface).

    * no terminal 1 digite (container 1): `docker run --name container1 --network neotech --rm --tty --interactive debian /bin/bash`

    * no terminal 1 digite (container 1): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 1 digite (container 1): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (container 2): `docker run --name container2 --network neotech --rm --tty --interactive debian /bin/bash`

    * no terminal 2 digite (container 2): `ip address show` observe as interfaces e anote o endereço de rede.

    * no terminal 2 digite (container 2): `ip route show` observe o endereço do gateway padrão.

    * no terminal 2 digite (container 2): `ping [endereço do container 1]`

    * no terminal 3 (host): observe o resultado.

    * no terminal 1 digite (container 1): `cat /etc/hosts`

    * no terminal 1 digite (container 1): `ping container2`

    * no terminal 2 digite (container 2): `cat /etc/hosts`

    * no terminal 2 digite (container 2): `ping container1`

    * no terminal 3 (host): observe o resultado.
