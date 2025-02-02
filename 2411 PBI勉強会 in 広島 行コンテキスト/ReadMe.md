# タイトル: 行コンテキストをのぞく時、行コンテキストもまたこちらをのぞいているのだ

# 0. はじめに
## 0.1. これは何？
Power BI 勉強会 in 広島で発表した内容。発表の機会を与えていただいた[Power BI 勉強会](https://powerbi.connpass.com/)に感謝。

https://powerbi.connpass.com/event/329171/

行コンテキストそのもの、また、陥りがちな失敗例とその回避方法についての説明。行コンテキストって計算列だけじゃないのよ。

（雑な絵）
![深淵をのぞく_集中線_レンズフレア](https://github.com/user-attachments/assets/4705c84b-908d-4328-beb9-5f9a39bb01c1)


## 0.2. Power BI 勉強会とは
楽しいところです。

https://powerbi.connpass.com/

## 0.3. サンプルファイルの開き方
サンプルファイルは `行コンテキスト.pbip` 。

プレビュー段階のPower BI Desktop プロジェクト（ `.pbip` ファイル）を使用しているので、プレビュー機能を有効にする必要あり。

https://learn.microsoft.com/ja-jp/power-bi/developer/projects/projects-overview

## 0.4. 先にフィルター コンテキストを
行コンテキストについて書くけど、その前にフィルター コンテキストの理解があると良い。イテレーター関数は、それらの組み合わせなので。

https://qiita.com/k_maki/items/f3d9b91a7546b1dd35d5

https://www.docswell.com/s/k_maki/KL1L9Q-2024-08-22-002442

（長い前置きはここまで）



# 1. 行コンテキストの例
ひとまずロジックは後回しで例示から。
## 例1: 計算列
例えば、次のような `売上` テーブルを考える。

|日付|商品ID|単価|数量|
|:-:|:-|-:|-:|
|2024/10/1|A|100|2|
|2024/10/1|A|100|2|
|2024/10/2|B|200|4|

`単価` 列と `数量` 列の値をかけて `売上_計算列` 列を作成しよう。テーブルを右クリックし、「新しい列」をクリック。数式バーに以下を入力する：

```
売上_計算列 = '売上'[単価] * '売上'[数量]
```

この計算列を定義すると、直ちに `売上` テーブルに `売上_計算列` 列が追加される。

|日付|商品ID|単価|数量|売上_計算列|
|:-:|:-|-:|-:|-:|
|2024/10/1|A|100|2|200|
|2024/10/1|A|100|2|200|
|2024/10/2|B|200|4|800|


## 例2: イテレーター関数
こっちは計算列じゃなくてメジャーね。テーブルを右クリックし、「新しいメジャー」をクリック。数式バーに以下を入力する：

```
売上_SUMX = SUMX('売上', '売上'[単価] * '売上'[数量])
```

メジャーなので、テーブルに列が追加されることはない。このメジャーをビジュアルで用いれば、売上（単価×数量）の合計が集計される。

例えば、テーブル ビジュアルで使ってみると次のようになる。

|商品ID|売上_SUMX|
|:-|-:|
|A|400|
|B|800|

## 例3: CALCULATEのブール式
あまり意識されていないかもしれないが、CALCULATE関数のフィルターとして使うブール式も行コンテキストで評価されている。
（これもイテレーター関数）

```DAX:売上_商品A
売上_商品A = CALCULATE(
    SUM([売上_計算列]),
    '売上'[商品ID] = "A"
)
```

後で説明。

# 2. メジャーと計算列の違い
## 2.1. メジャーと計算列
この記事はフィルター コンテキストを理解していることを前提としているので、よくあるようなメジャーと計算列の違いについては細かく説明しない。そうは言っても過去記事で計算列について説明していないので、ざくっと説明すれば以下。

**メジャー** : ビジュアルごとに計算される集計値の定義式

**計算列** : セマンティック モデルの更新時に計算されてテーブルに格納される値の定義式

個人的な考え
計算列はETLの延長なので、Power Queryでやれば事足りる場合が多く、使わなければならないケースは限られる（Power Queryが不得意な処理とか）。とっつきやすいから計算列というのをよく見かけるが、大人しくPower Queryを覚えた方が幸せになれますよと。Power Queryから逃げて計算列に頼りすぎると、計算列/計算テーブルで作った中間テーブル（ビジュアルで使わない）をたくさん作ったりしてメモリーを消費するので。非表示にすれば良いわけじゃないでしょ。

## 2.2. 外形的には: 集計関数の要否

```DAX:売上_商品A
売上 = '売上'[数量] * '売上'[単価]
```

は計算列なら大丈夫だけど、メジャーではエラーとなる。

エラーの内容は以下:

> テーブル '売上' の列 '数量' に対しては1つの値を特定できません。これは、1つの結果を取得するためにmin、max、count、sumなどの集計を指定せずに、メジャー数式が多数の値を含む列を参照している場合に発生する可能性があります。

メジャーは集計された結果を用意する必要があるので、列だけ渡されても、値がいっぱいあるからどう集計したらわからんわ、というのがエラーの内容。なので→


## 2.3. 本質的には: 評価コンテキストが違う
評価コンテキストが何かをひとことで言えば、どの行を計算対象とするかということ（どの列かはDAXで与えられている）。

計算列の場合は、テーブルの各行（一行のみ）のこと。これが **行コンテキスト** 。メジャーの場合は、スライサーやビジュアルによって絞り込まれた行（複数行となりうる）のこと。これが **フィルター コンテキスト** 。


# 3. 先の例を説明

## 例1: 計算列
先の例は、以下のようになっていた：

|日付|商品ID|単価|数量|売上_計算列|
|:-:|:-|-:|-:|-:|
|2024/10/1|A|100|2|200|
|2024/10/1|A|100|2|200|
|2024/10/2|B|200|4|800|

```
売上_計算列 = '売上'[単価] * '売上'[数量]
```

DAX式の`'売上'[単価]`や`'売上'[数量]`は列を表していて、どの行なのかはわからない。だが、計算列なので行コンテキストで評価＝各行で評価されるため、例えば、一行目は`'売上'[単価]=100`、`'売上'[数量]=2`となる。

まあ、計算列の場合は直感的に明らかかも。説明されると逆に混乱するかもしれないが、簡単な例で本質を押さえておかないと、発展的な概念には進めない（数学でよくあるパターン）。

## 例2: イテレーター関数
先の例で`売上_SUMX`メジャーは以下のようになっていた：

```DAX:売上_SUMX
売上_SUMX = SUMX('売上', '売上'[単価] * '売上'[数量])
```

これをテーブル ビジュアルで使ってみると次のようになる。

|商品ID|売上_SUMX|
|:-|-:|
|A|400|
|B|800|

どのように計算されるかを、データポイントごとに考えてみる。

### Step 1. フィルター コンテキストでテーブルがフィルター
例えば、商品ID=Aの行では、フィルター コンテキストで、計算対象の列は次のようになる。

|日付|商品ID|単価|数量|
|:-:|:-|-:|-:|
|2024/10/1|A|100|2|
|2024/10/1|A|100|2|


### Step 2. フィルターされたテーブルの各行をイテレート
このフィルターされたテーブルの各行で、`'売上'[単価] * '売上'[数量]`が計算される。

|日付|商品ID|単価|数量|'売上'[単価] * '売上'[数量]|
|:-:|:-|-:|-:|-:|
|2024/10/1|A|100|2|200|
|2024/10/1|A|100|2|200|

### Step 3.各行の結果を集計
各行の結果を集計する。SUMX関数なので合計。他にはAVERAGEX関数だと平均、MAXX関数だと最大となる。

### まとめ
Step 1～3を図示すると以下：

<img width="594" alt="SUMX関数図解" src="https://github.com/user-attachments/assets/6f9e6475-f88e-4838-966c-61d1216c0d3e">

## 例3: CALCULATEのブール式
CALCULATE関数のフィルターとして使うブール式が、実は行コンテキストで評価されているという話。言われてみればそう思えるが。。

```DAX:売上_商品A
売上_商品A = CALCULATE(
    SUM([売上_計算列]),
    '売上'[商品ID] = "A"
)
```

は、以下の短縮表現（シンタックス シュガー）

```DAX:売上_商品A
売上_商品A = CALCULATE(
    SUM([売上_計算列]),
    FILTER(
        ALL('売上'[商品ID]),
        '売上'[商品ID] = "A"
    )
)
```

このFILTER関数の第二引数が第一引数の行コンテキストで評価されている。

https://qiita.com/PowerBIxyz/items/06a3f1c37b66091a6a34

横道にそれるが、他にもSUM関数はSUMX関数の短縮表記だったりとかいろいろある。

# 4. 混ぜるなキケン⚠行コンテキストでメジャーを使うと⚠
行コンテキストとかメジャーとか意識せずに、勘で使っていると間違えるという話。結論を先に書くと、 **よく分からないうちは、行コンテキストでメジャーを使うべきではない** 。起こりうる問題を具体例で見てみよう。

## その1: CALCULATE関数の代わりにFILTER関数（良くない書き方）
商品Aのみの売上げを計算するには、CALCULATE関数を使って次のようにする。

```DAX:売上_商品A
売上_商品A = CALCULATE(
    SUM('売上'[売上_計算列]),
    '売上'[商品ID]="A"
)
```

<details><summary>（個人的メモ）短縮しないと</summary>

```DAX:売上_商品A_FILTER
売上_商品A_FILTER = CALCULATE(
    SUM('売上'[売上_計算列]),
    FILTER(
        ALL('売上'[商品ID]),
        '売上'[商品ID]="A"
    )
)
```

</details>


ここで、「オイ、フィルター コンテキストとか分らんけん、FILTER関数ば使おう。こいで良かやろ。数字もあっとっとさ。」とかやりたくなる人がいるかもしれない。

```DAX:売上_商品A_誤
売上_商品A_誤 = SUMX(
    FILTER(
        '売上',
        '売上'[商品ID]="A"
    ),
    SUM('売上'[売上_計算列])
)
```

ばってんさ。。。こやんなっとよ↓↓


![その1](https://github.com/user-attachments/assets/83da28d2-d665-48dd-93a3-be674d450084)


商品Bが空欄になるのは置いておくとして、商品Aの値が800となっていて。合計400よりも大きいから明らかに違う！


## その2: CALCULATE関数でFILTER関数＆メジャー（ベスト プラクティスだけど）
次は、メジャーをある程度書ける人でもうっかりやりがちなパターン。

メジャーを書く時のベストプラクティスとして、「フィルター引数として FILTER を使用しない
」がある。可能な限りブール式で記述しましょうというもの。

https://learn.microsoft.com/ja-jp/dax/best-practices/dax-avoid-avoid-filter-as-filter-argument

ただし、上記ドキュメントにもあるとおり、メジャーで評価する場合にはFILTER関数を使わなければならない。実はここに落とし穴がある。

具体例を見よう。

明細単位で、売上が400以上のものを合計するメジャーを考える。 `売上` テーブルは次のようになっていた：

|日付|商品ID|単価|数量|売上_計算列|
|:-:|:-|-:|-:|-:|
|2024/10/1|A|100|2|200|
|2024/10/1|A|100|2|200|
|2024/10/2|B|200|4|800|

ちょっとひねくれて、 `売上_計算列` を使わずにメジャーを書くと以下：

```DAX:売上_400以上
売上_400以上 = CALCULATE(
    SUM('売上'[売上_計算列]),
    FILTER(
        '売上',
        '売上'[単価]*'売上'[数量] >=400
    )
)
```

このメジャーが正解なんだけど、変に気を利かせて `売上_SUMX` メジャーを再利用しちゃうと、間違ったメジャーとなってしまう。

```DAX:売上_400以上_誤
売上_400以上_誤 = CALCULATE(
    SUM('売上'[売上_計算列]),
    FILTER(
        '売上',
        '売上'[売上_SUMX]>=400
    )
)
```

テーブルビジュアルで可視化してみると以下。

![その2](https://github.com/user-attachments/assets/fbf58828-2760-4b56-9bdf-b5ed6092938b)

商品Aの売上合計は400だけど、明細単位では200ずつなので、 `売上_400以上` のところに結果が表示されないのは望んだ結果。しかし、 `売上_400以上_誤` のところには値が出てる。なんじゃこりゃ？？

## 意図しない結果となる条件
意図しない結果となる理由はあるのだが（後述）、まあまあ難しいのでまずは **発生条件** を押さえておきたい。それは、 **行コンテキストでメジャーを使ったから** 。

よくわからないことが発生して、誤った結果とならないように、まずは **行コンテキストが何か、メジャーが何かをちゃんと理解しておくことが重要** 。厄介なのは、この誤った結果というのが毎回起こるわけでもない最初は合っていたのにということにもなりかねない。


大事なので2回書くと、 **行コンテキストとメジャーをちゃんと理解した上で、(よくわからないうちは)、行コンテキストでメジャーを使わないこと** 。理解していないと避けられないでしょ。


# 5. 次のステップ
行コンテキストでメジャーを使用して計算が誤った例を見てきた。

なぜ望んだ結果が得られなかったのか、その理由、すなわち、どのように計算されたのかを考えると、DAXの理解が深まる。その答えは **コンテキスト トランジション** 。仕組みを理解して使いこなせると新しい武器にできる。

…のだけれど、実際に武器として使えて、説明にちょうどよい例が思いついたらまた記事にしたいと思う。

一応、計算が誤った例を理解するだけだったら、↓の記事で説明してるので、ご参考まで。

https://qiita.com/k_maki/items/c24c5258fccb45cd4582

https://qiita.com/k_maki/items/2543156f215bf82847a6

# 参考

https://www.sqlbi.com/articles/context-transition-in-dax-explained-visually/

https://www.sqlbi.com/articles/understanding-context-transition/
