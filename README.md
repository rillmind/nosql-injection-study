<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="200" alt="Nest Logo" /></a>
</p>

[circleci-image]: https://img.shields.io/circleci/build/github/nestjs/nest/master?token=abc123def456
[circleci-url]: https://circleci.com/gh/nestjs/nest

  <p align="center">A progressive <a href="http://nodejs.org" target="_blank">Node.js</a> framework for building efficient and scalable server-side applications.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/dm/@nestjs/common.svg" alt="NPM Downloads" /></a>
<a href="https://circleci.com/gh/nestjs/nest" target="_blank"><img src="https://img.shields.io/circleci/build/github/nestjs/nest/master" alt="CircleCI" /></a>
<a href="https://coveralls.io/github/nestjs/nest?branch=master" target="_blank"><img src="https://coveralls.io/repos/github/nestjs/nest/badge.svg?branch=master#9" alt="Coverage" /></a>
<a href="https://discord.gg/G7Qnnhy" target="_blank"><img src="https://img.shields.io/badge/discord-online-brightgreen.svg" alt="Discord"/></a>
<a href="https://opencollective.com/nest#backer" target="_blank"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor" target="_blank"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec" target="_blank"><img src="https://img.shields.io/badge/Donate-PayPal-ff3f59.svg"/></a>
    <a href="https://opencollective.com/nest#sponsor"  target="_blank"><img src="https://img.shields.io/badge/Support%20us-Open%20Collective-41B883.svg" alt="Support us"></a>
  <a href="https://twitter.com/nestframework" target="_blank"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

# nosql-injection-study

Estudo de **NoSQL injection**: uma aplicação de login em NestJS + MongoDB, com um front-end em
JavaScript puro, em duas versões — uma vulnerável e outra corrigida.

## O que faz

Expõe duas rotas de autenticação (porta `3000`) que guardam o usuário no MongoDB:

| Método | Rota            | Ação                                            |
| ------ | --------------- | ----------------------------------------------- |
| POST   | `/auth/signup`  | cria um usuário (`{"username", "senha"}`)       |
| POST   | `/auth/signin`  | faz login; 200 com `{id, username}`, 404 se não existe, 422 se a senha estiver errada |

O front em `front/login/` (abra `html/login.html`) consome o `signin`.

A diferença entre as branches está no `signin` (`src/auth/auth.service.ts`):

- **`com-falha`**: monta a busca com `findOne({ username, senha })`, jogando os valores recebidos
  direto na consulta. Como o `senha` do DTO é `any`, um corpo como
  `{"username":"raul","senha":{"$ne":null}}` faz o Mongo casar com qualquer senha — é a NoSQL
  injection.
- **`sem-falha`** (branch padrão): busca só por `username` e compara a senha em separado, então o
  operador injetado não entra na consulta.

## Por que existe

Para estudar na prática como uma NoSQL injection acontece e como corrigi-la, comparando as duas
branches.

## Instalação

Pré-requisitos: Node.js 20, npm e um MongoDB em `localhost:27017` (por exemplo
`podman run -d --rm -p 27017:27017 docker.io/library/mongo:7`).

```bash
git clone https://github.com/rillmind/nosql-injection-study.git
cd nosql-injection-study
npm install
npm run start:prod        # ou npm run start:dev
```

O `.env` versionado usa `MONGO_URI = "mongodb://localhost:27017/nosqli"`. Em máquinas onde
`localhost` resolve para IPv6 (`::1`), a conexão falha (`MongoNetworkError`); nesse caso use
`127.0.0.1`:

```bash
MONGO_URI=mongodb://127.0.0.1:27017/nosqli npm run start:prod
```

Para ver a versão vulnerável, troque para a branch `com-falha` (`git checkout com-falha`).

## Como testar

Não há testes automatizados (`npm test` termina com "No tests found"). Com a API rodando:

```bash
curl -X POST localhost:3000/auth/signup -H 'content-type: application/json' \
  -d '{"username":"raul","senha":"123456"}'                       # 201
curl -X POST localhost:3000/auth/signin -H 'content-type: application/json' \
  -d '{"username":"raul","senha":"123456"}'                       # 200
curl -i -X POST localhost:3000/auth/signin -H 'content-type: application/json' \
  -d '{"username":"raul","senha":"errada"}'                       # 422 na branch sem-falha
curl -i -X POST localhost:3000/auth/signin -H 'content-type: application/json' \
  -d '{"username":"raul","senha":{"$ne":null}}'                   # 422/404 em sem-falha; 200 em com-falha
```

A última requisição é o teste da injeção: na branch `sem-falha` ela não autentica; na `com-falha`
ela loga sem saber a senha.

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Installation

```bash
$ npm install
```

## Running the app

```bash
# development
$ npm run start

# watch mode
$ npm run start:dev

# production mode
$ npm run start:prod
```

## Test

```bash
# unit tests
$ npm run test

# e2e tests
$ npm run test:e2e

# test coverage
$ npm run test:cov
```

## Support

Nest is an MIT-licensed open source project. It can grow thanks to the sponsors and support by the amazing backers. If you'd like to join them, please [read more here](https://docs.nestjs.com/support).

## Stay in touch

- Author - [Kamil Myśliwiec](https://kamilmysliwiec.com)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

Nest is [MIT licensed](LICENSE).
