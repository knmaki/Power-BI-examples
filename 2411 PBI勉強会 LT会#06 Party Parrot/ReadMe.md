# 1. はじめに
## 1.1. これは何？
Power BI LT 会 6で発表した内容。
Party ParrotをPower BIのレポートで動かす。

https://powerbi.connpass.com/event/336027/

↓をぐりぐり動かしたい。
![alt text](https://github.com/user-attachments/assets/be2414ce-1ddf-4a41-8fc9-2cbb3def5c34)

## 1.2. Power BI 勉強会とは
楽しいところです。

https://powerbi.connpass.com/

## 1.3. サンプルファイルの開き方
サンプルファイルは `party parrots.pbip` 。

プレビュー段階のPower BI Desktop プロジェクト（ `.pbip` ファイル）を使用しているので、プレビュー機能を有効にする必要あり。

https://learn.microsoft.com/ja-jp/power-bi/developer/projects/projects-overview


# 2. GIFの動かし方
以下を参考。

https://www.ehansalytics.com/blog/2021/10/27/add-an-animated-gif-to-your-power-bi-reports


## 2.1. ダメなケース（動かない）
`挿入`タブ -> `イメージ`ボタン -> ファイル選択

動かない。かなC。

![alt text](https://github.com/user-attachments/assets/48e25dbf-90a8-4b90-81c3-9fb54b0993f7)

## 2.2. OKなケース
### キャンバス全体
キャンバスを選択 -> 書式設定 -> `キャンバスの背景` -> `イメージ `の `参照...`からファイル選択 -> `透過性`=0%

![alt text](https://github.com/user-attachments/assets/4fb559e1-cdf7-44e1-8ac4-e216ae67233c)


### ボタン

`挿入`タブ -> `ボタン`ボタン -> `空白` -> 書式設定 -> `ボタンのスタイル` -> `フィル`のトグルをオン -> `参照...`からファイル選択 -> `透過性`=0%

![alt text](https://github.com/user-attachments/assets/6c324d1d-90ce-4056-bdd3-9171bda0ea51)

### たくさん動かしてみた
戦いは数だよ兄貴！！

![alt text](https://github.com/user-attachments/assets/691a66bc-21a9-4829-807a-76ae1ac55b7f)

リポジトリから.pbixファイルをダウンロードするともっとやばいやつがあるよ💖💖

# 3. たくさん動かすとどうなるか？？
大量にParrot君を入れるといろいろ大変だった。

- ページが無茶苦茶重くなる
  - CPU、メモリー、GPUの使用量が増える
  - 重くなるのはそのページを開いたとき。つまり、ページを開くまでメモリーにロードされない。
  - GIFを入れていないページに戻ると、CPU、GPUの使用量は減るが、メモリーは増えたまま
  - キャッシュとして置いているということ？
- レポートのサイズは大きくならない。pbixファイルに取り込まれる画像自体は一つのみであるため。（pbipファイルで確認）
- 無茶苦茶重いので、整列させるためにはpbip形式内のjsonファイルをPythonで直接編集しなければならなかった
- ビジュアルを大量に含むレポートはサービスに発行できない
- （関係ないけど）Windows Game BarでPower BI Desktop（ストアアプリ版）の画面録画ができない
