[← Voltar ao índice](../../../index.md)

# Fase 0 — M0: Security Lab Bootstrap

## Sessão 04 — Windows, validação do laboratório e Competency Gate

### Objetivo da sessão

Finalizar a construção do laboratório previsto no M0 através da instalação e configuração da terceira máquina, **Windows 11 Pro**, validar a comunicação completa entre os hosts, configurar mecanismos de recuperação através de snapshots e realizar o **Knowledge Check + Competency Gate** do módulo.

Ao final da sessão, o laboratório deveria possuir três sistemas independentes conectados através da rede privada `seclab`:

- Kali Linux
- Ubuntu Server
- Windows 11 Pro

---

## 1. Planejamento da VM Windows

Foi iniciada a criação da terceira máquina virtual do Security Lab.

Inicialmente foi considerada a utilização do **Windows 11 Home**, porém foi identificado que essa edição possui limitações que poderiam prejudicar experimentos futuros envolvendo domínio e Active Directory.

Por esse motivo, foi escolhida a edição:

`Windows 11 Pro`

Também foi discutida a possibilidade de utilizar **Windows Server**.

Foi concluído que Windows Server não deveria substituir o Windows 11 Pro, pois representam papéis diferentes dentro de um ambiente.

Em módulos futuros, poderá ser adicionada uma VM Windows Server exercendo o papel de servidor/Domain Controller, enquanto o Windows 11 Pro poderá atuar como workstation.

### Configuração da VM

| Recurso       | Configuração   |
| ------------- | -------------- |
| RAM           | 8 GB           |
| vCPU          | 8              |
| Disco virtual | 120 GB         |
| Alocação      | Dinâmica       |
| Firmware      | UEFI           |
| Sistema       | Windows 11 Pro |
| Hypervisor    | VirtualBox     |

Imagem utilizada:

`Win11_25H2_English_x64_v2.iso`

---

## 2. Instalação do Windows 11 Pro

Foi utilizada a instalação manual através da ISO oficial do Windows 11.

A mesma ISO disponibilizava múltiplas edições, incluindo:

- Windows 11 Pro
- Windows 11 Pro N
- Windows 11 Pro Education
- Windows 11 Pro for Workstations

Foi selecionado o **Windows 11 Pro** convencional.

Durante a instalação foi utilizada a opção:

`I don't have a product key`

Como se trata de uma máquina destinada exclusivamente ao laboratório, a ativação não era necessária para os objetivos atuais.

---

## 3. Criação de conta local

Durante o OOBE do Windows foi selecionada a configuração voltada para **work or school**.

Nas opções de login foi utilizada:

`Domain join instead`

Essa opção não realizou imediatamente a entrada da máquina em um domínio.

Ela permitiu continuar a instalação utilizando uma **conta local**, preservando a possibilidade de realizar posteriormente o processo real de domain join como parte dos estudos de Windows e Active Directory.

Foi criada a conta local:

`Johnny`

---

## 4. Troubleshooting da instalação

Durante a instalação, a VM permaneceu por vários minutos em uma tela preta.

Inicialmente existia a possibilidade de o processo ter travado.

Após uma reinicialização manual da VM, a instalação continuou normalmente para o OOBE.

A evidência observada indicou que o processo provavelmente estava aguardando ou falhou ao concluir corretamente uma reinicialização durante a instalação, sem que houvesse corrupção do sistema instalado.

---

## 5. Troubleshooting de vídeo

Após a instalação, o Windows permanecia limitado a aproximadamente:

`1024x768`

A resolução não podia ser alterada normalmente.

Como a VM já possuía `128 MB` de memória de vídeo, foi investigado o driver utilizado pelo Windows.

No Device Manager foi identificado:

`Microsoft Basic Display Adapter`

Isso indicava que o sistema estava utilizando um driver gráfico genérico.

### Guest Additions

Foi utilizada no VirtualBox a opção:

`Devices → Insert Guest Additions CD Image`

A ISO foi montada como uma unidade de CD virtual dentro do Windows.

Foi executado o instalador:

`VBoxWindowsAdditions-amd64`

Após a instalação e reinicialização, o Device Manager passou a mostrar:

`VirtualBox Graphics Adapter (WDDM)`

A partir desse momento, resize dinâmico e fullscreen passaram a funcionar corretamente.

### Evidência

O troubleshooting produziu a seguinte sequência:

```text
Microsoft Basic Display Adapter
        ↓
Hipótese: ausência dos drivers do VirtualBox
        ↓
Instalação do Guest Additions
        ↓
VirtualBox Graphics Adapter (WDDM)
        ↓
Resize e resolução funcionando
```

---

## 6. Baseline de rede do Windows

Antes de conectar o Windows à `seclab`, foi analisada sua configuração inicial utilizando apenas a interface NAT.

Foram identificadas as seguintes informações:

- Hostname: `secLab`
- IPv4: `10.0.2.15`
- Máscara: `255.255.255.0`
- DHCP: habilitado
- Default Gateway: `10.0.2.2`
- DHCP Server: `10.0.2.2`

Também foram realizados testes de conectividade.

### Teste de Internet

`ping 8.8.8.8`

Resultado:

- conectividade IP externa funcionando.

### Teste de DNS

`ping google.com`

O nome foi resolvido para um endereço IP e o host respondeu.

Resultado:

- resolução DNS funcionando;
- conectividade externa funcionando.

---

## 7. Adição do Windows à rede seclab

Foi adicionada uma segunda vNIC à VM Windows através do VirtualBox.

Configuração:

`Internal Network → seclab`

No Windows, a nova interface foi configurada manualmente com:

`192.168.50.30/24`

Sem default gateway.

A configuração final ficou:

| Máquina        | seclab             |
| -------------- | ------------------ |
| Kali Linux     | `192.168.50.10/24` |
| Ubuntu Server  | `192.168.50.20/24` |
| Windows 11 Pro | `192.168.50.30/24` |

Rede:

`192.168.50.0/24`

---

## 8. Routing Table do Windows

Após configurar `192.168.50.30/24`, foi analisada a tabela de roteamento do Windows.

Foram identificadas duas rotas conceitualmente importantes:

```text
0.0.0.0/0 → gateway 10.0.2.2
192.168.50.0/24 → On-link via 192.168.50.30
```

Isso demonstrou que:

- tráfego destinado à Internet continua utilizando a interface NAT e seu default gateway;
- tráfego destinado à `192.168.50.0/24` utiliza diretamente a interface conectada à `seclab`.

Como os hosts da `seclab` pertencem à mesma subnet, não é necessário gateway para comunicação entre eles.

---

## 9. Validação inicial da conectividade

A partir do Windows foram realizados testes contra as outras duas máquinas:

```text
Windows → Ubuntu
Windows → Kali
```

Ambos funcionaram.

Porém, ao realizar o caminho contrário:

```text
Ubuntu → Windows
```

o resultado foi:

`100% packet loss`

Foi identificado, portanto, um comportamento **assimétrico**.

A existência de comunicação em uma direção não era evidência suficiente para afirmar que a comunicação no sentido contrário também deveria funcionar.

---

## 10. Investigação do Windows Defender Firewall

Foi levantada a hipótese de que o Windows Defender Firewall poderia estar bloqueando os **ICMP Echo Requests inbound**.

A interface `seclab` do Windows estava classificada como:

`Public network`

Foram então investigadas as regras inbound relacionadas a ICMP.

Foi encontrada a regra:

`Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)`

A regra aplicável aos perfis **Private/Public** estava desabilitada.

### Experimento controlado

Em vez de desabilitar completamente o firewall, foi habilitada somente a regra responsável pelo comportamento que estava sendo investigado.

Após a alteração:

```text
Ubuntu → Windows
```

passou a funcionar.

Também foi testado:

```text
Kali → Windows
```

com sucesso.

### Aprendizado

O experimento demonstrou dois conceitos importantes.

Primeiro:

> Falha de `ping` não significa necessariamente ausência de conectividade de rede.

Segundo:

> Durante troubleshooting, alterar somente a variável diretamente relacionada à hipótese produz evidências melhores do que realizar mudanças amplas no ambiente.

Desabilitar completamente o firewall poderia indicar que ele estava envolvido, mas não identificaria de maneira tão precisa qual política estava causando o comportamento.

---

## 11. Topologia final do Security Lab

Ao final da configuração, o laboratório ficou estruturado da seguinte maneira:

```text
                     INTERNET
                        │
             ┌──────────┼──────────┐
             │          │          │
            NAT        NAT        NAT
             │          │          │
           Kali       Ubuntu     Windows
        10.0.2.15   10.0.2.15   10.0.2.15
             │          │          │
      192.168.50.10  .20          .30
             │          │          │
             └──────────┼──────────┘
                        │
              Internal Network
                    seclab
               192.168.50.0/24
```

Cada VM possui seu próprio contexto NAT no VirtualBox.

Por esse motivo, as três máquinas podem utilizar `10.0.2.15` simultaneamente sem conflito.

Na `seclab`, cada máquina possui um endereço único e estático.

---

## 12. Snapshots

Foi estudado o conceito de **snapshot** e sua diferença em relação a um backup.

Um snapshot representa um estado conhecido de uma VM que pode ser utilizado para retornar rapidamente a uma condição anterior após experimentos ou alterações destrutivas.

### Snapshot vs Backup

O snapshot é especialmente útil para:

- experimentos;
- alterações de configuração;
- instalações potencialmente destrutivas;
- retorno rápido a um estado conhecido.

Um backup possui outro objetivo: manter uma cópia independente dos dados ou da máquina para recuperação mesmo após perda da infraestrutura original.

Modelo mental:

> Quebrei a VM durante um experimento → Snapshot.

> Perdi a infraestrutura onde a VM existia → Backup.

---

## 13. Teste de restauração de snapshot

Foi criado no Ubuntu o snapshot:

`M0 - Ubuntu Baseline`

Após sua criação foi adicionado o arquivo:

`~/snapshot-test.txt`

A existência do arquivo foi confirmada.

Em seguida, a VM foi desligada e restaurada para o snapshot anterior.

Após o restore, o arquivo `snapshot-test.txt` não existia mais.

O experimento comprovou na prática que o estado da VM havia sido revertido para o estado registrado pelo snapshot.

---

## 14. Baselines do laboratório

Após validar o funcionamento dos snapshots, foram criados snapshots baseline das três máquinas:

- `M0 - Kali Baseline`
- `M0 - Ubuntu Baseline`
- `M0 - Windows Baseline`

Esses snapshots representam o estado funcional do laboratório ao final do M0 e poderão ser utilizados como pontos de recuperação durante experimentos futuros.

---

## 15. Repositório de estudos

Foi criado um repositório Git para registrar o histórico de aprendizado do laboratório.

Repositório:

`security-lab`

Estrutura inicial:

```text
security-lab/
└── phases/
    └── 0_security-foundations/
        └── m0_security-lab-bootstrap/
            ├── session-01.md
            ├── session-02.md
            ├── session-03.md
            └── session-04.md
```

Foi decidido criar novas pastas e estruturas apenas conforme forem necessárias, evitando adicionar antecipadamente uma organização que ainda não possui conteúdo real.

Também foi definido que o repositório não deve conter:

- senhas;
- tokens;
- private keys;
- credentials;
- arquivos `.env`;
- informações pessoais desnecessárias.

---

## 16. Documentação dos Challenges

Os challenges realizados durante o M0 foram separados em arquivos próprios para preservar não apenas os comandos utilizados, mas também o objetivo, raciocínio e evidências de cada experimento.

Foram documentados:

1. `challenge-01-discover-machine-ips.md`
2. `challenge-02-test-lab-connectivity.md`
3. `challenge-03-ssh-kali-to-ubuntu.md`
4. `challenge-04-http-service-kali-to-ubuntu.md`
5. `challenge-05-discover-open-port-with-nmap.md`
6. `challenge-06-prove-service-stopped.md`

Essa documentação passa a servir como histórico dos experimentos práticos realizados durante o módulo.

---

## 17. Docker vs Virtual Machines

Foi revisada a diferença conceitual entre containers e máquinas virtuais.

O modelo mental utilizado foi:

- **VM:** equivalente a uma casa independente.
- **Container:** equivalente a um apartamento que compartilha a infraestrutura do prédio.

Uma VM virtualiza uma máquina e possui seu próprio sistema operacional e kernel.

Containers isolam workloads, mas compartilham o kernel do host.

Modelo simplificado:

```text
Virtual Machines

Hardware
└── Host OS
    └── Hypervisor
        ├── VM → Guest OS → Kernel → Aplicação
        └── VM → Guest OS → Kernel → Aplicação
```

```text
Containers

Hardware
└── Host OS / Kernel
    └── Container Runtime
        ├── Container → Aplicação + dependências
        └── Container → Aplicação + dependências
```

Também foi observado que containers podem possuir suas próprias redes virtuais, interfaces, endereços e NAT.

A diferença fundamental não é a existência de networking virtual, mas o nível de abstração e isolamento utilizado.

Para o Security Lab, VMs são adequadas porque permitem estudar hosts e sistemas operacionais independentes.

---

## 18. Knowledge Check — Routing, ARP e ICMP

Durante o Competency Gate foi revisado o fluxo de comunicação dentro da `seclab`.

Ao enviar tráfego de Kali para Ubuntu:

```text
192.168.50.10 → 192.168.50.20
```

o sistema consulta sua tabela de roteamento e identifica que `192.168.50.20` pertence à rede diretamente conectada:

`192.168.50.0/24`

A interface correspondente é utilizada sem necessidade de gateway.

### ARP

Antes de entregar um frame Ethernet para outro host da mesma rede, é necessário conhecer seu MAC address.

Foi revisado o funcionamento do **ARP — Address Resolution Protocol**.

Modelo:

```text
Kali conhece:
192.168.50.20

        ↓

ARP Request
"Quem possui 192.168.50.20?"

        ↓

Ubuntu responde com seu MAC

        ↓

Kali associa:
IP → MAC

        ↓

Frame Ethernet pode ser endereçado
```

A associação pode ser observada através de:

`ip neigh`

### ICMP

Também foi consolidada a diferença entre ARP e ICMP.

ARP resolve:

`IP → MAC`

Enquanto o `ping` utiliza:

```text
ICMP Echo Request
        ↓
ICMP Echo Reply
```

Portanto, em um primeiro `ping` sem uma entrada ARP previamente conhecida, o modelo simplificado é:

```text
Routing
   ↓
ARP
   ↓
MAC conhecido
   ↓
ICMP Echo Request
   ↓
ICMP Echo Reply
```

---

## 19. Troubleshooting orientado por evidências

O Competency Gate também utilizou cenários hipotéticos de troubleshooting.

Um dos cenários apresentava:

```text
ping Ubuntu → sucesso
SSH Ubuntu  → sucesso
HTTP :8080  → falha
```

Foi concluído que essas evidências não permitem afirmar imediatamente que o serviço HTTP está parado.

A investigação poderia incluir:

- comparação a partir de outro host;
- Nmap externo;
- inspeção de sockets locais;
- identificação do processo associado à porta;
- endereço utilizado no bind;
- teste através de `localhost`;
- inspeção de firewall;
- comparação entre diferentes evidências.

Foi reforçado que um serviço em:

`127.0.0.1:8080`

possui comportamento diferente de:

`0.0.0.0:8080`

O primeiro está associado somente ao loopback, enquanto o segundo aceita conexões através das interfaces IPv4 disponíveis.

---

## 20. Metodologia consolidada

Ao longo do M0 foi consolidada uma metodologia para investigação técnica:

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

Foi reforçada a importância de não transformar uma observação em uma conclusão maior do que ela realmente suporta.

Exemplos:

> `ping` funcionando demonstra resposta a ICMP, não que todos os serviços da máquina estejam funcionando.

> `curl` falhando não demonstra sozinho que o serviço está parado.

> Uma porta inacessível não significa necessariamente ausência de conectividade com o host.

> Comunicação funcionando em uma direção não garante que o caminho inverso seja permitido.

---

## 21. Competency Gate do M0

Ao final da sessão foi realizado o **Knowledge Check + Competency Gate** do M0 sem consulta inicial às anotações.

Foram avaliados conceitos como:

- topologia do laboratório;
- NAT;
- Internal Network;
- endereçamento IPv4;
- subnet `/24`;
- interfaces de rede;
- routing table;
- default gateway;
- ARP;
- MAC addresses;
- ICMP;
- conectividade entre hosts;
- sockets e portas;
- SSH;
- HTTP;
- Nmap;
- firewall;
- snapshots;
- backup;
- containers;
- virtual machines;
- troubleshooting baseado em evidências.

O objetivo não era memorizar terminologia ou comandos, mas demonstrar capacidade de **explicar e operar o laboratório sem depender de um tutorial**.

Durante a avaliação foram identificadas pequenas confusões de terminologia, como:

- ARP × ICMP;
- Echo Request × Echo Reply;
- routing × bind.

Após os refinamentos, os modelos conceituais correspondentes foram demonstrados corretamente.

---

## 22. Resultado final

Ao final da sessão, o Security Lab possui:

- Kali Linux funcional;
- Ubuntu Server funcional;
- Windows 11 Pro funcional;
- rede privada `seclab`;
- endereçamento estático;
- acesso independente à Internet via NAT;
- comunicação entre os três hosts;
- SSH entre Kali e Ubuntu;
- ambiente preparado para experimentos de serviços;
- firewall do Windows investigado e validado;
- snapshots baseline das três VMs;
- repositório Git para documentação;
- challenges do M0 documentados;
- topologia compreendida e explicada sem tutorial.

O **Competency Gate do M0 foi concluído com sucesso**.

**Status: M0 — Security Lab Bootstrap concluído.**

[← Voltar ao índice](../../../index.md)
