[← Voltar ao índice](../../../../index.md)

# Challenge 01 — Discover Machine IPs

## Objetivo

Identificar os endereços IPv4 das três máquinas que compõem o Security Lab e compreender a função de cada interface de rede.

## Máquinas

### Kali Linux

- **NAT:** `10.0.2.15/24`
- **seclab:** `192.168.50.10/24`

### Ubuntu Server

- **NAT:** `10.0.2.15/24`
- **seclab:** `192.168.50.20/24`

### Windows 11 Pro

- **NAT:** `10.0.2.15/24`
- **seclab:** `192.168.50.30/24`

## Topologia identificada

Cada VM possui duas interfaces de rede com responsabilidades diferentes:

- **NIC 1 — NAT:** utilizada para acesso à Internet.
- **NIC 2 — Internal Network `seclab`:** utilizada para comunicação privada entre as máquinas do laboratório.

A rede privada utilizada pelo laboratório é:

`192.168.50.0/24`

## Resultado

Foi possível identificar corretamente os endereços das três máquinas e distinguir as interfaces utilizadas para Internet das interfaces pertencentes à rede privada do laboratório.

Também foi observado que as três VMs podem possuir simultaneamente o endereço `10.0.2.15` em suas interfaces NAT sem conflito, pois o NAT padrão do VirtualBox fornece um contexto NAT separado para cada VM.

[← Voltar ao índice](../../../../index.md)
