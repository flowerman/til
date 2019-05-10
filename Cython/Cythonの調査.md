## Cythonの調査

### Cythonとは？
Pythonを C/C++ に変換することにより高速化しようというのが主な狙い。  
ネイティブコードにコンパイルする。  
高速化の恩恵を受けるためには、ポイントを踏まえる必要がある。  
したがってコードをロード／コンパイルする時間は必要。
<br>

### では、そのポイントとは？
出来るだけボトルネックに特化する。（のが良いと思われる。）  
Pythonで言えばループにあたるが、Numpyで充足されるのであればそのほうがよい。  
可読性や、パイソニックが犠牲になる場合もあるので。
<br>

### 手軽に試すには ... Jupyter　ただし、これは Linux, macOSの場合
しかし、検討としてよい題材なので、ここでポテンシャルを確認しておいて、環境構築に手間取ることが予想されるWindowsへ行ってみる方針。
<br>

### 以下は、Jupyter Notebookでの確認
パッケージ導入：  
conda install cython  
バージョン：  
cython-0.29.7  
注意：  
マジックコマンドの直前にコメントを書くとハマる ...  
condaでのインストール方法は、Anaconda を想定したもの。  
Python3系や、仮想ディレクトリで Python を管理しているのであれば pip install cython で良い。  
また、書籍を出典とする記事で %load_ext cythonmagic でロードしているものがあるが、これはもう古いスタイルなので注意。%load_ext Cython で良い。  
<br>

Cythonロード

``` Jpyter Notebook
%load_ext Cython
```

pure Pythonで記載したコード

``` Jpyter Notebook
def pure_python(n):
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a
```

Cythonで記載したコード

``` Jpyter Notebook
%%cython
 
def cython(n):
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a
```

引数に型アノテーションを付与したCythonコード

``` Jpyter Notebook
%%cython
 
def cython_typed_anotation(int n):
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a  
```

関数内の変数すべての型アノテーションを付与した Cythonコード

``` Jpyter Notebook
%%cython
 
def cython_alltyped_anotetion(int n):
    cdef int a,b,i
    
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a  
```

実行速度を比較してみる

``` Jpyter Notebook
%timeit pure_python(5000)
 
%timeit cython(5000)
 
%timeit cython_typed_anotation(5000) 
 
%timeit cython_alltyped_anotetion(5000)

519 µs ± 5.93 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
430 µs ± 12 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
366 µs ± 14.3 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
2.87 µs ± 70.1 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
```
<br>

#### 所見：  
いくらなんでもやりすぎだろう ... という点で、最も劇的な効果が出ている。 
何度も呼び出されるような関数であれば、引数にアノテーション付与して型情報をサポートするだけでも、それなりの効果は上がると思われる。
<br>

#### その他／展望：
Windows上での確認は今後の課題。  
C#からの呼び出しに関しては、C# dynamic型からの pythonコードの呼び出しは可能（Iron Pythonではなくても）なので、そのへんからのアプローチをしてみる予定。
<br>

#### 参考：
[公式ドキュメント（和訳）](http://omake.accense.com/static/doc-ja/cython/index.html)  
[これは、本質をついてると思われる記事](https://qiita.com/pashango2/items/45cb85390193d97523ca)
<br>

更新履歴：  
2019.05.08 作成
