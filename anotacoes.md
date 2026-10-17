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
Mais uma tarefa agendada pelo cron. O script copia a senha de bandit23 para `/tmp`, mas calcula o nome do arquivo a partir de uma frase com o nome do usuário que o executa.

*Conceito:* variáveis, substituição de comandos e pipes no Bash. `$(...)` executa um comando e captura sua saída; `$variavel` é substituído pelo valor guardado nela. O `whoami` retorna o usuário que executa o script, que neste caso é bandit23, conforme a configuração do cron. O script monta a frase `I am user bandit23`, calcula seu hash com `md5sum` e usa `cut -d ' ' -f 1` para pegar só o primeiro campo, que é o hash. Esse resultado vira o nome do arquivo em `/tmp`.

*Comandos*
```bash
cat /etc/cron.d/cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh

# reproduzir o cálculo do nome do arquivo
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
# ler o arquivo usando o hash retornado no lugar de <hash>
cat /tmp/<hash>
```

*Onde travei:* o principal foi entender o script. Achei que precisava fazê-lo rodar, mas o cron já fazia isso automaticamente. Também tentei procurar `/tmp/$mytarget`, sem perceber que essa variável só era definida dentro do script e não existia no meu terminal. Precisava reproduzir o cálculo para descobrir o nome real do arquivo. Ao montar o comando, esqueci o `echo`: sem ele, o shell tenta executar `I` como um comando. Os espaços, as maiúsculas e a quebra de linha que o `echo` acrescenta fazem parte da entrada do hash.

### Bandit 23 → 24
Aqui criei meu primeiro script para o desafio. O cron executa um programa como bandit24, que procura arquivos em `/var/spool/bandit24/foo`, executa os arquivos regulares pertencentes a bandit23 e depois apaga os itens da pasta.

*Conceito:* o dono do arquivo e o usuário que o executa são coisas diferentes. Meu script pertence a bandit23, mas é executado pelo processo do cron como bandit24 e, por isso, consegue ler `/etc/bandit_pass/bandit24`. O comando dentro dele copia a senha para um arquivo em `/tmp`, que posso ler depois. Também precisei dar permissão de leitura e execução para que bandit24 pudesse executar o script.

*Comandos*
```bash
cat /etc/cron.d/cronjob_bandit24
cat /usr/bin/cronjob_bandit24.sh

# criar o script diretamente na pasta que o cron examina
TERM=xterm nano /var/spool/bandit24/foo/meuscript.sh
```

Conteúdo do meu script:
```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/senha
```

Depois de salvar com Ctrl+O, Enter e sair com Ctrl+X:
```bash
chmod 755 /var/spool/bandit24/foo/meuscript.sh
# aguardar a próxima execução do cron, que ocorre a cada minuto
cat /tmp/senha
# senha do próximo nível
```

*Onde travei:* primeiro, precisei separar o papel do script do cron do papel do script que eu criaria. O primeiro encontra e executa meu arquivo; o segundo contém o comando que quero executar com as permissões de bandit24. Depois, tive dificuldade com o `chmod`: `600` permite leitura e escrita só ao dono, mas não execução. Os valores são `4` para leitura, `2` para escrita e `1` para execução, somados em cada posição: dono, grupo e outros. Com `755`, o dono tem as três permissões, e grupo e outros têm leitura e execução. Também precisei entender como criar o arquivo na pasta certa usando o caminho completo no editor. O `nano` deu erro por não reconhecer `xterm-kitty`; usar `TERM=xterm nano ...` resolveu para aquele comando.

*Lembretes sobre o script do cron:*

- `shopt -s nullglob`: padrões sem correspondência somem da lista, em vez de virarem nomes literais.
- `cd ... || exit`: encerra se não conseguir entrar na pasta.
- `for i in * .*`: percorre os nomes comuns e os que começam com ponto; o teste seguinte exclui `.` e `..`.
- `stat --format "%U"`: consulta o dono do item. Isso não muda o usuário que vai executá-lo.
- `[ -f "$i" ]`: verifica se é um arquivo regular.
- `timeout -s 9 60 "./$i"`: tenta executar o arquivo e força o encerramento se ultrapassar 60 segundos.
- `rm -rf "./$i"`: apaga o item mesmo que ele não tenha sido executado, pois está fora do teste de dono e tipo. Por isso, é preciso salvar o script e ajustar as permissões antes que o cron o recolha; se for apagado antes, recriar e tentar novamente.

### Bandit 24 → 25
Um serviço na porta 30002 do localhost entrega a senha do bandit25 quando recebe a senha do bandit24 junto de um PIN numérico de quatro dígitos. O desafio pede testar as 10.000 combinações possíveis, de `0000` a `9999`.

*Conceito:* força bruta é testar todas as possibilidades até encontrar a correta. Fiz um script Bash com um `for` para gerar as tentativas. O `printf` monta cada linha com a senha atual, um espaço e o PIN: `%s` recebe o texto da senha, `%04d` formata o número com pelo menos quatro dígitos, completando com zeros à esquerda, e `\n` termina a linha. A estrutura lembra o `printf` de C, mas no Bash os argumentos são separados por espaços, sem parênteses nem vírgulas. O pipe `|` passa a saída do script para o `nc`, enviando as tentativas pela mesma conexão.

*Comandos*
Conteúdo de `/tmp/myscript.sh` (substituir `SENHA_ATUAL` pela senha do bandit24):

```bash
#!/bin/bash
for i in {0..9999}
do
    printf "%s %04d\n" "SENHA_ATUAL" "$i"
done
```

Depois de salvar o script:
```bash
chmod u+x /tmp/myscript.sh
/tmp/myscript.sh | nc localhost 30002
```

*Onde travei:* precisei entender como montar as tentativas dentro do script. Na primeira versão, escrevi `ehco` e coloquei `echo` e `printf` na mesma linha como se os dois fossem executar; nesse formato, o `echo` apenas imprimiria os argumentos seguintes. Resolvi usando só o `printf`. Também ajustei o intervalo de `{0..1000}` para `{0..9999}` e entendi como completar os PINs com zeros à esquerda. Depois que o script gerou as linhas corretamente, bastou enviar a saída para o serviço com `/tmp/myscript.sh | nc localhost 30002`.

### Bandit 25 → 26
Recebi uma chave SSH para acessar o bandit26. Copiei o conteúdo para um arquivo local, ajustei a permissão com `chmod 600` e conectei com `ssh -i`. A autenticação funcionava, mas a conexão encerrava logo depois de mostrar um texto.

*Conceito:* o SSH inicia o shell de login configurado para o usuário. Na entrada do bandit26 em `/etc/passwd`, esse programa é `/usr/bin/showtext`, em vez de `/bin/bash`. O script usa o `more` para mostrar um arquivo e depois encerra. Quando o texto cabe inteiro na tela, o `more` termina rapidamente e a sessão fecha. Diminuindo a altura da janela antes da conexão, o texto deixa de caber e o `more` pausa. A tecla `v` abre o Vim a partir do paginador; os comandos do editor permitem iniciar um shell durante a sessão, com as permissões do bandit26.

*Comandos*
No bandit25, investigar o shell configurado e ler o script:

```bash
grep '^bandit26:' /etc/passwd
cat /usr/bin/showtext
```

Depois de copiar a chave para o arquivo local `ssh25.rsa`:
```bash
chmod 600 ~/ssh25.rsa
# diminuir bastante a altura da janela antes de conectar
ssh -i ~/ssh25.rsa bandit26@bandit.labs.overthewire.org -p 2220
```

Quando aparecer `--More--`, pressionar `v` para abrir o editor. No Vim, pressionar `Esc` e usar a linha de comandos do próprio editor:
```vim
:help :shell
```

Para iniciar o Bash pelo Vim, definir o shell que ele deve usar e depois abri-lo:
```vim
:set shell=/bin/bash
:shell
```

*Onde travei:* achei estranho o SSH autenticar e desconectar imediatamente. Ao consultar `/etc/passwd`, descobri que o shell era o `showtext`. Precisei entender que deveria diminuir a janela antes de executar o `ssh -i`, para manter o `more` aberto, e usar `v` para entrar no editor. Dentro do Vim, pensei que precisava escrever um comando no arquivo, salvar e conectar de novo. Na verdade, precisava executar os comandos do próprio editor naquela sessão. O erro `E37: No write since last change` apareceu porque alterei o texto e tentei sair sem salvar. A ajuda `:help :shell` foi a pista para entender como abrir um shell, e a opção `shell` precisava apontar para o Bash, pois o shell de login do usuário era o `showtext`.

### Bandit 26 → 27
Depois de conseguir abrir um Bash como bandit26 pelo Vim no nível anterior, fui para a home do usuário e encontrei o executável `bandit27-do`. Usei esse programa para ler a senha do bandit27.

*Conceito:* o executável tem o bit SUID e pertence ao bandit27. Ao executá-lo, o processo ganha o usuário efetivo do dono do arquivo, permitindo executar o comando informado com as permissões de bandit27. Assim, o `cat` consegue ler `/etc/bandit_pass/bandit27`, que meu usuário bandit26 não poderia ler diretamente. É o mesmo conceito do nível 19 → 20.

*Comandos*
Se ainda estiver no Vim, pressionar `Esc` e abrir o Bash:

```vim
:set shell=/bin/bash
:shell
```

No shell do bandit26:
```bash
cd /home/bandit26
ls -l
./bandit27-do cat /etc/bandit_pass/bandit27
```

Com a senha obtida, conectar pelo terminal local:
```bash
ssh bandit27@bandit.labs.overthewire.org -p 2220
```

*Onde travei:* fiquei em dúvida se precisava continuar usando o bandit25, mas a próxima etapa já era feita como bandit26, no shell aberto pelo Vim. Também precisei entender como sair do contexto do editor para abrir o Bash e ir para `/home/bandit26`. Depois de chegar à pasta e encontrar o executável, resolvi com `./bandit27-do cat /etc/bandit_pass/bandit27`.

### Bandit 27 → 28
O enunciado fornece um repositório Git acessível por SSH. A senha de bandit27-git é a mesma de bandit27. Depois de clonar o repositório na máquina local, basta ler o `README` para encontrar a senha do próximo nível.

*Conceito:* `git clone` cria uma cópia local do repositório, incluindo seus arquivos e histórico. Na URL SSH, o usuário é `bandit27-git`, a porta é `2220` e o caminho completo do repositório é `/home/bandit27-git/repo`. O diretório pessoal do usuário e o repositório dentro dele são caminhos diferentes.

*Comandos*
No terminal local:
```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo bandit27-repo
cd bandit27-repo
ls -la
cat README
```

*Onde travei:* na primeira tentativa, usei `/home/bandit27-git/` na URL e recebi `does not appear to be a git repository`. Faltava o `/repo` no final do caminho. A conexão SSH chegava ao servidor, mas eu estava apontando para a pasta errada.

*Lembrete:* nos próximos desafios de Git, cada clone abaixo usa uma pasta local diferente. A senha pedida é sempre a do nível atual, mesmo que o usuário SSH termine em `-git`.

### Bandit 28 → 29
Mais um repositório Git. O `README.md` da versão atual mostra o usuário bandit29, mas a senha foi substituída por `xxxxxxxxxx`. A senha pode ser encontrada em uma versão anterior do arquivo.

*Conceito:* o Git guarda o histórico dos commits. Substituir uma senha no arquivo atual não apaga o conteúdo das versões anteriores. `git log` lista os commits; `git log -p` também mostra as alterações de cada um. `git show HASH:README.md` permite consultar o arquivo em um commit específico sem mudar os arquivos da pasta atual.

*Comandos*
No terminal local:
```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo bandit28-repo
cd bandit28-repo
cat README.md
git log --oneline
git log -p -- README.md
```

Para consultar uma versão específica, substituir `HASH_DO_COMMIT` pelo identificador de um commit anterior à remoção da senha:
```bash
git show HASH_DO_COMMIT:README.md
```

*Onde travei:* comecei olhando só o arquivo atual, que tinha a senha mascarada. Depois de perceber que precisava olhar o histórico, fiquei em dúvida se deveria restaurar a versão com algo como `git -r`. Para descobrir a senha, consultar o conteúdo antigo com `git show` já resolve; não é necessário restaurar o arquivo.

### Bandit 29 → 30
O repositório informa que não há senhas em produção. A pista leva a explorar outras branches: a senha do bandit30 está na branch de desenvolvimento, `dev`.

*Conceito:* branches mantêm linhas diferentes de desenvolvimento. O clone abre uma branch inicialmente, mas também traz referências às branches remotas, como `origin/dev`. `git branch -a` lista as branches locais e remotas. É possível ler um arquivo de outra branch com `git show origin/dev:README.md`, sem trocar de branch.

*Comandos*
No terminal local:
```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo bandit29-repo
cd bandit29-repo
cat README.md
git branch -a
git log --oneline --all --graph
git show origin/dev:README.md
```

Para entrar na branch de desenvolvimento e explorar seus arquivos:
```bash
git switch -c dev --track origin/dev
cat README.md
```

*Onde travei:* li a mensagem sobre não haver senhas em produção e cheguei a pensar que o próximo nível não tinha senha. O ponto era procurar fora da branch de produção. Esses enunciados de Git dão pouca informação, então precisei explorar arquivos, histórico e branches para descobrir a pista.

*Exploração extra:* depois de pegar a senha, fiquei curioso com a branch `sploits-dev`:
```bash
git log --oneline origin/sploits-dev
git ls-tree -r --name-only origin/sploits-dev
git show origin/sploits-dev:exploits/horde5.md
```

O arquivo parecia não mostrar nada. Ao conferir os bytes, apareceu `0a`: uma única quebra de linha. O nome do arquivo sugeria mais conteúdo do que realmente havia ali.

### Bandit 30 → 31
O arquivo principal do repositório só diz que está vazio. A senha está em um objeto do Git referenciado pela tag `secret`.

*Conceito:* uma tag é uma referência a um objeto do Git. Apesar de ser comum usá-la para marcar commits de versões, ela também pode apontar diretamente para um blob, que guarda conteúdo de arquivo. `git cat-file -t` informa o tipo do objeto e `git show` mostra seu conteúdo. Neste desafio, `secret` aponta para um blob com a senha, sem precisar procurar um arquivo dentro de um commit.

*Comandos*
No terminal local:
```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo bandit30-repo
cd bandit30-repo
cat README.md
git tag
git cat-file -t secret
# retorna: blob
git show secret
```

*Onde travei:* o README vazio não ajudava, e pedi uma dica sobre onde mais procurar. Depois de encontrar a tag `secret`, achei que precisava descobrir qual arquivo estava ligado a ela. O `git cat-file -t secret` retornou `blob`, e o `git show secret` mostrou diretamente a senha.

*Lembrete:* no Git, vale investigar os arquivos atuais, o histórico, as branches e as tags. Nem todo conteúdo guardado no repositório aparece na pasta de trabalho.

### Bandit 31 → 32

O repositório pede um arquivo `key.txt` com o conteúdo exato `May I come in?`. Depois de corrigir o arquivo, fiz commit e push para o servidor verificar e mostrar a senha do bandit32.

*Conceito:* `git add` prepara as alterações, `git commit` registra essas alterações localmente e `git push` envia os commits ao servidor. O servidor executa uma verificação do arquivo e devolve a senha nas linhas `remote:`. O push por SSH pede a senha do **bandit31** para autenticar como `bandit31-git`; é preciso informar essa senha e pressionar Enter antes de receber o resultado. Nada aparece na tela enquanto a senha é digitada ou colada.

*Comandos*
Dentro do repositório clonado:
```bash
printf 'May I come in?\n' > key.txt
git add -f key.txt
git commit -m "Add key.txt"
git push
```

Quando o push pedir `bandit31-git@bandit.labs.overthewire.org's password:`, informar a senha do bandit31 e pressionar Enter. Copiar a senha do bandit32 que aparecer nas linhas `remote:`; a verificação pode mostrar a senha mesmo que o push termine com `pre-receive hook declined`.

Depois, no terminal local:
```bash
ssh bandit32@bandit.labs.overthewire.org -p 2220
```

*Onde travei:* o arquivo estava com uma aspa simples extra no começo (`'May I come in?`). O primeiro commit informou que não havia alterações, e o push respondeu `Everything up-to-date`. Corrigi o conteúdo com `printf`, preparei o arquivo com `git add -f` e fiz um novo commit. Depois achei que o push tinha parado, mas só faltava informar a senha do bandit31 no prompt de autenticação.

*Lembrete:* `git add -f` permite adicionar arquivos ignorados pelo `.gitignore`. Porém, o commit desta tentativa mostrou `1 insertion(+), 1 deletion(-)`, indicando que o arquivo já era rastreado e teve seu conteúdo corrigido; a mensagem `Everything up-to-date`, sozinha, não prova que ele estava ignorado.

### Bandit 32 → 33
Depois dos desafios de Git, entrei em um ambiente com a mensagem `WELCOME TO THE UPPERCASE SHELL` e o prompt `>>`. Tudo que eu digitava era convertido para maiúsculas antes de ser executado.

*Conceito:* nomes de comandos no Linux diferenciam maiúsculas de minúsculas: `ls` vira `LS` e deixa de encontrar o programa. O parâmetro especial `$0` representa o nome usado para iniciar o shell ou script atual. Nesse ambiente, ele permite chamar o shell que interpreta os comandos. Como `$0` não tem letras, a conversão para maiúsculas não altera a referência; sua expansão acontece quando o shell interpreta a linha. Assim, abre um shell comum, no qual é possível executar comandos normalmente com a identidade efetiva de bandit33.

*Comandos*
No prompt `>>`:
```sh
$0
```

Depois de sair do uppercase shell:
```sh
whoami
# retorna: bandit33
cat /etc/bandit_pass/bandit33
```

Com a senha obtida, conectar pelo terminal local:
```bash
ssh bandit33@bandit.labs.overthewire.org -p 2220
ls
cat README.txt
```

*Onde travei:* achei que estava dentro de um editor de texto ou algo parecido. Precisei entender que era um shell especial, que alterava os comandos antes de executá-los. A dica foi procurar um parâmetro formado por `$` e um número, que não seria afetado pela conversão para maiúsculas. Depois de escapar, consegui concluir o nível.

*Lembrete:* `$0` não é sempre `/bin/sh`: seu valor depende de como o shell ou script foi iniciado. A saída funciona dessa forma no ambiente específico do desafio.

### Bandit 33 — conclusão
Ao entrar como bandit33 e ler o `README.txt`, cheguei ao final dos níveis disponíveis. Em 17/10/2026, o [site oficial informa que o nível 34 ainda não existe](https://overthewire.org/wargames/bandit/bandit34.html).

## Referências para revisão

- [Enunciados oficiais do Bandit](https://overthewire.org/wargames/bandit/)
- [Git: consultar objetos e versões com git show](https://git-scm.com/docs/git-show)
