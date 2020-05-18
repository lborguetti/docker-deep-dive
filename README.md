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
