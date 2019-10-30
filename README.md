# **MeCab-x64**

## **MeCab とは**
MeCab はオープンソースの形態素解析エンジンです。

公式サイトからダウンロードできる MeCab は Windows の32ビット版です。

https://taku910.github.io/mecab/

すでに64ビット版の MeCab のバイナリを公開している非公式サイトもあります。

ここでは、Windows の64ビット版の MeCab のビルド方法について記載します。

また、Windows 版のインストーラには付属していない jumandic 辞書と、
新語に対応した mecab-ipadic-neologd 辞書のビルド方法についても記載します。


## **MeCab-x64**
Windows の64ビット版の MeCab の作成方法は以下の通りです。

- MeCab(64ビット版)
 - MeCabのダウンロード
 - MeCab(32ビット版)のインストール
 - MeCab(64ビット版)のソースファイルの修正
 - MeCab(64ビット版)のビルド
 - MeCab(64ビット版)環境の作成
 - MeCab(64ビット版)の動作確認

- jumandic 辞書
 - mecab ソースファイルのダウンロード
 - jumandic 辞書のビルド
 - jumandic 辞書環境の作成
 - jumandic 辞書の動作確認

- mecab-ipadic-neologd 辞書
 - mecab-ipadic-neologd ソースファイルのダウンロード
 - mecab-ipadic-neologd 辞書のビルド
 - mecab-ipadic-neologd 辞書環境の作成
 - mecab-ipadic-neologd 辞書の動作確認

- おまけ


### **MeCabのダウンロード**
以下のサイトから MeCab のインストーラとソースファイルをダウンロードします。

https://taku910.github.io/mecab/

- インストーラ : mecab-0.996.exe
- ソースファイル : mecab-0.996.tar.gz


### MeCab(32ビット版)のインストール
インストーラ(mecab-0.996.exe)を実行し MeCab(32ビット版) をインストールします。

- MeCabセットアップ画面の「辞書の文字コードの選択」で、 **UTF-8** (デフォルトはSJIS) を指定します。
- MeCabセットアップ画面の「インストール先の指定」で、MeCabをインストールするパスを指定します。
ここでは "C:\MeCab-x64" にインストールするものとして説明しますので、適宜読み替えてください。

**Note:**  
ここで指定したパスの MeCab を64ビット化しますので、上書き可能なディレクトリを指定してください。  
"C:\\Program Files(x86)\\MeCab" のような Windows システムの管理下にあるパスにインストールした場合、セキュリティの観点から実行ファイルを上書きできない場合があります。
また、VirtualStore 機能により、環境構築したユーザ以外のユーザには変更が反映されない場合があります。

**Note:**  
インストールイメージのコピーを作成し、任意のパスに置いてしまえば、アンインストールしてしまってもかまいません。
MeCab(64ビット版)をコマンド指定のみで実行するには、MeCab(64ビット版)のあるパスを環境変数 PATH に設定すればよいだけです。

**Note:**  
MeCab を引数なし(--rcfileを指定しない)で実行した場合の設定ファイル(etc\\mecabrc)の読み込み位置はインストールパスになります。
MeCab 実行時に毎回指定するのが面倒な場合は、利用する場所を指定しておくとよいでしょう。
また、MeCab(64ビット版) のビルド時にもインストールパスを指定しますので、MeCab(64ビット版)として利用するパスを指定しておくとよいでしょう。


### **MeCab(64ビット版)のソースファイルの修正**
ソースファイル(mecab-0.996.tar.gz)を展開します。

展開したソースの mecab-0.996\\src ディレクトリに移動します。

~~~
cd mecab-0.996\src
~~~

**Makefile.msvc**  
Makefile.msvc.in を Makefile.msvc にコピーします。

~~~
copy Makefile.msvc.in Makefile.msvc
~~~

**Note:**  
ここでは、テンプレートファイルをもとに修正していますが、Linuxで ./configure を実行すれば、8,9行目が修正された Makefile.msvc ファイルが生成されます。

Makefile.msvc の6行目の /MACHINE:X86 を /MACHINE:X64 に修正します。

~~~
LDFLAGS = /nologo /OPT:REF /OPT:ICF /LTCG /NXCOMPAT /DYNAMICBASE /MACHINE:X64 ADVAPI32.LIB
~~~

Makefile.msvcの
8行目の @DIC_VERSION@ を 102 に、
9行目の @VERSION@ を 0.996 に、
11行目の c:\\\\Program Files\\\\mecab\\\\etc\\\\mecabrc を C:\MeCab-x64\\\\etc\\\\mecabrc に修正します。

~~~
DEFS =  -D_CRT_SECURE_NO_DEPRECATE -DMECAB_USE_THREAD \
        -DDLL_EXPORT -DHAVE_GETENV -DHAVE_WINDOWS_H -DDIC_VERSION=102 \
        -DVERSION="\"0.996\"" -DPACKAGE="\"mecab\"" \
        -DUNICODE -D_UNICODE \
        -DMECAB_DEFAULT_RC="\"C:\\MeCab-x64\\etc\\mecabrc\""
~~~

**common.h**  
common.h の17行目に iterator のインクルードを追加します。

~~~
#include <iterator>
~~~

**feature_index.cpp**  
feature_index.cpp の356行目のキャストを (size_t) から (unsigned int) に修正します。

~~~
            case 't':  os_ << (unsigned int)path->rnode->char_type;     break;
~~~

**writer.cpp**  
writer.cpp の260行目にキャスト (unsigned int) を追加します。

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

src 配下にモジュールが作成されます。


**Note:**  
Visual Studio 2017以外のバージョンを使用する場合は、適宜読み替えてください。


### **MeCab(64ビット版)環境の作成**
MeCab(32ビット版)環境を変更しますので、念のためバックアップしておいてください。

ビルドした MeCab(64ビット版) を、事前にインストールした MeCab(32ビット版) 環境の C:\MeCab-x64\bin 配下に上書きします。

~~~
> dir C:\MeCab-x64\bin
2019/10/30  16:59         1,914,880 libmecab.dll
2019/10/30  16:59           106,496 mecab-cost-train.exe
2019/10/30  16:59           106,496 mecab-dict-gen.exe
2019/10/30  16:59           106,496 mecab-dict-index.exe
2019/10/30  16:59           106,496 mecab-system-eval.exe
2019/10/30  16:59           106,496 mecab-test-gen.exe
2019/10/30  16:59           106,496 mecab.exe
~~~

ビルドした MeCab(64ビット版) を、事前にインストールした MeCab(32ビット版)環境の C:\MeCab-x64\sdk 配下に上書きします。

~~~
> dir C:\MeCab-x64\sdk
2019/10/30  16:59            29,940 libmecab.lib
2013/02/18  02:21            42,106 mecab.h
...
~~~

以上で MeCab(64ビット版) 環境ができました。


### **MeCab(64ビット版)の動作確認**
コマンドプロンプトを起動します。
コマンドプロンプトで以下のコマンドを実行します。

~~~
> cd C:\MeCab-x64\bin
> chcp 65001
> echo 吾輩は猫である | mecab --rcfile ..\etc\mecabrc
吾輩    名詞,代名詞,一般,*,*,*,吾輩,ワガハイ,ワガハイ
は      助詞,係助詞,*,*,*,*,は,ハ,ワ
猫      名詞,一般,*,*,*,*,猫,ネコ,ネコ
で      助動詞,*,*,*,特殊・ダ,連用形,だ,デ,デ
ある    助動詞,*,*,*,五段・ラ行アル,基本形,ある,アル,アル
EOS
~~~


## **jumandic 辞書**
jumandic 辞書のビルドは Linux を使用します。

**Note:**  
ipadic 辞書はインストーラ(32ビット版)に含まれています。
自分でビルドしたい場合は、master ソースファイルに含まれている mecab-ipadic に対して mecab-jumanic と同様の操作を行うことで作成することができます。


### **mecab ソースファイルのダウンロード**
mecab の git リポジトリから master ソースファイルをダウンロードします。
gitを使用している場合は clone してもよいでしょう。

https://github.com/taku910/mecab


### **jumandic 辞書のビルド**
ダウンロードした master ソースファイルを展開します。
以下のコマンドを実行し、jumandic をビルドします。

~~~
cd mecab-master/mecab-jumandic
./configure --with-charset=utf8
make
sudo make install
cd /usr/local/lib/mecab/dic
tar cvf /tmp/jumandic.tar jumandic/
~~~

作成された /tmp/jumandic.tar ファイルを、何らかの方法で Windows にコピーしてください。


### **jumandic 辞書環境の作成**
/tmp/jumandic.tar ファイルを展開し、MeCab(64ビット環境) の C:\MeCab-x64\dic パス配下に配置します。
以下のようなファイル構成になります。

~~~
dir C:\MeCab-x64\dic\jumandic
2019/10/30  16:24           262,464 char.bin
2019/10/30  16:24               240 dicrc
2019/10/30  16:24           114,693 left-id.def
2019/10/30  16:24         7,038,756 matrix.bin
2019/10/30  16:24             1,014 pos-id.def
2019/10/30  16:24             4,957 rewrite.def
2019/10/30  16:24           114,693 right-id.def
2019/10/30  16:24       136,308,720 sys.dic
2019/10/30  16:24             4,935 unk.dic
~~~

mecabrc ファイルをコピーして修正し、jumandic 辞書用の設定ファイルを作成します。

~~~
copy C:\MeCab-x64\etc\mecabrc C:\MeCab-x64\etc\mecabrc-jumandic
~~~

**mecabrc**  
6行目の $(rcpath)\..\dic\ipadic を $(rcpath)\..\dic\jumandic に変更します。

~~~
dicdir =  $(rcpath)\..\dic\jumandic
~~~

**Note:**  
jumandic 辞書をデフォルト辞書にする場合は、mecabrc ファイルを修正してください。


### **jumandic 辞書の動作確認**
コマンドプロンプトを起動します。
コマンドプロンプトで以下のコマンドを実行します。

~~~
> cd C:\MeCab-x64\bin
> chcp 65001
> echo 吾輩は猫である | mecab --rcfile ..\etc\mecabrc-jumandic
吾輩    名詞,普通名詞,*,*,吾輩,わがはい,代表表記:我が輩/わがはい カテゴリ:人
は      助詞,副助詞,*,*,は,は,*
猫      名詞,普通名詞,*,*,猫,ねこ,代表表記:猫/ねこ 漢字読み:訓 カテゴリ:動物
である  判定詞,*,判定詞,デアル列基本形,だ,である,*
EOS
~~~

ipadic 辞書の場合とは若干結果が変わっていることが分かります。


## **mecab-ipadic-neologd 辞書**
mecab-ipadic-neologd 辞書のビルドは Linux を使用します。


### **mecab-ipadic-neologd ソースファイルのダウンロード**
mecab-ipadic-neologd の git リポジトリから master ソースファイルをダウンロードします。
git を使用している場合は clone してもよいでしょう。

https://github.com/neologd/mecab-ipadic-neologd


### **mecab-ipadic-neologd 辞書のビルド**
ダウンロードした master ソースファイルを展開します。
以下のコマンドを実行し、mecab-ipadic-neologd をビルドします。

~~~
cd mecab-ipadic-neologd-master
./bin/install-mecab-ipadic-neologd -n -y
cd /usr/local/lib/mecab/dic
tar cvf /tmp/mecab-ipadic-neologd.tar mecab-ipadic-neologd/
~~~

作成された /tmp/mecab-ipadic-neologd.tar ファイルを、何らかの方法で Windows にコピーしてください。


### **mecab-ipadic-neologd 辞書環境の作成**
/tmp/mecab-ipadic-neologd.tar ファイルを展開し、MeCab(64ビット環境) の C:\MeCab-x64\dic パス配下に配置します。
以下のようなファイル構成になります。

~~~
dir C:\MeCab-x64\dic\mecab-ipadic-neologd
2019/10/30  16:18           262,496 char.bin
2019/10/30  16:18               693 dicrc
2019/10/30  16:18            74,686 left-id.def
2019/10/30  16:18         3,463,716 matrix.bin
2019/10/30  16:18             1,923 pos-id.def
2019/10/30  16:18             7,457 rewrite.def
2019/10/30  16:18            74,686 right-id.def
2019/10/30  16:18       890,360,962 sys.dic
2019/10/30  16:18             5,684 unk.dic
~~~

mecabrc ファイルをコピーして修正し、mecab-ipadic-neologd 辞書用の設定ファイルを作成します。

~~~
copy C:\MeCab-x64\etc\mecabrc C:\MeCab-x64\etc\mecabrc-ipadic-neologd
~~~

**mecabrc-ipadic-neologd**  
6行目の $(rcpath)\..\dic\ipadic を $(rcpath)\..\dic\mecabrc-ipadic-neologd に変更します。

~~~
dicdir =  $(rcpath)\..\dic\mecabrc-ipadic-neologd
~~~

**Note:**  
mecabrc-ipadic-neologd 辞書をデフォルト辞書にする場合は、mecabrc ファイルを修正してください。


### **mecab-ipadic-neologd 辞書の動作確認**
コマンドプロンプトを起動します。
コマンドプロンプトで以下のコマンドを実行します。

~~~
> cd C:\MeCab-x64\bin
> chcp 65001
> echo 吾輩は猫である | mecab --rcfile ..\etc\mecabrc-ipadic-neologd
吾輩は猫である  名詞,固有名詞,一般,*,*,*,吾輩は猫である,ワガハイハネコデアル,ワ ガハイワネコデアル
EOS
~~~

固有名詞として認識されました。

**Note:**  
mecab-ipadic-neologd 辞書は新語を取り入れて新しくなっているようですので、定期的にアップデートするとよいでしょう。


## **おまけ**
外国の人参の政権はないだろう。

ということで、新語を含む mecab-ipadic-neologd 辞書が良いようです。

~~~
> echo 外国人参政権 | mecab --rcfile ..\etc\mecabrc
外国    名詞,一般,*,*,*,*,外国,ガイコク,ガイコク
人参    名詞,一般,*,*,*,*,人参,ニンジン,ニンジン
政権    名詞,一般,*,*,*,*,政権,セイケン,セイケン
EOS
> echo 外国人参政権 | mecab --rcfile ..\etc\mecabrc-jumandic
外国    名詞,普通名詞,*,*,外国,がいこく,代表表記:外国/がいこく カテゴリ:場所-そ の他 ドメイン:政治
人参    名詞,普通名詞,*,*,人参,にんじん,代表表記:人参/にんじん カテゴリ:植物;人 工物-食べ物 ドメイン:料理・食事
政権    名詞,普通名詞,*,*,政権,せいけん,代表表記:政権/せいけん 組織名末尾 カテゴリ:抽象物 ドメイン:政治
EOS
> echo 外国人参政権 | mecab --rcfile ..\etc\mecabrc-ipadic-neologd
外国人参政権    名詞,固有名詞,一般,*,*,*,外国人参政権,ガイコクジンサンセイケン, ガイコクジンサンセイケン
EOS
~~~

