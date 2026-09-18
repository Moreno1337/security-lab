# Fase 0 — M0: Security Lab Bootstrap

## Sessão 01 — Virtualização e instalação da Kali Linux

### Objetivo da sessão

Iniciar efetivamente a construção do laboratório de segurança previsto no M0, compreendendo as decisões envolvidas na criação de uma máquina virtual e realizando a instalação da primeira máquina do laboratório: **Kali Linux**.

---

## 1. Virtualização e Hypervisor

Foi estudado o conceito de **virtualização** e o papel de um **hypervisor**.

O hypervisor fornece a camada responsável por criar e administrar máquinas virtuais, permitindo distribuir recursos do host entre diferentes guests e mantendo isolamento entre eles.

Também foram estudados os dois principais tipos de hypervisor:

- **Type 1:** executado diretamente sobre o hardware.
- **Type 2:** executado sobre um sistema operacional host.

Foram pesquisadas alternativas como **VirtualBox** e **Hyper-V**.

Como o host utiliza Windows 11 Home e o objetivo é construir um laboratório local simples e flexível, foi escolhido o **Oracle VirtualBox**.

---

## 2. ISO e arquitetura

Foi estudado o conceito de **imagem ISO**.

Uma ISO representa uma imagem de uma mídia óptica. No contexto da virtualização, o VirtualBox consegue apresentá-la para a máquina virtual como uma mídia virtual de instalação.

Também foi estudada a diferença entre:

- **Installer ISO:** utilizada para realizar a instalação manual do sistema operacional.
- **Pre-built VM:** máquina virtual previamente instalada e configurada.

Para o laboratório foi escolhida a instalação através da ISO, pois passar pelo processo completo de instalação oferece maior valor pedagógico.

### Arquitetura amd64 / x86-64

Foi estudado o significado de **amd64/x86-64**, entendendo o AMD64 como uma extensão de 64 bits da arquitetura x86 e que essa arquitetura também é utilizada por processadores Intel modernos.

Imagem utilizada:

`kali-linux-2026.2-installer-amd64.iso`

---

## 3. Recursos da máquina virtual

Antes da criação da VM, foram estudados os principais recursos disponibilizados pelo host para uma máquina virtual:

- RAM
- vCPU
- armazenamento virtual

Também foi estudada a diferença entre discos virtuais:

- **Fixed:** espaço reservado previamente no armazenamento do host.
- **Dynamically allocated:** arquivo do disco virtual cresce conforme o espaço é utilizado pelo guest, até o limite configurado.

### Configuração escolhida para a Kali

| Recurso           | Configuração |
| ----------------- | ------------ |
| RAM               | 4 GB         |
| vCPU              | 4            |
| Disco virtual     | 50 GB        |
| Alocação do disco | Dinâmica     |
| Firmware          | Legacy BIOS  |
| Hypervisor        | VirtualBox   |
| Sistema           | Kali Linux   |

---

## 4. BIOS e UEFI

Foi estudado o papel de **BIOS e UEFI** no processo de inicialização de uma máquina.

O modelo mental construído foi:

`Power On → BIOS/UEFI → inicialização do hardware → dispositivo bootável → bootloader → sistema operacional`

Foi entendido que UEFI é a alternativa moderna ao BIOS tradicional, mas que adicionar essa complexidade não contribuiria significativamente para os objetivos atuais do M0.

Por esse motivo, a Kali foi configurada utilizando **Legacy BIOS**.

---

## 5. Troubleshooting do instalador

Durante a instalação ocorreu o primeiro troubleshooting relevante do laboratório.

O **Graphical Installer** da Kali ficava repetidamente travado na etapa:

`Starting up the partitioner`

O problema foi reproduzido mais de uma vez.

### Verificação da ISO

Uma das hipóteses investigadas foi corrupção da imagem ISO.

Foi realizada a verificação do **SHA-256** do arquivo local e o resultado foi comparado com o hash publicado oficialmente pela Kali.

Os hashes eram idênticos.

Com isso, corrupção da ISO foi eliminada como causa provável do problema.

### Segunda tentativa

Na tentativa seguinte foram alteradas duas variáveis:

1. Utilização do instalador textual (`Install`) em vez do `Graphical Install`.
2. Remoção do caractere `~` que havia sido utilizado em um nome durante a instalação.

Após essas alterações, o particionador iniciou normalmente.

Como duas variáveis foram modificadas simultaneamente, não foi possível determinar com segurança qual delas estava relacionada ao problema.

### Aprendizado de troubleshooting

Uma lição importante foi registrada:

> Sempre que possível, alterar apenas uma variável por experimento.

Dessa forma é possível estabelecer uma relação mais confiável entre uma alteração e o resultado observado.

---

## 6. Particionamento e LVM

Durante a instalação foi realizado um estudo introdutório sobre **LVM (Logical Volume Manager)**.

O entendimento construído foi que LVM adiciona uma camada de abstração ao gerenciamento de armazenamento, permitindo maior flexibilidade na administração dos volumes.

Entre as possibilidades estudadas:

- expansão e redução de volumes;
- gerenciamento mais flexível do armazenamento;
- utilização de pools;
- snapshots.

Apesar das vantagens, foi concluído que adicionar LVM neste momento aumentaria a complexidade sem contribuir diretamente para os objetivos atuais do M0.

### Particionamento escolhido

Foi utilizado:

`Guided — use entire disk`

seguido de:

`All files in one partition`

Portanto:

- sem LVM;
- sem encrypted LVM;
- sem particionamento manual;
- todos os arquivos na mesma partição.

Também foi confirmado que o disco apresentado ao instalador era o **VBOX HARDDISK virtual de aproximadamente 50 GB**, e não o SSD físico do host.

---

## 7. Seleção de software

Na etapa de seleção de software foram mantidas as opções padrão relevantes da Kali:

- XFCE
- Top 10 tools
- Default/recommended tools

GNOME, KDE Plasma e outros componentes adicionais não foram selecionados.

A decisão seguiu o princípio utilizado durante o M0:

> Não adicionar complexidade sem que ela resolva um problema ou contribua para o objetivo atual de aprendizado.

---

## 8. GRUB e processo de boot

Foi estudado o papel básico do **GRUB**.

GRUB funciona como bootloader, sendo responsável por carregar o sistema operacional após o firmware localizar um dispositivo inicializável.

Modelo mental construído:

```text
VM é ligada
    ↓
BIOS
    ↓
Localiza dispositivo bootável
    ↓
GRUB
    ↓
Kernel Linux
    ↓
Kali Linux
```
