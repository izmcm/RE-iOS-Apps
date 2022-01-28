# [Módulo 3] Análise Estática

Até agora, você aprendeu a configurar seu computador e dispositivo móvel com as ferramentas necessárias para descriptografar aplicativos iOS e copiá-los para seu computador. Agora você aprenderá a analisar um aplicativo iOS inspecionando todos os seus arquivos, frameworks (dependências) e até o binário. O nome desse módulo é `Análise Estática` porque vamos analisar o aplicativo estaticamente, sem executá-lo. 

Este é um módulo interativo, o que significa que os passos o colocarão na direção certa e você mesmo encontrará os problemas. Mas não se preocupe, se você se sentir perdido ou não conseguir encontrar nenhum problema, todas as soluções estão no final do módulo (juntamente com explicações sobre _por que eles são considerados problemas_ e algumas _soluções recomendadas_).

Depois de descriptografar um aplicativo iOS, você ficará com um arquivo do tipo `.ipa`. Esse arquivo é basicamente um arquivo `.zip` com a extensão trocada. Isso significa que ele inclui o binário da sua aplicação, todos os frameworks de terceiros que ela incorpora, arquivos de configurações, arquivos de mídia como imagens e vídeos, elementos de UI como [storyboards](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/DesigningwithStoryboards.html) e [nibs](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html), fontes customizadas e quaisquer outras coisas que os desenvolvedores puseram lá.

Para ilustrar as vulnerabilidades mais comuns, [@ivRodriguezCA](https://twitter.com/ivRodriguezCA) criou um aplicativo iOS inseguro chamado [`CoinZa`](../CoinZa.ipa). O `CoinZa` é escrito em Objective-C, o que torna mais simples a exploração - o que significa que aplicativos escritos em Swift são mais difíceis de serem explorados.

## O que você vai precisar nesse módulo?
* um computador (de preferência, um MacBook)
* o ipa do aplicativo CoinZa ([download](../CoinZa.ipa))
* class-dump e um ARM Disassembler

## Sumário
  - [Extração de arquivos](#extração-de-arquivos)
  - [Análise dos arquivos embutidos](#análise-dos-arquivos-embutidos)
    - [Arquivos `.plist`](#arquivos-plist)
    - [Arquivos `.mom` e `.momd`](#arquivos-mom-e-momd)
    - [Outros arquivos](#outros-arquivos)
  - [Análise de frameworks de terceiros (dependências)](#análise-de-frameworks-de-terceiros-dependências)
  - [Recuperação das classes da aplicação](#recuperação-das-classes-da-aplicação)
  - [Desmontagem e Decompilção um `.ipa` com Hopper](#desmontagem-e-decompilção-um-ipa-com-hopper)
    - [Rastreando uma String](#rastreando-uma-string)
  - [Conclusões](#conclusões)
  - [Soluções](#soluções)

_Nota: Execute cada um dos passos abaixo com o aplicativo `CoinZa` e anote o que encontrou para depois comparar com as soluções._

## Extração de arquivos
A extração do conteúdo de um `.ipa` é tão simples quando trocar sua extensão para `.zip` e descompactar ele. Você pode fazer isso manualmente, mas se quiser usar o terminal seria algo como

```bash
mv CoinZa.ipa CoinZa.zip
unzip CoinZa.zip
```

Depois de descompactar o `zip`, o conteúdo dele ficará num arquivo chamado `CoinZa.app` dentro da pasta `Payload`. Clicar com o botão direito no `CoinZa.app` fará aparecer uma mensagem de `Show Package Contents` (ou `Mostrar Conteúdo do Pacote`). Com isso, você terá acesso a todos os arquivos citados anteriormente e poderá até copiá-los para um local que possa ser acessado mais facilmente.

## Análise dos arquivos embutidos
Nosso objetivo final é entender o máximo possível o que os desenvolvedores estão enviando junto com cada um dos seus apps. É uma boa ideia começar procurando por problemas clássicos, coisas que **não deveriam estar ali**. No iOS, esses problemas são tipicamente arquivos de configuração, arquivos de dados de exemplo, arquivos de conexão com banco de dados ou até arquivos de chaves privadas pra conexões SSH. E sim, todos eles acontecem no mundo real!

### Arquivos `.plist`
Os arquivos `Info.plist` e outros arquivos com extensão `.plist` estão disponíveis em todos os aplicativos da App Store. Arquivos `.plist` podem guardar muitas informações sobre o aplicativo, em especial o `Info.plis` que carrega informações de configuração como:
- Se o app habilita conexões inseguras, desabilitando as proteções da política de App Transport Security (ATS). Isso por ser visto por meio da chave [`NSAppTransportSecurity`](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity).
- Se o app aceita [`Scheme URLs`](https://developer.apple.com/documentation/uikit/core_app/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) customizadas e quais são elas. Isso pode ser visto procurando a chave `CFBundleURLTypes`.


### Arquivos `.mom` e `.momd` 
Essas extensões se referem a modelos de CoreData compilados. Esses arquivos podem ser decompilados usando a ferramenta citada no [módulo 1](../Module-1/README.md): `momodec`.

### Outros arquivos
Além dos dois citados acima, outros arquivos também podem ser ferramentas interessantes nessa análise: 
- Arquivos `.json` são comuns para armazenar chaves privadas e outras informações de configuração
- Arquivos `.txt` também podem servir de local para armazenamento de informações de configuração e até recursos do app que deveriam ser protegidos por uma assinatura, por exemplo
- Arquivos de `mídia` também podem ser recursos especiais que não deveriam estar disponíveis localmente na aplicação

## Análise de frameworks de terceiros (dependências)
A maioria absoluta dos aplicativos iOS usam pelo menos um framework de terceiro. Muitos usam centenas deles! Isso merece uma atenção especial porque aumenta muito a superfície de ataque, principalmente quando desenvolvedores esquecem de atualizar suas dependências por falta de gerenciamento adequado. Muitas vezes, se o app está funcionando bem, não se tem um incentivo para essas atualizações, mesmo que isso possa representar vulnerabilidades de segurança. 

As dependências de um aplicativo podem ser encontradas na pasta `Frameworks`. Fique sempre atento às bibliotecas e às suas versões presentes nos seus respectivos `Info.plist`. **Vale sempre a pena procurar no Google quais as vulnerabilidades conhecidas para aquela biblioteca naquela versão específica.**

## Recuperação das classes da aplicação
Uma parte essencial da análise estática de qualquer aplicativo é reunir informações sobre que métodos e classes estão contidos ali. Esse passo pode fornecer muitas informações porque a declaração de métodos muito descritivos é um dos pontos principais de uma boa engenharia de software. Por outro lado, esses nomes também podem dar uma ótima ideia para um atacante das features que a aplicação possui. Há uma boa forma de evitar dar informações para atacantes ao mesmo tempo que mantém uma boa engenharia de software no seu projeto: [ofuscando o código](https://github.com/rockbruno/swiftshield).

Em aplicativos ofuscados o resultado dessa etapa pode ser apenas um monte de strings aleatórias. Entretanto, não são todos os apps que mantém uma ofuscação e, por isso, é sempre importante passar por aqui. Para isso, o `class-dump` falado no [módulo 1](../Module-1/README.md) será usado. 

Navegue até a pasta onde está o binário executável do `class-dump` e forneça o endereço do binário do aplicativo `CoinZa`. Grave o resultado num arquivo `.txt` para analisar melhor depois.

```bash
cd path/to/class-dump
./class-dump path/to/Payload/CoinZa.app/CoinZa > dump.txt
```

O aplicativo `dump.txt` gerado deve conter uma lista enorme de métodos e classes. Procure por classes interessantes e tente advinhar sobre o que é essa aplicação.

## Desmontagem e Decompilção um `.ipa` com Hopper
Depois de analisar a saída do `class-dump`, você pode ver que mesmo esse aplicativo sendo **muito** pequeno, a quantidade de classses e métodos resultantes no `dump.txt` foi bem grande. A maioria dos aplicativos na App Store são dezenas ou centenas de vezes maiores do que esse, o que significa que esse trabalho pode ficar extramemente dificil rapidamente. Por isso, é importante priorizar seu trabalho e focar nos casos mais interessantes.

Para começar a falar desse tópico, primeiro temos que entender que compilação e montagem são duas etapas responsáveis por transformar o código que escrevemos no Xcode num aplicativo na App Store. De forma extremamente resumida, podemos pensar que o código fonte escrito em uma linguagem de alto nível (como Swift ou Obj-C, por exemplo) passa por um compilador e vira um código de máquina (no nosso caso, um ARMv7 ou ARM64 Assembly). O código assembly resultante então passa por uma espécie de montador (ou assembler) para virar o código binário, aquele monte de 0 e 1 que conhecemos bem.

O que um disassembler faz é o inverso dessa última etapa: ele pega o código binário que temos no nosso celular e transforma num código assembly do qual podemos tirar algumas informações.

Para desmontar e descompilar o app, vamos abrir o Hopper e arrastar e soltar o arquivo `.ipa` nele. _Você verá que este arquivo é um [`FAT binário`](https://en.wikipedia.org/wiki/Fat_binary), o que significa que contém código para mais de uma arquitetura. Neste caso, ele contém código para as arquiteturas `ARMv7` e `ARM64`, ou seja, o app pode ser instalado em dispositivos mais antigos como o iPhone 5 é um dispositivo `ARMv7`.

![Hopper](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Module-3/hopper1.png?raw=true)

O Hopper nos dá a opção de escolher de qual arquitetura queremos desmontar o arquivo e, embora `ARMv7` seja uma arquitetura mais fácil de ser lida por ter um menor conjunto de instruções, o Hopper trabalha muito melhor com `ARM64`, então vamos escolher ela. Na próxima tela, devem aparecer algumas opções para o arquivo [Mach-o file](https://en.wikipedia.org/wiki/Mach-O), mas as opções pré-definidas são o suficiente. 

O processo de desmontagem deve demorar alguns minutos. Ao finalizar, no canto esquerdo da tela, podemos ver algumas tabs:
*  `Proc.` (ou `Procedures`) nos mostra a lista de métodos que o Hopper foi capaz de encontrar no binário
*  `Str` (ou `Strings`) nos mostra a lista de todas as strings contidas no binário. É uma ferramente muito útil para procurar por palavras como `secret`, `private`, `test` ou `debug` e rastrear seus usos. 

![Hopper Procedures](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Module-3/hopper2.png?raw=true)

### Rastreando uma String
É muito comum os desenvolvedores deixarem classes de teste que fornecem uma boa visão do funcionamento do app e, às vezes, existem até modos de desenvolvedor que podemos ativar para obter funcionalidades extras. Podemos achar essas vulnerabilidades rastreando strings:

1. Na tab `Str`, procure por `isProVersion` e clique no resultado
2. Na janela principal, selecione a string `isProVersion` e clique com o botão direito
3. Selecione `References to aIsproversion`
4. Você será redirecionado para a seção de `cfstring`
5. Selecione `cfstring_isProVersion` e aperte com o botão direito para selecionar `References to cfstring_isProVersion`
6. Uma nova janela deve aparecer com a lista de métodos que usam essa string.
7. Selecione o método `[AddFundsViewController viewDidAppear:]` e clique em `Go`
8. Na janela principal você será redirecionado para o código assembly do método `viewDidAppear` da classe `AddFundsViewController`
9. Na parte superior selecione a opção com texto `if(b)`, o `Pseudo-code Mode`, para decompilar o código assembly

![Hopper Pseudo-code Mode](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Module-3/hopper3.png?raw=true)

10. Como resultado, você verá que a string `isProVersion` é a chave de um objeto guardado no [`NSUserDefaults`](https://developer.apple.com/documentation/foundation/nsuserdefaults). **Isso é uma falha grave de segurança porque o UserDefaults não deve ser usado para armazenar dados sensíveis porque pode ser facilmente manipulado**

#### **Mas o que acontece se a versão Pro for ativada?**
11. Vá até o outro método visto na janela do item 6, que também usa a string `isProVersion`. O método será o `didUpdateWalletBalance:` da classe `WalletDetailViewController`.
12. Desative a caixa `Remove potentially dead code` pois, às vezes, a tentativa de otimização do Hopper pode comprometer o pseudo-código. Observe o seguinte trecho do pseudo-código:

```c
r2 = @"isProVersion";
if (objc_msgSend(r0, @selector(boolForKey:)) != 0x0) {
        r8 = 0x1001f0000;
        r2 = @"isProVersion";
        r1 = @selector(stringWithFormat:);
        var_60 = d8 * 0x1001ad2e0;
        r2 = @"Since you are a pro user we added an extra 20%% and it's on us!\nYour balance will actually increase by US$%f.";
        r0 = objc_msgSend(@class(NSString), r1);
        r29 = r29;
} else {
        r8 = 0x1001f0000;
        r2 = @"isProVersion";
        var_60 = d8;
        r2 = @"Funds purchased successfully, your balance will increase by US$ %f.";
        r0 = objc_msgSend(@class(NSString), @selector(stringWithFormat:));
        r29 = r29;
}
```

O trecho acima deixa claro que ativar a opção `isProVersion` dará 20% a mais de dinheiro no saldo do usuário. Isso é basicamente dinheiro de graça por causa de duas vulnerabilidades: checagem apenas no lado da aplicação e armazenamento inseguro de dados. 

Você pode continuar procurando por mais strings e métodos interessantes. Ignore classes com prefixo `FIR` porque elas fazem parte do framework Firebase. 

## Conclusões
- A análise estática pode levar o tempo que você quiser. Você pode ir o mais fundo que conseguir, principalmente porque essas mesmas técnicas são aplicáveis não só em apps mas também nos frameworks que aquele app carrega. Às vezes será necessário passar muitos dias ou semanas realizando esse tipo de análise para entender o suficiente do funcionamento interno daquele código.

- Muitos desenvolvedores não percebem que qualquer arquivo que incorporem em seu aplicativo será muito fácil de extrair e analisar.

- Como pesquisadores, é sempre uma boa ideia verificar os frameworks de terceiros incluídos no aplicativo, suas versões e possíveis vulnerabilidades já conhecidas e reportadas
  
- Reúna o máximo de informações que puder nesta etapa, pois você as usará na etapa de análise dinâmica.

## Soluções
Todas as soluções podem ser vistas [aqui](Solutions.md).
