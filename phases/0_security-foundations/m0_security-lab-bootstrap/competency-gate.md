[← Voltar ao índice](../../../index.md)

# Fase 0 — M0: Security Lab Bootstrap

## Competency Gate

### Objetivo

Validar se os conhecimentos e habilidades desenvolvidos durante o **M0 — Security Lab Bootstrap** foram realmente assimilados antes de avançar para o próximo módulo.

O Competency Gate foi realizado sem consulta inicial às anotações e priorizou a capacidade de:

- explicar os conceitos com palavras próprias;
- aplicar os conhecimentos em cenários diferentes dos experimentos originais;
- interpretar evidências;
- formular hipóteses;
- conduzir troubleshooting de maneira estruturada;
- operar e compreender a topologia do laboratório sem depender de tutorial.

O objetivo não foi avaliar memorização de comandos ou terminologia, mas verificar a existência de um **modelo mental funcional do laboratório construído durante o M0**.

---

## 1. Topologia do Security Lab

Foi solicitado explicar como as três máquinas estão conectadas entre si e à Internet.

Topologia avaliada:

```text
Kali
NAT:     10.0.2.15
seclab:  192.168.50.10

Ubuntu
NAT:     10.0.2.15
seclab:  192.168.50.20

Windows
NAT:     10.0.2.15
seclab:  192.168.50.30
```

Foi demonstrado o entendimento de que cada VM possui duas interfaces com responsabilidades distintas:

- **NAT:** acesso a redes externas/Internet;
- **Internal Network `seclab`:** comunicação privada entre os hosts do laboratório.

Também foi explicado corretamente que a `seclab` utiliza:

`192.168.50.0/24`

e que os três endereços `10.0.2.15` não entram em conflito porque cada VM utiliza seu próprio contexto NAT padrão fornecido pelo VirtualBox.

### Refinamento realizado

Inicialmente, ICMP foi associado à capacidade das máquinas de se comunicarem dentro da `seclab`.

Foi refinado que **ICMP é apenas um dos protocolos que podem utilizar essa conectividade**, tendo sido utilizado através do `ping` para testá-la.

A comunicação da rede não depende de ICMP.

**Resultado:** Aprovado.

---

## 2. Routing e escolha de interface

Foi apresentado o seguinte cenário na Kali:

```text
eth0 → 10.0.2.15/24
       default via 10.0.2.2

eth1 → 192.168.50.10/24
```

E a requisição:

```bash
curl http://192.168.50.20:8080
```

Foi solicitado explicar como o Linux decide utilizar `eth1` em vez de `eth0`.

Foi demonstrado o entendimento de que a configuração de `192.168.50.10/24` cria uma rota diretamente conectada para:

`192.168.50.0/24`

Conceitualmente:

```text
192.168.50.0/24 dev eth1
default via 10.0.2.2 dev eth0
```

Como `192.168.50.20` pertence à rede `192.168.50.0/24`, o tráfego é enviado através da `eth1`.

Também foi explicado corretamente que nenhum gateway é necessário, pois Kali e Ubuntu pertencem à mesma rede diretamente conectada.

**Resultado:** Aprovado.

---

## 3. ARP e entrega Ethernet

Foi solicitado explicar como a Kali consegue entregar tráfego ao Ubuntu conhecendo inicialmente apenas:

`192.168.50.20`

Foi identificado corretamente que, para construir o frame Ethernet destinado ao Ubuntu, a Kali precisa conhecer seu **MAC address**.

O processo explicado foi:

```text
Kali conhece o IP 192.168.50.20
        ↓
ARP Request
"Quem possui 192.168.50.20?"
        ↓
Ubuntu recebe o broadcast
        ↓
ARP Reply
"192.168.50.20 corresponde ao meu MAC"
        ↓
Kali aprende IP → MAC
        ↓
Frame Ethernet pode ser endereçado
```

### Refinamento realizado

O funcionamento do mecanismo foi explicado corretamente, porém inicialmente houve uma confusão entre os nomes **ARP** e **ICMP Echo Request/Reply**.

Foi consolidada a distinção:

```text
ARP
IP → MAC

ICMP
Echo Request ↔ Echo Reply
```

Também foi relacionado o processo à tabela/cache de vizinhos observável através de:

```bash
ip neigh
```

**Resultado:** Aprovado após refinamento de terminologia.

---

## 4. Fluxo de um ping

Foi solicitado reconstruir o que acontece ao executar:

```bash
ping 192.168.50.20
```

considerando que a Kali ainda não possui uma entrada ARP referente ao Ubuntu.

Foi demonstrado corretamente o fluxo:

```text
Routing
   ↓
Identificação da eth1
   ↓
ARP Request
   ↓
ARP Reply
   ↓
IP → MAC conhecido
   ↓
ICMP Echo Request
   ↓
Ubuntu recebe
   ↓
ICMP Echo Reply
   ↓
Kali recebe resposta
```

Também foi identificado que o Ubuntu pode aprender informações sobre o remetente através da comunicação ARP.

**Resultado:** Aprovado.

---

## 5. Troubleshooting de serviço inacessível

Foi apresentado o cenário:

```text
ping 192.168.50.20
→ funciona

curl http://192.168.50.20:8080
→ falha
```

Foi solicitado determinar o que poderia e o que não poderia ser concluído.

Foi identificado corretamente que:

- existe evidência de comunicação IP/ICMP entre Kali e Ubuntu;
- a falha do `curl` não prova que toda a comunicação entre os hosts está quebrada;
- a falha também não prova, isoladamente, que o serviço HTTP está parado.

Foi proposta uma investigação utilizando diferentes evidências, incluindo:

- teste a partir de outro host;
- SSH;
- inspeção das portas locais;
- identificação do processo;
- Nmap externo;
- testes locais no próprio Ubuntu;
- análise do endereço utilizado para bind;
- análise de firewall.

### Refinamento realizado

Inicialmente foi considerada a possibilidade de reiniciar o serviço relativamente cedo durante a investigação.

Foi reforçada a importância de **observar antes de alterar**.

Por exemplo:

```text
127.0.0.1:8080
```

e:

```text
0.0.0.0:8080
```

podem produzir comportamentos externos diferentes mesmo com o processo ativo.

A metodologia consolidada foi:

```text
Observar
   ↓
Formular hipótese
   ↓
Buscar evidência
   ↓
Alterar uma variável
   ↓
Testar novamente
   ↓
Comparar
```

**Resultado:** Aprovado após refinamento da metodologia.

---

## 6. NAT vs Internal Network

Foi apresentado um cenário onde as interfaces NAT das três VMs seriam removidas, permanecendo somente:

```text
Kali     192.168.50.10/24 ─┐
Ubuntu   192.168.50.20/24 ──┼── seclab
Windows  192.168.50.30/24 ──┘
```

Sem default gateway.

Foi solicitado prever o comportamento das seguintes comunicações:

```text
Kali → Ubuntu
Kali → Windows
Kali → 8.8.8.8
Kali → google.com
```

Foi demonstrado corretamente que:

- Kali → Ubuntu continuaria funcionando;
- Kali → Windows continuaria funcionando;
- Kali → `8.8.8.8` deixaria de funcionar por ausência de rota para redes externas/default gateway;
- Kali → `google.com` também perderia a conectividade externa e introduziria adicionalmente a dependência de resolução DNS.

Foi demonstrada a distinção entre:

- comunicação dentro de uma rede diretamente conectada;
- roteamento para redes externas;
- resolução DNS.

**Resultado:** Aprovado.

---

## 7. Firewall e comunicação assimétrica

Foi revisado o comportamento observado anteriormente:

```text
Windows → Ubuntu    OK
Windows → Kali      OK

Ubuntu → Windows    FAIL
Kali → Windows      FAIL
```

Foi solicitado explicar por que comunicação em uma direção não garante comunicação no sentido contrário.

Foi demonstrado corretamente que cada host pode possuir políticas próprias de firewall e que tráfego iniciado localmente e tráfego recebido como uma nova comunicação inbound podem ser tratados de maneira diferente.

Também foi analisada a diferença entre:

```text
Desabilitar completamente o firewall
```

e:

```text
Habilitar somente ICMPv4 Echo Request inbound
```

Foram identificadas duas vantagens da segunda abordagem:

1. menor exposição e respeito ao princípio do menor privilégio;
2. experimento mais controlado e evidência mais específica.

### Refinamento realizado

Houve uma pequena troca entre **Echo Request** e **Echo Reply**.

Para:

```text
Kali → Windows
```

o Windows precisa aceitar o **ICMP Echo Request inbound** e posteriormente gerar o Echo Reply.

**Resultado:** Aprovado após refinamento de terminologia.

---

## 8. Snapshots e recuperação

Foi apresentado um cenário onde diversas alterações quebrariam a VM Ubuntu após a criação do snapshot:

`M0 - Ubuntu Baseline`

Foi solicitado explicar o comportamento esperado após restaurá-lo e diferenciá-lo de um backup.

Foi demonstrado corretamente que a restauração retorna a VM ao estado representado pelo snapshot, descartando as alterações posteriores caso o estado atual não seja preservado separadamente.

Também foi construída corretamente a distinção conceitual:

```text
Snapshot
→ rollback rápido
→ ligado à VM/infraestrutura do hypervisor
→ útil durante experimentos
```

```text
Backup
→ cópia independente
→ recuperação após perda da infraestrutura original
→ armazenamento preferencialmente separado
```

Modelo mental utilizado:

> Instalei algo e quebrei a VM → Snapshot.

> O datacenter foi perdido junto com as máquinas → Backup.

### Refinamento realizado

Foi evitada a definição de que o snapshot simplesmente "sobrescreve" a VM, pois a implementação interna pode variar.

O conceito adotado foi:

> A VM é revertida ao estado representado pelo snapshot.

**Resultado:** Aprovado.

---

## 9. Virtual Machines vs Containers

Foi apresentada a proposta hipotética de substituir:

```text
Kali VM
Ubuntu VM
Windows VM
```

por três containers Docker.

Foi solicitado explicar por que as duas arquiteturas não seriam equivalentes para os objetivos do Security Lab.

Foi demonstrado o entendimento de que:

- uma VM representa uma máquina virtualizada;
- cada VM possui seu próprio sistema operacional/kernel;
- containers isolam workloads utilizando o kernel do host;
- múltiplos containers podem compartilhar o mesmo kernel.

A analogia utilizada foi:

> VM é como uma casa independente.

> Container é como um apartamento que compartilha a infraestrutura do prédio.

### Refinamento realizado

Inicialmente foi sugerido que containers não permitiriam construir uma topologia de rede semelhante.

Foi refinado que containers também podem possuir:

- redes virtuais;
- interfaces;
- endereços IP;
- NAT;
- isolamento de rede.

A diferença fundamental não está na possibilidade de criar networking virtual, mas no **nível de abstração e isolamento**.

Para o Security Lab, VMs são mais adequadas porque permitem estudar sistemas operacionais completos e independentes.

**Resultado:** Aprovado após refinamento.

---

## 10. Cenário final de troubleshooting

Como etapa final, foi apresentado o cenário:

```text
Kali
192.168.50.10

Ubuntu
192.168.50.20

Windows
192.168.50.30
```

Resultados observados:

```text
ping 192.168.50.20       → funciona
ssh johnny@192.168.50.20 → funciona
curl 192.168.50.20:8080  → falha

ping 192.168.50.30       → falha

ping 8.8.8.8             → funciona
ping google.com          → funciona
```

O objetivo era conduzir uma investigação sem assumir previamente as causas.

### Ubuntu :8080

Foi identificado que:

- ICMP com Ubuntu funciona;
- SSH/TCP 22 funciona;
- somente a tentativa contra `8080` apresenta falha entre as evidências fornecidas.

A investigação proposta incluiu:

```text
Nmap externo
    ↓
SSH no Ubuntu
    ↓
Inspeção dos sockets
    ↓
Verificação da porta 8080
    ↓
Identificação do processo
    ↓
Verificação do listening address/bind
    ↓
Teste local via localhost
    ↓
Novas hipóteses conforme as evidências
```

### Windows

Para o Windows foi proposta uma investigação começando pelas condições mais fundamentais:

```text
VM está ligada?
    ↓
vNIC seclab existe?
    ↓
Interface está ativa?
    ↓
192.168.50.30 está configurado?
    ↓
Windows consegue pingar Kali?
    ↓
Qual é o network profile?
    ↓
Quais regras de firewall estão aplicadas?
```

Foi demonstrada a capacidade de utilizar comunicação no sentido inverso como evidência adicional para identificar possíveis comportamentos assimétricos.

### Refinamentos realizados

Foi corrigido o uso do termo:

`routing da porta`

para:

`listening address / bind`

Também foi reforçada novamente a diferença entre **ICMP Echo Request** e **ICMP Echo Reply**.

**Resultado:** Aprovado.

---

# Avaliação Final

Durante o Competency Gate foram demonstrados conhecimentos funcionais sobre:

- virtualização;
- hypervisors;
- máquinas virtuais;
- containers;
- interfaces de rede;
- IPv4;
- subnet `/24`;
- NAT;
- Internal Network;
- routing;
- default gateway;
- ARP;
- MAC addresses;
- ICMP;
- DNS;
- sockets;
- portas;
- processos;
- SSH;
- HTTP;
- Nmap;
- firewall;
- network profiles do Windows;
- snapshots;
- backups;
- troubleshooting orientado por evidências.

Também foi demonstrada capacidade de relacionar esses conceitos em vez de tratá-los isoladamente.

Exemplo:

```text
Aplicação
    ↓
Serviço / Processo
    ↓
Socket / Porta
    ↓
TCP/IP
    ↓
Routing
    ↓
ARP / Ethernet
    ↓
Interface
    ↓
Rede virtual
    ↓
Host de destino
```

---

## Pontos de refinamento identificados

O Competency Gate identificou algumas pequenas confusões que foram corrigidas durante a avaliação:

| Confusão inicial              | Distinção consolidada                                                           |
| ----------------------------- | ------------------------------------------------------------------------------- |
| ARP × ICMP                    | ARP resolve IP → MAC; ICMP inclui Echo Request/Reply                            |
| Echo Request × Echo Reply     | Request é enviado ao alvo; Reply é sua resposta                                 |
| Routing × Bind                | Routing decide o caminho do tráfego; bind define onde o serviço aceita conexões |
| Ping × conectividade completa | Ping fornece evidência sobre ICMP, não sobre todos os protocolos/serviços       |
| Container networking          | Containers também podem possuir networking virtual                              |
| Restart precoce               | Investigar e coletar evidências antes de alterar o ambiente                     |

Os pontos identificados foram tratados como refinamentos de terminologia ou precisão, sem indicar deficiência no modelo mental fundamental.

---

## Metodologia demonstrada

Um dos principais resultados do M0 foi a consolidação da metodologia:

```text
Hipótese
   ↓
Investigação
   ↓
Evidência
   ↓
Experimento controlado
   ↓
Comparação
   ↓
Conclusão
```

Durante os cenários apresentados, foi demonstrada capacidade de evitar conclusões maiores do que as evidências disponíveis permitem.

Também foi demonstrada preferência por alterações pequenas e controladas em vez de modificar múltiplas variáveis simultaneamente.

---

# Resultado

O objetivo do Competency Gate era verificar se o Security Lab poderia ser **operado, explicado e investigado sem dependência de tutorial**.

Esse requisito foi demonstrado.

As respostas apresentaram compreensão suficiente da topologia construída, dos mecanismos básicos envolvidos na comunicação entre os hosts e da metodologia necessária para investigar problemas dentro do ambiente.

**Competency Gate: APROVADO**

**M0 — Security Lab Bootstrap: CONCLUÍDO**

[← Voltar ao índice](../../../index.md)
