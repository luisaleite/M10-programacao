Backstage é uma plataforma de código aberto para construção de portais de desenvolvedores, foi criado pelo spotify. Ela oferece uma interface unificada para visualizar e interagir com todos os serviços e infraestruturas de uma organização. Com o Backstage, as equipes podem catalogar, descobrir e colaborar em serviços, além de automatizar processos de desenvolvimento e operações.

Objetivo:

- criar o um aplicativo backstage
- construir uma imagem no docker

Passo a Passo:

- Criação do aplicativo bacstage seguindo o passo a passo desse link: https://backstage.io/docs/getting-started/
    - Usei esse comando:
    
    ```jsx
    npx @backstage/create-app@latest
    ```
    
    e coloquei o nome “luisa” no aplicativo.
    
    ![Untitled](imagens\Untitled (1).png)
    
    - Depois usei esses dois comando:
    
    ```jsx
    cd luisa 
    yarn dev
    ```
    
    Ao final digitando no navegador “[http://localhost:3000](http://localhost:3000/)” aparece o nosso aplicativo backstage.
    
    ![Untitled](imagens\Untitled (2).png)
    

O artigo **Construindo uma imagem Docker** começa com:

- Com a construção de host:

Estes foram os comandos usados no diretorio do aplicativo:

yarn install --frozen-lockfile

yarn tsc

yarn build:backend --config ../../app-config.yaml

![Untitled](imagens\Untitled (3).png)

No caminho “packages/backend/Dockerfile” encontra esse codigo: 

```docker
FROM node:18-bookworm-slim

# Install isolate-vm dependencies, these are needed by the @backstage/plugin-scaffolder-backend.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential && \
    yarn config set python /usr/bin/python3

# Install sqlite3 dependencies. You can skip this if you don't use sqlite3 in the image,
# in which case you should also move better-sqlite3 to "devDependencies" in package.json.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev

# From here on we use the least-privileged `node` user to run the backend.
USER node

# This should create the app dir as `node`.
# If it is instead created as `root` then the `tar` command below will
# fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV production

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
```

Substitua ENV NODE_ENV production por ENV NODE_ENV development

Depois use esses dois comandos: 

docker image build . -f packages/backend/Dockerfile --tag backstage

docker run -it -p 7007:7007 backstage

![Untitled](imagens\Untitled (4).png)

![Untitled](imagens\Untitled (5).png)

Ao final depois de roda esses dois comando, acesso no navegador “localhost:7007”.

![Untitled](imagens\Untitled (6).png)