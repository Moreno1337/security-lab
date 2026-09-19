[← Voltar ao índice](../../../../index.md)

# Challenge 03 — SSH from Kali to Ubuntu

## Objetivo

Estabelecer uma conexão SSH da Kali Linux para o Ubuntu Server através da rede privada `seclab`.

## Estado inicial

As máquinas estavam conectadas à mesma rede:

- **Kali Linux:** `192.168.50.10`
- **Ubuntu Server:** `192.168.50.20`

Antes de tentar a conexão, foi investigado se o Ubuntu possuía algum serviço escutando na porta TCP 22.

## Investigação

No Ubuntu foi utilizado:

`sudo ss -tulpn`

Inicialmente não havia nenhum socket TCP na porta `22` em estado `LISTEN`.

Também foi investigado o estado do serviço SSH através do `systemd`.

Foi identificado que o OpenSSH Server ainda não estava disponível para receber conexões.

## Configuração

O OpenSSH Server foi instalado no Ubuntu.

Após a instalação, uma nova inspeção mostrou a porta `22` em estado `LISTEN`:

- `0.0.0.0:22`
- `[::]:22`

Também foi observado que o `systemd` utilizava **socket activation**, permitindo que `ssh.socket` mantivesse o socket disponível e acionasse o serviço quando necessário.

## Conexão

A partir da Kali foi realizada a conexão:

`ssh johnny@192.168.50.20`

A conexão foi estabelecida com sucesso através da `seclab`.

## Host Key

No primeiro acesso, o cliente SSH informou que ainda não conhecia a identidade do servidor e apresentou sua fingerprint.

Após a confirmação, a host key foi registrada em `known_hosts`.

O comportamento demonstrou o mecanismo utilizado pelo SSH para lembrar a identidade de hosts previamente acessados e detectar possíveis mudanças em conexões futuras.

## Resultado

Foi comprovada comunicação em nível de aplicação entre:

`Kali → seclab → TCP/22 → Ubuntu → SSH`

O challenge também demonstrou a relação entre serviço, processo, socket e porta de rede, além do processo inicial de confiança da host key realizado pelo cliente SSH.

[← Voltar ao índice](../../../../index.md)
