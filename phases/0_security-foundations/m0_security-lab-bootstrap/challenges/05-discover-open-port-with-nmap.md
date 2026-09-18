# Challenge 05 — Discover Open Port with Nmap

## Objetivo

Utilizar a Kali Linux para investigar remotamente o Ubuntu Server e identificar quais portas estavam abertas, sem depender apenas da inspeção local do servidor.

## Cenário

No Ubuntu Server existiam dois serviços acessíveis pela rede:

- SSH na porta TCP `22`.
- HTTP temporário na porta TCP `8080`.

O alvo possuía o endereço:

`192.168.50.20`

## Investigação

A partir da Kali Linux foi utilizado o Nmap para realizar o scan do Ubuntu Server.

O scan permitiu observar o alvo externamente e identificar as portas que estavam aceitando conexões.

Foram encontradas:

- `22/tcp` — open
- `8080/tcp` — open

## Interpretação

Uma porta reportada como `open` pelo Nmap indica que existe algo no host aceitando conexões naquela porta.

Isso fornece uma perspectiva diferente de ferramentas executadas localmente no servidor.

No Ubuntu, ferramentas como `ss` permitem observar os sockets locais.

Na Kali, o Nmap permite investigar quais portas do Ubuntu são alcançáveis pela rede.

## Correlação

As evidências obtidas puderam ser relacionadas:

`Processo HTTP no Ubuntu → Socket TCP → Porta 8080 → Detectada remotamente pela Kali`

Da mesma forma, o serviço SSH disponível na porta `22` também foi identificado pelo scan.

## Resultado

Foi possível identificar remotamente as portas abertas no Ubuntu Server utilizando Nmap.

O challenge demonstrou a diferença entre observar o estado de uma máquina localmente e investigar sua superfície de rede a partir de outro host.

Também introduziu o Nmap como ferramenta para descoberta e enumeração de serviços acessíveis em um alvo.
