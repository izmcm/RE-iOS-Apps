# [Módulo 3] Análise Estática

Até agora, você aprendeu a configurar seu computador e dispositivo móvel com as ferramentas necessárias para descriptografar aplicativos iOS e copiá-los para seu computador. Agora você aprenderá a analisar um aplicativo iOS inspecionando todos os seus arquivos, frameworks (dependências) e até o binário. O nome desse módulo é `Análise Estática` porque vamos analisar o aplicativo estaticamente, sem executá-lo. 

Este é um módulo interativo, o que significa que os passos o colocarão na direção certa e você mesmo encontrará os problemas. Mas não se preocupe, se você se sentir perdido ou não conseguir encontrar nenhum problema, todas as soluções estão no final do módulo (juntamente com explicações sobre _por que eles são considerados problemas_ e algumas _soluções recomendadas_).

Depois de descriptografar um aplicativo iOS, você ficará com um arquivo do tipo `.ipa`. Esse arquivo é basicamente um arquivo `.zip` com a extensão trocada. Isso significa que ele inclui o binário da sua aplicação, todos os frameworks de terceiros que ela incorpora, arquivos de configurações, arquivos de mídia como imagens e vídeos, elementos de UI como [storyboards](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/DesigningwithStoryboards.html) e [nibs](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html), fontes customizadas e quaisquer outras coisas que os desenvolvedores puseram lá.

Para ilustrar as vulnerabilidades mais comuns, [@ivRodriguezCA](https://twitter.com/ivRodriguezCA) criou um aplicativo iOS inseguro chamado [`CoinZa`](../CoinZa.ipa). O `CoinZa` é escrito em Objective-C, o que torna mais simples a exploração - o que significa que aplicativos escritos em Swift são mais difíceis de serem explorados.

## O que você vai precisar nesse módulo?
* um computador (de preferência, um MacBook)
* o ipa do aplicativo CoinZa ([download](../CoinZa.ipa))
* class-dump e um ARM Disassembler

**Execute cada um dos passos abaixo com o aplicativo `CoinZa` e anote o que encontrou para depois comparat com as soluções.**

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

## Recuperar as classes da aplicação
Uma parte essencial da análise estática de qualquer aplicativo é reunir informações sobre que métodos e classes estão contidos ali. Esse passo pode fornecer muitas informações porque a declaração de métodos muito descritivos é um dos pontos principais de uma boa engenharia de software. Por outro lado, esses nomes também podem dar uma ótima ideia para um atacante das features que a aplicação possui. Há uma boa forma de evitar dar informações para atacantes ao mesmo tempo que mantém uma boa engenharia de software no seu projeto: [ofuscando o código](https://github.com/rockbruno/swiftshield).

Em aplicativos ofuscados o resultado dessa etapa pode ser apenas um monte de strings aleatórias. Entretanto, não são todos os apps que mantém uma ofuscação e, por isso, é sempre importante passar por aqui. Para isso, o `class-dump` falado no [módulo 1](../Module-1/README.md) será usado. 

Navegue até a pasta onde está o binário executável do `class-dump` e forneça o endereço do binário do aplicativo `CoinZa`. Grave o resultado num arquivo `.txt` para analisar melhor depois.

```bash
cd path/to/class-dump
./class-dump path/to/Payload/CoinZa.app/CoinZa > dump.txt
```

O aplicativo `dump.txt` gerado deve conter uma lista enorme de métodos e classes. Procure por classes interessantes e tente advinhar sobre o que é essa aplicação.

## Desmontando e decompilando [Hopper]
After the initial reconnaissance work, you've reached (IMO) the most exciting part of this module, understanding the actual behaviour of the application methods. After searching through the classes and methods in the `class-dump-z` output, you could see that this application is very small; but most of the applications are significantly bigger and have far more classes and methods. Because of this, it's important that you can prioritize your work and focus on the more interesting cases.
- To disassemble and decompile the binary open Hopper and drag-n-drop the CoinZa binary in Hopper's active window. _Note: You'll see that this binary is a [`FAT` binary](https://en.wikipedia.org/wiki/Fat_binary), which means that it contains code for more than one architecture. In this case it contains code for the `ARMv7` and `ARM64` architectures because this application targets a minimum version of iOS 10 and the minimum supported devices on iOS 10 are the iPhone 5, iPod Touch 6th Gen and iPad 4th Gen, which are `ARMv7` devices._

![Hopper](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Module-3/hopper1.png?raw=true)
- Hopper will ask you which architecture you want to disassemble. You can choose which ever you want though I'd recommend the `ARMv7` since it has a smaller and simpler instruction set but Hopper has some trouble disassembling some parts of the application on `ARMv7` and to explain better I'll be using the `ARM64` disassembled code.
- After selecting the architecture, it will ask you to set some options for the [Mach-o file](https://en.wikipedia.org/wiki/Mach-O). The defaults should suffice.
- Hopper will then begin to disassemble the binary. It shouldn't take too long since, again, this is a small app. But I've had some instances where it took about 45min to finish, and this was on a MacPro 6-core Xeon with 64GB RAM.
- Once Hopper finishes disassembling, select the `Procedures` tab on the left panel and you'll be able to see the list of method names that hopper was able to find.

![Hopper Procedures](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Module-3/hopper2.png?raw=true)
- If you select the `Str` tab (next to the `Procedures` one) as you probably guessed is the list of all the String-looking or printable characters within the binary. This is another favourite of mine since you can start searching for words like `secret`, `private`, `test` or `debug` and trace their usage. More often than not, developers leave test classes that provide a good insight. Sometimes there are even developer modes that we can enable to get extra functionality out of the application.
- To trace the usage of a string:
    - Search for a string, for example search for `isProVersion`.
    - On the main window, select the string and right-click on it.
    - On the menu select `References to aIsproversion`.
    - Hopper will take you to a the `cfstring` section, which is where the c-string literals are listed.
    - Select the `cfstring_isProVersion` and right-click on it and select `References to cfstring_isProVersion`.
    - Hopper will now show you a window with a list of methods. As you probably guessed, this is the list of methods that use the `isProVersion` string.
    - Select the first instance of `[AddFundsViewController viewDidAppear:]` and click Go.
    - On the main window you'll now see the assembly code of the `viewDidAppear` method of the `AddFundsViewController` class. If this is a bit confusing for you, Hopper has also a decompiler function.
    - In the middle of the top options bar select the `Pseudo-code Mode` tab (the one with the `if(b)` text).

    ![Hopper Pseudo-code Mode](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Module-3/hopper3.png?raw=true)
    - You'll be able to see that the string `isProVersion` is actually a key of an object stored in the [`NSUserDefaults`](https://developer.apple.com/documentation/foundation/nsuserdefaults) shared settings. I'll explain more on this in a bit.
- With the string search exercise you found evidence of features potentially guarded by a `ProVersion` state. You also saw in the previous exercise that you can load the `pseudo-code` of a method.
- To analyze a method with its `pseudo-code`: _Note: use the `ARM64` disassembly for this exercise._
    - Click on the `Procedures` tab and search for `WalletDetailViewController` and select the `didUpdateWalletBalance:` method.
    - Uncheck the `Remove potentially dead code` checkbox. Sometimes Hopper tries to optimize the decompiled code or just gets it wrong and the `pseudo-code` has some missing information. I usually uncheck this checkbox in case that happened.
    - I want to bring your attention to this section of the `pseudo-code`:
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
    - What you can see is that enabling the `ProVersion` state will be beneficial to an attacker since it will grant them an extra 20% of _something_. You don't know what that _something_ is yet, but looks like you should take a note about this finding. 😉 Specially since it looks like the check is done on the client side.
    - I'll leave it to you to keep digging around and take notes of interesting methods and classes. Analyze as many classes and methods as you can because they will help on the next module.
- **Tip 1:** Ignore all classes with the `FIR` prefix, they are part of the Firebase framework and are outside of the scope of this analysis.
- **Tip 2:** If you are using the trial version of `Hopper` take into account that it will self-close every 30min.


## Conclusões
- A static analysis on an iOS application can take you as little or as long as you want. You can go as deep as you can. Specially because the same techniques used to inspect the main application binary can be used to reverse engineer the 3rd party frameworks' binaries. I personally spend many days, and sometimes even many weeks, performing static analysis on iOS applications. _Note: The first mobile bug I was ever rewarded for on [HackerOne](https://hackerone.com) was a weak encryption vulnerability, specifically an insecure encryption key generation, basically I was able to predict past and future encryption keys. This was possible because I spent a lot of time understanding their key generation algorithm and was finally able to understand its behaviour without even running the application, all via static analysis._
- Many developers don't realize that any file they embed in their application will be very easy to extract and analyze.
- As researchers is very good idea to check the 3rd party frameworks bundled with the application.
- Gather as much information as you can on this step because you'll use it in the dynamic analysis step.

## Soluções
Find the solutions [here](Solutions.md).
