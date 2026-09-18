# Fase 0 — M0: Security Lab Bootstrap

## Sessão 03 — Pós-instalação da Kali e configuração da rede do laboratório

### Objetivo da sessão

Realizar as configurações pós-instalação da **Kali Linux**, solucionar o problema de resolução da máquina virtual, validar sua conectividade e construir a primeira parte da topologia de rede do Security Lab.

---

## 1. VirtualBox Guest Additions

Foi estudado o conceito de **VirtualBox Guest Additions**.

Guest Additions é um conjunto de drivers e componentes instalados dentro do sistema operacional guest para melhorar sua integração com o VirtualBox.

Entre as funcionalidades fornecidas estão:

- integração do mouse;
- shared clipboard;
- drag and drop;
- integração gráfica;
- resolução dinâmica da tela.

Foi verificado se os componentes necessários estavam instalados utilizando:

`dpkg -l | grep -i virtualbox`

Foram encontrados:

- `virtualbox-guest-utils`
- `virtualbox-guest-x11`

Também foram verificados os módulos relacionados ao VirtualBox carregados pelo kernel:

`lsmod | grep -i vbox`

Foram encontrados módulos como:

- `vboxsf`
- `vboxguest`

---

## 2. Troubleshooting da resolução da Kali

Mesmo com Guest Additions instalado, a Kali permanecia utilizando resolução de aproximadamente **1280x800**, sem acompanhar corretamente o tamanho da janela do VirtualBox.

Foi iniciado um processo de investigação para identificar a causa.

### Sessão gráfica e VBoxClient

Foi verificado o tipo de sessão gráfica:

`echo $XDG_SESSION_TYPE`

Resultado:

`x11`

Também foram verificados os processos do `VBoxClient`, sendo encontrados componentes relacionados a:

- clipboard;
- seamless mode;
- drag and drop;
- sessão VMSVGA.

Essas evidências indicavam que os componentes de integração do VirtualBox estavam instalados e em execução.

### Investigação com xrandr

Foi utilizado:

`xrandr`

A resolução atual era:

`1280x800`

Entretanto, resoluções maiores estavam disponíveis, incluindo `1920x1080`.

Foi realizado o teste:

`xrandr --output Virtual-1 --mode 1920x1080`

A resolução era alterada momentaneamente, mas retornava automaticamente para `1280x800`.

Isso indicou que o problema não era simplesmente a incapacidade do sistema de utilizar uma resolução maior.

---

## 3. Scaled Mode vs. resolução real

Durante a investigação foi testado o **Scaled Mode** do VirtualBox.

Com ele ativado, a imagem da Kali preenchia corretamente a janela, porém o `xrandr` continuava reportando:

`1280x800`

Foi entendido que o Scaled Mode não altera a resolução utilizada pelo guest. Ele apenas redimensiona visualmente a imagem produzida pela VM.

Modelo simplificado:

```text
Guest renderiza em 1280x800
        ↓
VirtualBox amplia a imagem
        ↓
Janela é preenchida
```

Isso explica a perda de nitidez observada.

O comportamento desejado era que a própria Kali alterasse sua resolução conforme o tamanho da janela.

---

## 4. Identificação da causa do problema gráfico

Foi então analisada a configuração de display da VM no VirtualBox.

A configuração encontrada possuía:

| Configuração        | Valor        |
| ------------------- | ------------ |
| Video Memory        | 16 MB        |
| Graphics Controller | VMSVGA       |
| Virtual Monitors    | 1            |
| Scale Factor        | 100%         |
| 3D Acceleration     | Desabilitado |

A quantidade de **Video Memory** foi levantada como possível causa.

Seguindo o princípio aprendido anteriormente de alterar apenas uma variável por experimento, foi modificada somente a memória de vídeo:

`16 MB → 128 MB`

As demais configurações foram mantidas.

Após iniciar novamente a Kali, o **Auto-resize Guest Display passou a funcionar corretamente**.

A resolução da Kali passou a acompanhar o tamanho da janela do VirtualBox sem necessidade do Scaled Mode.

### Aprendizado de troubleshooting

O processo reforçou novamente o método:

```text
Problema
    ↓
Coleta de evidências
    ↓
Hipótese
    ↓
Experimento controlado
    ↓
Resultado
    ↓
Conclusão
```

Neste caso, a alteração isolada da VRAM resolveu o comportamento observado.

---

## 5. Validação básica da Kali

Com o problema gráfico resolvido, foi realizada uma validação básica da instalação.

Foram utilizados comandos como:

- `hostnamectl status`
- `ip a`
- `ip route`
- `ping`

Foi confirmado:

| Informação    | Resultado              |
| ------------- | ---------------------- |
| Hostname      | `kali`                 |
| Sistema       | Kali GNU/Linux Rolling |
| Arquitetura   | x86-64                 |
| Virtualização | Oracle/VirtualBox      |

A interface de rede inicialmente disponível era:

`eth0 → 10.0.2.15/24`

Também estava presente a interface de loopback:

`lo → 127.0.0.1/8`

---

## 6. Conectividade IP e DNS

Foi analisada a tabela de rotas da Kali.

A rota padrão utilizava:

`default via 10.0.2.2 dev eth0`

Foi entendido que `10.0.2.2` representa o gateway disponibilizado pelo NAT do VirtualBox.

Foram realizados dois testes diferentes.

Primeiro:

`ping 8.8.8.8`

O teste foi bem-sucedido, comprovando conectividade IP externa.

Depois:

`ping google.com`

O hostname foi resolvido para um endereço IP e a comunicação também funcionou.

Foi estabelecida uma distinção importante:

> Possuir conectividade IP não significa necessariamente que a resolução DNS também esteja funcionando.

Um cenário futuro como:

```text
ping 8.8.8.8
→ funciona

ping example.com
→ falha na resolução do nome
```

poderia indicar conectividade externa funcional com um problema específico relacionado a DNS.

---

## 7. VirtualBox NAT

Foi estudado com maior profundidade o funcionamento do modo **NAT** utilizado pela Kali.

A configuração inicial era:

```text
Kali
10.0.2.15
    ↓
VirtualBox NAT
10.0.2.2
    ↓
Host / rede externa
    ↓
Internet
```

Foi entendido que o VirtualBox fornece uma rede virtual privada para a VM e realiza a tradução necessária para permitir comunicação com redes externas.

Nesse cenário:

- a Kali recebe um endereço privado;
- o VirtualBox fornece sua configuração através da rede NAT;
- `10.0.2.2` funciona como gateway;
- conexões iniciadas pela VM conseguem acessar a Internet;
- respostas dessas conexões conseguem retornar;
- máquinas externas não conseguem simplesmente iniciar uma conexão diretamente para `10.0.2.15`.

Também foi entendido que duas VMs utilizando o **NAT padrão** do VirtualBox não formam automaticamente uma mesma LAN virtual compartilhada.

---

## 8. Modos de rede do VirtualBox

Foram estudados diferentes modos de networking disponíveis no VirtualBox:

- NAT;
- NAT Network;
- Bridged Adapter;
- Host-only Adapter;
- Internal Network;
- Generic Driver;
- Cloud Network;
- Not Attached.

O **NAT Network** foi inicialmente considerado para o Security Lab, pois permitiria comunicação entre VMs e acesso à Internet através de uma única rede.

Entretanto, foi discutida uma segunda arquitetura utilizando duas interfaces por VM:

```text
Adapter 1 → NAT
            Internet

Adapter 2 → Internal Network
            Security Lab
```

Essa arquitetura foi escolhida por separar explicitamente a rede utilizada para acesso externo da rede privada utilizada pelos experimentos do laboratório.

---

## 9. Internal Network e arquitetura do laboratório

Foi estudado o funcionamento da **Internal Network** do VirtualBox.

O modelo mental utilizado foi semelhante a um **switch virtual**, conectando as máquinas participantes daquela rede.

A Internal Network criada para o laboratório recebeu o nome:

`seclab`

A rede escolhida foi:

`192.168.50.0/24`

O planejamento inicial de endereçamento ficou:

| Máquina       | Endereço na seclab |
| ------------- | ------------------ |
| Kali          | `192.168.50.10/24` |
| Ubuntu Server | `192.168.50.20/24` |
| Windows       | `192.168.50.30/24` |

As interfaces da `seclab` não utilizarão default gateway.

Cada máquina terá uma segunda interface NAT responsável pela comunicação com a Internet.

---

## 10. Subnetting e CIDR

Foi estudado o básico de **subnetting IPv4 e CIDR** necessário para compreender a rede criada.

Foi utilizado como exemplo:

`192.168.50.10/24`

Um endereço IPv4 possui **32 bits**.

O `/24` determina que:

```text
24 bits → identificação da rede
 8 bits → identificação dos hosts
```

Para:

`192.168.50.0/24`

temos:

| Elemento        | Endereço         |
| --------------- | ---------------- |
| Network address | `192.168.50.0`   |
| Primeiro host   | `192.168.50.1`   |
| Último host     | `192.168.50.254` |
| Broadcast       | `192.168.50.255` |

A quantidade tradicional de endereços disponíveis para hosts é:

`2^8 - 2 = 254`

Foi reforçado que `192.168.50.0` representa a própria rede e `192.168.50.255` representa o endereço de broadcast, portanto não são endereços comuns atribuíveis a hosts nesse cenário.

---

## 11. Comunicação local e ARP

Foi estudado como uma máquina determina se precisa ou não utilizar um roteador para alcançar outro endereço.

Considerando:

```text
Kali
192.168.50.10/24

Ubuntu
192.168.50.20/24
```

ambos pertencem à rede:

`192.168.50.0/24`

Portanto, a comunicação ocorre diretamente dentro do segmento local.

Para IPv4, o **ARP (Address Resolution Protocol)** permite descobrir qual endereço MAC corresponde ao endereço IP desejado.

Modelo simplificado:

```text
Kali
192.168.50.10
      │
      │ "Quem possui 192.168.50.20?"
      ↓
   ARP Request
      ↓
Ubuntu
192.168.50.20
      │
      │ "Esse IP é meu. Este é meu MAC."
      ↓
   ARP Reply
```

Depois dessa resolução, a comunicação pode ocorrer diretamente através da rede local.

---

## 12. Default gateway

Também foi estudado o papel do **default gateway**.

Foi corrigida uma confusão inicial entre broadcast address e gateway.

> O broadcast address não é um default gateway.

O gateway precisa ser o endereço IP de uma interface pertencente a um dispositivo capaz de encaminhar pacotes para outras redes.

No caso da `seclab`, todas as máquinas planejadas pertencem à mesma subnet e não existe um roteador conectado à Internal Network.

Portanto:

```text
seclab
192.168.50.0/24
        │
        ├── Kali
        ├── Ubuntu
        └── Windows
```

não necessita de default gateway para comunicação entre essas máquinas.

O acesso a outras redes será realizado através da interface NAT de cada VM.

---

## 13. Configuração da segunda interface da Kali

Foi adicionado um segundo adaptador de rede à Kali no VirtualBox.

A configuração passou a ser:

### Adapter 1

```text
Mode: NAT
```

Responsável pelo acesso à Internet.

### Adapter 2

```text
Mode: Internal Network
Name: seclab
```

Responsável pela comunicação privada do Security Lab.

Dentro da Kali, a segunda interface foi configurada com endereço estático:

`eth1 → 192.168.50.10/24`

Sem default gateway.

---

## 14. Estado final das interfaces da Kali

Após a configuração, a Kali passou a possuir duas interfaces ativas:

```text
lo
└── 127.0.0.1/8

eth0
└── 10.0.2.15/24
    └── NAT

eth1
└── 192.168.50.10/24
    └── Internal Network: seclab
```

A função de cada interface ficou claramente separada:

```text
eth0 → acesso externo / Internet
eth1 → comunicação interna do Security Lab
```

---

## 15. Validação da tabela de rotas

Foi utilizada a tabela de rotas para verificar se o sistema havia interpretado corretamente a arquitetura.

O resultado relevante foi:

```text
default via 10.0.2.2 dev eth0
10.0.2.0/24 dev eth0
192.168.50.0/24 dev eth1
```

Para um destino externo, como:

`8.8.8.8`

não existe uma rota específica, portanto é utilizada a rota padrão:

```text
8.8.8.8
    ↓
default route
    ↓
10.0.2.2
    ↓
eth0
    ↓
Internet
```

Para um futuro destino da `seclab`, como:

`192.168.50.20`

existe uma rota diretamente conectada:

```text
192.168.50.20
      ↓
192.168.50.0/24
      ↓
eth1
      ↓
seclab
```

Nenhum gateway é necessário nesse segundo caso.

---

## 16. Topologia definida para o Security Lab

Ao final da sessão, a arquitetura planejada para o laboratório ficou:

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

Neste momento apenas a Kali estava implementada:

```text
Kali
│
├── eth0
│   ├── 10.0.2.15/24
│   └── NAT → Internet
│
└── eth1
    ├── 192.168.50.10/24
    └── seclab
```

---

## 17. Resultado da sessão

A sessão foi encerrada com a **Kali Linux preparada para integrar o Security Lab**.

Estado final:

```text
Kali Linux
│
├── Guest Additions funcionando
│
├── Display
│   ├── VMSVGA
│   ├── 128 MB VRAM
│   └── Auto-resize funcionando
│
├── eth0
│   ├── 10.0.2.15/24
│   ├── NAT
│   ├── Gateway 10.0.2.2
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
- X11 e VBoxClient
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
- Conectividade IP e DNS
- VirtualBox NAT
- Modos de rede do VirtualBox
- NAT Network
- Internal Network
- IPv4 e CIDR
- Subnetting `/24`
- Network e broadcast address
- ARP
- Endereços MAC
- Default gateway
- Tabela de rotas
- Múltiplas interfaces de rede
- IP estático

---

## Checkpoint

A **Kali Linux está instalada, operacional e com seu lado da topologia de rede do laboratório configurado**.

```text
Internet
   ↑
VirtualBox NAT
   ↑
 eth0
10.0.2.15
   │
 KALI
   │
 eth1
192.168.50.10
   ↓
seclab
192.168.50.0/24
```

O próximo passo será criar a **Ubuntu Server VM**, conectá-la simultaneamente ao NAT e à `seclab` e configurá-la como `192.168.50.20/24`.

Com isso, será possível realizar o primeiro teste de comunicação **Kali ↔ Ubuntu** dentro da rede privada do Security Lab.
