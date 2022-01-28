# Introdução

Este curso destina-se a ajudar qualquer pessoa que queira começar a aprender sobre segurança em aplicativos iOS, entendendo o básico de manipulação em tempo de execução e _binary patching_. Como o público-alvo é de iniciantes e _talvez_ pesquisadores intermediários/desenvolvedores/engenheiros/etc, não há pré-requisitos obrigatórios. No lugar disso, segue uma lista de _nice-to-have_ (é-bom-ter) ou conceitos recomendados e ferramentas que podem ajudar a navegar mais facilmente pelos próximos módulos:   

## Conceitos
* Conhecimento básico em [SSH](https://en.wikipedia.org/wiki/Secure_Shell)
* Entendimento básico de [Objective-C](https://en.wikipedia.org/wiki/Objective-C) e [Swift](https://en.wikipedia.org/wiki/Swift_programming_language)
* Conhecimento básico em [Javascript](https://en.wikipedia.org/wiki/JavaScript)
* Conhecimento básico em [HTML](https://en.wikipedia.org/wiki/HTML)
* Entendimento básico de ataques [Man-in-the-Middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)

## Ferramentas
* **Um computador, idealmente um MacBook:** todo o material abordado aqui foi criado usando um MacBook, mas quase todas as ferramentas utilizadas possuem uma versão Windows ou alternativa, exceto Xcode
* **Cabo Lightning:** um cabo para conectar seu iPhone ao seu computador. Evite usar cabos Lightning para USB-C, especialmente os que vêm nas caixas do iPhone's, [porque já foram detectados problemas na utilização desse tipo de cabo para execução de jailbreak, por exemplo](https://twitter.com/checkra1n/status/1195452064958222337). Procure usar cabos Lightning para USB-A ou cabos de terceiros.
* **iPhone com jailbreak:** para dispositivos com processador A11 ou anteriores (no caso, iPhone X ou anteriores), é fortemente recomendado o uso do [checkra1n](https://checkra.in), independente da versão do iOS. Caso contrário, dê uma olhada nas opções oferecidas pelo [unc0ver](https://unc0ver.dev/).

### Mais importante de tudo, curiosidade!

---
Além disso, para manter as expectativas em ordem, aqui vai uma lista do que não será coberto por esse curso (pelo menos não nessa ou em uma versão próxima, mas talvez num futuro mais distante).

## O que NÃO vai ser coberto por esse curso
- Exploração do iOS (o sistema operacional)
- Exploração do backend de aplicativos mobile
- Entendimento profundo sobre assembly ARM 

## Nota
Não se preocupe se você não tiver um dispositivo com jailbreak em mãos. Ainda assim, há muita coisa que você pode aprender nesse curso. O que não for possível, pode pular e voltar depois 