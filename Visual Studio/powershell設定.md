# Visual Studio で PowerShell を起動するとき、スクリプトの実行を自動的に有効化する

PowerShellのデフォルトの実行ポリシーではスクリプトを実行できない。都度実行ポリシーを変更するのは手間なので、起動時の引数を変更する。

ターミナルを開き、設定画面を開く。

引数に `-ExecutionPolicy RemoteSigned` を追加して保存する。

```cmd
-NoExit -ExecutionPolicy RemoteSigned -Command "& { Import-Module """$env:VSAPPIDDIR\..\Tools\Microsoft.VisualStudio.DevShell.dll"""; Enter-VsDevShell -SkipAutomaticLocation -SetDefaultWindowTitle -InstallPath $env:VSAPPIDDIR\..\..\}"
```
