# [Módulo 1] Setup do Ambiente

Neste módulo, vamos aprender como configurar os seus dispositivos com todas as ferramentas que você vai usar durante o curso. A partir desse ponto, vamos assumir que você já tem um dispositivo com jailbreak. O processo é bastante chato, mas você só vai precisar fazê-lo **uma vez**, mesmo que o seu iPhone perca o jailbreak após desligar.

## O que você vai precisar nesse módulo?
* um iPhone **com jailbreak**
* um computador (de preferência, um MacBook)

_Nota: Se você precisa de ajuda para fazer o jailbreak, há muito recurso online. Um dos melhores sites para isso é [iDownloadblog](https://www.idownloadblog.com/jailbreak/)._

## Sumário
  - [No Computador](#no-computador)
    - [Servidor SSH  ](#servidor-ssh)
    - [Cydia Impactor](#cydia-impactor)
    - [ARM Disassembler](#arm-disassembler)
    - [Frida](#frida)
    - [frida-ios-dump](#frida-ios-dump)
    - [Bettercap](#bettercap)
    - [class-dump](#class-dump)
  - [No iPhone com Jailbreak](#no-iphone-com-jailbreak)
    - [Cliente SSH](#cliente-ssh)
    - [Cydia](#cydia)
    - [Cycript](#cycript)
    - [Frida](#frida-1)
  - [Conclusões](#conclusões)

## No Computador

### Servidor SSH  
Eles permitirão você estabelecer uma conexão [SSH-USB](https://iphonedevwiki.net/index.php/SSH_Over_USB) entre o seu dispositivo com jailbreak e o seu computador. O [iPhoneTunnel](https://code.google.com/archive/p/iphonetunnel-mac/downloads) possui uma interface gráfica com a qual é possível interagir, já o [iTunnel](https://code.google.com/archive/p/iphonetunnel-usbmuxconnectbyport/downloads) e o [iProxy](https://command-not-found.com/iproxy) funcionam via linha de comando.


### [Cydia Impactor](http://www.cydiaimpactor.com/)
Cydia Impactor vai permitir que você instale aplicativos iOS assinados com uma conta de desenvolvedor em seu dispositivo com jailbreak. É necessário ter uma conta de desenvolvedor paga para isso. Há alternativas como o [AppSync Unified](https://kubadownload.com/news/appsync-unified/) e a [AltStore](https://iosninja.io/altstore), além do combo [iOS App Signer](https://www.iosappsigner.com/) + [Apple Configurator 2](https://apps.apple.com/us/app/apple-configurator-2/id1037126344?mt=12)

### ARM Disassembler
O mais recomendado aqui é o [Hopper](https://www.hopperapp.com/), mas não é necessário pagar a versão completa porque o trial é suficiente. O [IDA Pro](https://hex-rays.com/ida-pro/) é uma alternativa entre os pagos e o [Ghidra](https://ghidra-sre.org/) e o [Cutter](https://cutter.re/) são alternativas open source e inteiramente gratuitas.

### [Frida](https://www.frida.re/docs/ios/)
Frida é uma ferramente poderosa e open source para manipulação de aplicativos em tempo de execução. Também é super fácil de instalar

```shell
sudo pip install frida-tools
```

### [frida-ios-dump](https://github.com/AloneMonkey/frida-ios-dump)
O frida-ios-dump permite descriptografar aplicativos iOS e transferi-los automáticamente para o seu computador. Caso a instalação das dependências dê erro, tente remover o `--upgrade` e ficar apenas com

```shell
sudo pip install -r requirements.txt 
```

### [Bettercap](https://www.bettercap.org/installation/)
Bettercap permite monitorar o tráfego de rede de um dispositivo e efetuar ataques de Man in the Middle remotamente

### [class-dump](https://github.com/nygard/class-dump)
Essa ferramente permite extrair os headers das funções em Objective-C presentes no aplicativo a partir do seu binário

## No iPhone com Jailbreak
### [Cydia](https://cydia.saurik.com/)
O Cydia é uma espécie de loja não oficial para instalação de softwares de terceiros no iOS, como as ferramentas que vamos precisar. O checkra1n, por exemplo, já vem com o Cydia. Imagino que o unc0ver também dê essa possibilidade

### Cliente SSH
Com o jailbreak feito, o cliente SSH é instalado automaticamente no seu dispositivo. Para testar, inicie o iPhoneTunnel ou semelhante. A porta padrão do dispositivo para jailbreak feito com checkra1n é `44` mas algumas outras ferramentas usam `22`. Nos dois casos, a porta padrão no seu computador vai ser `2222`. Inicie a conexão com essas portas configuradas, abra o terminal e digite
```shell
ssh -p 2222 root@localhost
```
Se a conexão funcionar, o seu dispositivo deve pedir uma senha `root`. A senha padrão é `alpine` mas você pode [alterá-la](https://cydia.saurik.com/password.html). Uma vez que a conexão for estabelecida, você poderá navegar pelo dispositivo usando o terminal. 

### [Cycript](https://cydia.saurik.com/package/cycript/)
o Cycript pode ser encontrado e instalado via Cydia diretamente no seu iPhone. Ele permite a modificação de aplicativos em tempo de execução por meiod e um console interativo.

### Frida
Vá até a aba `Sources` e adicione `https://build.frida.re`. Depois vá em `Search`, procure por `frida` e instale.

