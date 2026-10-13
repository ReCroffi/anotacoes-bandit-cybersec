# Anotações pessoais — OverTheWire: Bandit

Aqui vou deixar algumas anotações sobre o Bandit, do OverTheWire, como os comandos usados e por que eles funcionaram. A senha do último nível em que parei fica no arquivo `senha-atual.md`, que é local e não vai para o GitHub.


## Níveis
### Bandit 0
Aqui o desafio é entrar no jogo via SSH.
Foram fornecidos o host (bandit.labs.overthewire.org), o nome de usuário (bandit0), a senha (bandit0) e a porta (2220).

*Comandos*
```bash 
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

### Bandit 0 → 1
Uma vez conectado via SSH ao nível 0, o desafio é encontrar a senha para o próximo nível. O enunciado diz que ela está num arquivo chamado readme no diretório home.

*Conceito:* o básico do terminal: listar o que tem na pasta e ler o conteúdo de um arquivo.

*Comandos*
```bash
cat readme
```

*Onde travei:*
N/A

### Bandit 1 → 2
Com a senha adquirida no readme do nível 0, fiz login no nível 1 via SSH, usando bandit1@bandit.labs.overthewire.org e a senha.
Aqui, o desafio era parecido: pegar a senha em um arquivo. Porém, esse arquivo se chamava *-*.

*Conceito:* os programas tratam argumentos que começam com `-` como opções, e, para o `cat`, o `-` sozinho significa "ler do teclado". Escrevendo o caminho com `./`, o nome deixa de parecer opção.

*Comandos*
```bash
cat ./-
```

*Onde travei:* tive que pesquisar como o terminal lê esse tipo de nome.

### Bandit 2 → 3
Mais uma vez, aqui o desafio é pegar a senha num arquivo, que, nesse caso, se chama *--spaces in this filename--*.

*Conceito:* o shell separa as palavras nos espaços antes de passá-las para o programa. As aspas fazem o nome inteiro virar uma palavra só.

*Comandos*
```bash
cat ./"--spaces in this filename--"
```

*Onde travei:* aqui, novamente, tive que pesquisar como fazer. Além disso, não tinha entendido pelo enunciado que o nome do arquivo era esse (*--spaces in this filename--*).

### Bandit 3 → 4
Agora a senha estava num arquivo oculto dentro de um diretório.

*Conceito:* arquivo cujo nome começa com ponto é oculto: o `ls` comum não mostra, o `ls -a` mostra. O `...` do nome é só um nome que começa com ponto.

*Comandos*
```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```

*Onde travei:* nunca tinha visto um arquivo oculto com três pontos no início do nome, apenas com um; precisei de algumas tentativas até sacar.

### Bandit 4 → 5
Aqui a senha está no único arquivo legível por humanos (human-readable) dentro do diretório inhere.

*Conceito:* o nome do arquivo não diz o que tem dentro. O `file` olha o conteúdo e diz se é texto (ASCII text) ou dados binários (data).

*Comandos*
```bash
cd inhere
file ./*
cat ./-file07
```

*Onde travei:* tive que procurar o que era um arquivo legível por humanos (human-readable file) e como descobrir qual dos arquivos da pasta inhere tinha essa característica.

### Bandit 5 → 6
Já nesse nível, a senha estava num arquivo no diretório inhere com características específicas: legível por humanos, exatamente 1033 bytes de tamanho e não executável.

*Conceito:* o `find` busca por propriedades, não só pelo nome: tipo (`-type f`), tamanho (`-size`, onde o `c` quer dizer bytes) e permissão. O `!` nega o teste que vem depois.

*Comandos*
```bash
find inhere -type f -size 1033c ! -executable
```

*Onde travei:* novamente, fui pesquisar como buscar arquivos pelas suas características.

### Bandit 6 → 7
A senha está em um arquivo em algum lugar do servidor com dono bandit7, grupo bandit6 e exatamente 33 bytes.

*Conceito:* todo arquivo tem dono e grupo. Buscando a partir da raiz `/`, aparecem muitos "Permission denied", que saem pelo canal de erro (stderr, número 2). O `2>/dev/null` joga esses erros fora.

*Comandos*
```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

*Onde travei:* fui buscar como fazer.

### Bandit 7 → 8
A senha está no arquivo data.txt logo ao lado da palavra millionth.

*Conceito:* o `grep` mostra só as linhas que contêm um padrão.

*Comandos*
```bash
grep "millionth" data.txt
```

*Onde travei:* fui buscar como o grep funciona.

### Bandit 8 → 9
A senha está no arquivo data.txt e é a única linha de texto que ocorre exatamente uma vez (sem repetições).

*Conceito:* o `uniq` só compara linhas vizinhas, então, antes, o `sort` precisa deixar as linhas iguais juntas. O pipe `|` passa a saída de um comando para o próximo, e o `-u` mostra só as linhas que não se repetem.

*Comandos*
```bash
sort data.txt | uniq -u
```

*Onde travei:* tive que ir pesquisar sobre o sort.

### Bandit 9 → 10
A senha está no arquivo data.txt, numa das poucas linhas legíveis por humanos, logo depois de vários caracteres `=`.

*Conceito:* o `data.txt` é binário, então o `cat` mostra lixo e o `grep` comum só diz "binary file matches". A senha está numa linha legível depois de vários `=`.

*Comandos*
```bash
# -a faz o grep tratar o binário como texto
# ====+ = três "=" e depois um ou mais, ou seja, pelo menos 4 seguidos
grep -a -E '====+' data.txt
```

*Onde travei:* quantos `=` eram "vários". Não precisa saber: o grep acha o padrão dentro da linha, então um padrão menor também pega sequências maiores. O `strings` é o outro caminho, mas precisa de um `grep` junto pelo pipe.

### Bandit 10 → 11
A senha está armazenada no arquivo data.txt, codificada em formato base64.

*Conceito:* base64 é codificação, não criptografia: transforma bytes em texto, e qualquer um desfaz. O `-d` decodifica; sem ele, o comando codifica.

*Comandos*
```bash
base64 -d data.txt
```

*Onde travei:* fui pesquisar sobre Base64 e como decodificar.

### Bandit 11 → 12
A senha está armazenada no arquivo data.txt, onde todas as letras minúsculas e maiúsculas foram rotacionadas em 13 posições (cifra de César / ROT13).

*Conceito:* o ROT13 troca cada letra pela que está 13 posições à frente. Como o alfabeto tem 26 letras, aplicar duas vezes volta ao original. O `tr` troca cada caractere do primeiro conjunto pelo da mesma posição no segundo.

*Comandos*
```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt
```

*Onde travei:* fui pesquisar se existia um comando pronto no Bash. Encontrei o `tr` e fiquei um tempo até entender o que ele faz.

### Bandit 12 → 13
A senha está no arquivo data.txt, que é um hexdump de um arquivo comprimido várias vezes. O enunciado sugere criar uma pasta própria em /tmp, copiar o data.txt para lá e trabalhar nela.

*Conceito:* o arquivo é um hexdump de algo comprimido várias vezes. A extensão não diz o formato; quem diz são os primeiros bytes, e o `file` lê isso. É um ciclo: identificar, renomear se precisar, descomprimir, identificar de novo, até o `file` dizer "ASCII text".

*Comandos*
```bash
xxd -r data.txt data.bin   # desfaz o hexdump
file data.bin              # diz o formato
mv data.bin data.gz        # gzip quer a extensão .gz
gzip -d data.gz            # troca data.gz por data (o .gz some!)
```

*Onde travei:* usei `base64` no binário, mas não era base64, e `base64 arquivo` sem opção codifica em vez de decodificar. Também achei que o arquivo tinha sumido depois do `gzip -d`: ele só trocou de nome.

*Lembretes:*

- `mv` renomeia e move. `mv -i` pergunta antes de sobrescrever.
- O bzip2 funciona igual ao gzip, com a extensão dele.
- O tar empacota em vez de comprimir.

### Bandit 13 → 14
A senha do próximo nível fica guardada em /etc/bandit_pass/bandit14 e só pode ser lida pelo usuário bandit14. Neste nível você não recebe a próxima senha. Em vez disso, recebe uma chave SSH privada que pode ser usada para entrar no próximo nível. Olhe os comandos que permitiram sua entrada nos níveis anteriores do Bandit e descubra como usar a chave neste nível.

Se precisar de ajuda com este nível, tem um arquivo de dica na pasta home.
Leia com atenção as mensagens de erro, porque elas ajudam bastante.

*Conceito:* a chave privada permite autenticar sem digitar a senha da conta. O servidor precisa autorizar a chave pública correspondente para o usuário escolhido; não é o dono do arquivo da chave privada que determina em qual conta eu entro. O `-i` diz ao SSH qual chave usar. Neste exercício, a conexão por localhost foi bloqueada, então copiei a chave para o meu PC; como é um arquivo de texto, deu para copiar e colar. O SSH recusa uma chave privada acessível por outros usuários (e com `000` nem eu consigo ler); `chmod 600` deixa leitura e escrita só para o dono. Já `chmod u+r` apenas acrescenta leitura para o dono, sem remover permissões dos outros. No `chmod` com letras: quem (`u`/`g`/`o`/`a`) + operação (`+`/`-`/`=`) + permissão (`r`/`w`/`x`).

*Comandos*
```bash
cat sshkey.private
#copiei o texto do arquivo e criei uma versão dele localmente
#após isso alterei a permissão 
chmod 600 sshkey.private #arquivo local
ssh -i ~/sshkey.private bandit14@bandit.labs.overthewire.org -p 2220

```

*Onde travei:* travei em tudo :P. Precisei buscar dicas para entender o que era necessário fazer. Depois, ao criar o arquivo, precisei descobrir como alterar a permissão e como o SSH lê um arquivo no lugar da senha. Além disso, escrevi a porta errada (2200), quando o correto é 2220.

### Bandit 14 → 15
A senha para o próximo nível pode ser obtida enviando a senha do nível atual na porta 30000 no localhost.

*Conceito:* `localhost` é usado para acessar serviços de rede no próprio dispositivo em que eles estão rodando. O `nc` e o `ncat` são comandos que permitem ler e escrever dados por uma conexão de rede.

*Comandos*
```bash
cat /etc/bandit_pass/bandit14 # pegar a senha do nível atual
nc localhost 30000
#senha aqui
#Correct!
#senha do próximo nível
```

*Onde travei:* aqui já são conceitos novos para mim, tudo é uma trava e preciso buscar como fazer.

### Bandit 15 → 16
Esse nível é parecido com o anterior, porém deve-se enviar a senha para a porta 30001 no localhost usando criptografia SSL/TLS.

Fiz usando `ncat --ssl`. Outra ferramenta para conectar é `openssl s_client -connect localhost:30001`, que também mostra informações sobre o handshake e o certificado. São duas formas de estabelecer a conexão TLS.

*Conceito:* TLS protege os dados enviados pela conexão. Antes de enviar a senha, o cliente e o servidor fazem um handshake para negociar a conexão protegida. O `ncat --ssl` estabelece essa conexão e permite enviar e receber texto; o `nc` usado no nível anterior enviava os dados sem TLS. Criptografar a conexão e verificar a identidade do servidor são coisas distintas: usar `--ssl` sozinho no Ncat não verifica o certificado do servidor.

*Comandos*
```bash
ncat --ssl localhost 30001
#cola a senha do nível atual
#Correct!
#senha do próximo nível

```

*Onde travei:* imaginei que seria parecido com o anterior. Fui ver se o `nc` ou o `ncat` conectava usando criptografia e achei o parâmetro `--ssl`.

### Bandit 16 → 17
Neste nível, não pegamos uma senha, assim como no 13 → 14. Para pegar a credencial, é preciso enviar a senha para uma porta no intervalo de 31000 a 32000. Existem servidores escutando em algumas dessas portas, mas apenas um devolve a credencial.

*Conceito:* Port scan → método para descobrir quais portas estão abertas em um host; neste exercício, o alvo é `localhost`. Uma porta aberta indica que há um serviço aceitando conexões, mas não diz sozinha qual serviço é nem se ele usa TLS. O *Nmap* é um scanner de rede: `-p` seleciona as portas e `-sV` tenta identificar os serviços e suas versões, podendo também reconhecer TLS. Depois ainda preciso me conectar e interpretar a resposta para descobrir qual serviço devolve a credencial.

*Comandos*
```bash
nmap -p 31000-32000 localhost
# retorna a lista de portas no localhost
# tentei até encontrar a correta, que devolveu a credencial. 
# aí o restante é como no 13→14, pegar a credencial num arquivo novo, usar o chmod para as permissões e entrar com ssh -i
```

*Onde travei:* TUDO! Comecei com o `netstat`, mas não saía do lugar. Fui para o `nmap`, e aí aconteceu o bizarro. Rodei ```nmap -p-``` e recebi algumas portas (entre 1 e 65535), peguei as do intervalo do exercício e tentei o handshake com openssl (e ncat --ssl). Uma delas retornou a credencial, mas eu esperava uma senha e a ignorei.
Tentei novamente e fui pesquisar como fazer. A credencial apareceu várias vezes, mas eu a ignorava. Até que meu amigo Claudinho (Claude Code) me lembrou que não era necessariamente uma senha, mas sim uma credencial de entrada. Foi aí que, depois de 20 minutos, percebi que já tinha resolvido o problema.


Depois de resolver, vi que existe uma forma mais direta: o `nmap` mostra quais serviços estão escutando nas portas, o que ajuda a escolher o próximo passo.
```nmap -sV localhost -p 31000-32000```

### Bandit 17 → 18
A senha do próximo nível está em passwords.new. Ela é a única linha que foi alterada entre passwords.new e passwords.old.

*Conceito:* comparar o texto de dois arquivos.

*Comandos*
```bash
diff passwords.new passwords.old
42c42
< <senha do próximo nível>
---
> <linha antiga do passwords.old>
```

*Onde travei:* na interpretação da saída. Quando vi que `< ...` indicava a linha do primeiro arquivo, que, no meu caso, era o `passwords.new`, resolvi.

### Bandit 18 → 19
A senha está num arquivo chamado *readme*, porém alguém alterou o *.bashrc* para te expulsar assim que você loga.

*Conceito:* como usar o SSH além do shell interativo, enviando um comando para o servidor sem "entrar" nele.

*Comandos*
```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

*Onde travei:* primeiro, achei que mexer no `.bashrc` do bandit17 resolveria. Depois, fui buscar formas de usar o SSH. Tentei `ssh -t`, `ssh -T` e `ssh -v`, mas nenhum deu certo. Até que vi que poderia enviar comandos pela própria solicitação de entrada.

### Bandit 19 → 20
Usar um arquivo setuid e, com isso, pegar a senha que se encontra em ```/etc/bandit_pass/bandit20```.

*Conceito:* setuid e setgid são bits especiais de permissão. Em um executável, o setuid faz o processo executar com o ID efetivo do dono do arquivo, e o setgid faz o equivalente para o grupo, quando o sistema permite esse comportamento. Neste nível, o `bandit20-do` usa setuid para executar o comando com a identidade efetiva de `bandit20`, permitindo ler a senha que meu usuário não consegue acessar diretamente. Em diretórios, o setgid tem outra função: novos arquivos e subdiretórios herdam o grupo do diretório. No Linux, o setuid em diretórios não tem esse efeito.

*Comandos*
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

*Onde travei:* primeiro, nunca tinha ouvido falar de `setuid`. O desafio pede para executar o arquivo sem argumentos para ver como funciona e, depois, sugere `./bandit20-do whoami`. A partir daí, percebi a ligação com o nível anterior: é possível passar um comando junto com o arquivo.

### Bandit 20 → 21
Um arquivo setuid que faz uma conexão com o localhost em uma porta que **você especifica**, passando-a como argumento. Ele lê uma linha de texto pela conexão e a compara com a senha obtida no nível anterior (19 → 20). Se a senha estiver correta, transmite a senha do próximo nível.

*Conceito:* comunicação entre cliente e servidor via TCP. Usei o ```nc``` como servidor, escutando em uma porta escolhida por mim, e o ```suconnect``` como cliente. O servidor enviou a senha atual; o cliente validou e respondeu com a próxima senha. O pipe ```|``` levou o texto do ```echo``` até o ```nc```, e o ```&``` deixou o servidor rodando em segundo plano.

*Comandos*
```bash
echo -n "senha do 19 → 20" | nc -l -p 1234 & 
./suconnect 1234
#senha do proximo retornada
```

*Onde travei:* tive dificuldade em entender o enunciado, que dizia que a porta era especificada por mim. Rodei `cat suconnect`, e lá dizia como usar, passando a porta como argumento. Mas não sabia qual porta! Fiquei travado e fui procurar ajuda. Em duas soluções, vi que usavam a porta 1234 e não entendia de onde ela vinha. Até que percebi o trecho: *on the port you specify*. Como a porta podia ser escolhida por mim, tive que pensar em como enviar o texto com a senha e, ao mesmo tempo, chamar o `suconnect`. Foi aí que entraram o pipe `|` (que leva a saída do comando à esquerda para a entrada do comando à direita) e o `&` no final da linha, que deixa o comando rodando em segundo plano e libera o terminal.


### Bandit 21 → 22
Um programa é executado automaticamente em intervalos regulares pelo cron, o agendador de tarefas. A configuração fica em `/etc/cron.d/`.

*Conceito:* investigar tarefas agendadas com cron e as permissões dos arquivos que elas produzem. Um script executado como outro usuário copiou um segredo para um arquivo com leitura permitida a todos, expondo seu conteúdo.

*Comandos*
```bash
cat /etc/cron.d/cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
#senha do próximo nível
```

*Onde travei:* depois de ler o conteúdo de `cronjob_bandit22`, não soube o que fazer.
 ```@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null```
```* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null ```

Fui atrás de entender como esse script funcionava e o que fazia.

- ```@reboot bandit22 ...```: executa o script como bandit22 quando o serviço cron inicia.
- ``` * * * * * bandit22 ...```: executa o mesmo script a cada minuto.
- ```&> /dev/null```: descarta a saída padrão e os erros da execução.
- ```chmod 644 ...```: dá leitura e escrita ao dono, e leitura ao grupo e aos outros usuários.
- ```cat /etc/bandit_pass/bandit22 > /tmp/...```: lê a senha usando as permissões de bandit22 e grava no arquivo temporário.

Assim, foi possível ler a senha: apesar de eu não ter acesso ao arquivo original, o script a salvava em um arquivo temporário.

### Bandit 22 → 23


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 23 → 24


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 24 → 25


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 25 → 26


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 26 → 27


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 27 → 28


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 28 → 29


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 29 → 30


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 30 → 31


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 31 → 32


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 32 → 33


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*
