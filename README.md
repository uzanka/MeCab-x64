# **MeCab-x64**

## **MeCab とは**
MeCabはオープンソースの形態素解析エンジンです。

公式サイトからダウンロードできるMeCabはWindowsの32ビット版です。

https://taku910.github.io/mecab/

すでに64ビット版のMeCabのバイナリを公開している非公式サイトもあります。

ここでは、Windowsの64ビット版のMeCabのビルド方法について記載します。


## **MeCab-Windows-x64**
Windowsの64ビット版のMeCabの作成方法は以下の通りです。

- MeCabのダウンロード
- MeCab(32ビット版)のインストール
- MeCab(64ビット版)のソースファイルの修正
- MeCab(64ビット版)のビルド
- MeCab(64ビット版)環境の作成
- MeCab(64ビット版)の動作確認


### **MeCabのダウンロード**
以下のサイトからMeCabのインストーラとソースファイルをダウンロードします。

https://taku910.github.io/mecab/

- インストーラ : mecab-0.996.exe
- ソースファイル : mecab-0.996.tar.gz


### MeCab(32ビット版)のインストール
インストーラ(mecab-0.996.exe)を実行しMeCab(32ビット版)をインストールします。

- MeCabセットアップ画面の「辞書の文字コードの選択」で、 **UTF-8** (デフォルトはSJIS) を指定します。
- MeCabセットアップ画面の「インストール先の指定」で、MeCabをインストールするパスを指定します。

**Note:**  
ここで指定したパスのMeCabを64ビット化しますので、上書き可能なディレクトリを指定してください。  
"C:\\Program Files(x86)\\MeCab" のようなWindowsシステムの管理下にあるパスにインストールした場合、セキュリティの観点から実行ファイルを上書きできない場合があります。
また、VirtualStore機能により、環境構築したユーザ以外のユーザには変更が反映されない場合があります。

**Note:**  
インストールイメージのコピーを作成し、任意のパスに置いてしまえば、アンインストールしてしまってもかまいません。
MeCab(64ビット版)をコマンド指定のみで実行するには、MeCab(64ビット版)のあるパスを環境変数PATHに設定すればよいだけです。

**Note:**  
MeCabを引数なし(--rcfileを指定しない)で実行した場合の設定ファイル(etc\\mecabrc)の読み込み位置はインストールパスになります。
MeCab実行時に毎回指定するのが面倒な場合は、利用する場所を指定しておくとよいでしょう。
また、MeCab(64ビット版)のビルド時にもインストールパスを指定しますので、MeCab(64ビット版)として利用するパスを指定しておくとよいでしょう。


### **MeCab(64ビット版)のソースファイルの修正**
ソースファイル(mecab-0.996.tar.gz)を展開します。

展開したソースのmecab-0.996\\srcディレクトリに移動します。

~~~
cd mecab-0.996\src
~~~

**Makefile.msvc**  
Makefile.msvc.inをMakefile.msvcにコピーします。

~~~
copy Makefile.msvc.in Makefile.msvc
~~~

**Note:**  
ここでは、テンプレートファイルをもとに修正していますが、Linuxで ./configure を実行すれば、8,9行目が修正されたMakefile.msvcファイルが生成されます。

Makefile.msvcの6行目の /MACHINE:X86 を /MACHINE:X64 に修正します。

~~~
LDFLAGS = /nologo /OPT:REF /OPT:ICF /LTCG /NXCOMPAT /DYNAMICBASE /MACHINE:X64 ADVAPI32.LIB
~~~

Makefile.msvcの
8行目の @DIC_VERSION@ を 102 に、
9行目の @VERSION@ を 0.996 に、
11行目の c:\\\\Program Files\\\\mecab\\\\etc\\\\mecabrc を {MeCab(32ビット版)のインストールで指定したパス}\\\\etc\\\\mecabrc に修正します。

~~~
DEFS =  -D_CRT_SECURE_NO_DEPRECATE -DMECAB_USE_THREAD \
        -DDLL_EXPORT -DHAVE_GETENV -DHAVE_WINDOWS_H -DDIC_VERSION=102 \
        -DVERSION="\"0.996\"" -DPACKAGE="\"mecab\"" \
        -DUNICODE -D_UNICODE \
        -DMECAB_DEFAULT_RC="\"C:\\MeCab-x64\\etc\\mecabrc\""
~~~

**common.h**  
common.hの17行目にiteratorのインクルードを追加します。

~~~
#include <iterator>
~~~

**feature_index.cpp**  
feature_index.cppの356行目のキャストを (size_t) から (unsigned int) に修正します。

~~~
            case 't':  os_ << (unsigned int)path->rnode->char_type;     break;
~~~

**writer.cpp**  
writer.cppの260行目にキャスト (unsigned int) を追加します。

~~~
          case 'L': *os << (unsigned int)lattice->size(); break;
~~~


### **MeCab(64ビット版)のビルド**
「VS 2017 用 x64 Native Tools コマンドプロンプト」を起動します。
ソースファイルのあるディレクトリに移動し、ビルドを実行します。

~~~
cd {ソースファイルを展開したパス}\mecab-0.996\src
nmake -f Makefile.msvc
~~~

src配下にモジュールが作成されます。


**Note:**  
Visual Studio 2017以外のバージョンを使用する場合は、適宜読み替えてください。


### **MeCab(64ビット版)環境の作成**
MeCab(32ビット版)環境を変更しますので、念のためバックアップしておいてください。

ビルドしたMeCab(64ビット版)を、事前にインストールしたMeCab(32ビット版)環境のbin配下に上書きします。

~~~
mecab.exe
mecab-cost-train.exe
mecab-dict-gen.exe
mecab-dict-index.exe
mecab-system-eval.exe
mecab-test-gen.exe
libmecab.dll
~~~

ビルドしたMeCab(64ビット版)を、事前にインストールしたMeCab(32ビット版)環境のsdk配下に上書きします。

~~~
libmecab.lib
mecab.h
~~~

以上でMeCab(64ビット版)環境ができました。


### **MeCab(64ビット版)の動作確認**
コマンドプロンプトを起動します。
コマンドプロンプトで以下のコマンドを実行します。

~~~
> cd {MeCab(64ビット版)パス}\bin
> chcp 65001
> echo 吾輩は猫である | mecab --rcfile ..\etc\mecabrc
吾輩    名詞,代名詞,一般,*,*,*,吾輩,ワガハイ,ワガハイ
は      助詞,係助詞,*,*,*,*,は,ハ,ワ
猫      名詞,一般,*,*,*,*,猫,ネコ,ネコ
で      助動詞,*,*,*,特殊・ダ,連用形,だ,デ,デ
ある    助動詞,*,*,*,五段・ラ行アル,基本形,ある,アル,アル
EOS
~~~

