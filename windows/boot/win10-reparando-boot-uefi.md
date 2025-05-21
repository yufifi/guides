# 🛠️ Guia: Criar partição EFI e restaurar o boot UEFI do Windows via Live Boot

Este guia descreve o processo de recriar manualmente a partição EFI e restaurar o boot UEFI de uma instalação Windows existente, utilizando um ambiente Live Boot (Windows PE ou instalador do Windows com `Shift + F10`).

---

## 📋 Cenário

Você está com:
- Windows instalado em modo **UEFI** com partição corrompida ou ausente de boot.
- Acesso a um ambiente de **Live Boot (CMD)**.
- Partições listadas via `diskpart` semelhantes a:
  - Partição 1: 16MB – MSR (Microsoft Reserved)
  - Partição 2: 476GB – Primária (instalação do Windows)
  - Partição 3: 652MB – Recuperação
  - Espaço não alocado: 344KB (não utilizável)

A partição EFI está ausente e será recriada manualmente.

---

## 🧱 Etapa 1 – Reduzir a partição principal para criar espaço EFI

1. Acesse o Prompt de Comando (`Shift + F10` no instalador).
2. Execute `diskpart`:

```cmd
diskpart
list disk
select disk 0            # Substitua pelo número do disco correto
list partition
select partition 2       # Partição onde o Windows está instalado
shrink desired=250       # Libera 250MB (ou mais) para a partição EFI
```

> 💡 O comando `shrink` pode falhar se a partição estiver cheia ou fragmentada. Tente com `desired=300` ou use ferramentas mais avançadas (GParted) se necessário.

---

## 📁 Etapa 2 – Criar e formatar a partição EFI

```cmd
create partition efi size=100
format fs=fat32 quick
assign letter=S
exit
```

- Isso cria a partição EFI (obrigatória para boot UEFI).
- A letra `S:` será usada temporariamente.

---

## 🧭 Etapa 3 – Identificar a partição correta do Windows

No prompt:

```cmd
diskpart
list volume
```

- Localize o volume que contém o Windows. Normalmente é o volume com ~476GB e sistema de arquivos `NTFS`.
- Atribua uma letra, se necessário:

```cmd
select volume X           # Substitua X pelo número do volume do Windows
assign letter=W
exit
```

Verifique se a pasta `W:\Windows` existe:

```cmd
dir W:\Windows
```

---

## 🧩 Etapa 4 – Restaurar os arquivos de boot com `bcdboot`

Execute:

```cmd
bcdboot W:\Windows /s S: /f UEFI
```

- `W:` → Instalação do Windows.
- `S:` → Partição EFI recém-criada.
- `/f UEFI` → Garante que o sistema bootará em modo UEFI.

> 🔍 Se ocorrer erro `c000000f`, certifique-se que:
> - `W:\Windows` realmente contém uma instalação válida.
> - A partição EFI (`S:`) está formatada em FAT32.
> - A ISO do Live Boot usada é uma imagem de instalação completa.

Use o parâmetro `/v` para mais detalhes:
```cmd
bcdboot W:\Windows /s S: /f UEFI /v
```

---

## ✅ Etapa 5 – Verifique a estrutura da partição EFI

```cmd
dir S:\EFI\Microsoft\Boot
```

Você deve ver arquivos como:
- `bootmgfw.efi`
- `BCD`
- Outros arquivos `.efi`

---

## 🔁 Etapa 6 – Reinicie e teste

Remova o pendrive ou disco de boot e reinicie o computador. O Windows deve iniciar normalmente em modo UEFI.

---

## 🧯 Solução alternativa (caso o `bcdboot` continue falhando)

- Use um ambiente mais completo como o **Windows PE criado com ADK + WinPE Add-on**.
- Reinstale o Windows e permita que o instalador crie automaticamente todas as partições (⚠️ apaga os dados).

---

## 🧾 Referências

- [`bcdboot` Microsoft Docs](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcdboot-command-line-options)
- [UEFI/GPT Partitioning Requirements](https://learn.microsoft.com/en-us/windows/win32/medfound/uefi)

---

📦 **Licença**: MIT  
📂 **Autor**: [Yuri Filgueira (yufifi)]  
📅 **Última atualização**: 21-05-2025
