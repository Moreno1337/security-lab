# Challenge 02 — Test Lab Connectivity

## Objetivo

Validar a comunicação entre as máquinas conectadas à rede privada `seclab` e comprovar que a topologia configurada permite comunicação direta entre os hosts.

## Topologia

A rede utilizada pelo laboratório é:

`192.168.50.0/24`

Hosts:

- **Kali Linux:** `192.168.50.10`
- **Ubuntu Server:** `192.168.50.20`
- **Windows 11 Pro:** `192.168.50.30`

## Validação

A conectividade foi testada utilizando ICMP através do comando `ping`.

Foi comprovada comunicação entre Kali e Ubuntu e, posteriormente, entre Windows e as duas máquinas Linux.

Durante os testes foi identificado um comportamento assimétrico:

- Windows → Kali/Ubuntu: funcionava.
- Kali/Ubuntu → Windows: não recebia resposta.

## Investigação

A interface `seclab` do Windows estava classificada com o perfil de rede **Public**.

Foi investigado o Microsoft Defender Firewall e identificada uma regra de entrada para:

`Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)`

A regra aplicável aos perfis **Private/Public** estava desabilitada.

A hipótese formulada foi que o firewall estava impedindo ICMP Echo Requests iniciados pelas outras máquinas contra o Windows.

## Experimento

Foi habilitada somente a regra de entrada responsável por permitir `ICMPv4 Echo Request` nos perfis Private/Public, mantendo o restante da configuração inalterado.

Após a alteração:

- Ubuntu → Windows: sucesso.
- Kali → Windows: sucesso.
- Packet loss: `0%`.

## Resultado

A comunicação entre as três máquinas da `seclab` foi comprovada.

O experimento também demonstrou que uma falha de `ping` não prova, isoladamente, ausência de conectividade de rede. Firewalls e outras políticas podem permitir determinado tráfego em uma direção e bloqueá-lo na direção oposta.

A alteração controlada de uma única regra do firewall forneceu evidência de que o bloqueio de ICMP inbound no Windows era responsável pelo comportamento observado.
