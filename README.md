# Criando um WinPE Bootável para Instalar o Windows a partir de um Arquivo WIM

Este tutorial guiará você no processo de criação de um ambiente **WinPE bootável** e na configuração de um instalador do Windows usando um único arquivo **WIM**. Isso permite a instalação personalizada do Windows sem necessidade de um instalador tradicional.

---

## ✨ Passo 1: Instalar o Windows ADK e o WinPE Add-on

### 1. Instalar o Windows ADK

1. Baixe e instale o **Windows ADK**: [Download aqui](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install).
2. Durante a instalação, **selecione**:
   - ✅ **Deployment Tools** (Ferramentas de Implantação)

### 2. Instalar o WinPE Add-on

1. Baixe e instale o **WinPE Add-on**: [Download aqui](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install#winpe-add-on-for-the-adk).
2. Durante a instalação, **selecione**:
   - ✅ **Windows Preinstallation Environment (WinPE)**

Reinicie o computador após a instalação.

---

## ✨ Passo 2: Criar o Ambiente WinPE

Abra o **Prompt de Comando como Administrador** e execute:

```cmd
copype amd64 C:\WinPE_amd64
```

*(Para 32 bits, use `x86` em vez de `amd64`.)*

Para testar a ISO base do WinPE (opcional):

```cmd
MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE_Base.iso
```

---

## ✨ Passo 3: Copiar o Arquivo WIM e Criar Scripts

### 1. Copiar o Arquivo `.WIM` para o WinPE

```cmd
xcopy "C:\Caminho\Para\Windows.wim" C:\WinPE_amd64\media\sources\
```

### 2. Criar o Script de Instalação Automatizado

Crie um arquivo `install_windows.bat` e adicione:

```bat
@echo off
title Instalador Automatizado do Windows
color 0A
echo ==========================================
echo  INSTALADOR DO WINDOWS
echo ==========================================
echo Iniciando o processo de instalação...
pause

:: Verificar a unidade do WinPE
for %%I in (D E F G H) do if exist %%I:\sources\Windows.wim set WIMDRIVE=%%I

if "%WIMDRIVE%"=="" (
    echo ERRO: Arquivo WIM não encontrado!
    pause
    exit
)

:: Preparar o disco
diskpart /s X:\diskpart_script.txt

:: Aplicar a imagem do Windows
dism /Apply-Image /ImageFile:%WIMDRIVE%:\sources\Windows.wim /Index:1 /ApplyDir:C:\

:: Configurar o boot
bcdboot C:\Windows /s C: /f BIOS

:: Reiniciar o sistema
wpeutil reboot
```

### 3. Criar o Script de Particionamento do Disco

Crie um arquivo `diskpart_script.txt` e adicione:

```txt
select disk 0
clean
create partition primary
format fs=ntfs quick
assign letter=C
exit
```

### 4. Copiar os Scripts para o WinPE

```cmd
xcopy "C:\Caminho\Para\install_windows.bat" C:\WinPE_amd64\media\
xcopy "C:\Caminho\Para\diskpart_script.txt" C:\WinPE_amd64\media\
```

---

## ✨ Passo 4: Configurar o WinPE para Rodar o Instalador

1. Edite `startnet.cmd` dentro da pasta `System32` do WinPE:

```cmd
notepad C:\WinPE_amd64\media\Windows\System32\startnet.cmd
```

2. Adicione ao final:

```bat
wpeinit
X:\install_windows.bat
```

Salve e feche.

---

## ✨ Passo 5: Criar a ISO Bootável

Gerar a ISO final do WinPE:

```cmd
MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE.iso
```

---

## ✨ Passo 6: Testar o WinPE

### 1. Testar em uma Máquina Virtual
Se estiver usando **VirtualBox ou Hyper-V**:
- Crie uma nova VM
- Configure a **ISO como mídia de boot**
- Inicie a VM e teste a instalação

### 2. Criar um Pendrive Bootável
Para criar um **pendrive bootável**, insira um pendrive e use:

```cmd
MakeWinPEMedia /UFD C:\WinPE_amd64 E:
```

*(Substitua `E:` pela letra do seu pendrive.)*

---

## 🚀 O que acontece ao iniciar a ISO?

1. O WinPE inicia automaticamente.
2. O script `install_windows.bat` é executado e verifica a unidade correta do **arquivo WIM**.
3. O **disco é formatado** e a **imagem do Windows é aplicada**.
4. O **boot é configurado** e o sistema **reinicia automaticamente**.
5. O Windows inicia e finaliza a instalação!

---

## ✔ Conclusão

Agora você tem uma **ISO WinPE bootável** que permite instalar **Windows** de forma automatizada usando um arquivo `.WIM`. 

Se precisar de mais ajustes ou melhorias, contribua no repositório! 🚀😃
