# Challenge "Impossible Password" da plataforma hackthebox:

imagem 1 

## Entendendo o problema
Trata-se de um challenge de engenharia reversa.
Após baixar e descompactar o arquivo do challenge, procura-se entender como o problema funciona.

impossible_password.bin
imagem 2

Um asterisco (*) é printado na tela seguido por uma requisição de input.
imagem 3

Ao tentar colocar algo, o programa printa entre colchetes ([ ]) o que foi colocado como entrada. Outra coisa que é possível identificar é que ele sempre vai printar entre colchetes
somente os 20 primeiros caracteres da entrada.
imagem 4

## Analisando o arquivo
Para saber mais sobre o arquivo, pode-se usar o comando file para analisar de
qual tipo de arquivo se trata:
imagem5

Logo, trata-se de um arquivo ELF, ou seja, define um tipo de padrão unificado para a interface binária das aplicações em sistemas Unix. Esse formato é utilizado como padrão para arquivos executáveis, código objeto,
bibliotecas compartilhadas e arquivos com core dumps. Dentro das seções de um arquivo ELF procura-se por strings dentro das seções de data. Para isso, usa-se uma funcionalidade da biblioteca de análise de binários para
engenharia reversa Radare2.
imagem 6
imagem 7

Aqui, encontra-se uma string interessante “SuperSeKretKey”. 
Também é possível ver a limitação da string com 20 caracteres.
Ao executar novamente e colocando a string encontrada como entrada:
imagem 8

Como das outras vezes a entrada foi printada entre os colchetes, mas, dessa
vez a execução não foi finalizada.
O programa printa dois asteríscos ** e pede por uma nova entrada.
Ao colocar alguma segunda entrada o programa finaliza sua execução.
Logo, parece que o programa está esperando por uma string específica.

### ltrace
ltrace é um utilitário de depuração no Linux.
Exibe as chamadas que um aplicativo de user space faz para bibliotecas
compartilhadas.

imagem 9

Como pode-se notar, tem-se logo no início entre os scanfs, uma comparação de strings, time(0) seguido de vários rand e no final uma outra comparação strcmp que compara a entrada colocada pelo usuário com uma string
aparentemente gerada.

imagem 10

Se tentarmos pegar a string gerada e colocar, gera-se uma nova diferente.
que é esperado, uma vez que provavelmente a string de entrada está sendo comparada com uma string sendo gerada seedeada por time(0).
Logo, é possível tentar resolver esse problema através da manipulação do tempo do sistema, entretanto, segue-se analisando o arquivo usando a biblioteca Radare2 para analisar as instruções Assembly do arquivo.

## Engenharia Reversa
Através da ferramenta Radare2, abre-se o arquivo com os seguintes parâmetros:
imagem 11
imagem 12
Realiza-se um seek para a main e printando o assembly:
imagem 13
Primeiramente, é possível ver várias declarações de inteiros de 64 bits e uma string que contém a primeira key.
imagem 14
É possível analisar a estrutura do programa e as chamadas de funções vistas
com o ltrace.
imagem 15
Após a primeira chamada strcmp(), está sendo comparado o input do usuário com a string SuperSeKretKey.
Se for igual irá realizar um jump para 0x00400925 continuando a execução do programa.
Senão, irá executar a próxima instrução e irá chamar exit().
Seguindo com a execução, após o segundo scanf é feita uma chamada fcn.0040078d:
imagem 16
imagem 17
Trata-se da geração pseudo-aleatória da segunda string.

Voltando para main, analise-se que após a segunda comparação de string vai
retornar o valor para o registrador eax.
imagem 18

A próxima instrução é test, que realiza uma operação AND bit a bit com os dois
operandos.

Isso vai setar o Zero_Flag(ZF) para 1 se eax < 0.

A próxima instrução jne 0x00400976 significa que se ZF ≠ 1 irá realizar um jump para a instrução em 0x00400976, ou seja, leave() terminando a execução
do programa.

Ou seja, com o input errado não será chamada fcn.00400978() seguindo com a execução do programa, onde provavelmente está localizada a key do challenge.

Logo, basta realizar-se um patch em 0x00400968:
Para isso, realiza-se um seek até o endereço, e sobescreve-se(wa) nele
uma instrução jump para a próxima linha de execução desejada.

imagem 19

## Executando após o patch
Após realizar o patch, executa-se o programa seguindo a linha de execução encontrada para a chamada fcn.00400978.
Se tudo funcionar bem, não deve ser necessário depender do segundo input de usuário para chamar a função e continuar com o fluxo de execução.

imagem 20

Dessa forma, obtém-se a flag do challenge independendo da segunda entrada.
