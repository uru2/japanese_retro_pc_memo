# CD-SD STDをWindows環境でビルド
Web上で [CD-SD STD](https://www.vector.co.jp/soft/dos/hardware/se000298.html) のビルド方法を解説しているサイトはいくつか存在しますが、手順が古くビルドに必要なツールの入手が困難となってしまいました。  
そこでアセンブラに [JWasm](https://github.com/Baron-von-Riedesel/JWasm) を利用することでLINKやEXE2BIN(代替品含む)が不要であることがわかったため、新しい手法として記録を残しておきます。

ダウンロードの仕方、lzh/zip展開方法、コマンド実行方法、CONFIG.SYS設定方法などの基本的なことはここでは説明しません。

## 1. ダウンロード
### 1.1. CD-SD STDダウンロード
ベクターより [CD-SD STDのアーカイブ](https://www.vector.co.jp/soft/dl/dos/hardware/se000298.html) をダウンロード。  
必須なのは "CD-SD STD 8" の `cdsdstd2.lzh`。8Aを適用したい場合は `std8a.lzh` もダウンロード。

### 1.2. JWasmダウンロード
GitHubの [JWasm Releases](https://github.com/Baron-von-Riedesel/JWasm/releases) より "win32" 用のビルド済みバイナリのアーカイブをダウンロード。

## 2. ファイル展開
### 2.1. CD-SD STD
`cdsdstd2.lzh` に含まれる `SOURCE.LZH` を展開、展開先ディレクトリは `SOURCE` とする。  
8A適用は同名のソースコードを置き換え。

### 2.2. JWasm
win32バイナリアーカイブ内の `JWasm.exe` を `SOURCE` にコピー。

## 3. ビルド
`SOURCE` ディレクトリ内で下記コマンドを実行。

```
jwasm -bin -FoCDSD.SYS -DDRIVE={Drive} -DIF_TYPE={IF_Type} -DDRV_TYPE={Drv_Type} CDSD.ASM
```

`{Drive}` `{IF_Type}` `{Drv_Type}` の設定値は `ASM.DOC` を参照。

よほど古いドライブでなければ下記コマンドでよいかと、要ASPIマネージャー。

```
jwasm -bin -FoCDSD.SYS -DDRIVE=SCSI2 -DIF_TYPE=ASPI -DDRV_TYPE=XA CDSD.ASM
```

## 4. ビルド完了
`CDSD.SYS` が作成されていれば完了。

# その他
## ビルドに関して
### MS-DOS環境上でのビルド
JWasm "dos"バイナリの `JWASMR.EXE` か `JWASMD.EXE` を使えばMS-DOS環境内でビルド可能です。

- `JWASMR.EXE` x86リアルモードでも動作可能なバイナリでCD-SDのソースコードも小規模なので通常はこちらでOK、ただexe自体が大きいのでメモリの空きには注意。  
- `JWASMD.EXE` 32bit(386)プロテクトモード用なので実行にはプロテクトメモリとDPMIホスト必須、MASM代替としては実質こちらが実用的かと。

### マイクロソフト製ツールを「合法的」に入手しビルド
オープンソース化されたMS-DOSのリポジトリから [ビルドに必要なツール](https://github.com/microsoft/MS-DOS/tree/main/v4.0/src/TOOLS) の入手はできます。  
ただしx64版Windows上では [MS-DOS Player](http://takeda-toshiya.my.coocan.jp/msdos/) 経由でないと各ツールの実行ができません。

### IF_Type=PC9801 の指定
[Thor-Hammerのホームページ](https://akiyuki.boy.jp/old/t98next/midi/) にある「T98-NEXT(7th Beta以降)用CD-ROMモジュール」内のドキュメントによるとPC-9801-55用CD-DA再生ルーチンにはバグがあるとのこと。

### IF_Type=FM の指定
FM TOWNSでは問題があるようなので [YSSCSICD.SYS と SYSDRV.EXE](http://ysflight.in.coocan.jp/FM/towns/internalCDROM/YSSCSICD_J.html) を使いましょう。

## PC-98用SCSI I/Fで専用のASPIマネージャーがない場合
一部メーカーではASPIマネージャーの提供がないのですが [HENRRY](https://www.vector.co.jp/soft/dos/hardware/se000679.html) が使えるかもしれません。

手元にある緑電子 MDC-925L / MDC-926Rsでは利用可能です。

ちなみにメーカー純正ASPIマネージャーが存在する ICM IF-2769 / Logitec LHA-301 でHENRRYの利用も可能なため、 **非推奨ですが** Cバス用I/Fであれば他社のASPIマネージャーが使える可能性があります。

## MSCDEXの入手
MS-DOS上でCDアクセスするために必要な `MSCDEX.EXE` はマイクロソフトの製品であるため、本来であれば同社のOS付属のもの(ライセンス供給分含む)を利用すべきです。(昔マイクロソフトのダウンロードセンターで最終版?の2.23が入手できましたが…)

そのため代替となる [SHSUCDX](http://adoxa.altervista.org/shsucdx/) の利用をおすすめします。何よりメモリ常駐サイズが小さいのでこれだけで使う価値があります。
