# Snort IDPS + Docker Laboratory (Kali Linux)

> Projeto Pratico de Seguranca de Redes: Criacao de um ambiente isolado com Docker para simulacao, analise de trafego e deteccao de intrusoes (IDS) em tempo real utilizando o Snort 2.
> 
> 

---

## Sobre o Projeto

Como estudante de Ciberseguranca, desenvolvi este laboratorio para entender na pratica a diferenca entre um Firewall (que analisa apenas IPs e portas) e um IDPS (Intrusion Detection & Prevention System), que realiza a Inspecao Profunda de Pacotes (Deep Packet Inspection - DPI) lendo o conteudo (payload) das requisicoes.

Neste projeto, utilizei o Docker para subir um ambiente Linux leve e seguro contendo o Snort, permitindo testar regras de deteccao (alert) contra varreduras de rede, tentativas de SQL Injection, Command Injection, Path Traversal e pings sem poluir o sistema hospedeiro (Kali Linux).

---

## Arquitetura e Tecnologias

* Sistema Hospedeiro: Kali Linux (Simulador do Atacante / Gerador de Trafego)


* Containerization: Docker (Ubuntu Base ultraleve)


* IDPS Core: Snort 2


* Testes Praticos: cURL, Ping (ICMP), Nmap



---

## Regras Criadas (local.rules)

Todas as regras foram configuradas no modo de alerta (alert) para fins de monitoramento e testes de invasao no laboratorio:

* ICMP Ping (sid:1000001): Alerta na tela sempre que qualquer maquina enviar pacotes de Ping (ICMP Echo Request com itype:8).


* Command Injection (sid:1000003): Identifica tentativas de ler o arquivo de senhas /etc/passwd via requisicao HTTP na porta 80.


* Path Traversal (sid:1000004): Identifica requisicoes HTTP que tentam subir diretorios utilizando ../ na porta 80.


* SQL Injection (sid:1000005): Identifica injecao de comandos SQL comuns (' or 1=1) em aplicacoes web na porta 80.


* Nmap SYN Scan (sid:1000007): Identifica varreduras furtivas do Nmap (SYN Scan sem confirmacao de conexao, flags:S; ack:0).


* Nmap Xmas Scan (sid:1000008): Identifica pacotes de varredura agressiva do Nmap com as flags FIN, PSH e URG ativas simultaneamente (flags:FPU).



---

## Passo a Passo de Execucao & Explicativo dos Comandos

### 1. Construcao da Imagem Docker

docker build -t lab-snort .

O que faz: Le as instrucoes do Dockerfile (que baixa uma imagem base limpa do Ubuntu) e instala o Snort, utilitarios de rede e bibliotecas necessarias para criar a imagem customizada chamada lab-snort.

---

### 2. Execucao do Container Isolado

docker run -it --network=host --name meu-snort -v $(pwd)/local.rules:/etc/snort/rules/local.rules lab-snort

O que faz:

* -it: Abre o terminal do container de forma interativa.


* --network=host: Faz o container compartilhar a interface de rede diretamente com o Kali Linux para escutar os pacotes que passam pela maquina.


* -v ...: Cria um volume que espelha o arquivo local.rules da sua maquina fisica para dentro do container. Assim, qualquer alteracao na regra e refletida instantaneamente sem precisar rebuildar a imagem.



---

### 3. Validacao de Sintaxe das Regras

snort -T -c /etc/snort/rules/local.rules

O que faz: Executa o Snort no modo de teste (-T) carregando o arquivo de regras (-c). Garante que nao ha erros de digitacao ou sintaxe antes de colocar a rede em monitoramento.

---

### 4. Iniciar o Snort no Modo IDS (Deteccao)

snort -A console -q -c /etc/snort/rules/local.rules -i eth0

O que faz:

* -A console: Imprime os alertas disparados diretamente na tela do terminal.


* -q: Ativa o modo silencioso (quiet), omitindo mensagens de carregamento e banners.


* -i eth0: Define qual interface de rede sera monitorada pelo Snort.



---

## Simulacao de Ataques (Testes Praticos)

Executados a partir de um segundo terminal no Kali Linux para validar os disparos:

# 1. Testar Alerta de Ping (ICMP)

ping -c 2 127.0.0.1

# 2. Testar Alerta de Command Injection

curl "[suspicious link removed]"

# 3. Testar Alerta de Path Traversal

curl "[suspicious link removed]"

# 4. Testar Alerta de SQL Injection

curl "[http://127.0.0.1:80/index.php?id=1'or'1'='1](https://www.google.com/search?q=http://127.0.0.1:80/index.php%3Fid%3D1%27or%271%27%3D%271)"

# 5. Testar Alerta de Nmap SYN Scan

nmap -sS 127.0.0.1

# 6. Testar Alerta de Nmap Xmas Scan

nmap -sX 127.0.0.1

---

## Aprendizados Alcancados

* Analise Profunda de Pacotes (DPI): Compreensao pratica de como a clausula content: e seletores de cabecalho (flags:, itype:) filtram assinaturas maliciosas no payload.


* Conteineres na Ciberseguranca: Como isolar ferramentas de seguranca em ambientes reprodutiveis via Docker.


* Engenharia de Deteccao: Balanceamento entre criacao de assinaturas precisas contra ameacas conhecidas e monitoramento de varreduras de rede.
