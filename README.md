# 颯式
颯式は、「クイックソート種より高速」を目指した、マージソートの改良アルゴリズムです。

以下の特徴があります。  
* 比較ソート
* 安定ソート
* 外部領域：N
* 平均時間：O(N log N)
* 最悪時間：O(N log N)
* 昇順済み：O(N)
* 降順済み：O(2N)
* 再帰：無し

<br>

# 基本となるアルゴリズム
* 外部領域を、2Nの連続帯として見立てます。
* 値を外部領域に置く際、以下のルールを適用します。
  * （最大値 ≦ 値）であれば、昇順列の上側に置き、最大値を更新します。
  * （値 ＜ 最小値）であれば、降順列の下側に置き、最小値を更新します。
  * （最小値 ≦ 値 ＜ 最大値）であれば、新しい値（最大値であり最小値）を昇順列に置き、それまでに並べた値群をPartとします。
* Part群をマージします。

## 具体的な流れ
~~~
外部領域を、2Nの連続帯として見立てます

4 5 1 2 7 6 3 8|入力列
. . . . . . . .|外部領域
→昇順列  降順列←|実際

               |4 5 1 2 7 6 3 8|入力列
. . . . . . . . . . . . . . . .|外部領域
        降順列←｜→昇順列        |2N見立て
~~~
~~~
新しい値（最大値であり最小値）を昇順列に置く
               |. 5 1 2 7 6 3 8
. . . . . . . . 4 . . . . . . .
~~~
~~~
次の値は（最大値 ≦ 値）なので、昇順列の上側に置き、最大値を更新
               |. . 1 2 7 6 3 8
. . . . . . . . 4 5 . . . . . .
~~~
~~~
次の値は（値 ＜ 最小値）なので、降順列の下側に置き、最小値を更新
               |. . . 2 7 6 3 8
. . . . . . . 1 4 5 . . . . . .
~~~
~~~
次の値は（最小値 ≦ 値 ＜ 最大値）なので、新しい値（最大値であり最小値）を昇順列に置く（Part：145が完成）
               |. . . . 7 6 3 8
. . . . . . .|1 4 5|2 . . . . .
~~~
~~~
次の値は（最大値 ≦ 値）なので、昇順列の上側に置き、最大値を更新
               |. . . . . 6 3 8
. . . . . . .|1 4 5|2 7 . . . .
~~~
~~~
次の値は（最小値 ≦ 値 ＜ 最大値）なので、新しい値（最大値であり最小値）を昇順列に置く（Part：27が完成）
               |. . . . . . 3 8
. . . . . . .|1 4 5|2 7|6 . . .
~~~
~~~
次の値は（値 ＜ 最小値）なので、降順列の下側に置き、最小値を更新
               |. . . . . . . 8
. . . . . . 3|1 4 5|2 7|6 . . .
~~~
~~~
次の値は（最大値 ≦ 値）なので、昇順列の上側に置く（Part：368が完成）
               |. . . . . . . .
. . . . . .|3|1 4 5|2 7|6 8|. .
~~~
~~~
最終的な外部領域
4 5|2 7|6 8|. .  昇順列見立て
. . . . . .|3|1  降順列見立て
4 5|2 7|6 8|3|1  実際の内容
~~~
~~~
生成したPart群をマージします  
145 27 368  
12457 368  
12345678  
ソート完了  
~~~

<br>

# 改良
基本アルゴリズムから更に、以下の改良を施します。
* Partの長さを確保する為、インサートソートを行います。
* 再帰をしないように、逐次マージを行います。

<br>

# ビルド＆テスト
検証を行った環境は以下のとおりです。
  * Windows 10 Pro 64bit  
  * Core i7-8700 3.20GHz  

## **Msvc**
Microsoft(R) C/C++ Optimizing Compiler Version 19.15.26732.1 for x64  
Microsoft (R) Incremental Linker Version 14.15.26732.1  
~~~
cl Main.cpp -Ox -EHsc -Fe:TestMsvc.exe  
TestMsvc.exe  
~~~

## **clang++**
clang version 7.0.0 (tags/RELEASE_700/final)  
Target: x86_64-w64-windows-gnu  
~~~
clang++ Main.cpp -O3 -o TestClang++.exe  
TestClang++.exe  
~~~

## **g++**
gcc version 8.2.0 (Rev3, Built by MSYS2 project)  
Target: x86_64-w64-mingw32  
~~~
g++ Main.cpp -O3 -o TestG++.exe  
TestG++.exe  
~~~

<br>

# 乱数ベンチマーク
同じシードから生成したfloat値をソートしました。  
単位は秒で、数値が低いほど高速です。

## **Msvc**
|件数|std::sort|std::stable_sort|颯式|
|-:|-:|-:|-:|
|10,000|0.00083858|*0.00069774*|**0.00060311**|
|1,000,000|0.07069080|*0.06401268*|**0.06192394**|
|100,000,000|8.96780856|*8.58470773*|**8.15964417**|

他のコンパイラと比較しても、std::sortが遅く、std::stable_sortが速い結果となったMsvc。  
初っ端から意外な結果が出ましたが、この特性のお陰で勝つことができました。  

## **clang++**
|件数|std::sort|std::stable_sort|颯式|
|-:|-:|-:|-:|
|10,000|*0.00040838*|0.00044848|**0.00040581**|
|1,000,000|**0.05941431**|0.06861489|*0.06335664*|
|100,000,000|**7.61420540**|9.02416852|*8.35873630*|

今一つぱっとしなかったclang++。  
ソースレベルで最適化を行う必要があるなど、コンパイラの最適化ロジックに疑問が残る結果となりました。  

## **g++**
|件数|std::sort|std::stable_sort|颯式|
|-:|-:|-:|-:|
|10,000|0.00041512|0.00045907|**0.00038881**|
|1,000,000|**0.05990257**|0.06931713|*0.06063945*|
|100,000,000|**7.63235313**|9.07213184|*7.94998035*|

善戦したのがg++。  
「1,000,000」でも僅差となりました。  
「100,000,000」で差が開いたのは、インプレースかアウトプレースかによる、キャッシュ効率の影響と思われます。  

<br>

# 特性ベンチマーク
以下は全て、float値「100,000,000」件でソートしました。  
単位は秒で、数値が低いほど高速です。

## 昇順済み 
||std::sort|std::stable_sort|颯式|
|-:|-:|-:|-:|
|Msvc|**0.21892567**|1.25088036|*0.24510241*|
|clang++|*1.00117798*|1.44316028|**0.25334411**|
|g++|1.33518179|*1.30792267*|**0.22993043**|

## 降順済み
||std::sort|std::stable_sort|颯式|
|-:|-:|-:|-:|
|Msvc|*0.27433638*|1.50812926|**0.26809872**|
|clang++|*0.92900829*|1.64602567|**0.28246292**|
|g++|*1.21591655*|1.55615578|**0.25077546**|

## 同値
||std::sort|std::stable_sort|颯式|
|-:|-:|-:|-:|
|Msvc|**0.06657813**|1.25241026|*0.24396806*|
|clang++|*0.97352844*|1.42485664|**0.24551785**|
|g++|*1.22685844*|1.29003928|**0.22957563**|

<br>

# 余談
如何だったでしょうか？  

std::stable_sortには全勝したものの、std::sortに常勝するのは容易でないことが分かります。  
しかし、環境や運用次第では、std::sortに勝る可能性を秘めているかもしれません。  

マージソート種がクイックソート種に勝てる日は来るのでしょうか？  

ソートアルゴリズムには、まだ浪漫が残っています。  
