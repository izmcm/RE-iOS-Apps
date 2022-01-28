# Engenharia Reversa em Aplicativos iOS

### **[do original, por [@ivRodriguezCA](https://twitter.com/ivRodriguezCA)]**  

Bem vindos ao meu curso `Engenharia Reversa em Aplicativos iOS`. Se você está aqui, provavelmente significa que você compartilha meu interesse por segurança e exploração de aplicativos iOS. _Ou você apenas clicou no link errado_ 😂

Todas as vulnerabilidades que eu vou mostrar aqui são reais, foram encontradas em aplicativos em produção por pesquisadores de segurança, incluindo eu mesmo, como parte de programas de bug bounty ou apenas pesquisas regulares. Uma das razões pelas quais você não vê frequentemente writeups com esses tipos de vulnerabilidades é porque muitas companhias proibem a publicação desse tipo de conteúdo. 

Ajudamos essas empresas relatando esses problemas e somos recompensados por isso, mas ninguém além do(s) pesquisador(es) e da equipe de engenharia da empresa aprenderá com essas experiências. Esta é parte da razão pela qual decidi criar este curso, com um aplicativo iOS falso que contém _todas_ as vulnerabilidades que encontrei em minha própria pesquisa ou nas poucas publicações de outros pesquisadores. Embora já existam alguns projetos¹ com o objetivo de ensinar problemas comuns em aplicativos iOS, senti que precisávamos de um que mostrasse o tipo de vulnerabilidade que vimos em aplicativos baixados da App Store.

Este curso é dividido em 5 módulos que levarão você do zero até a possibilidade de fazer engenharia reversa de aplicativos baixados da App Store. Cada módulo destina-se a explicar uma única parte do processo em uma série de instruções passo a passo que devem guiá-lo até o sucesso.

Esta é minha primeira tentativa de criar um curso online, então tenha paciência comigo se não for o melhor. Eu adoro feedback e mesmo que você odeie, me avise; mas espero que você goste deste passeio e aprenda algo novo. Sim, eu sou um n00b!

Se você encontrar erros de digitação, erros ou conceitos errados, por favor, seja gentil e me diga para que eu possa corrigi-los e todos nós aprendemos!

## Módulos

- [Pré-requisitos](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Prerequisites.md)
- [Introdução](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Introduction.md)
- [Module 1 - Setup do ambiente](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Module-1/README.md)
- [Module 2 - Descriptografando aplicativos iOS](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Module-2/README.md)
- [Module 3 - Análise Estática](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Module-3/README.md)
- [Module 4 - Análise Dinâmica e Hacking](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Module-4/README.md)
- [Module 5 - Binary Patching](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Module-5/README.md)
- [Pensamentos finais](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Final-Thoughts.md)
- [Recursos](https://github.com/ivRodriguezCA/RE-iOS-Apps/blob/master/Resources.md)

## EPUB 
Graças à brilhante [ideia](https://github.com/ivRodriguezCA/RE-iOS-Apps/issues/7) de [natalia-osa](https://github.com/natalia-osa), agora há uma versão do curso no formato `.epub` que você pode baixar [aqui](https://github.com/ivRodriguezCA/RE-iOS-Apps-Extras-Github/blob/master/Files/RE-iOS-Applications-v1.1.epub). Como Natalia mencionou, isso é para facilitar o consumo do conteúdo. Obrigado novamente por esta ideia fantástica, Natalia 🙏🏼.

## Licença

Copyright 2019 Ivan Rodriguez `<ios [at] ivrodriguez.com>`

É concedida permissão, gratuitamente, a qualquer pessoa que obtenha uma cópia deste software e dos arquivos de documentação associados (o "Software"), para lidar com o Software sem restrições, incluindo, sem limitação, os direitos de usar, copiar, modificar, mesclar, publicar, distribuir, sublicenciar e/ou vender cópias do Software e permitir que as pessoas a quem o Software é fornecido o façam, sujeito às seguintes condições:

O aviso de direitos autorais acima e este aviso de permissão devem ser incluídos em todas as cópias ou partes substanciais do Software.

O SOFTWARE É FORNECIDO "COMO ESTÁ", SEM GARANTIA DE QUALQUER TIPO, EXPRESSA OU IMPLÍCITA, INCLUINDO MAS NÃO LIMITADO ÀS GARANTIAS DE COMERCIALIZAÇÃO, ADEQUAÇÃO A UM DETERMINADO FIM E NÃO VIOLAÇÃO. EM NENHUMA CIRCUNSTÂNCIA OS AUTORES OU DETENTORES DE DIREITOS AUTORAIS SERÃO RESPONSÁVEIS POR QUALQUER REIVINDICAÇÃO, DANOS OU OUTRA RESPONSABILIDADE, SEJA EM UMA AÇÃO DE CONTRATO, ATO ILÍCITO OU DE OUTRA FORMA, DECORRENTE DE, DE OU EM CONEXÃO COM O SOFTWARE OU O USO OU OUTRAS NEGOCIAÇÕES NO PROGRAMAS.

## Doações
Eu realmente não aceito doações porque faço isso para compartilhar o que aprendo com a comunidade. Se você quiser me apoiar, compartilhe este conteúdo e ajude a alcançar mais pessoas. Eu também tenho uma loja online [nullswag.com](https://nullswag.com) com roupas legais se você quiser comprar algo lá.

## Aviso
Eu criei este curso por conta própria e não reflete as opiniões do meu empregador, todos os comentários e opiniões são meus.

## Insenção de Danos
O uso deste curso ou material é, em todos os momentos, "por sua conta e risco". Se você estiver insatisfeito com qualquer aspecto do curso, qualquer um destes termos e condições ou quaisquer outras políticas, sua única solução é interromper o uso do curso. Em nenhuma hipótese eu, o curso, ou seus fornecedores, seremos responsáveis ​​perante qualquer usuário ou terceiro, por quaisquer danos resultantes do uso ou incapacidade de usar este curso ou o material deste site, seja com base em garantia, contrato, delito, ou qualquer outra teoria legal, e se o site é ou não avisado da possibilidade de tais danos. Use qualquer software e técnicas descritas neste curso, em todos os momentos, "por sua conta e risco", não me responsabilizo por quaisquer perdas, danos ou responsabilidades decorrentes ou relacionadas a este curso. Em nenhum caso serei responsável por quaisquer danos indiretos, especiais, punitivos, exemplares, incidentais ou consequenciais. Esta limitação será aplicada independentemente da outra parte ter sido avisada ou não da possibilidade de tais danos.

## Privacidade
Não estou coletando nenhuma informação pessoalmente. Como todo o curso está hospedado no Github, essa é a [política de privacidade](https://help.github.com/en/articles/github-privacy-statement) que você deveria ler.


¹ Eu adoro o trabalho que [@prateekg147](https://twitter.com/prateekg147) fez com [DVIA](http://damnvulnerableiosapp.com/) e OWASP fez com [iGoat](https://www.owasp.org/index.php/OWASP_iGoat_Tool_Project). Eles são ótimas ferramentas para começar a aprender os detalhes internos de um aplicativo iOS e alguns dos bugs que os desenvolvedores introduziram no passado, mas acho que muitos dos problemas mostrados são apenas teóricos ou impraticáveis e podem ser comparados a um "_auto-hack_" . É como olhar para o código-fonte de uma página da Web em um navegador da Web, você consegue entender o código estático (HTML/Javascript) do site, mas qualquer modificação que você fizer não afetará outros usuários. Eu queria mostrar vulnerabilidades que podem prejudicar a empresa que criou o aplicativo ou seus usuários finais.
