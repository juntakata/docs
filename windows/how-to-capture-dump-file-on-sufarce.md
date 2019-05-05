# Surface でのメモリ ダンプファイルの採取手順

Surface は右 Ctrl キーがないため、以下のレジストリでスキャン コードを登録します。WinRE で起動させる場合は、電源ボタンを押下後、F8 を数回連打すれば「お待ちください...」のメッセージと共に Windows RE (Recovery Environment) で起動してきます。

## 概要

メモリ ダンプを取得することで、現象発生時のメモリの状態を Memory.dmp ファイルとして保存し、
そこから現象発生時のオペレーティング システムのメモリの状態や、行われていた処理などの詳細を確認することができます。
 
## 影響

正規の手順でシャットダウンさせることにはならないため、最悪の場合、Dump ファイルの採取によって、
システムに必要なファイルが破損するなどの致命的な問題を引き起こす可能性がありますことを、予めご留意ください。
 
## 再起動の必要性

必要 (事前準備時に必要です。メモリダンプ採取時にも再起動されます)
 
## 採取ファイル

%systemroot%\Memory.dmp
  
## 手順

以下の手順にて、事象発生時に左 Ctrl キー + スペース キーの 2 回押下の操作で、完全メモリ ダンプが端末に保存されるように事前設定します。
 
1. PageFile の設定
 
    [初期サイズ]、[最大サイズ] の両方に物理メモリ + 300 Mbyte 以上の値を入力します。  
    (例えば 4096MB メモリの場合は 4396MB)

    キー: HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SessionManager\Memory Management  
    名前: PagingFiles  
    タイプ: REG_MULTI_SZ  
    設定値: <ページ ファイル保存先> <初期サイズ (MB)> <最大サイズ (MB)>  
    (例: c:\pagefile.sys 4396 4396)
 
2. 完全メモリ ダンプを保存する設定
 
   下記のレジストリ値を設定します。
 
   キー: HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\CrashControl  
   名前: CrashDumpEnabled  
   タイプ: REG_DWORD  
   設定値: 1  
 
   キー: HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\CrashControl  
   名前: AlwaysKeepMemoryDump  
   タイプ: REG_DWORD  
   設定値: 1  
 
3. キーボードから STOP エラーを発生させる設定
 
    下記のレジストリ値を設定します。
 
    キー: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\kbdhid\Parameters  
    名前: CrashOnCtrlScroll  
    種類: REG_DWORD  
    設定値: 0  
 
    以下のレジストリ キー配下に新規に CrashDump サブキーを追加します。
 
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\kbdhid
 
    Dump1Keys と Dump2Key を追加し、下記のレジストリ値を設定します。下記は、例として左 Ctrl と Space キーを 2 回押下することで強制メモリ ダンプが取得できるようにしています。
    
    キー: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\kbdhid\CrashDump
    
    名前: Dump1Keys  
    種類: REG_DWORD  
    設定値: 0x20 (アサインしたキー: 左 Ctrl)  
 
    名前: Dump2Key  
    種類: REG_DWORD  
    設定値: 0x3D (アサインしたキー : Space)  
 
    > 注意
    > Dump1Keys は 末尾に s がつき、Dump2Key には s がつきません。
 
4. OS を再起動します。
 
5. 再起動後 左 Ctrl キーを押しながらスペース キーを 2 回押下することでブルースクリーンが発生し、完全メモリ ダンプが保存されることを確認します。
 
6.	C:\Windows\Memory.dmp フォルダーファイルをご取得頂きご提供頂きますようお願いいたします。