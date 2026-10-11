# Personal notes Over The Wire - Bandit

Aqui vou deixar algumas anotações sobre o bandit do Over The Wire, como os comandos usados e por que eles funcionaram. A senha do último nível em que parei fica no arquivo `senha-atual.md`, que é local e não vai pro GitHub.


## Níveis
### Bandit 0 
Aqui o desafio é entrar no jogo via SSH.
Foi fornecido o host (bandit.labs.overthewire.org), username (bandit0), senha (bandit0) e porta (2220).

*Comandos*
```bash 
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

### Bandit 0 → 1
Uma vez conectado ao SSH do nível 0, o desafio é encontrar a senha pro próximo nível. O enunciado diz que ela está num arquivo chamado readme no diretório home.

*Conceito:* o básico do terminal: listar o que tem na pasta e ler o conteúdo de um arquivo.

*Comandos*
```bash
cat readme
```

*Onde travei:* 
N/A

### Bandit 1 → 2
Com a senha adquirida no readme do nível 0, fiz login no nível 1 via SSH, usando bandit1@bandit.labs.overthewire.org e a senha.
Aqui o desafio era parecido, pegar a senha num arquivo, porém esse arquivo se chamava *-*.

*Conceito:* os programas tratam argumento que começa com `-` como opção, e pro `cat` o `-` sozinho significa "ler do teclado". Escrevendo o caminho com `./`, o nome deixa de parecer opção.

*Comandos*
```bash
cat ./-
```

*Onde travei:* tive que pesquisar como o terminal lê esse tipo de nome.

### Bandit 2 → 3
Mais uma vez, aqui o desafio é pegar a senha num arquivo, que nesse caso se chama *--spaces in this filename--*.

*Conceito:* o shell separa as palavras nos espaços antes de passar pro programa. As aspas fazem o nome inteiro virar uma palavra só.

*Comandos*
```bash
cat ./"--spaces in this filename--"
```

*Onde travei:* aqui novamente eu tive que ir pesquisar como fazer, além de não ter entendido no enunciado que o nome do arquivo era esse (*--spaces in this filename--*).

### Bandit 3 → 4
Agora a senha estava num arquivo oculto dentro de um diretório.

*Conceito:* arquivo cujo nome começa com ponto é oculto: o `ls` comum não mostra, o `ls -a` mostra. O `...` do nome é só um nome que começa com ponto.

*Comandos*
```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```

*Onde travei:* nunca tinha visto arquivo oculto com ... no início do nome, sempre com 1 ponto apenas; precisei de algumas tentativas até sacar.

### Bandit 4 → 5
Aqui a senha está no único arquivo legível por humanos (human-readable) dentro do diretório inhere.

*Conceito:* o nome do arquivo não diz o que tem dentro. O `file` olha o conteúdo e diz se é texto (ASCII text) ou dados binários (data).

*Comandos*
```bash
cd inhere
file ./*
cat ./-file07
```

*Onde travei:* Tive que procurar o que era um human-readable file e como descobrir qual dos arquivos na pasta inhere era um.

### Bandit 5 → 6
Já nesse a senha estava num arquivo no diretório inhere com características específicas: legível por humanos, exatamente 1033 bytes de tamanho e não executável.

*Conceito:* o `find` busca por propriedades, não só pelo nome: tipo (`-type f`), tamanho (`-size`, onde o `c` quer dizer bytes) e permissão. O `!` nega o teste que vem depois.

*Comandos*
```bash
find inhere -type f -size 1033c ! -executable
```

*Onde travei:* novamente, fui pesquisar como buscar arquivos através de suas características.

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

*Conceito:* o `uniq` só compara linhas vizinhas, então antes o `sort` precisa deixar as linhas iguais juntas. O pipe `|` passa a saída de um comando pro próximo, e o `-u` mostra só as linhas que não se repetem.

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

*Onde travei:* fui pesquisar sobre base64, e como decodificar.

### Bandit 11 → 12
A senha está armazenada no arquivo data.txt, onde todas as letras minúsculas e maiúsculas foram rotacionadas em 13 posições (cifra de César / ROT13).

*Conceito:* o ROT13 troca cada letra pela que está 13 posições à frente. Como o alfabeto tem 26 letras, aplicar duas vezes volta ao original. O `tr` troca cada caractere do primeiro conjunto pelo da mesma posição no segundo.

*Comandos*
```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt
```

*Onde travei:* fui pesquisar se existia um comando bash pronto, me deparei com o tr e fiquei um tempo até entender o que ele faz.

### Bandit 12 → 13
A senha está no arquivo data.txt, que é um hexdump de um arquivo comprimido várias vezes. O enunciado sugere criar uma pasta própria em /tmp, copiar o data.txt pra lá e trabalhar nela.

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
A senha do próximo nível fica guardada em /etc/bandit_pass/bandit14 e só pode ser lida pelo usuário bandit14. Neste nível você não recebe a próxima senha. Em vez disso, recebe uma chave SSH privada que pode ser usada para entrar no próximo nível. Olhe os comandos que te deixaram entrar nos níveis anteriores do Bandit e descubra como usar a chave neste nível.

Se precisar de ajuda com este nível, tem um arquivo de dica na pasta home.
Leia com atenção as mensagens de erro, porque elas ajudam bastante.

*Conceito:* a chave privada substitui a senha: quem tem o arquivo entra como o dono dela, e o `-i` diz pro ssh qual chave usar. O Bandit bloqueia conexão de dentro do próprio servidor (localhost), então a chave tem que ir pro meu PC; como é só texto, basta copiar e colar. O ssh recusa chave que outros podem ler (e com `000` nem eu consigo ler); o certo é só o dono ter acesso (`600` ou `u+r`). No `chmod` com letras: quem (`u`/`g`/`o`/`a`) + operação (`+`/`-`/`=`) + permissão (`r`/`w`/`x`).

*Comandos*
```bash
cat sshkey.private
#copiei o texto do arquivo e criei uma versão dele localmente
#após isso alterei a permissão 
chmod 600 sshkey.private #arquivo local
ssh -i ~/sshkey.private bandit14@bandit.labs.overthewire.org -p 2220

```

*Onde travei:* travei em tudo :P, precisei buscar dicas pra entender o que era necessário fazer, depois ao criar o arquivo eu precisei ver como alterar a permissão e ver como o ssh lê um arquivo no lugar da senha. Além disso, escrevi a porta errada (2200), quando o correto é 2220.

### Bandit 14 → 15
A senha para o próximo nível pode ser obtida enviando a senha do nível atual na porta 30000 no localhost.

*Conceito:* localhost é usado para acessar serviços de rede no mesmo dispositivo que está rodando os serviços. nc ou ncat é um comando que permite ler e escrever dados através da conexão de rede.

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

Fiz usando ncat --ssl, que é a versão rápida. Usando ```openssl s_client -connect localhost:30001``` sai a versão completa e mais "oficial".

*Conceito:* 

*Comandos*
```bash
ncat --ssl localhost 30001
#cola a senha do nível atual
#Correct!
#senha do próximo nível

```

*Onde travei:* imaginei que seria parecido com o anterior e fui ver se nc/ncat conectava com porta criptografada e achei o parâmetro --ssl

### Bandit 16 → 17
Esse aqui não pegamos a senha, igual ao nível 13→14. Pra pegar a credencial é preciso enviar a senha para uma porta que está no range 31000 - 32000. Existem algumas portas com servidor escutando elas, e apenas 1 devolve a credencial. 

*Conceito:* Port scan → método para encontrar portas abertas num localhost. 
*Nmap* é um scanner de rede

*Comandos*
```bash
nmap -p 31000-32000 localhost
# retorna a lista de portas no localhost
# tentei até encontrar a correta, que devolveu a credencial. 
# aí o restante é como no 13→14, pegar a credencial num arquivo novo, usar o chmod para as permissões e entrar com ssh -i
```

*Onde travei:* TUDO!  Comecei com netstat, não saía do lugar. Fui pro nmap, aí que aconteceu o bizarro. Rodei ```nmap -p-``` e recebi algumas portas (entre 1 e 65535), peguei as do range do exercício e tentei o handshake com openssl (e ncat --ssl). Uma delas me retornou a credencial mas eu esperava que fosse ser uma senha, ignorei. 
Tentei novamente, fui pesquisar como fazer, e por várias vezes aparecia a credencial e eu ignorava. Até que meu amigo Claudinho (Claude Code) me lembrou que não era necessariamente uma senha, mas sim uma credencial de entrada. Foi aí que depois de 20 minutos eu percebi que já tinha resolvido o problema. 


Após a minha solução vi que existe uma forma mais direta, onde o ```nmap``` mostra quais serviços estão escutando nas portas, o que deixa mais assertivo o caminho. 
```nmap -sV localhost -p 31000-32000```

### Bandit 17 → 18
A senha do próximo nível está em passwords.new. Ela é a única linha que foi alterada entre passwords.new e passwords.old.

*Conceito:* comparar texto de dois arquivos. 

*Comandos*
```bash
diff passwords.new passwords.old
42c42
< <senha do próximo nível>
---
> <linha antiga do passwords.old>
```

*Onde travei:* Na interpretação da saída. Uma vez que vi que < ... era a linha no primeiro arquivo, que no meu caso era o passwords.new, resolvi.

### Bandit 18 → 19
A senha está num arquivo chamado *readme*, porém alguém alterou o *.bashrc* para te expulsar assim que você loga. 

*Conceito:* Como usar o ssh além do shell interativo, enviando um comando pro servidor sem "entrar" nele.

*Comandos*
```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

*Onde travei:* Primeiro achei que poderia mexer no .bashrc do bandit17 que resolveria. Depois fui buscar modos de usar o ssh, tentei ```ssh -t```, ```ssh -T``` e ```ssh -v``` e nenhum deu certo. Até que vi que poderia enviar comandos através da própria solicitação de entrada. 

### Bandit 19 → 20
Usar um arquivo setuid e com isso pegar a senha que se encontra em ```/etc/bandit_pass/bandit20```.

*Conceito:* Setuid (e setgid) (abreviações de "set user ID upon execution" e "set group ID upon execution", que significam, respectivamente, "definir ID de usuário sob execução" e "definir ID de grupo sob execução") são sinalizadores de direitos de acesso do Unix que permitem que usuários rodem um executável com as permissões do dono ou grupo do executável, respectivamente, e alterem o comportamento em diretórios.

*Comandos*
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

*Onde travei:* Primeiramente nunca ouvi falar de ```setuid```. O desafio fala pra rodar ele normalmente sem argumentos para ver como funciona, e daí sugere o ```./bandit20-do whoami``` depois. A partir daí o link com o anterior vem, é possível passar um comando junto com o arquivo.

### Bandit 20 → 21


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

### Bandit 21 → 22


*Conceito:*

*Comandos*
```bash

```

*Onde travei:*

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
