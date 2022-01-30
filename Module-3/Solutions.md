# [Module 3] Soluções

## Sumário
  - [Arquivo de configuração](#arquivo-de-configuração)
  - [Chave secreta única](#chave-secreta-única)
  - [Conexão insegura de rede](#conexão-insegura-de-rede)
  - [URL Schemes](#url-schemes)
  - [Dependências desatualizadas](#dependências-desatualizadas)

## Arquivo de configuração
Um dos primeiros arquivos interessantes que podemos ver na pasta do aplicativo é um arquivo com extensão json: `coinza-c7e97-firebase-adminsdk-ok3f8-df3457e3e8.json`. Trata-se de um arquivo de configuração para um projeto do [Firebase](https://firebase.google.com/) e, nesse caso, o maior perigo é o fato desse arquivo conter informações como a **chave privada** que deve ser usada para se conectar com os serviços do Firebase. Como o nome diz, a chave privada deveria ficar privada, secreta, e jamais ser disponibilizada no `.ipa` de um app.

- **Problema:** esse arquivo de configuração dá a um atacante as informações necessárias para acessar o serviço de backend do Firebase do aplicativo e tudo que estiver armazenado lá
- **Correção Recomendada:** esses tipos de arquivo de configuração que contém uma chave privada devem ser armazenados num servidor seguro e não no app do celular. É necessário adicionar um fluxo de autenticação entre o app no celular e um backend próprio e esse backend próprio que deve usar esse tipo de arquivo para se comunicar com o Firebase.


## Chave secreta única
Vasculhando o arquivo `Info.plist`, podemos ver uma key chamada `SQLCIPHER_KEY`. Trata-se de uma chave criptográfica usada pelo [SQLCipher](https://www.zetetic.net/sqlcipher/) para criptografar o banco de dados do app.

- **Problema:** embora os desenvolvedores estivessem pensando em proteger os dados dos seus usuários criptografando, eles fizeram isso inserindo uma chave criptográfica única que **todas** as instalações do aplicativo vão usar. Esse é um problema de segurança um pouco mais difícil de explorar, mas digamos que um invasor consiga ter acesso ao backup da vítima sem uma criptografia adicional. Esse banco de dados que deveria estar protegido por criptografia na verdade não estaria, porque qualquer um tem acesso à sua chave criptográfica para descriptografá-lo
- **Correção Recomendada:** o certo a se fazer nessa situação é gerar uma chave criptográfica por instalação e armazená-la no [keychain](https://developer.apple.com/documentation/security/keychain_services). Desse modo, mesmo que um atacante tenha acesso ao banco de dados do aplicativo de uma vítima, ele não conseguirá descriptografá-lo. 

## Conexão insegura de rede
Também no arquivo `Info.plis` um olhar mais atento mostra a utilização da chave `NSAllowsArbitraryLoads`. Essa chave teoricamente desativa o [App Transport Security](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity), política do iOS de garantia de comunicações seguras. 

Mas aqui o problema é um pouco mais complexo. Na verdade, essa chave realmente tinha esse objetivo, mas seguindo o escrito na [documentação oficial da Apple](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity/nsallowsarbitraryloads):

> In iOS 10 and later and macOS 10.12 and later, the value of the NSAllowsArbitraryLoads key is ignored—and the default value of NO used instead—if any of the following keys are present in your app’s Information Property List file:
>
> * NSAllowsArbitraryLoadsForMedia
>
> * NSAllowsArbitraryLoadsInWebContent
>
> * NSAllowsLocalNetworking

e também [nessa parte da documentação](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity):

>If you specify a value for any of the global exceptions besides NSAllowsArbitraryLoads, then the ATS behavior depends on the version of the OS on which your app runs:
>
> **iOS 9.0 or macOS 10.11**   
>   ATS uses the NSAllowsArbitraryLoads value that you set, or NO by default, and ignores the other global exceptions.
> 
> **iOS 10.0 or later or macOS 10.12 or later**   
>   ATS ignores the NSAllowsArbitraryLoads value that you set and instead obeys the other key or keys.

Considerando o disposto na documentação, podemos checar o valor para a chave `MinimumOSVersion` também no `Info.plist` e ver que se trata de um aplicativo que roda em iOS 10.0 ou superior. Procurando pelas outras chaves potencialmente perigosas existentes para essa versão do iOS (`NSAllowsArbitraryLoadsForMedia`, `NSAllowsArbitraryLoadsInWebContent` ou `NSAllowsLocalNetworking`) vemos que nenhuma delas está definida nesse aplicativo. Assim, o iOS deve ignorar o valor definido manualmente para `NSAllowsArbitraryLoads` e isso **não deve mais ser um problema de segurança**.

Independente disso, é importante sempre usar conexões seguras para quaisquer serviços, aplicativos ou servidores implementados.

## URL Schemes
Outra informação presente no `Info.plist` pode ser vista na chave `CFBundleURLTypes`. Essa chave não nos mostra um problema de segurança por si só, mas a forma como o aplicativo está lidando com essa URL pode ser um problema futuro que tentaremos explorar nos próximos módulos, então essa é uma informação importante de ser coletada.

Em `CFBundleURLTypes` vemos que o aplicativo contém a string `coinza`. Isso significa que esse aplicativo pode ser iniciado por outros aplicativos usando `coinza://` e que provavelmente podemos passar parâmetros para ela também

## Dependências desatualizadas 
Por último, dando uma olhada na pasta `Frameworks` vemos as dependências que esse aplicativo carrega. No `Info.plist` de cada uma das bibliotecas é possível ver a versão da biblioteca embarcada na chave `Bundle version string (short)`. A biblioteca `AFNetworking` embarcada no aplicativo está na versão `2.5.1`, uma versão [muito conhecida](https://blog.mindedsecurity.com/2015/03/ssl-mitm-attack-in-afnetworking-251-do.html) por possuir uma vulnerabilidade ao ataque [Man-in-the-Middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

- **Problema:** mesmo que frameworks de terceiros ajudem na economia de tempo e recursos durante o desenvolvimento de um app, eles podem facilmente introduzir vulnerabilidades que só serão corrigidas quando uma atualização na biblioteca for feita. Em muitos casos, de bibliotecas famosas e com comunidade ativa, as correções são feitas rapidamente, como no caso da AFNetworking (embora não fosse uma [correção completa](https://github.com/AFNetworking/AFNetworking/blob/2.5.2/AFNetworking/AFSecurityPolicy.m#L257-L265)). Mesmo assim, todos os aplicativos que possuiam a versão problemática dessa biblioteca tiveram que passar por um longo processo de atualização interno, envio para App Store, espera do review da Apple e, o pior de todos, boa vontade do usuário de atualizar o aplicativo. Esse processo leva muito tempo e, enquanto não for finalizado, os usuários do seu aplicativo estarão vulneráveis. Então, sempre vale a pena pensar bastante e verificar se a biblioteca que você quer adicionar é _realmente_ necessária. Além disso, não usar bibliotecas desconhecidas ou sem comunidade ativa também é importante. 
- **Correção Recomendada:** a correção é relativamente simples: basta atualizar o framework. Mas a recomendação real é usar a quantidade mínima de estruturas de terceiros em seus aplicativos, sempre priorizando os frameworks que a Apple nos fornece.

