# 店舗LP（中継ページ）運用手順

作成：2026/11/18　対象：広告設定担当（田中）

## 1. これは何か

広告のリンク先に挟むページ。動画をもう一度見せて、地図（Googleビジネスプロフィール）か予約を押させ、その行動を計測する。
公開場所は Liberty のサブドメイン `https://go.movliberty.com/` 配下。クライアントごとにフォルダが分かれる。

| クライアント | URL | 状態 |
|---|---|---|
| 肉屋の台所（9店舗） | `https://go.movliberty.com/nikuya/` | 公開中 |

## 2. 広告に設定するリンク先（肉屋の台所）

店舗ごとに `?s=店舗ID` を付ける。付けない場合は位置情報で最寄り店を自動表示。

| 店舗 | リンク先 |
|---|---|
| 新宿東口2号店 | `https://go.movliberty.com/nikuya/?s=shinjuku2` |
| 新宿店 | `https://go.movliberty.com/nikuya/?s=shinjuku` |
| Ginza Premium | `https://go.movliberty.com/nikuya/?s=ginza` |
| 五反田店 | `https://go.movliberty.com/nikuya/?s=gotanda` |
| 渋谷道玄坂店 | `https://go.movliberty.com/nikuya/?s=shibuya-dogenzaka` |
| 渋谷宮益坂店 | `https://go.movliberty.com/nikuya/?s=shibuya-miyamasuzaka` |
| 飯田橋店 | `https://go.movliberty.com/nikuya/?s=iidabashi` |
| 上野店 | `https://go.movliberty.com/nikuya/?s=ueno` |
| 上野公園前店 | `https://go.movliberty.com/nikuya/?s=ueno-koenmae` |

- 言語を固定したい広告セット（英語圏向け等）は末尾に `&lang=en`（`ko` / `zh` も可）。指定なしは端末言語で自動。
- Meta広告のURLパラメータ欄は使わなくてよい（UTMを足す場合は `&utm_source=...` を末尾に追加して問題ない）。

## 3. 広告セットの作り方（決め事）

1. リンク先は上の表のURL。直行（Googleマップ直）と比較したい店舗は、広告セットを2つに分けて同予算にする
2. 最適化イベントは、開始時は「ランディングページビュー」。FindLocation（地図クリック）が週50件を超えたら FindLocation に切り替える
3. クーポン・LINE・フォームはページに載せない（テンプレの固定ルール）

## 4. 計測（ピクセル）

ページには Liberty 共通のMetaピクセル **「Liberty_店舗LP」（ID: 1451261403128793、Liberty株式会社BM所有）** が入っている。全クライアントのLPでこの1本を使い、店舗はイベントのパラメータで区別する。

新しいクライアントの広告アカウントでこのピクセルを使うには：ビジネス設定 → データソース → データセットとピクセル → Liberty_店舗LP → 「リンク済みのアセット」→ 広告アカウントを追加（1クリック）。肉屋の台所（広告アカウントID 1493282545825754）はリンク済み。

送っているイベント：

| 動作 | Meta標準イベント | カスタムイベント |
|---|---|---|
| 地図で開く | FindLocation | map_click |
| 予約する | Schedule | reserve_click |
| 電話する | Contact | phone_click |
| 動画75%視聴 | — | video_75 |
| 店舗切替 | — | store_pick / store_auto |
| 言語切替 | — | lang_ja / lang_en … |

イベントは店舗IDつき（`store=nikuya-gotanda` など）で送られるので、イベントマネージャで店舗別に集計できる。

ピクセルIDの変更・追加は `config.json`（`{"pixel_id":"…","ga4_id":"…"}`）を書き換えるだけ。変更は林さん経由で依頼（GitHubの操作は不要）。

## 5. 店舗・内容を変えたいとき

| やりたいこと | 依頼の仕方 |
|---|---|
| 店舗を追加・住所や営業時間を直す | 店舗名・変更内容を林さんにチャットで送る → 数分で反映 |
| 新しいクライアントのLPを作る | 中継ページ作成テンプレ（店舗名 / GoogleマップURL / 公式サイト / 予約リンク / Instagram / 動画 / 商品写真 / 配色 / 言語）を埋めて送る |
| 先方が自社ドメインで出したい | 先方のドメイン管理者にDNS1行（CNAME）を追加してもらう。依頼文は林さんが用意 |

## 6. 動作確認のしかた

1. スマホで `https://go.movliberty.com/nikuya/?s=gotanda` を開き、店名が「五反田店」になっているか見る
2. 右上の言語を切り替えて英語・韓国語・中国語が出るか見る
3. 「地図で開く」を押してGoogleマップの該当店が開くか見る
4. Meta イベントマネージャ → 該当ピクセル → テストイベントで、FindLocation が届いているか見る（ピクセル設定後）

## 7. 初回だけだったもの（次のクライアントでは不要）

GitHubの接続・公開リポジトリ作成・GitHub Pages有効化・go.movliberty.com のDNS・HTTPS・Metaピクセル作成・ドメイン認証は、2026/11/17〜18に一度だけ行った基盤づくり。次のクライアントからは「LPのフォルダを置く → ピクセルを広告アカウントにリンク → 広告のリンク先に入れる」の3つだけ。所要は配置数分＋リンク1クリック。

## 8. 触らないもの

- GitHub（hayashi-Liberty/liberty-pages）の設定・DNS（Squarespace）の設定。壊れると全クライアントのLPが止まる
- 変更はすべて林さん経由で依頼する
