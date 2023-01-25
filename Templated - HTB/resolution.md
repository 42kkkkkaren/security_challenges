# Challenge “Templated” da plataforma hackthebox: 

![wikipedia ctr um](https://github.com/42kkkkkaren/security_challenges/blob/main/Challenge%2018/pictures/reschallenge181)

Primeiramente, ao procurar por pistas pela página web, nota-se a mensagem: “Proudly powered by Flask/Jinja2”.

## **Flask Jinja2**
É um mecanismo de template da web para a linguagem de programação Python que facilita a criação de páginas HTML. Serve para permitir que as informações trocadas entre uma aplicação escrita em Python e suas páginas HTML seja feita de forma mais simples e intuitiva, garantindo que o desenvolvedor consiga criar templates de forma mais fácil para suas aplicações. Basicamente, quando criamos um template com Jinja2 e incorporamos código Python nas páginas HTML, a própria ferramenta traduz o código Python e incorpora à página HTML, já que o Browser não consegue exibir código diferente do HTML.

## **WERKZEUG**
Pesquisando por fraquezas na library werkzeug, encontra-se que uma vulnerabilidade identificada é a execução remota de código (RCE), é um tipo de vulnerabilidade de segurança que permite que invasores executem código arbitrário em uma máquina remota, conectando-se a ela por meio de redes públicas ou privadas, gerando impactos severos no sistema.

![wikipedia ctr um](https://github.com/42kkkkkaren/security_challenges/blob/main/Challenge%2018/pictures/reschallenge181)

Geralmente, o console está armazenado em http://www.site.com/console.

![wikipedia ctr um](https://github.com/42kkkkkaren/security_challenges/blob/main/Challenge%2018/pictures/reschallenge181)

## **DirBuster**
É um aplicativo java multi-threaded projetado para diretórios de força bruta e nomes de arquivos em servidores da web. 

## **Server Side Template Injection (SSTI)**
É uma vulnerabilidade que ocorre quando um servidor processa o input do usuário como um template de algum tipo. Os templates podem ser usados quando apenas pequenos detalhes de uma página precisam mudar de circunstância para circunstância. Em vez de criar uma página totalmente nova por pessoa que acessa o site, ele simplesmente renderiza o endereço remoto na variável enquanto reutiliza o restante do HTML para cada solicitação de pessoa que o servidor recebe para esse terminal.
```
<h1>Bem vindo à página !!</h1>
<u>Essa página está sendo acessada pelo seguinte endereço de IP: {{ip}}</u>
```
Isso pode ser abusado, já que alguns mecanismos de template suportam algumas funcionalidades bastante complexas, que eventualmente permitem que os desenvolvedores executem comandos ou arquivem o conteúdo diretamente do template.
Logo, quando o poder de gerar e renderizar templates é dado a um usuário, isso pode levar ao acesso total ao sistema, como o usuário executando o servidor da web.

## **Method Resolution Order (MRO)**
É a ordem na qual o Python procura um método em uma hierarquia de classes. Ele desempenha um papel fundamental no contexto de herança múltipla, pois um único método pode ser encontrado em várias superclasses. Logo, podemos usar a função MRO para exibir todas as classes, que serão utilizadas para acessar os diretórios e realizar o RCE na biblioteca do sistema operacional (os).
```
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```
Como sabemos que o servidor está sendo executado no Flask, que é uma biblioteca Python, podemos usar o Method Resolution Order (MRO) para percorrer a request library no Flask para importar a biblioteca do sistema operacional. Uma vez que tenhamos acesso à biblioteca do sistema operacional, basicamente temos acesso ao shell no servidor onde podemos executar qualquer comando e ele será impresso no template e renderizado para nós.

Como podemos ver, o output gerado pelo comando é renderizado como template:

![wikipedia ctr um](https://github.com/42kkkkkaren/security_challenges/blob/main/Challenge%2018/pictures/reschallenge181)

Dessa forma, ao executar-se o comando “id” da biblioteca os, descobrimos que estamos sob modo de operação root, ou seja, temos permissão de privilégio total. Logo, executamos o comando ls para listar todos os arquivos e diretórios:

![wikipedia ctr um](https://github.com/42kkkkkaren/security_challenges/blob/main/Challenge%2018/pictures/reschallenge181)

Como estamos procurando por uma flag, é possível notar um arquivo de texto flag.txt, e ao executar o comando cat para verificarmos o conteúdo obtemos nossa flag:

![wikipedia ctr um](https://github.com/42kkkkkaren/security_challenges/blob/main/Challenge%2018/pictures/reschallenge181)

> HTB{t3mpl4t3s_4r3_m0r3_p0w3rfu1_th4n_u_th1nk!}
