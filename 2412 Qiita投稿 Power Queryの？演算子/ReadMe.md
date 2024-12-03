

# 1. はじめに
## 1.1. きっかけ
ヘルパー クエリが出るやつ（例えば[こちら](https://qiita.com/k_maki/items/cab7a340ffc717ef1a4d#21-%E3%83%98%E3%83%AB%E3%83%91%E3%83%BC-%E3%82%AF%E3%82%A8%E3%83%AA%E5%87%BA%E7%8F%BE)）をやっていると、数式内に `?` が出てた。

![？が出た.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/239172/9090e5af-0aa8-bedd-5bc6-92c13e582d11.png)

あれ？ `?` って何だっけと思ったのが調べたきっかけ。


## 1.2. Microsoft Learnを読もう
当たり前だけど、MS Learnにちゃんと書いてあった。

https://learn.microsoft.com/ja-jp/powerquery-m/m-spec-operators


その他にも参考になった記事など：

https://gorilla.bi/power-query/coalesce/

https://bengribaudo.com/blog/2020/01/06/4844/power-query-m-primer-part14-control-structure


疑問に思ったら勉強会のネタにしよう（みんなで勉強すると楽しいよ）：

https://powerbi.connpass.com/


## 1.3. サンプル ファイルについて
いつもどおりこちらのリポジトリにアップ。 `.pbip` ファイルを使うので、プレビュー機能をオンにする（参考→[MS Learn](https://learn.microsoft.com/ja-jp/power-bi/developer/projects/projects-overview)）。

サンプル ファイルのセマンティック モデルは、ほぼ空。Power Queryエディターの方にサンプル クエリがあるのでそっちを見てね。各ステップを追えば、結果を確認できる。

![サンプル クエリ一覧.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/239172/99194a49-b44f-abf6-9961-c679d7959df6.png)


# 2. ?演算子（項目およびフィールドへのアクセス）
`?` で項目およびフィールドへのアクセスを行うわけではないけど、組み合わせると動作が変わるよと。こういう風にしたい人もいるよねということで追加されている機能。

## 2.1. 項目アクセス
https://learn.microsoft.com/ja-jp/powerquery-m/m-spec-operators#item-access

ListやTableの要素にアクセスする。

### 2.1.1. ?なし
存在しない要素にアクセスしようとすると、エラーとなる。

```PowerQuery:?なし
let
	#"💖" = "",
	
	// Listの範囲内の要素
	#"case1 List" = {"a","b","c"}, 
	// 0番目の要素にアクセス -> a
	#"case1 結果" = {"a","b","c"}{0},

	// Listの範囲内の要素
	#"case2 List" = {1, [A=2], 3} ,
	// 1番目の要素にアクセス -> [A=2]
	#"case2 結果" = {1, [A=2], 3}{1} ,

	// Listの範囲外の要素
	#"case3 List" =  {true, false} ,            
	// 2番目の要素（リストの範囲外）にアクセス -> error
	#"case3 結果⚠" =  {true, false}{2} 

	// Tableに存在するRecord
	#"case4 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// Tableの0行目にアクセス -> [A=0,B=1]
	#"case4 結果" =  #table({"A","B"},{{0,1},{2,1}}){0},

	// Tableに存在するRecord
	#"case5 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// TableのうちA=2のものにアクセス -> [A=2,B=1] 
	#"case5 結果" =  #table({"A","B"},{{0,1},{2,1}}){[A=2]},

	// Tablに存在しないRecord
	#"case6 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// TableのうちB=3のもの（存在しない）にアクセス -> error 
	#"case6 結果⚠" =  #table({"A","B"},{{0,1},{2,1}}){[B=3]},

	// Tableに複数存在するRecord
	#"case7 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// TableのうちB=1のもの（2件存在）にアクセス -> error 
	#"case7 結果⚠" =  #table({"A","B"},{{0,1},{2,1}}){[B=1]},

	ret = "各ステップを見てね"
in
	ret
```

### 2.1.2. ?あり
存在しない要素にアクセスしようとしても、エラーとならず、nullを返す。こっちの方が良いこともあるだろう。

ただし、2つ以上の要素が該当する場合はエラーとなる。

```PowerQuery:?あり
let
	#"💖" = "",
	// Listの範囲内の要素
	#"case1 List" = {"a","b","c"}, 
	// 0番目の要素にアクセス -> a
	#"case1 結果" = {"a","b","c"}{0}?,

	// Listの範囲内の要素
	#"case2 List" = {1, [A=2], 3} ,
	// 1番目の要素にアクセス -> [A=2]
	#"case2 結果" = {1, [A=2], 3}{1}? ,

	// Listの範囲外の要素
	#"case3 List" =  {true, false} ,            
	// 2番目の要素（リストの範囲外）にアクセス -> null
	#"case3 結果∅" =  {true, false}{2}?,

	// Tableに存在するRecord
	#"case4 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// Tableの0行目にアクセス -> [A=0,B=1]
	#"case4 結果" =  #table({"A","B"},{{0,1},{2,1}}){0}?,
		
	// Tableに存在するRecord
	#"case5 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// TableのうちA=2のものにアクセス -> [A=2,B=1] 
	#"case5 結果" =  #table({"A","B"},{{0,1},{2,1}}){[A=2]}?,

	// Tablに存在しないRecord
	#"case6 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// TableのうちB=3のもの（存在しない）にアクセス -> null
	#"case6 結果∅" =  #table({"A","B"},{{0,1},{2,1}}){[B=3]}?,

	// Tableに複数存在するRecord
	#"case7 Table" =  #table({"A","B"},{{0,1},{2,1}}),
	// TableのうちB=1のもの（2件存在）にアクセス -> error 
	#"case7 結果⚠" =  #table({"A","B"},{{0,1},{2,1}}){[B=1]}?,

	ret = "各ステップを見てね"
in
	ret
```



## 2.2. フィールド アクセス
https://learn.microsoft.com/ja-jp/powerquery-m/m-spec-operators#field-access


### 2.2.1. 項目取得
特定のフィールドの値を取得する。フィールドが存在しない場合に、 `?` を使えばエラーを回避できるのは、Listの場合と同様。

```PowerQuery:項目取得
let
	#"💖" = "",

	// レコードのフィールドにアクセス
	#"case 1 Record" = [A=1,B=2],
	
	// 存在するフィールド -> 2
	#"case 1-1 結果" = [A=1,B=2][B],

	// 存在しないフィールド（?なし） -> error
	#"case 1-2 結果⚠" = [A=1,B=2][C],

	// 存在しないフィールド（?あり） -> null
	#"case 1-3 結果∅" = [A=1,B=2][C]?,

	ret = "各ステップを見てね"
in
	ret	
```

### 2.2.2. 射影
直前のと似てるけど、複数フィールドからなるRecordのフィールド数を減らすのが射影。前者がRecordに格納された値を取得することに対して、こちらが取得するのはRecordのまま。ただし、フィールド数を減らす。

（数学出身としては射影という言葉はめちゃしっくりくるが、一般的じゃないなぁ）

射影の場合に `?` を利用すると、フィールドがない場合にはnullが格納されたRecordを返す。

```PowerQuery:射影
let
	#"💖" = "",

	// Recordの一部フィールドからRecordを作成
	#"case 1 Record" = [A=1,B=2],        

	// 存在するフィールド -> [B=2]
	#"case 1-1 結果" = [A=1,B=2][[B]],       

	// 存在しないフィールド（?なし） -> error
	#"case 1-2 結果⚠" = [A=1,B=2][[C]], 

	// 存在しないフィールド（?あり） -> [B=2,C=null]
	#"case 1-3 結果" = [A=1,B=2][[B], [C]]?,
	
	ret = "各ステップを見てね"
in
	ret
```

# 3. ??演算子（コアレス演算子）

https://learn.microsoft.com/ja-jp/powerquery-m/m-spec-operators#coalesce-operator

`?` 以外にも、2つ重ねた `??` もある。棚ぼた的発見。

`A ?? B` という式は `if A <> null then A else B` と同じ。

```PowerQuery:Coalesce
let
	#"💖" = "",

	// nullに値を足す -> null
	#"case 1" = null + 5,
	
	// if文を使うと（#"case 1"の値はnull） -> 5
	#"case 2 if文" = (if #"case 1" <> null then #"case 1" else 0) + 5,

	// Coalesce演算子を使うとすっきり -> 5
	#"case 3 Coalesce" = (null ?? 0) + 5,

	// 関数なら別の回避方法も -> 5
	#"case 4 List.Sum" = List.Sum({#"case 1", #"case 3"}),

	// 引数リストの値がすべてnullなら -> null
	#"case 5 List.Sum全部null" = List.Sum({null, null}),

	// 引数リストに0のみのリストを追加する -> 0
	#"case 6 List.Sumに0追加" = List.Sum({null, null} & {0}),

	// Coalesce演算子でも -> 0
	#"case 7 List.SumとCoalesce" = List.Sum({null, null}) ?? 0,

	ret = "各ステップを見てね"
in
	ret	
```

まあ、こんなもんかという気もするが、意外と有用だったり。 `Table.AddColumn` 関数で列を追加する場合などで、nullならこっちの列みたいなことを何回も書いたりするケースでは、かなりすっきりする。

```PowerQuery:??なし
if [列A] <> null then [列A] else if [列B] <> [列B] then [列B] else [列C] 
```


```PowerQuery:??あり
[列]A ?? [列B] ?? [列C]
```

結構違うでしょ？

注意点としては、上記サンプルの `List.Sum` のように、 `??` を使わなくても良いこともあること。ソースDB側が Coalesce演算子に対応（＝SQLのCOALESCE関数に対応）してれば、クエリ フォールディングが効くが、そうでない場合は効かなくなる。 `List.Sum` にゼロのリストを追加するようなトリックでクエリ フォールディングを保てるなら、そっちの方が良い。
（この辺は具体例を知らないので、あまり詳しく書けない）

クエリ フォールディングについては↓を参考に:

https://qiita.com/akihiro_suto/items/15fb8804684e39ec4bb6


# 4. 結論
疑問に思ったら調べることが大事。わからないってことは、そこにスキルアップのチャンスがある！

