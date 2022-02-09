---
title: "Windows10にWSL/Ubuntuを最短でインストールしたい"
date: 2022-02-09T00:00:00+09:00
draft: false
tags:
  - etc
  - linux
---

全くWindows、PowerShell、仮想環境、Pythonも知らない人に、実行環境構築するために教えた時のメモです。

### カーネルアップデート実行

PowerShellのコマンドプロンプトを実行します、

```powershell
Invoke-WebRequest -Uri "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi" -OutFile "wsl_update_x64.msi"
./wsl_update_x64.msi
```
### WSL/Ubuntuインストール

PowerShellのコマンドプロンプトで、以下を実行します。

```powershell
wsl --install -d Ubuntu
wsl --set-default Ubuntu
```

### Ubuntuに入る

PowerShellのコマンドプロンプトで、以下を実行します。

```powershell
wsl
```

Ubuntuのコマンドプロンプトに入れたと思うので、以下を実行します。

```sh
python3 --version
```
