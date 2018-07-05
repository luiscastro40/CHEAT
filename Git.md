#  Git

## Commit and Push

- git **status**                  : mostra os ficheiros para commit
- git **add .**                   : adiciona os ficheiros para commit
- git **commit -m "..."**         : realiza um commit com a mensagem "..."
- git **push origin nomebranch**  : fazer upload para o repositório remoto


## Manipulação de Branchs

- git **branch**                  : lista todos os branchs locais
- git **branch -av**              : lista todos os branchs locais e remotos
- git **branch new_branch**       : cria um novo branch com o nome 'new_banch'
- git **checkout nome_branch**    : muda para o branch 'nome_branch'
- git **merge 'nome_branch'**     : junta o 'nome_branch' com o branch atual



## Logs de commits

- git **log**                     : mostra o log de commits
- git **log --oneline**           : mostra numa linha todos os commit

## Mais comandos

- git **pull**                    : transfere as alterações existente no remoto para o local
- git **diff**                    : mostra as diferenças existentes entre os ficheiros antes do commit 
- git **clone**                   : clona o repositório
- git **rebase**                  : é parecido com um merge no entanto não altera a ordem cronologica nem reescreve outros commits

## Exemplo (Rebase)

Branch master com 3 commits (A, B e C).

A<---B<---C
          |
          |
       |Master|
          |
         HEAD

Exsplicação:
git checkout master
Após fazer o commit C, criei um novo branch com o nome design.

        HEAD
          |
       |design|
          |
          |
A<---B<---C
          |
          |
       |Master|
Comando:

git commit -m "Commit C"
Fiz um commit (D) no branch design

                  HEAD
                    |
                |design|
                    |
                    |
                .---D
               /
A<---B<---C<--´
          |
          |
       |Master|
Comando:

git checkout -b design
git commit -m "Commit D"
e voltei ao branch master.

                |design|
                    |
                    |
                .---D
               /
A<---B<---C<--´
          |
          |
       |Master|
          |
         HEAD
Comando:

git checkout master
Fiz um commit (E) no branch master.

                |design|
                    |
                    |
                .---D
               /
A<---B<---C<--´-----E
                    |
                    |
                |Master|
                    |
                   HEAD
Comando:

git commit -m "Commit E"
E fazendo o rebase:

                |design|
                    |
                    |
                .---D<----E'
               /          |
A<---B<---C<--´           |
                       |Master|
Comando:

git checkout design
git rebase master
Como pode notar, com o rebase da master na branch design, os commits a mais (E) da master vão para o topo da branch design.

Assim, como pode notar ao final, os commits ficaram como você disse:

master: A, B, C, D, E
design: A, B, C, D
