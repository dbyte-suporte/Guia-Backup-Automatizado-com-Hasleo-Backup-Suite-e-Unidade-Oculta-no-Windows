# Backup Automatizado do Windows com Hasleo Backup Suite + Unidade de Backup Oculta + Sergei Strelec

> Guia revisado para Windows 10/11, com foco em **backup automático**, **proteção contra exclusão acidental** e **restauração offline** usando WinPE.
>
> **Última revisão:** 31/08/2026

## Visão geral

Este guia monta uma estratégia simples:

```text
┌──────────────────────┐
│ Windows 10/11        │
│                      │
│ Hasleo Backup Suite  │
│       │              │
│       ▼              │
│ Disco de backup      │
│ Partição Z:          │
│ (oculta no Explorer) │
└──────────────────────┘
           │
           │ restauração de emergência
           ▼
┌─────────────────────────────┐
│ Pendrive WinPE              │
│ Sergei Strelec              │
│ + Hasleo Backup Suite       │
└─────────────────────────────┘
           │
           ▼
    Restauração do Windows
```

O objetivo é **ocultar a unidade de backup da interface normal do Windows**, não torná-la criptograficamente segura ou invisível para todo o sistema. O mecanismo `NoDrives`, usado neste tutorial, afeta principalmente a exibição das letras no Explorer.

---

## ⚠️ Antes de começar: correções importantes em relação ao procedimento original

Alguns pontos do procedimento original precisam de ajustes:

1. **`NoDrives` não é um mecanismo de segurança.**  
   Ele oculta a letra da unidade no Explorer, mas a unidade continua montada e pode ser acessada por programas, pelo Gerenciamento de Disco, pelo Prompt/PowerShell e por outros meios.

2. **Não remova a letra `Z:` depois de configurar o Hasleo**, se o agendamento depender desse caminho.  
   O tutorial mantém `Z:` e apenas oculta a letra visualmente.

3. **O arquivo de imagem não deve ser presumido como `.bim`.**  
   O formato e a extensão efetivamente utilizados pelo Hasleo devem ser confirmados na instalação/versão em uso. Não use uma extensão como critério para localizar a imagem: selecione a imagem pelo próprio Hasleo.

4. **Não é obrigatório instalar o Hasleo dentro do Sergei Strelec.**  
   A alternativa mais segura é usar a mídia WinPE de emergência e, quando necessário, executar uma versão compatível/portable do Hasleo ou criar a própria mídia WinPE do Hasleo. Teste previamente o método escolhido.

5. **Pendrive de 1–2 GB não é uma recomendação universal.**  
   Para uma mídia WinPE específica do Hasleo, siga o tamanho mínimo indicado pela versão atual do programa. Para Sergei Strelec, use um pendrive com folga; **16 GB é uma escolha prática**.

6. **Não trate um único disco interno como backup contra falha física.**  
   Se o disco de origem e o disco de backup estiverem no mesmo computador, uma falha elétrica, roubo, ransomware ou dano físico pode atingir ambos. Para dados importantes, mantenha pelo menos uma segunda cópia offline ou externa.

---

# Sumário

> **Navegação:** estes links usam as âncoras geradas automaticamente pelo GitHub. O GitHub transforma letras em minúsculas, converte espaços em `-` e remove pontuação; por isso, por exemplo, `Z:` vira `#atribuir-a-letra-z` e não `#atribuir-a-letra-z:`. citeturn0search0

# Sumário

1. [Pré-requisitos](#1-pré-requisitos)
2. [Downloads oficiais](#2-downloads-oficiais)
3. [Arquitetura recomendada](#3-arquitetura-recomendada)
4. [Preparar o disco de backup](#4-preparar-o-disco-de-backup)
5. [Atribuir a letra Z](#5-atribuir-a-letra-z)
6. [Ocultar Z no Explorer](#6-ocultar-z-no-explorer)
7. [Configurar o Hasleo Backup Suite](#7-configurar-o-hasleo-backup-suite)
8. [Criar e preparar o pendrive WinPE](#8-criar-e-preparar-o-pendrive-winpe)
9. [Usar Ventoy como alternativa](#9-usar-ventoy-como-alternativa)
10. [Integrar o Hasleo ao ambiente de recuperação](#10-integrar-o-hasleo-ao-ambiente-de-recuperação)
11. [Procedimento de restauração](#11-procedimento-de-restauração)
12. [Problemas comuns](#12-problemas-comuns)
13. [Boas práticas](#13-boas-práticas)
14. [Checklist final](#14-checklist-final)
15. [Referências e downloads](#referências-e-downloads)
16. [Aviso de segurança](#aviso-de-segurança)
17. [Licenciamento e distribuição](#licenciamento-e-distribuição)
18. [Resumo operacional](#resumo-operacional)

---

# 1. Pré-requisitos

## Hardware

Recomendação:

- PC com Windows 10 ou Windows 11.
- Segundo HD/SSD para armazenar os backups.
- Preferencialmente um disco de backup com capacidade significativamente maior que o espaço ocupado no Windows.
- Pendrive de **16 GB ou maior** para o ambiente Sergei Strelec.
- Um segundo pendrive, opcional, para uma mídia WinPE dedicada ao Hasleo.
- Para maior segurança, uma segunda unidade de backup externa/offline.

### Exemplo

```text
SSD 1 — Windows
C:\
├── Windows
├── Program Files
└── Usuários

SSD/HDD 2 — Backup
Z:\
└── Backups\
    └── PC-01\
        ├── imagens completas
        └── incrementais/diferenciais

Pendrive — Recuperação
├── Sergei Strelec WinPE
└── ferramentas de recuperação
```

## Software

Você precisará de:

- Hasleo Backup Suite Free.
- Sergei Strelec WinPE.
- Rufus **ou** Ventoy.
- Windows 10/11 já instalado e funcionando para configurar o sistema.

---

# 2. Downloads oficiais

## Hasleo Backup Suite

Use o site oficial da Hasleo:

- https://www.hasleo.com/

A página oficial atualmente apresenta o **Hasleo Backup Suite V5.9** e informa suporte a backup/restauração, modos full/incremental/differential, agendamento e política de retenção de imagens.

> **Importante:** prefira sempre baixar o instalador diretamente do domínio oficial da Hasleo e confira a versão antes de seguir o tutorial.

## Sergei Strelec WinPE

Página do WinPE 11-10-8 Sergei Strelec em inglês:

- https://sergeistrelec.name/winpe-10-8-sergei-strelec-english/280-winpe-11-10-8-sergei-strelec-x86x64native-x86-20260414-english-version.html

No momento desta revisão, a página lista a versão **2026.04.14 English**, com os respectivos hashes de verificação.

> O Sergei Strelec é distribuído como uma imagem WinPE contendo várias ferramentas. Baixe a imagem de uma fonte confiável e verifique o hash publicado pelo autor antes de utilizá-la.

## Rufus

Site oficial:

- https://rufus.ie/

O Rufus cria pendrives inicializáveis a partir de imagens ISO. A versão atual exibida no site oficial durante esta revisão é a **4.15**.

## Ventoy

Site oficial:

- https://www.ventoy.net/

O Ventoy permite instalar o bootloader uma vez no pendrive e depois copiar arquivos ISO para a partição do pendrive. A página oficial lista a versão **1.1.17** durante esta revisão.

### Rufus ou Ventoy?

| Situação | Recomendação |
|---|---|
| Quero um pendrive dedicado ao Sergei Strelec | Rufus |
| Quero vários ISOs no mesmo pendrive | Ventoy |
| Quero adicionar novas ISOs facilmente | Ventoy |
| Quero o método tradicional de gravação | Rufus |

---

# 3. Arquitetura recomendada

Uma configuração robusta pode ser:

```text
                         INTERNET
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Downloads oficiais  │
                 │ Hasleo / Strelec    │
                 │ Rufus / Ventoy      │
                 └─────────────────────┘

┌──────────────────────────────────────────────────────┐
│ PC                                                   │
│                                                      │
│  SSD 1                                               │
│  └── Windows 10/11                                   │
│                                                      │
│  SSD/HDD 2                                           │
│  └── Z: Backup                                       │
│      └── imagens Hasleo                              │
│                                                      │
│  Pendrive                                            │
│  └── Sergei Strelec WinPE                            │
└──────────────────────────────────────────────────────┘
```

### Princípio importante

**Backup não é sinônimo de disco oculto.**

Ocultar `Z:` reduz a chance de um usuário apagar ou modificar arquivos por engano, mas não protege contra:

- ransomware;
- malware com privilégios administrativos;
- falha física do disco;
- incêndio;
- roubo;
- corrupção do sistema de arquivos;
- exclusão intencional.

Para dados realmente importantes, utilize a regra prática **3-2-1**:

- 3 cópias dos dados;
- 2 mídias diferentes;
- 1 cópia offline/off-site.

---

# 4. Preparar o disco de backup

## 4.1 Identifique corretamente o disco

Abra:

**Win + X → Gerenciamento de Disco**

Antes de formatar ou alterar qualquer partição, confirme:

- capacidade;
- modelo do disco;
- partições;
- letra atual;
- volume utilizado.

> **ATENÇÃO:** nunca execute `clean`, formatação ou exclusão de partição sem confirmar visualmente que está trabalhando no disco de backup.

## 4.2 Sistema de arquivos

Para um disco de backup usado principalmente no Windows, **NTFS** é uma escolha apropriada.

Sugestão:

```text
Disco: HDD/SSD secundário
Partição: NTFS
Nome do volume: BACKUP
Letra: Z:
```

## 4.3 Estrutura de pastas

Crie:

```text
Z:\
└── Backups\
    └── NOME-DO-PC\
```

Por exemplo:

```text
Z:\Backups\DESKTOP-01\
```

Evite guardar documentos pessoais misturados diretamente na raiz da partição.

---

# 5. Atribuir a letra Z:

1. Pressione `Win + X`.
2. Abra **Gerenciamento de Disco**.
3. Localize o volume correto.
4. Clique com o botão direito.
5. Selecione **Alterar letra de unidade e caminhos...**
6. Escolha **Z:**.
7. Confirme.

Verifique no Explorer:

```text
Este Computador
└── BACKUP (Z:)
```

Antes de esconder a unidade, faça um teste simples:

```powershell
Test-Path Z:\
```

O resultado esperado é:

```text
True
```

---

# 6. Ocultar Z: no Explorer

## 6.1 O que este método realmente faz

O valor de política `NoDrives` controla quais letras de unidade são exibidas pelo Explorer.

Para a letra `Z:`, o valor decimal é:

```text
33554432
```

Isso corresponde a:

```text
2^(26) = 33554432
```

porque:

```text
A = 2^0
B = 2^1
C = 2^2
...
Z = 2^25
```

> O ponto mais importante: **isso não desmonta o volume e não constitui uma proteção de segurança.**

## 6.2 Método pelo Registro

1. Pressione `Win + R`.
2. Digite:

```text
regedit
```

3. Navegue até:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer
```

4. Se `Explorer` não existir:
   - botão direito em `Policies`;
   - **Novo → Chave**;
   - nomeie como `Explorer`.

5. No painel direito:
   - botão direito;
   - **Novo → Valor DWORD (32 bits)**.

6. Nome:

```text
NoDrives
```

7. Abra o valor.
8. Selecione **Decimal**.
9. Informe:

```text
33554432
```

10. Clique em **OK**.

11. Reinicie o Windows ou reinicie o processo Explorer.

## 6.3 Teste

Depois da alteração:

- `Z:` deve desaparecer do Explorer;
- o volume continua montado;
- o Hasleo ainda deve conseguir utilizá-lo.

Teste no PowerShell:

```powershell
Test-Path Z:\
```

Se retornar:

```text
True
```

a unidade continua acessível ao sistema.

### Importante

Se o Hasleo não conseguir acessar o destino após essa alteração, **não prossiga com o agendamento**. Remova temporariamente o `NoDrives`, valide o funcionamento do backup e somente depois teste novamente.

---

# 7. Configurar o Hasleo Backup Suite

A interface pode mudar entre versões. Os nomes abaixo correspondem à lógica atual do produto e podem apresentar pequenas diferenças de tradução.

## 7.1 Criar o primeiro backup

1. Abra o **Hasleo Backup Suite**.
2. Escolha **System Backup**.
3. Confira cuidadosamente as partições selecionadas.
4. Em **Destination**, escolha:

```text
Z:\Backups\NOME-DO-PC\
```

5. Configure compressão, criptografia e demais opções disponíveis conforme sua necessidade.
6. Salve a tarefa.

A página oficial informa que o Hasleo suporta backup completo, incremental e diferencial, além de agendamento e política de retenção.

## 7.2 Primeiro backup: prefira Full

Para o primeiro backup, recomendo:

```text
Primeira execução
└── FULL
```

Depois:

```text
Full
 ├── Incremental
 ├── Incremental
 ├── Incremental
 └── novo Full conforme política
```

A escolha entre incremental e diferencial depende da estratégia de retenção e do espaço disponível.

### Incremental

Vantagens:

- normalmente menor consumo de espaço;
- execução mais rápida após o primeiro backup.

Desvantagens:

- a cadeia pode ficar dependente de vários backups anteriores;
- restaurações podem exigir mais arquivos da cadeia.

### Differential

Vantagens:

- cada diferencial depende do Full correspondente;
- restauração geralmente é conceitualmente mais simples.

Desvantagens:

- arquivos diferenciais crescem ao longo do tempo;
- consome mais espaço que uma cadeia incremental equivalente.

---

# 7.3 Configurar o agendamento

Entre em **Schedule** e escolha:

- Diário;
- Semanal;
- Mensal;
- horário.

Uma configuração de exemplo:

```text
Backup:
    Tipo: System Backup
    Frequência: Diário
    Horário: 02:00
    Primeiro backup: Full
    Posteriores: Incremental
```

Adapte o horário ao uso real do computador.

## 7.4 Configurar retenção

Use **Image Reserve Scheme** ou a opção equivalente de retenção.

Exemplo:

```text
Manter:
    3 Full
    + cadeia de incrementais associada
```

A quantidade correta depende da capacidade do disco.

### Não configure retenção agressiva

Evite algo como:

```text
Manter somente 1 imagem
```

se esse for seu único backup.

Uma falha silenciosa pode tornar a última imagem inválida antes de você perceber.

---

# 7.5 Ativar criptografia

Se a versão do Hasleo utilizada oferecer criptografia da imagem, considere ativá-la quando o disco de backup puder ser fisicamente acessado por terceiros.

**Mas guarde a senha em um local seguro.**

Um backup criptografado sem a senha pode ser tão inútil quanto um backup inexistente.

---

# 7.6 Executar o primeiro backup

Clique em:

```text
Proceed
```

Aguarde a conclusão.

Não considere o sistema pronto ainda.

Primeiro:

1. aguarde o backup terminar;
2. confira se a tarefa informa sucesso;
3. use a função de verificação/check da própria aplicação, quando disponível;
4. faça um teste real de restauração.

---

# 8. Criar e preparar o pendrive WinPE

## Opção A — Rufus

1. Baixe o Rufus do site oficial.
2. Conecte o pendrive.
3. Abra o Rufus.
4. Selecione o pendrive correto.
5. Em **Boot selection**, selecione a ISO do Sergei Strelec.
6. Escolha o esquema de partição adequado ao computador.

Para computadores modernos:

```text
Partition scheme: GPT
Target system: UEFI
```

Se você precisa compatibilidade com hardware legado, analise o modo de boot antes de escolher MBR/BIOS.

7. Inicie a gravação.

> A gravação normalmente apaga o conteúdo do pendrive.

---

# 9. Usar Ventoy como alternativa

O Ventoy é particularmente interessante para um pendrive de manutenção.

A página oficial informa que ele permite copiar múltiplas imagens ISO/WIM/IMG/VHD(x)/EFI e selecioná-las no menu de boot.

## Instalação

1. Baixe o Ventoy.
2. Extraia o arquivo.
3. Execute:

```text
Ventoy2Disk.exe
```

4. Confirme cuidadosamente o dispositivo USB.
5. Clique em **Install**.

Depois:

```text
Pendrive
│
├── Ventoy
│
├── Strelec.iso
├── Windows.iso
├── outro-WinPE.iso
└── ferramentas.iso
```

Basta copiar as ISOs para o pendrive.

Na inicialização, o Ventoy exibirá um menu.

---

# 10. Integrar o Hasleo ao ambiente de recuperação

Esta é uma parte crítica.

## Método recomendado

Não dependa de simplesmente copiar a pasta:

```text
C:\Program Files\Hasleo\Hasleo Backup Suite
```

para o pendrive.

Um programa Windows pode depender de:

- serviços;
- drivers;
- DLLs;
- componentes de sistema;
- registros;
- arquitetura específica;
- componentes WinPE.

Portanto, **copiar a pasta não garante que o Hasleo funcionará no WinPE**.

## Estratégias

### Estratégia 1 — Mídia WinPE criada pelo próprio Hasleo

Se a versão atual do Hasleo disponibilizar a criação de mídia WinPE/rescue media, prefira esse método para uma mídia dedicada ao Hasleo.

Vantagem:

```text
Hasleo
  ↓
WinPE compatível
  ↓
Restauração
```

### Estratégia 2 — Sergei Strelec

Use o Strelec como ambiente de diagnóstico e recuperação geral.

Se a versão utilizada já contiver uma ferramenta de backup compatível, use-a.

Se precisar especificamente do Hasleo:

1. obtenha uma versão que o fornecedor permita executar em ambiente portátil/WinPE;
2. teste-a;
3. coloque-a no pendrive;
4. inicialize o WinPE;
5. confirme que o aplicativo abre;
6. confirme que ele detecta o disco de backup;
7. confirme que ele reconhece uma imagem real.

### Não faça

Não assuma:

```text
copiei a pasta do Hasleo
        ↓
vai funcionar
```

Esse método pode falhar justamente quando você mais precisa dele.

---

# 11. Procedimento de restauração

## 11.1 Preparação

Antes de qualquer restauração, confirme:

- imagem correta;
- computador correto;
- disco de destino correto;
- data do backup;
- integridade da imagem;
- modo de boot (UEFI/Legacy);
- existência de BitLocker;
- esquema de partição GPT/MBR.

---

## 11.2 Inicializar o Sergei Strelec

1. Conecte o pendrive.
2. Ligue/reinicie o computador.
3. Abra o Boot Menu.

Teclas comuns:

```text
F12
F11
F8
ESC
```

A tecla varia conforme fabricante/modelo.

4. Escolha o pendrive.
5. Inicialize o WinPE 64-bit quando apropriado.

---

# 11.3 Localizar o disco de backup

No WinPE, as letras de unidade **podem mudar**.

Exemplo:

```text
Windows normal:
    C: = Windows
    Z: = Backup

WinPE:
    C: = talvez Windows
    D: = talvez EFI
    E: = pendrive
    F: = Backup
```

**Não presuma que o backup continuará sendo `Z:` no WinPE.**

Identifique o volume por:

- nome;
- capacidade;
- modelo do disco;
- estrutura de pastas.

Procure:

```text
Backups
└── NOME-DO-PC
```

---

# 11.4 Abrir o Hasleo

Abra o Hasleo pelo método de recuperação previamente testado.

Selecione:

```text
Restore
```

e depois procure a imagem de backup.

Use o navegador da própria aplicação.

> Não dependa de uma extensão específica como `.bim`. O formato deve ser identificado pela própria aplicação e pela versão do Hasleo que criou a imagem.

---

# 11.5 Escolher o destino

Esta é a etapa de maior risco.

Se a intenção é restaurar o Windows inteiro, confirme que o destino é:

```text
DISCO DO WINDOWS
```

e **não**:

```text
DISCO DE BACKUP
```

Compare:

- modelo;
- capacidade;
- número do disco;
- partições;
- identificação do fabricante.

Se houver dúvida, pare.

---

# 11.6 Restaurar

Após conferir tudo:

1. selecione a imagem;
2. selecione o disco de destino;
3. confirme o layout;
4. inicie a restauração;
5. aguarde a conclusão.

A restauração pode sobrescrever as partições do destino.

---

# 11.7 Reiniciar

Ao terminar:

1. feche o Hasleo;
2. desligue/reinicie;
3. remova o pendrive;
4. permita que o computador inicialize pelo SSD/HDD do Windows.

Se o Windows não inicializar, o problema pode estar relacionado ao boot/EFI/BCD e não necessariamente à imagem.

---

# 12. Problemas comuns

## Problema: Z: desapareceu do Explorer e o Hasleo não encontra o disco

Teste:

```powershell
Test-Path Z:\
```

Se retornar `True`, a unidade está acessível ao sistema.

Se o Hasleo ainda não funcionar:

1. desative temporariamente `NoDrives`;
2. reinicie o Explorer;
3. teste novamente;
4. valide o backup;
5. somente depois volte a ocultar.

---

## Problema: o backup agendado não executa

Verifique:

- computador ligado no horário;
- Hasleo instalado corretamente;
- tarefa existente;
- destino acessível;
- espaço livre;
- permissões;
- histórico/log da tarefa;
- se o volume continua montado como `Z:`.

Não dependa apenas de o ícone da unidade estar oculto.

---

## Problema: o WinPE não encontra o SSD

Pode faltar driver de armazenamento.

Verifique no BIOS/UEFI:

- modo SATA;
- AHCI/RAID/VMD;
- NVMe;
- controladora de armazenamento.

Em computadores modernos com Intel VMD/RAID, um WinPE sem o driver correspondente pode não enxergar o SSD do Windows.

---

## Problema: BitLocker

Se o Windows ou o disco estiver protegido por BitLocker, o ambiente WinPE poderá exigir:

- senha;
- chave de recuperação;
- desbloqueio da unidade.

**Guarde a chave de recuperação do BitLocker fora do próprio computador.**

---

## Problema: restauração terminou, mas o Windows não inicia

Possibilidades:

- EFI/ESP não restaurada corretamente;
- BCD inconsistente;
- ordem de boot UEFI alterada;
- restauração em disco diferente;
- diferença entre GPT e MBR;
- modo BIOS/UEFI incorreto;
- problema de driver.

Nesse cenário, use as ferramentas de reparo do WinPE/Windows e confirme o modo de boot.

---

# 13. Boas práticas

## 13.1 Tenha uma mídia de recuperação pronta

Não espere o Windows quebrar para descobrir que:

```text
o pendrive não inicializa
```

Teste o pendrive enquanto o computador está funcionando.

---

## 13.2 Teste a restauração

Um backup que nunca foi restaurado é apenas uma hipótese de backup.

Faça pelo menos um teste periódico:

```text
Backup
  ↓
Verificação
  ↓
Boot WinPE
  ↓
Detecção da imagem
  ↓
Detecção do disco
  ↓
Restauração/teste
```

---

## 13.3 Mantenha uma segunda cópia

Exemplo:

```text
SSD Windows
    │
    ├── Backup automático → HDD interno
    │
    └── Backup periódico → HDD externo
```

Melhor ainda:

```text
PC
 │
 ├── Backup 1 → HDD interno
 │
 ├── Backup 2 → HDD externo
 │
 └── Backup 3 → armazenamento externo/off-site
```

---

## 13.4 Não deixe o backup permanentemente exposto

Ocultar `Z:` é apenas uma proteção contra erro humano.

Para proteção adicional contra ransomware, considere:

- backup em disco externo desconectado quando não estiver sendo usado;
- armazenamento com controle de acesso;
- cópia offline;
- retenção de múltiplas versões.

---

## 13.5 Verifique o espaço

Mantenha uma margem de espaço livre.

Exemplo:

```text
Capacidade do disco: 4 TB
Ocupação recomendada: não chegar a 100%
```

Se a retenção estiver configurada para apagar backups antigos, confirme regularmente se o comportamento corresponde ao esperado.

---

# 14. Checklist final

## Configuração

- [ ] Windows 10/11 funcionando.
- [ ] Hasleo instalado.
- [ ] Disco de backup identificado corretamente.
- [ ] Partição formatada.
- [ ] Letra `Z:` atribuída.
- [ ] Pasta `Z:\Backups\NOME-DO-PC\` criada.
- [ ] `NoDrives` configurado.
- [ ] `Z:` oculto no Explorer.
- [ ] `Test-Path Z:\` retorna `True`.

## Backup

- [ ] Primeiro Full executado.
- [ ] Backup concluído sem erro.
- [ ] Imagem verificada.
- [ ] Agendamento configurado.
- [ ] Retenção configurada.
- [ ] Espaço livre conferido.
- [ ] Criptografia configurada, se necessária.
- [ ] Senha/chave armazenada em local seguro.

## Recuperação

- [ ] Sergei Strelec baixado de fonte confiável.
- [ ] Hash da ISO conferido.
- [ ] Pendrive criado.
- [ ] Pendrive inicializa.
- [ ] WinPE detecta o disco do Windows.
- [ ] WinPE detecta o disco de backup.
- [ ] Hasleo funciona no ambiente escolhido.
- [ ] Imagem é localizada.
- [ ] Disco de destino é identificado corretamente.
- [ ] Procedimento de restauração foi testado.

---

# Referências e downloads

| Ferramenta | Finalidade | Fonte |
|---|---|---|
| Hasleo Backup Suite | Backup/restauração do Windows | https://www.hasleo.com/ |
| Sergei Strelec WinPE | Ambiente de recuperação | https://sergeistrelec.name/ |
| Rufus | Criar USB bootável | https://rufus.ie/ |
| Ventoy | Multi-boot por ISO | https://www.ventoy.net/ |

## Fontes consultadas nesta revisão

- Hasleo Software: https://www.hasleo.com/
- Sergei Strelec — WinPE 11-10-8 English 2026.04.14: https://sergeistrelec.name/winpe-10-8-sergei-strelec-english/280-winpe-11-10-8-sergei-strelec-x86x64native-x86-20260414-english-version.html
- Rufus: https://rufus.ie/
- Ventoy: https://www.ventoy.net/en/download.html
- Ventoy — documentação de instalação: https://www.ventoy.net/en/doc_start.html

---

# Aviso de segurança

Este procedimento envolve operações de backup, restauração, particionamento e potencial sobrescrita de discos.

**Um erro na identificação do disco pode causar perda permanente de dados.**

Antes de restaurar:

1. confirme o modelo do disco;
2. confirme a capacidade;
3. confirme as partições;
4. confirme a imagem;
5. tenha uma segunda cópia dos dados importantes.

Nunca teste um procedimento de restauração pela primeira vez em uma situação de emergência sem antes validá-lo em condições controladas.

---

## Licenciamento e distribuição

Os softwares utilizados neste tutorial possuem seus próprios termos de licença e distribuição.

Para obter os componentes necessários:

* Baixe o **Hasleo Backup Suite** diretamente do site oficial da Hasleo.
* Baixe o **Sergei Strelec WinPE** a partir da página oficial do projeto.
* Baixe o **Rufus** diretamente do site oficial do projeto.
* Baixe o **Ventoy** diretamente do site oficial do projeto.

### Não distribua os arquivos de software junto com este tutorial

Ao compartilhar ou publicar este tutorial, mantenha apenas os arquivos que você possui autorização para distribuir.

Não inclua, salvo se a respectiva licença autorizar expressamente:

* ISOs do Sergei Strelec;
* instaladores proprietários do Hasleo;
* executáveis de terceiros;
* cópias de softwares comerciais;
* imagens de backup contendo dados pessoais.

Os links apresentados neste tutorial servem para orientar o usuário até as **fontes oficiais de download**.

> **Nota:** os termos de licença podem mudar. Antes de redistribuir qualquer software, arquivo ISO ou executável, consulte a licença vigente do respectivo projeto ou fabricante.

---

## Resumo operacional

```text
CONFIGURAÇÃO
    │
    ├── Instalar Hasleo
    │
    ├── Preparar disco de backup
    │
    ├── Atribuir Z:
    │
    ├── Ocultar Z: com NoDrives
    │
    ├── Criar Full
    │
    ├── Configurar Incremental/Differential
    │
    └── Configurar retenção
             │
             ▼
        BACKUP AUTOMÁTICO
             │
             ▼
      TESTAR RESTAURAÇÃO
             │
             ▼
        WINPE STRELEC
             │
             ▼
       LOCALIZAR IMAGEM
             │
             ▼
       CONFIRMAR DESTINO
             │
             ▼
          RESTAURAR
             │
             ▼
      WINDOWS OPERACIONAL
```
