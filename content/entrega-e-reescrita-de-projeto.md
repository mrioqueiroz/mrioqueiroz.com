+++
title = "Resutados de entrega e reescrita de projeto"
description = "Um pouco sobre a automação de um processo utilizando Python e posterior rerescrita do projeto em Rust."
date = 2020-12-14
+++

Há alguns meses atrás fui chamado para um projeto: se tratava de uma rotina
manual que envolvia o processamento de arquivos. Esse processamento seria feito
após a finalização de operações rotineiras realizadas pela empresa. A ideia era
dar mais velocidade e confiabilidade ao processo.

Este é o resumo da experiência da automação desse processo, do aprendizado de
uma nova linguagem e dos resultados da reescrita do projeto nesta nova
linguagem.

# Contexto

O serviço envolvia um certo número de colaboradores que, ao fim de cada
operação, geravam dois arquivos que posteriormente precisavam ser validados e
mesclados para envio aos clientes.

O primeiro arquivo continha os dados da operação e o outro arquivo era o
comprovante da operação, que também continha informações pessoais. Os dois
arquivos deveriam ser combinados em um novo par de arquivos:

- Um contendo todas as informações da operação, bem como o comprovante contendo
informações de uso pessoal;
- Outro contendo os mesmos dados, porém com as informações de uso pessoal
removidas.

Os dois arquivos gerados eram enviados para o cliente, sendo o primeiro (com as
informações de uso pessoal) para arquivo e acompanhamento da operação e o
segundo (sem as informações de uso pessoal) para envio a fornecedores, bancos,
órgãos públicos etc.

Após terem sido realizadas as operações de *todos* os clientes, a tarefa de
verificação dos dados, junção dos documentos e remoção das informações de uso
pessoal era realizada manualmente.

No início, quando o número de clientes era bem menor, isso não causava tanto
impacto na rotina. Porém, com um número cada vez maior de clientes, a
realização desta tarefa de maneira manual passa a comprometer o andamento do
processo e o envio dos resultados das operações.

## Desafios

Os desafios que essa abordagem manual trazia eram:

- Manter a padronização durante o salvamento dos arquivos visando facilitar o
trabalho posterior de pesquisa e junção dos documentos;
- Durante a junção dos arquivos, ter o cuidado de não misturar documentos de
clientes;
- Ter o cuidado de remover todas as informações de uso pessoal;
- Risco de atraso no envio dos resultados, conforme o número de clientes
aumenta.

# Versão 1.0

Na implementação inicial escolhi Python como ferramenta para execução da ideia.
Em menos de um dia tivemos um resultado com a correta identificação dos dados
nos documentos (nomes e números de operação), com as validações necessárias
para garantir que os pares corretos estavam sendo gerados e a remoção das
informações de uso pessoal, além de um código extremamente simples.

Tendo como base um número de 1500 clientes (ou seja, 3000 arquivos para serem
processados), tivemos como tempo de execução um total de \~11min em um
computador com SSD e cerca de 1h15min em um computador com HDD (uma diferença
considerável, que acabou me surpreendendo e que vale a pena ser levada em
conta, visto que muitas empresas ainda não contam com SSD).

Com isso, tivemos uma boa economia de tempo, se levarmos em consideração que o
processo, se realizado manualmente, poderia levar diversas horas de trabalho de
alguns funcionários. Tivemos ainda um ganho maior de confiança no resultado
gerado, com as validações de conteúdo feitas pelo programa antes de gerar os
novos arquivos.

A implementação teve algumas pequenas barreiras:

- A instalação das dependências, que geralmente trazem alguma surpresa
desagradável ao executar em um computador diferente daquele do desenvolvedor;
- A provável falta de familiaridade com o uso do terminal/ambiente virtual por
quem precisar executar o *script* na ausência de alguma pessoa com mais
familiaridade.

É importante considerar também uma possível dificuldade na implementação do
programa em outros computadores por alguém ainda não familiarizado com Python,
em caso de necessidade futura.

No final, o projeto atendeu à necessidade inicial de dar velocidade ao processo
e aumentar a confiabilidade dos resultados, trouxe valor ao processo da empresa
e aos clientes. Até o momento continua funcionando conforme o esperado.

# Versão 2.0

Continuando os estudos de programação e a busca melhores formas de atender a
esse tipo de necessidade nas empresas, comecei a ler um pouco sobre *Rust* (uma
linguagem de programação que começou a subir cada vez mais na minha lista de
prioridades de estudo) e resolvi tentar reescrever este projeto nela (*porque
nada melhor do que um projeto vinculado a uma necessidade real pra aprender uma
nova linguagem*).

Com uma noção geral da sintaxe, comecei tentando traduzir função por função do
código anterior para a nova linguagem. Após acompanhar as mensagens de erro do
compilador, mais algumas pesquisas sobre os conceitos novos (*ownership*,
*borrowing*, *lifetimes*, *moves* etc.) as coisas foram se encaixando e tive o
resultado inicial, com um ganho de performance perceptível.

Em seguida, pesquisei um pouco mais sobre as estruturas da linguagem, como
deixar o código mais idiomático, remover redundâncias, uso de memória etc. e
consegui um ganho ainda mais considerável de performance, além de uma boa
redução no uso de memória, o que me possibilitou adicionar mais
funcionalidades:

- O programa agora não busca os arquivos em uma pasta específica. Os documentos
são buscados em toda a cadeia de diretórios em que o executável se encontra;
- O programa não depende mais de qualquer padronização no nome dos arquivos,
extensões, nomes de pastas ou arquivos diversos nos diretórios:
  - Os PDFs serão identificados mesmo se tiverem outra extensão, ou mesmo sem
  qualquer extensão;
  - Outros documentos, mesmo em PDF, que não estejam relacionados à operação,
  serão desconsiderados com base nos respectivos conteúdos.
- Adicionados outros tipos de validação do conteúdo dos arquivos,
possibilitando reconhecer e combinar arquivos de períodos distintos.

Nesta versão, o tempo de execução do programa reduziu para ~4min em um
computador com SSD e ~25min em um computador com HDD, considerando uma mesma
base de 1500 clientes/3000 arquivos.

Foi um ganho considerável. Mas sinto que ainda há muito espaço para melhorar e
já estou com algumas ideias para a versão 3.0. Tenho buscado esse ganho de
performance visando outras atividades que possuem uma frequência ainda maior e
que envolvem quantidades mais significativas de arquivos.
