# Projeto Cairo: Counter Smart Contract

Este projeto implementa um contrato inteligente simples em [Cairo Language](https://book.cairo-lang.org/title-page.html) que mantém um contador. O objetivo deste README é documentar o processo de desenvolvimento para me lembrar dos passos que segui no futuro, implantação e configuração do ambiente de desenvolvimento usando o Rancher Desktop para rodar a Starknet Devnet.

## Ferramentas Utilizadas

Para desenvolver e testar este projeto, utilizei as seguintes ferramentas:

- **Git Bash** ou um terminal compatível
- **Rancher Desktop** (que inclui Docker)
- **Starkli** (para gerenciar e implantar contratos)
- **Postman** (para validar se a DevNet está rodando corretamente)

## Configuração do Ambiente

### 1. Preparando o Ambiente com Rancher Desktop

Utilizei o Rancher Desktop, que já traz o Docker integrado, para rodar a imagem Starknet Devnet.

Abri o Rancher Desktop e desativei Kubernetes, pois usaremos apenas o Docker e evitamos mensagens de erros referente ao uso do Kubernetes.

Executei o seguinte comando no terminal para iniciar a Starknet Devnet com um seed específico (passado em aula):

```bash
docker run -d -p 5050:5050 shardlabs/starknet-devnet-rs --seed 2525635640
```

### 2. Validando se Starkent DevNet está rodando corretamente

Com o Postman instalado adicionei esse curl

```bash
curl http://localhost:5050/is_alive
```
**A resposta deve ser ***Alive!!!*** se tudo estiver funcionando corretamente.**

## Criando o Signer e Obtendo a Conta

### 1. Criando o Signer

Dentro do projeto Counter criei uma pasta oculta chamada .c-wallets. Para isso utilizei o comando abaixo
```bash
mkdir .c-wallets
```
Após a criação da pasta .c-wallets, criei um arquivo keystore que armazenará a chave privada da conta. Para isso utilizei o seguinte comando:

```bash
starkli signer keystore from-key .c-wallets/account0_keystore.json
```
> **Nota:** Nesse passo será solicitado a private key da conta (Account address - conta essa passada em aula) e a password (eu quem devo definir e será utilizada nos próximos passos, por isso é bom anotar)

Dando tudo certo, este comando cria um arquivo keystore em .c-wallets/account0_keystore.json.

### 2. Obtendo as Informações da Conta

Após criar o signer, fiz o fetch as informações da conta passada em aula. Para isso utilizei o seguinte comando:
```bash
starkli account fetch <account_address> --output .c-wallets/account0_account.json --rpc http://localhost:5050
```

Dando tudo certo, este comando cria um arquivo keystore em .c-wallets/account0_account.json com as informações da conta.

##  Declarando e Deployando o Contrato

### 1. Declarando o Contrato

Antes de declarar o contrato, compilei o mesmo usando o comando abaixo. 
``` bash
scarb build
```
Este comando compila o projeto .cairo e gera uma pasta chamada target, que deverá conter uma pasta chamada dev e dentro dela terão 3 arquivos:

`counter_CounterContract.compiled_contract_class.json`

`counter_CounterContract.contract_class.json`

`counter.starknet_artifacts.json`

Com o projeto compilado com sucesso, usei esse comando para declarar o contrato.

```bash
starkli declare target/dev/counter_CounterContract.contract_class.json --account .c-wallets/account0.json --rpc http://localhost:5050 --keystore .c-wallets/account0_keystore.json
```
> **Nota:** Nesse passo será solicitado a password que defini anteriormente no passo criando o Signer.

Dando tudo certo, terei a informação do class hash que fica no log. algo como isso abaixo:

`Declaring Cairo 1 class: 0x03a74dd7efac9048233ae4dcfd3f4e6439794aa762bfbdsdfgac12562ace4ffe2`

> **Nota:** Guardar o Class hash pois usarei na fase do deploy do contrato.

### 2. Fazendo o deploy do Contrato

```bash
starkli deploy <class_hash_obtido_na_fase_de_declarar__contrato> <inputs_do_contrato> --account .c-wallets/account0.json --rpc http://localhost:5050 --keystore .c-wallets/account0_keystore.json
```
> **Nota:** Nesse ponto usei os inputs_do_contrutor quem tem que ser convertido em hexadecimal. Utilizei o exemplo da aula, mas há convesores online para isso.
Também foi solicitado o keystore password que defini anteriormente no passo criando o Signer.

Dando tudo certo, terei a informação do hash do contrato que foi deployado. Algo como isso abaixo:

```
Contract deployed:
0x066b8e549cd99c686e8c2c0cd624bac9b994d46637lpl4abc17282de0ce98dfa
```
> **Nota:** Guardar o Hash do contrato que foi deployado, pois usarei para interação com o contrato.

 ## Interagindo com o Contrato

Para interagir com o contrato, usei os comandos abaixo comandos.

### 1. Chamando método get_contador(somente leitura)
```bash
starkli call --rpc http://localhost:5050 <hash_do_contrato_deployado> get_contador
```

### 2. Chamando método increase_contador(escrita)
```bash
starkli invoke --rpc http://localhost:5050 --account .c-wallets/account0_account.json --keystore .c-wallets/account0_keystore.json <hash_do_contrato_deployado> increase_contador
```


## referências:
[Preparando Ambiente de Desenvolvimento Starknet - Muller Esposito](https://medium.com/starknet-in-brazil/preparando-ambiente-de-desenvolvimento-starknet-9e0663c1e0e5)

[Instalação do Scarb - Muller Esposito](https://medium.com/starknet-in-brazil/preparando-ambiente-de-desenvolvimento-starknet-9e0663c1e0e5)

[Setting up your environment](https://docs.starknet.io/quick-start/environment-setup/)

[Welcome to Starkli](https://book.starkli.rs/introduction)

[The Cairo Programming Language](https://book.cairo-lang.org/ch00-00-introduction.html)
