+++
title = "Continuação do projeto de reconhecimento de caracteres em Captcha"
date = 2020-12-14
+++

Apesar do projeto de quebra de CAPTCHA ter sido apenas um passatempo nesses
dias de quarentena, alguns colegas perguntaram sobre o andamento e resolvi
trabalhar novamente no código. Então, atendendo a pedidos, vou fazer essa
atualização e explicar um pouco do funcionamento e ferramentas utilizadas.

Quem assistiu ao [vídeo que publiquei no
YouTube](https://www.youtube.com/watch?v=uqHhLTrEU9U), pôde observar que o
programa está atualmente tentando fazer o **reconhecimento de apenas um tipo de
imagem** e tenta fazer a extração de **exatos seis caracteres** (sendo estes
possíveis aprimoramentos para as próximas versões).

No momento, a maioria das imagens que eu testei **estão retornando com pelo
menos um caractere errado** após o processamento.

É bem complicado encontrar os parâmetros que vão funcionar para todos os casos:
a alteração de um parâmetro que permita fazer o reconhecimento dos caracteres
de uma imagem com 100% de acurácia, pode fazer com que o reconhecimento de
outra imagem retorne completamente errado.

Para piorar a situação, o código atual contém trechos com níveis de laço de
repetição maiores do que seria ideal; dessa forma, tentar aumentar certos
parâmetros faz com que a performance do programa caia bastante (pra quem quiser
saber mais, recomendo pesquisar sobre Big O Notation):

> "...é usado para classificar algoritmos pela forma como eles respondem (ex.,
> no tempo de processamento ou espaço de trabalho requerido) a mudanças no
> tamanho da entrada."
>
> [Wikipédia](https://pt.wikipedia.org/wiki/Grande-O)

Mas enfim...

# Falando um pouco do funcionamento

Para que tudo funcione, o primeiro passo é transformar a imagem em preto e
branco utilizando a biblioteca `Pillow`:

```python
color_image.convert("1")
```

Em seguida é preciso **remover as manchas da imagem**. Isso é feito através da
contagem dos pixels por cores: caso seja identificado uma sequência de pixels
escuros menor que certo limite, estes são transformados em pixels brancos.
Dessa forma, restarão apenas os grupos maiores de pixels escuros (que seriam os
caracteres).

Essa foi a parte mais complicada. Porém, depois de muita pesquisa, eu encontrei
no Stack Overflow [esta pergunta](https://stackoverflow.com/q/15319528) do
usuário [djadmin](https://stackoverflow.com/users/1618788/djadmin), que me
ajudou bastante no processo (recomendo também que vejam o projeto no
[GitHub](https://github.com/djadmin/decodeCaptcha/)).

> Como o projeto dele é um pouco antigo e foi feito para rodar em Python 2,
> [fiz um fork](https://github.com/mrioqueiroz/decodeCaptcha) e ajustei para
> que rode em Python 3. Quem quiser testar, só instalar as dependências e
> executar. Caso ocorra algum erro, me avise.

O programa faz essa "limpeza" cerca de cinco vezes e salva uma imagem para cada
uma delas. Mais do que isso a imagem fica quase totalmente branca e o processo
de OCR fica comprometido.

O próximo passo é fazer o reconhecimento dos caracteres (ver
`Tesseract`/`PyTesseract`).

Com o `Tesseract`, faço a leitura de cada uma das images salvas no
processo anterior utilizando diversos layouts de análise (parâmetro
`psm`, conforme
[documentação](https://github.com/tesseract-ocr/tesseract/blob/master/doc/tesseract.1.asc)).
Para cada imagem de analisada, os caracteres reconhecidos são
armazenados em uma lista.

Após as análises de todas as imagens, é feita a contagem dos padrões de
caracteres que mais ocorrem utilizando a biblioteca `collections`. **O
resultado final que aparece na tela é aquele que mais se repete**.

## E sobre o código...

Com relação ao código final, continuo trabalhando nele pra que fique mais
compreensível que pra que tenha mais flexibilidade quando for tentar diferentes
imagens. Digo isso porque eu mesmo tive dificuldades de entender algumas partes
após alguns dias. Também tenho que verificar as dependências (lembro que tive
alguns problemas com isso quando comecei o projeto, principalmente com o
`Tesseract`).

Em breve disponibilizo no [GitHub](https://github.com/mrioqueiroz),
possivelmente com um container Docker configurado e tudo mais.

Mas vendo o [fork do projeto que citei
anteriormente](https://github.com/mrioqueiroz/decodeCaptcha), dá pra ter uma
boa noção de como tudo funciona.

Até o próximo post com a continuação dessa saga. Valeu!
