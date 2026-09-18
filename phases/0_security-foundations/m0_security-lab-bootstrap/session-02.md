# Fase 0 — M0: Security Lab Bootstrap

## Sessão 02 — Pós-instalação da Kali e configuração da rede do laboratório

### Objetivo da sessão

Continuar a preparação da Kali Linux após sua instalação, validar o funcionamento básico da máquina e iniciar a construção da topologia de rede do Security Lab.

A sessão envolveu principalmente:

- integração gráfica entre Kali e VirtualBox;
- troubleshooting da resolução da VM;
- validação básica do sistema;
- fundamentos de networking;
- modos de rede do VirtualBox;
- NAT;
- subnetting e CIDR;
- ARP e roteamento;
- criação da rede privada `seclab`;
- configuração da Kali com duas interfaces de rede.

---

## 1. VirtualBox Guest Additions

A sessão começou investigando como melhorar a experiência de utilização da Kali dentro do VirtualBox, principalmente porque o display da VM permanecia em uma resolução pequena.

Foi estudado o conceito de **VirtualBox Guest Additions**.

O entendimento construído foi que Guest Additions são drivers e componentes de software instalados dentro do sistema operacional guest para melhorar sua integração com o VirtualBox.

Entre as funcionalidades fornecidas estão:

- integração do mouse;
- shared clipboard;
- drag and drop;
- integração gráfica;
- resolução dinâmica;
- outras funcionalidades de comunicação entre guest e hypervisor.

Foi entendido que essas funcionalidades reduzem o "gap" existente entre o sistema operacional guest e o ambiente virtual fornecido pelo hypervisor.

---

## 2. Troubleshooting da resolução

Apesar da presença do Guest Additions, a Kali permanecia em aproximadamente **1280x800**, mesmo com a janela do VirtualBox maximizada.

Foi iniciado um processo de troubleshooting para identificar a causa.

### Verificação dos pacotes instalados

Foi utilizado:

```bash
dpkg -l | grep -i virtualbox
```

Foram encontrados, entre outros:

```text
virtualbox-guest-utils
virtualbox-guest-x11
```

O prefixo `ii` apresentado pelo `dpkg` indicava que os pacotes estavam instalados e configurados.

Isso mostrou que os componentes do Guest Additions já estavam presentes na Kali.

---

## 3. Verificação dos kernel modules

Em seguida foi investigado se os módulos relacionados ao VirtualBox estavam carregados pelo kernel.

Foi utilizado:

```bash
lsmod | grep -i vbox
```

Foram encontrados:

```text
vboxsf
vboxguest
```

O módulo `vboxvideo` não apareceu.

Inicialmente isso levantou a hipótese de que algum componente gráfico pudesse estar ausente, mas não foi assumido que essa era a causa, pois stacks Linux modernas podem utilizar componentes diferentes dos apresentados em tutoriais ou documentações antigas.

A principal lição foi diferenciar:

> "software instalado"

de:

> "componente efetivamente carregado/em execução".

---

## 4. X11 e VBoxClient

Foi investigado qual tipo de sessão gráfica estava sendo utilizada.

Com:

```bash
echo $XDG_SESSION_TYPE
```

o resultado foi:

```text
x11
```

Isso confirmou que a Kali estava utilizando uma sessão **X11**.

Também foram investigados os processos `VBoxClient`.

Foram encontrados processos relacionados a funcionalidades como:

```text
VBoxClient --clipboard
VBoxClient --seamless
VBoxClient --draganddrop
VBoxClient --vmsvga-session
```

Portanto, havia evidências de que os componentes de integração do VirtualBox estavam instalados e em execução.

---

## 5. Investigação com xrandr

Foi utilizado o `xrandr` para verificar a configuração gráfica disponível para a sessão X11.

O resultado mostrava:

```text
current: 1280 x 800
```

Porém também estavam disponíveis resoluções maiores, incluindo:

```text
1920x1080
1920x1200
2048x1152
...
```

Isso demonstrou que a stack gráfica da Kali **era capaz de trabalhar com resoluções maiores**.

Foi realizado um teste manual:

```bash
xrandr --output Virtual-1 --mode 1920x1080
```

A resolução mudava momentaneamente para `1920x1080`, mas logo retornava automaticamente para `1280x800`.

Isso foi uma evidência importante:

> O problema não era simplesmente incapacidade da Kali de utilizar uma resolução maior.

Algum componente estava alterando novamente a resolução após o comando manual.

---

## 6. Scaled Mode vs. resolução real

Foi encontrado no VirtualBox o **Scaled Mode**.

Ao ativá-lo, o desktop da Kali passou a preencher o espaço disponível da janela.

Entretanto, ao verificar novamente com `xrandr`, a resolução continuava sendo:

```text
1280x800
```

Foi então compreendida a diferença entre **scaling** e **alteração real de resolução**.

### Scaled Mode

O guest continua renderizando, por exemplo:

```text
1280x800
```

e o VirtualBox apenas aumenta visualmente essa imagem para preencher a janela.

Isso pode causar perda de nitidez e aparência borrada.

### Auto-resize real

O comportamento desejado era:

```text
Janela aumenta
      ↓
VirtualBox informa nova área ao guest
      ↓
Kali altera sua própria resolução
      ↓
Desktop é renderizado nativamente na nova resolução
```

Por isso o Scaled Mode foi considerado apenas um workaround, e não a solução definitiva.

---

## 7. Identificação da causa do problema gráfico

Foi então verificada a configuração de display da VM no VirtualBox.

A configuração encontrada era:

```text
Video Memory:         16 MB
Graphics Controller:  VMSVGA
Virtual Monitors:     1
Scale Factor:         100%
3D Acceleration:      Disabled
```

A quantidade de **Video Memory** chamou atenção.

Foi realizado um experimento controlado alterando **somente uma variável**:

```text
Video Memory
16 MB → 128 MB
```

Foram mantidos:

- VMSVGA;
- 3D Acceleration desabilitado;
- 1 monitor;
- Scale Factor em 100%.

Após iniciar novamente a Kali, o **Auto-resize Guest Display passou a funcionar corretamente**.

A resolução passou a acompanhar o tamanho da janela sem necessidade do Scaled Mode.

### Conclusão do troubleshooting

O problema estava relacionado à quantidade insuficiente de memória de vídeo configurada para a VM.

O processo reforçou novamente o método:

```text
Problema
   ↓
Coleta de evidências
   ↓
Hipóteses
   ↓
Experimentos controlados
   ↓
Eliminação de hipóteses
   ↓
Causa identificada
```

---

## 8. Validação básica da Kali

Com o problema gráfico resolvido, foi iniciada a validação básica da Kali.

Foram investigadas:

- identidade da máquina;
- versão do sistema;
- interfaces de rede;
- endereços IP;
- tabela de rotas;
- conectividade externa;
- resolução DNS.

Entre os comandos utilizados estavam:

```bash
hostnamectl status
ip a
ip route
ping
```

Foi confirmado:

```text
Hostname: kali
Sistema: Kali GNU/Linux Rolling
Arquitetura: x86-64
Virtualização: Oracle/VirtualBox
```

---

## 9. Interface de rede inicial

Inicialmente a Kali possuía:

```text
eth0
└── 10.0.2.15/24
```

Também estava presente a interface de loopback:

```text
lo
└── 127.0.0.1/8
```

A interface `lo` representa a própria máquina e permite comunicação local através de loopback.

A `eth0` era a interface conectada ao modo NAT do VirtualBox.

---

## 10. Rotas e conectividade

O comando:

```bash
ip route
```

mostrou uma rota padrão semelhante a:

```text
default via 10.0.2.2 dev eth0
```

Foi entendido que `10.0.2.2` funciona como gateway da rede NAT virtual fornecida pelo VirtualBox.

Foram realizados dois testes distintos.

### Conectividade IP

```bash
ping 8.8.8.8
```

O teste funcionou.

Isso demonstrou que a Kali possuía conectividade IP externa.

### Resolução de nomes

Também foi realizado:

```bash
ping google.com
```

O hostname foi resolvido para um endereço IP e o destino respondeu.

Isso permitiu separar dois conceitos importantes:

```text
Conseguir alcançar um IP
          ≠
Conseguir resolver um hostname
```

Por exemplo, futuramente um cenário como:

```text
ping 8.8.8.8
→ funciona

ping example.com
→ falha na resolução
```

indicaria que a conectividade IP pode estar funcionando enquanto existe um problema específico relacionado a DNS.

---

## 11. VirtualBox NAT

Foi estudado o funcionamento do modo **NAT** do VirtualBox.

O modelo mental construído foi aproximadamente:

```text
Kali
10.0.2.15
    │
    ▼
VirtualBox NAT
10.0.2.2
    │
    ▼
Host / rede física
    │
    ▼
Internet
```

O VirtualBox fornece serviços virtuais que permitem à VM receber sua configuração de rede e utilizar o NAT para acessar redes externas.

Foi entendido que:

- a VM possui um endereço privado;
- o VirtualBox atua como intermediário entre a VM e a rede externa;
- conexões iniciadas pela VM podem sair para a Internet;
- respostas dessas conexões conseguem retornar para a VM;
- uma máquina externa não consegue simplesmente iniciar uma conexão diretamente para `10.0.2.15`.

Também foi utilizada uma analogia de um hotel:

> A VM funciona como um hóspede e o VirtualBox como o hotel. Quando o hóspede envia algo para fora, a comunicação passa pela portaria e é apresentada externamente através do hotel, enquanto o número do quarto permanece um detalhe interno.

---

## 12. Modos de rede do VirtualBox

Foram pesquisados diferentes modos de networking disponíveis no VirtualBox:

- NAT
- NAT Network
- Bridged Adapter
- Host-only Adapter
- Internal Network
- Generic Driver
- Cloud Network
- Not Attached

### NAT

Permite acesso externo, mas o NAT padrão de cada VM não cria automaticamente uma rede compartilhada entre as VMs.

### Bridged Adapter

Colocaria a VM diretamente na rede física utilizada pelo host.

Foi descartado porque não havia necessidade de expor as máquinas do Security Lab diretamente à rede doméstica.

### Host-only

Permite comunicação envolvendo host e guests, mas não oferece por si só a saída para Internet desejada.

Também não havia requisito atual para comunicação direta entre host e VMs.

### NAT Network

Foi inicialmente considerado como solução porque permite:

```text
VM ↔ VM
VM ↔ Internet
```

através de uma única rede.

### Internal Network

Permite criar uma rede privada entre as VMs.

O modelo mental utilizado foi semelhante a um **switch virtual** conectando apenas as máquinas participantes daquela Internal Network.

---

## 13. Arquitetura escolhida para o laboratório

Inicialmente, **NAT Network** parecia a solução mais simples.

Entretanto, foi considerada uma segunda arquitetura utilizando **duas interfaces por VM**:

```text
Adapter 1 → NAT
            acesso à Internet

Adapter 2 → Internal Network
            comunicação privada do laboratório
```

Essa arquitetura foi escolhida.

A principal vantagem é separar explicitamente:

```text
Rede de saída
      ≠
Rede do Security Lab
```

Isso também permite futuramente desligar o adaptador NAT e manter as máquinas comunicando apenas dentro do ambiente isolado.

---

## 14. Internal Network e DHCP

Foi estudado o comportamento da **Internal Network**.

O entendimento construído foi que ela funciona conceitualmente como um switch virtual entre as máquinas conectadas àquela rede.

Não foi utilizado DHCP nessa rede.

Por isso foi decidido configurar **endereços IPv4 estáticos** para as máquinas do laboratório.

---

## 15. Subnetting e CIDR

Antes de definir os IPs foi estudado o básico de **IPv4 subnetting e CIDR**.

O exemplo utilizado foi:

```text
192.168.50.10/24
```

Um endereço IPv4 possui 32 bits.

O `/24` significa que os primeiros 24 bits representam a parte de rede, restando 8 bits para hosts.

Representação:

```text
192.168.50.10

11000000.10101000.00110010.00001010
└──────────── 24 bits ────────────┘
              network
```

Para:

```text
192.168.50.0/24
```

temos:

```text
Network address:   192.168.50.0
Broadcast address: 192.168.50.255
```

Faixa tradicional de hosts utilizáveis:

```text
192.168.50.1
        ↓
192.168.50.254
```

Quantidade:

```text
2^8 - 2
= 256 - 2
= 254 hosts
```

Foi reforçado que:

- `192.168.50.0` identifica a rede;
- `192.168.50.255` é o broadcast;
- nenhum dos dois deve ser tratado como endereço comum de host nesse cenário.

---

## 16. Comunicação local e ARP

Também foi estudado como uma máquina determina se um destino é local.

A máquina utiliza seu endereço e sua máscara para determinar a rede à qual pertence.

Exemplo:

```text
Kali:
192.168.50.10/24

Ubuntu:
192.168.50.20/24
```

Ambos pertencem a:

```text
192.168.50.0/24
```

Portanto, a comunicação pode ocorrer diretamente no segmento local.

Para IPv4, entra em cena o **ARP (Address Resolution Protocol)**.

Modelo simplificado:

```text
Kali:
"Quem possui 192.168.50.20?"
              │
              │ ARP Request
              ▼
         Rede local
              │
              ▼
Ubuntu:
"Esse IP é meu.
Este é meu MAC."
```

Assim, a Kali consegue associar o endereço IP do destino ao seu endereço MAC e realizar a comunicação no segmento local.

---

## 17. Default gateway

Também foi corrigido um erro conceitual importante.

Inicialmente houve confusão entre:

- broadcast address;
- default gateway.

Foi entendido que:

> **Broadcast address não é default gateway.**

O gateway precisa ser o endereço de uma interface pertencente a um dispositivo capaz de rotear tráfego para outras redes.

Por exemplo:

```text
Host:     192.168.1.50/24
Gateway:  192.168.1.1
```

O endereço `.1` não possui significado especial por si só; ele apenas foi atribuído à interface do roteador naquele exemplo.

---

## 18. Internal Network sem gateway

Como nossa Internal Network conecta diretamente as máquinas da mesma subnet, ela não precisa de default gateway.

Modelo:

```text
             Internal Network
                  seclab
                    │
        ┌───────────┼───────────┐
        │           │           │
      Kali        Ubuntu      Windows
```

É conceitualmente semelhante a três computadores conectados ao mesmo switch sem um roteador conectado a esse switch.

As máquinas conseguem conversar entre si porque pertencem à mesma subnet.

Para acessar outras redes, cada VM utiliza sua outra interface:

```text
eth0 → NAT → Internet
```

Portanto, a interface da Internal Network fica **sem default gateway**.

---

## 19. Endereçamento definido para o Security Lab

Foi criada/definida a Internal Network:

```text
seclab
```

Subnet escolhida:

```text
192.168.50.0/24
```

Planejamento de endereçamento:

| Máquina       | IP na seclab       |
| ------------- | ------------------ |
| Kali          | `192.168.50.10/24` |
| Ubuntu Server | `192.168.50.20/24` |
| Windows       | `192.168.50.30/24` |

Nenhuma dessas interfaces utilizará default gateway.

Cada VM terá outra interface NAT responsável pela saída para Internet.

---

## 20. Segunda interface da Kali

Foi adicionado um segundo adaptador virtual à Kali.

### Adapter 1

```text
Mode: NAT
```

Responsável por:

```text
Internet
```

### Adapter 2

```text
Mode: Internal Network
Name: seclab
```

Responsável pela comunicação privada do laboratório.

Foi entendido que a configuração realizada no VirtualBox cria e conecta o **hardware virtual**, mas o endereço IP da interface pode ser configurado dentro do próprio sistema operacional guest.

---

## 21. Configuração final das interfaces da Kali

Após algum troubleshooting no Linux depois da adição da segunda placa, ambas as interfaces foram configuradas corretamente.

Estado final:

```text
lo
└── 127.0.0.1/8

eth0
└── 10.0.2.15/24
    NAT

eth1
└── 192.168.50.10/24
    Internal Network: seclab
```

A `eth0` permanece responsável pela conectividade externa.

A `eth1` passa a representar a presença da Kali dentro da rede privada do Security Lab.

---

## 22. Validação da tabela de rotas

Foi utilizado novamente:

```bash
ip route
```

A tabela apresentou essencialmente:

```text
default via 10.0.2.2 dev eth0
10.0.2.0/24 dev eth0
192.168.50.0/24 dev eth1
```

A interpretação foi:

### Destinos da Internet

Um destino como:

```text
8.8.8.8
```

não pertence a nenhuma das redes diretamente conectadas.

Portanto:

```text
8.8.8.8
    ↓
default route
    ↓
10.0.2.2
    ↓
eth0
    ↓
NAT / Internet
```

### Destinos da seclab

Um futuro destino como:

```text
192.168.50.20
```

pertence a:

```text
192.168.50.0/24
```

Portanto:

```text
192.168.50.20
      ↓
192.168.50.0/24
      ↓
eth1
      ↓
comunicação local
```

Nenhum gateway é necessário nessa rota.

---

## 23. Topologia atual do laboratório

Ao final da sessão, a arquitetura planejada estava assim:

```text
                       INTERNET
                           │
                     VirtualBox NAT
                           │
              ┌────────────┼────────────┐
              │            │            │
            eth0          eth0         eth0
            Kali         Ubuntu       Windows
              │            │            │
            eth1          eth1         eth1
              │            │            │
              └────────────┼────────────┘
                           │
                    Internal Network
                         seclab
                    192.168.50.0/24
```

No momento apenas a Kali estava implementada:

```text
Kali

eth0
10.0.2.15/24
│
└── NAT → Internet

eth1
192.168.50.10/24
│
└── seclab
```

Ubuntu e Windows ainda serão adicionados.

---

## 24. Resultado da sessão

A sessão foi encerrada com a Kali completamente funcional para o estágio atual do laboratório.

Estado final:

```text
Kali Linux
│
├── Sistema instalado e funcional
├── GRUB funcionando
├── XFCE funcionando
│
├── Guest Additions
│   └── funcionando
│
├── Display
│   ├── VMSVGA
│   ├── 128 MB VRAM
│   └── Auto-resize funcionando
│
├── eth0
│   ├── 10.0.2.15/24
│   ├── NAT
│   ├── default gateway 10.0.2.2
│   └── Internet + DNS funcionando
│
└── eth1
    ├── 192.168.50.10/24
    ├── Internal Network
    ├── seclab
    └── sem default gateway
```

### Conhecimentos trabalhados

- VirtualBox Guest Additions
- Kernel modules
- X11
- VBoxClient
- `dpkg`
- `lsmod`
- `xrandr`
- Scaled Mode vs. resolução real
- Troubleshooting baseado em evidências
- Interfaces de rede Linux
- `hostnamectl`
- `ip a`
- `ip route`
- `ping`
- Conectividade IP
- DNS
- VirtualBox NAT
- Modos de rede do VirtualBox
- NAT Network
- Internal Network
- Subnetting
- CIDR
- Network address
- Broadcast address
- ARP
- MAC address
- Default gateway
- Rotas diretamente conectadas
- Múltiplas interfaces de rede
- IP estático

---

## Checkpoint

**Kali Linux preparada e conectada à arquitetura inicial do Security Lab.**

Topologia da Kali validada:

```text
Internet
   ↑
  NAT
   ↑
 eth0
10.0.2.15
   │
 KALI
   │
 eth1
192.168.50.10
   │
   ↓
seclab
192.168.50.0/24
```

### Próxima sessão

O próximo passo será criar a **Ubuntu Server VM** de forma mais autônoma, utilizando os conhecimentos adquiridos durante a criação da Kali.

Endereçamento planejado:

```text
Ubuntu NAT      → acesso à Internet
Ubuntu seclab   → 192.168.50.20/24
```

Com Kali e Ubuntu conectadas à `seclab`, será possível realizar o primeiro teste real de comunicação **VM ↔ VM** do laboratório e posteriormente avançar para SSH, serviços HTTP e descoberta de portas.
