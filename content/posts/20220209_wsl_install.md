---
title: "Windows10にWSL/Ubuntu/Python/Pip環境を最短でインストールしたい"
date: 2022-02-09T00:00:00+09:00
draft: false
tags:
  - etc
  - linux
---

全くWindows、PowerShell、仮想環境、Pythonも知らない人に、実行環境構築するために教えた時のメモです。

### カーネルアップデート実行

PowerShellのコマンドプロンプトで、以下を実行します。[^1]

```powershell
Invoke-WebRequest -Uri "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi" -OutFile "wsl_update_x64.msi"
./wsl_update_x64.msi
```
### WSL/Ubuntuインストール

PowerShellのコマンドプロンプトで、以下を実行します。[^2]

```powershell
wsl --install -d Ubuntu
wsl --set-default Ubuntu
```

### Ubuntuに入る

PowerShellのコマンドプロンプトで、以下を実行します。

```powershell
wsl
```

Ubuntuのコマンドプロンプトに入れたと思うので、以下を実行して、Python3が有効か確認します。

```sh
python3 --version
```

### pip3のインストール

Ubuntuのコマンドプロンプトで以下を実行します。

```sh
yes | apt update && apt install python3-pip
```

以下を実行して、pip3が有効か確認します。

```sh
pip --version
```

[^1]: 英語のウィンドウが出ますが、'next'をクリックして、'finish'まで行けば大丈夫です。
[^2]: "Retype new password:"と言うウィンドウが出ますが、それは、そのまま閉じて下さい。
