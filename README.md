# Anotações do Bandit (OverTheWire)

Repositório com as minhas anotações do [Bandit](https://overthewire.org/wargames/bandit/), o primeiro wargame do OverTheWire. O Bandit ensina o básico de Linux e de linha de comando em forma de jogo: cada nível esconde a senha do próximo, e para achar essa senha é preciso usar algum comando ou conceito novo.

Faz parte dos meus estudos de **cibersegurança**, como aquecimento antes de partir para conteúdos de pentest.

## Como as anotações estão organizadas

Tudo fica no arquivo [`anotacoes.md`](anotacoes.md). Cada nível segue o mesmo formato:

- **Enunciado:** o que o nível pede, resumido;
- **Conceito:** a ideia por trás do nível e por que o comando funciona;
- **Comandos:** o que foi usado para resolver;
- **Onde travei:** as dificuldades e os erros no caminho, que é a parte que mais ajuda quando volto para revisar.

Alguns níveis também têm _Lembretes_, com detalhes que vale guardar para depois.

## Progresso

- [x] Níveis 0 → 20
- [ ] Níveis 20 → 34

## O que já foi visto

- Navegação e leitura de arquivos: `ls`, `cd`, `cat`, `file`
- Nomes de arquivo difíceis: `-`, espaços e arquivos ocultos
- Busca de arquivos por propriedades: `find`
- Filtrar e ordenar texto: `grep`, `sort`, `uniq`, `strings`
- Codificação e cifras: `base64`, `tr` (ROT13)
- Hexdump e arquivos comprimidos: `xxd`, `gzip`, `bzip2`, `tar`
- SSH com chave privada e permissões de arquivo: `ssh -i`, `chmod`
- Rede: `nc`/`ncat`, conexão com SSL/TLS (`ncat --ssl`, `openssl s_client`) e varredura de portas com `nmap`
- Comparar arquivos: `diff`
- Rodar comandos pelo SSH sem abrir um shell interativo
- Arquivos com setuid

## Observação

As anotações registram como cada nível foi resolvido. Para quem ainda vai jogar o Bandit, a graça é tentar sozinho primeiro.
