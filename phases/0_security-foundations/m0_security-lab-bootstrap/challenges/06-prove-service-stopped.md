[← Voltar ao índice](../../../../index.md)

# Challenge 06 — Prove Service Stopped

## Objetivo

Interromper o serviço HTTP executado no Ubuntu Server e comprovar, a partir da Kali Linux, que ele deixou de estar disponível.

A intenção não era apenas parar o processo, mas obter evidências externas de que o estado do alvo realmente havia mudado.

## Estado inicial

Antes da interrupção, o Ubuntu possuía:

- SSH disponível na porta TCP `22`.
- HTTP disponível na porta TCP `8080`.

A partir da Kali, o Nmap identificava ambas as portas como abertas e o `curl` conseguia acessar o serviço HTTP.

## Interrupção

O servidor HTTP iniciado anteriormente com Python foi encerrado no Ubuntu.

Após a interrupção, o processo deixou de manter um socket escutando na porta `8080`.

## Validação com curl

A partir da Kali foi realizada uma nova tentativa de acesso:

`curl http://192.168.50.20:8080`

A conexão não pôde ser estabelecida.

Esse resultado era compatível com a hipótese de que o serviço havia sido interrompido, mas isoladamente não seria suficiente para determinar a causa exata da falha.

## Validação com Nmap

Um novo scan foi realizado contra o Ubuntu.

A porta `8080`, anteriormente identificada como aberta, deixou de aparecer como um serviço disponível.

A porta `22`, utilizada pelo SSH, continuou aberta.

Isso demonstrou que a mudança estava especificamente relacionada ao serviço HTTP e não à perda geral de conectividade com o Ubuntu.

## Comparação

Antes:

- `22/tcp` — open
- `8080/tcp` — open
- `curl` → resposta HTTP

Depois:

- `22/tcp` — open
- `8080/tcp` — não disponível
- `curl` → conexão não estabelecida

## Resultado

Foi comprovado através de múltiplas evidências que o serviço HTTP deixou de estar disponível após sua interrupção.

O challenge reforçou que uma mensagem de erro isolada não deve ser tratada automaticamente como prova de uma causa específica.

O processo utilizado foi:

`Hipótese → alteração controlada → observação → comparação → conclusão`

Também foi possível distinguir uma indisponibilidade específica de serviço de uma falha geral de conectividade com o host.

[← Voltar ao índice](../../../../index.md)
