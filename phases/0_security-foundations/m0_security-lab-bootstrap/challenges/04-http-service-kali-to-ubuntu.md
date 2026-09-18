# Challenge 04 — Access HTTP Service from Kali

## Objetivo

Executar um serviço HTTP no Ubuntu Server e acessá-lo remotamente a partir da Kali Linux através da rede privada `seclab`.

## Topologia

As máquinas envolvidas foram:

- **Kali Linux:** `192.168.50.10`
- **Ubuntu Server:** `192.168.50.20`

A comunicação ocorreu exclusivamente através da rede privada `192.168.50.0/24`.

## Serviço HTTP

No Ubuntu foi iniciado um servidor HTTP temporário utilizando Python:

`python3 -m http.server 8080`

O serviço passou a escutar conexões TCP na porta `8080`.

## Validação local

Foi verificado que o processo estava executando e que existia um socket em estado `LISTEN` associado à porta `8080`.

Isso permitiu relacionar:

`Processo → Socket → Porta TCP 8080`

## Acesso remoto

A partir da Kali foi utilizado `curl` para realizar uma requisição HTTP ao Ubuntu:

`curl http://192.168.50.20:8080`

O servidor respondeu à requisição, comprovando que o serviço estava acessível remotamente através da `seclab`.

## Resultado

Foi demonstrado que possuir conectividade IP entre duas máquinas não é o mesmo que possuir um serviço de aplicação disponível.

Para que a Kali conseguisse acessar o servidor HTTP foi necessário que:

1. Existisse conectividade entre Kali e Ubuntu.
2. Um processo estivesse executando no Ubuntu.
3. O processo estivesse escutando em uma porta de rede.
4. A porta estivesse acessível pela interface utilizada na `seclab`.
5. A Kali realizasse uma conexão TCP com essa porta utilizando o protocolo HTTP.

O challenge demonstrou comunicação em nível de aplicação entre:

`Kali → seclab → TCP/8080 → Ubuntu → HTTP Server`
