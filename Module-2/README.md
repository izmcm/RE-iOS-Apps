# [Módulo 2] Descriptografando Aplicativos iOS

Depois de configurar seu ambiente, você está pronto para começar fazendo o download de qualquer aplicativo da App Store e descriptografando ele. Como um pesquisador de segurança, esse é o primeiro passo que precisará fazer antes de começar qualquer análise. Isso é necessário porque o iOS criptografa toda e qualquer aplicação instalada via App Store usando uma tecnologia DRM chamada [FairPlay](https://en.wikipedia.org/wiki/FairPlay). 

Embora exista alguma movimentação por parte da comunidade de criar sistemas que sejam capazes de realizar operações em cima de dados criptografados e obter resultados idênticos às operações realizadas com dados descriptografados, como a [Criptografia Homomórfica](https://en.wikipedia.org/wiki/Homomorphic_encryption), esses sistemas ainda não estão disponíveis em sistemas operacionais como o iOS. Isso significa que para executar uma aplicação, o iOS precisa descriptografá-la antes e é disso que vamos tirar proveito. Ao carregar a aplicação em memória (de forma descriptografada), vamos recuperar essa porção da memória onde ela está sendo executado e gravar em disco para poder ter acesso ao arquivo _.ipa_ da aplicação.

## O que você vai precisar nesse módulo?
* um iPhone **com jailbreak**
* um computador (de preferência, um MacBook)
* servidor SSH, Frida e frida-ios-dump

_Nota: Caso você não tenha um dispositivo com jailbreak em mãos, pule este módulo e use o arquivo CoinZa.ipa disponibilizado [aqui](../CoinZa.ipa) nos próximos módulos._

## Sumário
  - [Passo a Passo](#passo-a-passo)
      - [1. Instalação de qualquer aplicativo via App Store](#1-instalação-de-qualquer-aplicativo-via-app-store)
      - [2. Conexão SSH (computador - iPhone)](#2-conexão-ssh-computador---iphone)
      - [3. Uso do frida-ios-dump](#3-uso-o-frida-ios-dump)
  - [Bônus](#bônus)

## Passo a Passo
### 1. Instalação de qualquer aplicativo via App Store
### 2. Conexão SSH (computador - iPhone)
Você pode fazer isso usando qualquer uma das ferramentas citadas anteriormente. Iniciando a conexão do seu computador, configure a porta do iPhone para 44 (checkra1n) ou 22 e a do seu computador para 2222. 

Usando `iProxy`, por exemplo:
```bash
iproxy 2222 44
```
Ou até via interface gráfica com o `iPhoneTunnel` na barra superior do MacBook.

### 3. Uso o frida-ios-dump
* Vá até a pasta onde você fez o download do `frida-ios-dump` falado no [módulo anterior](../Module-1/README.md#frida-ios-dump) no seu computador.
* Abra o aplicativo e use o comando do Frida 
```bash
frida-ps -Ua
```
Esse comando deve listar todos os aplicativos que estão abertos (incluindo os em segundo plano) no seu dispositivo, junto com o número do processo e o bundle id da aplicação. **Guarde o bundle id do aplicativo que você quiser recuperar.**
* Descriptografe o aplicativo rodando o arquivo `dump.py` existente na pasta do `frida-ios-dump`
```bash
./dump.py <bundle id>
```
Se for necessário inserir uma senha, será a senha padrão `alpine` que falamos anteriormente OU a senha inserida por você caso tenha alterado ela.
* Ao término do processo, um arquivo `.ipa` estará no mesmo diretório em que você rodou o script. Esse arquivo é a versão descriptografada da aplicação que você baixou na App Store.

## Bônus
Se você olhou todos esses passos e pensou "muito disso pode ser automatizado, não?", você está absolutamente certo! Esse [script](https://github.com/ivRodriguezCA/decrypt-ios-apps-script) open source automatiza todos os passos, desde a inicialização do iTunnel até a cópia do aplicativo para o computador.
