# npa-rta-2019
[警察庁オープンデータ：交通事故統計情報2019](https://www.npa.go.jp/publications/statistics/koutsuu/opendata/index_opendata.html) で公開されている CSV データを Mapbox Vector Tile 形式に変換したデータセットです。

# デモ
データの確認とクリッピングダウンロードを目的としたデモです。

- 地図上のサークルをクリックすることで属性情報ががポップアップ表示されます
- 画面右上の **【Get GeoJSON】** ボタンをクリックすることで現在見えている交通事故情報を GeoJSON FeatureCollection としてダウンロードできます

<https://indigo-lab.github.io/npa-rta-2019/>

[![npa-rta-2019](https://repository-images.githubusercontent.com/386204191/ce32d9b8-6fda-4f1a-a14f-dc080123671e)](https://indigo-lab.github.io/npa-rta-2019/)

# タイル仕様

URL         | <https://indigo-lab.github.io/npa-rta-2019/{z}/{x}/{y}.pbf>
----------- | -----------------------------------------------------------------------------------
データソース  | [警察庁オープンデータ：交通事故統計情報2019,本票・補充票・高速票](https://www.npa.go.jp/publications/statistics/koutsuu/opendata/2019/opendata_2019.html)
ズームレベル  | 0〜14 (※)
作成日時     | 2021年7月16日

※ 0〜14のズームレベルでは,個々の pbf ファイルが 500kbytes 以内に収まるように、近しい POI を集約する処理が行われています



## レイヤー定義

以下のレイヤーを含んでいます。

source-layer | description
------------ | --------------------------------------------
npa-rta-2019 | 交通事故情報 (Point) を収納したレイヤー


## ID 情報

本票・高速票・補充票にはそれぞれ「都道府県コード」「警察署等コード」「本票番号」が含まれており、
これを連結したものがキーとなって、本票・高速票・補充票を関連付けることができます。

また、このキーに対して本票は 1:1、高速票は 0:1、 補充票は 0:n の対応関係にあり、
基本的には本票を主たるデータとして、高速票を最大ひとつ、補充表を複数関係づけることが可能です。

「都道府県コード」「警察署等コード」「本票番号」を連結して数値としてパースしたものを GeoJSON Feature の id として採用しています。


## ジオメトリ情報

ジオメトリ情報は、ソースデータの「本票」に含まれる以下のフィールドの情報を加工して Point として作成しました。

- 地点_緯度(北緯) : ddmmss.sss のような、9桁固定長文字列
- 地点_経度(東経) : dddmmss.sss のような、10桁固定長文字列

ただし、緯度＝経度＝0 であるようなレコードが 70 ほどあり、これらは緯度経度不明として除外しています。
また、ズームレベルに応じた座標値の丸めが行われていることにも注意してください。


## 属性情報

原則としてもともとの CSV のヘッダーフィールドの項目名をそのまま GeoJSON Feature の properties のキーとして採用します。
ただし以下の例外があります。

- 高速票の「事故類型」は「事故類型（高速票）」とする。（本票の「事故類型」と衝突しているため）
- 補充票のプロパティ名には補充票ごとに末尾に `#0` `#1` のような連番を付与する。（補充票は複数存在しうるので、それぞれを区別するため）

なお、以下のプロパティは収録していません。

- 「資料区分」
- 高速票、補充票の「都道府県コード」「警察署等コード」「本票番号」 (本票のものと共通であるため)
- 「地点　緯度（北緯）」「地点　経度（東経）」 (GeoJSON Geometry として保持されるため)
- 値が空白であるようなプロパティ

元データの値は原則として文字列だが、以下のプロパティは整数値として保持するものとする。

- 「死亡者数」
- 「障害者数」

また、以下のプロパティを追加しています。

- 「高速票数」 : このレコードに付与されている、高速票の数。0 または 1。
- 「補充票数」 : このレコードに付与されている、補充票の数。0 以上の整数。
- clustered : この Feature が tippecanoe によってクラスタリングされている場合は true
- point_count : この Feature が tippecanoe によってクラスタリングされている場合のクラスタ数
- point_count_sqrt : point_count の平方根

### プロパティ名の正規化

コード表、CSV のあいだのゆらぎの解消を目的として、すべてのプロパティ名に対して以下の正規化処理を行っています。

- 前後の空白（全角・半角）の除去
- 全角カッコを半角カッコに置換する
- 途中の空白（全角・半角）は連続するものをひとつにまとめた上で _ (アンダースコア)に置換する


## サンプル

以下の MVT としてエンコードされる前の GeoJSON の一例です。

```sample.json
{
  "geometry": {
    "type": "Point",
    "coordinates": [
      139.7230052947998,
      35.66107949916439
    ]
  },
  "type": "Feature",
  "properties": {
    "高速票数": 0,
    "補充票数": 1,
    "都道府県コード": "30",
    "警察署等コード": "111",
    "本票番号": "0047",
    "事故内容": "2",
    "死者数": 0,
    "負傷者数": 2,
    "路線コード": "20120",
    "上下線": "0",
    "地点コード": "0000",
    "市区町村コード": "103",
    "発生日時_年": "2019",
    "発生日時_月": "02",
    "発生日時_日": "16",
    "発生日時_時": "15",
    "発生日時_分": "05",
    "昼夜": "12",
    "天候": "2",
    "地形": "1",
    "路面状態": "1",
    "道路形状": "01",
    "環状交差点の直径": "00",
    "信号機": "1",
    "一時停止規制_標識(当事者A)": "09",
    "一時停止規制_表示(当事者A)": "22",
    "一時停止規制_標識(当事者B)": "09",
    "一時停止規制_表示(当事者B)": "22",
    "車道幅員": "18",
    "道路線形": "9",
    "衝突地点": "20",
    "ゾーン規制": "70",
    "中央分離帯施設等": "4",
    "歩車道区分": "2",
    "事故類型": "21",
    "年齢(当事者A)": "35",
    "年齢(当事者B)": "35",
    "当事者種別(当事者A)": "17",
    "当事者種別(当事者B)": "03",
    "用途別(当事者A)": "01",
    "用途別(当事者B)": "31",
    "車両形状(当事者A)": "11",
    "車両形状(当事者B)": "01",
    "速度規制(指定のみ)(当事者A)": "04",
    "速度規制(指定のみ)(当事者B)": "04",
    "車両の衝突部位(当事者A)": "10",
    "車両の衝突部位(当事者B)": "30",
    "車両の損壊程度(当事者A)": "3",
    "車両の損壊程度(当事者B)": "2",
    "エアバッグの装備(当事者A)": "2",
    "エアバッグの装備(当事者B)": "2",
    "サイドエアバッグの装備(当事者A)": "2",
    "サイドエアバッグの装備(当事者B)": "2",
    "人身損傷程度(当事者A)": "4",
    "人身損傷程度(当事者B)": "2",
    "曜日(発生年月日)": "7",
    "祝日(発生年月日)": "3",
    "補充票番号#0": "001",
    "当事者種別#0": "03",
    "乗車別#0": "2",
    "乗車等の区分#0": "01",
    "エアバッグの装備#0": "2",
    "サイドエアバッグの装備#0": "2",
    "人身損傷程度#0": "2"
  },
  "id": 301110047
}
```

# ライセンス

本データセットは [CC-BY-4.0](LICENSE) で提供されます。
使用の際にはこのレポジトリへのリンクを提示してください。

また、本データセットは [警察庁オープンデータ：交通事故統計情報2019](https://www.npa.go.jp/publications/statistics/koutsuu/opendata/2019/opendata_2019.html) を
加工して作成したものです。
本データセットの使用・加工にあたっては、[警察庁Webサイト|利用規約](https://www.npa.go.jp/rules/index.html) を確認し、権利者の権利を侵害しないように留意してください。

# 技術情報

## 属性拡張

デモページでは、ポップアップ画面およびダウンロードされる GeoJSON Feature に対して属性拡張を行っています。
前出のサンプル GeoJSON は属性拡張によって以下のように属性が追加されます。

```enrichment.json
{
  "geometry": {
    "type": "Point",
    "coordinates": [
      139.7230052947998,
      35.66107949916439
    ]
  },
  "type": "Feature",
  "properties": {
    "高速票数": 0,
    "補充票数": 1,
    "都道府県コード": "30",
    "警察署等コード": "111",
    "本票番号": "0047",
    "事故内容": "2",
    "死者数": 0,
    "負傷者数": 2,
    "路線コード": "20120",
    "上下線": "0",
    "地点コード": "0000",
    "市区町村コード": "103",
    "発生日時_年": "2019",
    "発生日時_月": "02",
    "発生日時_日": "16",
    "発生日時_時": "15",
    "発生日時_分": "05",
    "昼夜": "12",
    "天候": "2",
    "地形": "1",
    "路面状態": "1",
    "道路形状": "01",
    "環状交差点の直径": "00",
    "信号機": "1",
    "一時停止規制_標識(当事者A)": "09",
    "一時停止規制_表示(当事者A)": "22",
    "一時停止規制_標識(当事者B)": "09",
    "一時停止規制_表示(当事者B)": "22",
    "車道幅員": "18",
    "道路線形": "9",
    "衝突地点": "20",
    "ゾーン規制": "70",
    "中央分離帯施設等": "4",
    "歩車道区分": "2",
    "事故類型": "21",
    "年齢(当事者A)": "35",
    "年齢(当事者B)": "35",
    "当事者種別(当事者A)": "17",
    "当事者種別(当事者B)": "03",
    "用途別(当事者A)": "01",
    "用途別(当事者B)": "31",
    "車両形状(当事者A)": "11",
    "車両形状(当事者B)": "01",
    "速度規制(指定のみ)(当事者A)": "04",
    "速度規制(指定のみ)(当事者B)": "04",
    "車両の衝突部位(当事者A)": "10",
    "車両の衝突部位(当事者B)": "30",
    "車両の損壊程度(当事者A)": "3",
    "車両の損壊程度(当事者B)": "2",
    "エアバッグの装備(当事者A)": "2",
    "エアバッグの装備(当事者B)": "2",
    "サイドエアバッグの装備(当事者A)": "2",
    "サイドエアバッグの装備(当事者B)": "2",
    "人身損傷程度(当事者A)": "4",
    "人身損傷程度(当事者B)": "2",
    "曜日(発生年月日)": "7",
    "祝日(発生年月日)": "3",
    "補充票番号#0": "001",
    "当事者種別#0": "03",
    "乗車別#0": "2",
    "乗車等の区分#0": "01",
    "エアバッグの装備#0": "2",
    "サイドエアバッグの装備#0": "2",
    "人身損傷程度#0": "2",
    "都道府県コード@": "東京",
    "警察署等コード@": "東京>麻布",
    "事故内容@": "負傷",
    "路線コード@": "一般都道府県道(2012)",
    "上下線@": "対象外",
    "昼夜@": "昼-昼",
    "天候@": "曇",
    "地形@": "市街地-人口集中",
    "路面状態@": "舗装-乾燥",
    "道路形状@": "交差点-その他",
    "環状交差点の直径@": "環状交差点以外",
    "信号機@": "点灯-3灯式",
    "車道幅員@": "交差点-大(13.0m以上)-中",
    "道路線形@": "直線-平坦",
    "衝突地点@": "その他",
    "ゾーン規制@": "規制なし",
    "中央分離帯施設等@": "中央線-ペイント",
    "歩車道区分@": "区分あり-縁石・ブロック等",
    "事故類型@": "車両相互"
  },
  "id": 301110047
}
```

GeoJSON properties のうち、コード表を参照することでコード値に対応するラベルが得られる場合には、
以下のように `${もともとのプロパティ名}@` のように、末尾に @ を付与したプロパティに対して
ラベル値が設定されます。

```properties.json
{
  "都道府県コード": "30",
  "都道府県コード@": "東京"
}
```

## コードリスト

コードリストの原本は [コード表](https://www.npa.go.jp/publications/statistics/koutsuu/opendata/2019/opendata_2019.html) として PDF ファイルが提供されています。
こちらを JSON-LD に加工したものを <./codebook_2019.json> として収録しています。

## 属性拡張のための JavaScript 関数

デモページで使用している属性拡張のための JavaScript 関数を <./enrichment.bundle.js> として収録しています。

# 更新履歴

## 2021-07-20

- 初版公開
