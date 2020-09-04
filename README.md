# C-Cpp_makefile
C/C++用makefile，ファイルの依存関係をmakefileが自動で判断し，オブジェクトの種類毎にディレクトリ分割してくれる

# シンプルで応用の聞くmakefile
C/C++のビルド自動化にかなり役立つmakefileのサイトがあったので，改良したmakefileと解説を残しておきます．

参考サイト：<a href="http://urin.github.io/posts/2013/simple-makefile-for-clang" rel="nofollow">シンプルで応用の効くmakefileとその解説</a>, 2020/9/4閲覧  
makefileのgit：<a href="https://github.com/Yuuki-Katsusaka/C-Cpp_makefile" rel="nofollow">https://github.com/Yuuki-Katsusaka/C-Cpp_makefile</a>

## ディレクトリ構成
カレントディレクトリ名が&quot;example&quot;でmakeを実行した場合，以下のようにディレクトリが分かれファイルが生成される


```
 ./&quot;example&quot;
 |-- makefile 
 |-- bin
 |   `-- exe &lt;- (TARGET) 実行ファイル
 |-- include
 |   `-- example.h
 |-- obj &lt;- (OBJDIR) 中間ファイル生成先ディレクトリ
 |   |-- example.d &lt;- (DEPENDS) 依存関係ファイル
 |   `-- example.o &lt;- (OBJECTS) オブジェクトファイル
 `-- source
    `-- example.cpp
```


## 使用方法
### make
依存関係を考慮し，差分コンパイルを行う．
sourceディレクトリ内のソースファイルはもちろん，includeディレクトリ内のヘッダファイルの更新も自動的に検出して差分コンパイルされる．サイトによればライブラリの更新も検出するらしいが未確認．

### make all
全ての中間ファイル(オブジェクトファイル，依存関係ファイル)と実行ファイルを削除し，強制的に全ソースをコンパイルする．

### make run
実行ファイルを実行する．（初期設定なら実行ファイル名は&quot;exe&quot;）

### make clean
全ての中間ファイル(オブジェクトファイル，依存関係ファイル)と実行ファイルを削除する．

## 解説
makefile内で使用されているマクロの解説．

### コンパイラの指定(COMPILER)
コンパイラはCOMPILERの値を用いる。初期値はg++。C言語のみの場合はgccに変更する．

### コンパイルオプション(CFLAGS)
コンパイルオプションとしてCFLAGSの値を用いる。-Dオプションによる#defineの追加、最適化オプション、コードカバレッジ用の-coverageなどを用いる場合はここに記述する。

以下は初期値の解説。

* -Wall -Wextra -Winit-self -Wno-missing-field-initializers
    * -Wall : コンパイルワーニングのレベルを最大にする。
    * -Wextra : 歴史的理由により-Wallを使用でも抑制される警告を抑制しない。つまり可能な限り全ての警告を出す。
    * -W* （それ以外） : 詳しくはWarning Options - Using the GNU Compiler Collection (GCC)を参照されたし。
* -g
    * デバッグオプション。
    * gdbでのデバッグを可能にする。
* -MMD -MP
    * ソースファイルの依存関係を中間ファイルに出力する。
    * 依存関係ファイルはソースファイル名の拡張子を.dに変更したものとなり、OBJDIRで指定したディレクトリに生成される。
    * この依存関係ファイルはmakefile最後のinclude文にてインクルードされることで依存しているヘッダファイル等が変更された場合に自動的に再コンパイルされるようになる。

### コマンドライン引数の指定(ARGV)
コマンドライン引数に与える値としてARGVを用いる．初期値は空．

### リンクオプション(LDFLAGS)
(動作未確認)
リンクオプションとしてLDFLAGSの値を用いる。初期値は空。 動的ライブラリをリンクする-lオプションを用いる場合はここに記述する。 パスの通っていない動的ライブラリをリンクするならここにそのファイル名(*.soみたいな)を書いても良い。

### ライブラリの指定(LIBS)
(動作未確認)
静的リンクするライブラリとしてLIBSの値を用いる。初期値は空。 静的ライブラリ（*.a）を用いる場合、空白区切りでファイル名を記述する。 ここで指定したライブラリが更新された場合、makeは再コンパイルが必要だと認識する。

### インクルードパスの指定(INCLUDE)
インクルードパスとしてINCLUDEの値を用いる。初期値は-I./include。 ソースファイル中の#includeファイル検索パスに加えるパスを-Iオプションにて指定する。-Iオプションとディレクトリ名の間に空白を書くことはできない。 複数ディレクトリを指定したい場合は-Iオプションを空白区切りで複数指定する。

### 実行ファイルの指定(TARGET)
実行ファイル名としてTARGETの値を用いる。 初期値は ./bin/exe

### 中間ファイル生成先ディレクトリの指定 (OBJDIR)
中間ファイル生成先ディレクトリとしてOBJDIRの値を用いる。初期値は./obj。 このフォルダにオブジェクトファイル(*.o)や依存関係ファイル(*.d)が生成される。

### ソースファイルの指定 (SOURCES)
コンパイル対象となるソースファイルとしてSOURCESの値を用いる。初期値は$(wildcard $(SRCDIR)/*.cpp)。 SRCDIRに存在する拡張子cppのファイル全てをコンパイル対象とすることを意味する。別の拡張子(.cなど)に変更したい場合は、makefile内のcppを全て変更する。

### オブジェクトファイルの指定 (OBJECTS)
オブジェクトファイルとしてOBJECTSの値を用いる。 以下は初期値 $(addprefix $(OBJDIR)/, $(notdir $(SOURCES:.cpp=.o)))の解説。 オブジェクトファイルの生成先ディレクトリはOBJDIR。 オブジェクトファイル名はソースファイル(SOURCES)の拡張子を.oに置換したもの。 OBJDIRが空の場合はmakefileと同一のディレクトリに生成される。

### 依存関係ファイルの指定 (DEPENDS)
依存関係ファイルとしてDEPENDSの値を用いる。 初期値$(OBJECTS:.o=.d)はオブジェクトファイルの拡張子を.dに置換したもの。
