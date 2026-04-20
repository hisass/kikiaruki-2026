# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

**利き歩きナビ 2026** — 日本酒試飲・蔵元巡りイベント用のモバイルPWAアプリ。ビルドシステムなし。依存パッケージなし。`index.html` 1ファイルに HTML/CSS/JS がすべて収まっている。

## 開発・確認方法

```bash
# ローカルサーバーで動作確認（CORS制約回避のため直接開くより推奨）
python3 -m http.server 8080
# → http://localhost:8080 で確認

# または直接ブラウザで開く
open index.html
```

テスト自動化なし。動作確認はブラウザの手動テストのみ。

## アーキテクチャ

`index.html` のみで完結（約1100行）。構成は以下の順：

1. `<head>` — PWA manifest メタ、CDN（qrcodejs）、CSS変数・スタイル定義
2. `<body>` — モバイルファーストのUI（ヘッダー・タブ・カード・モーダル）
3. `<script>` — アプリロジック全体

### データモデル

`D`（AppData）が LocalStorage に永続化されるメインオブジェクト：

```js
D = {
  usr: { name },             // ユーザー名
  sv: {                      // 店舗ごとの記録
    [storeId]: {
      v,   // 訪問フラグ
      vt,  // チェックイン時刻
      dr,  // 日本酒ドリンク記録 { [sakeId]: { c, r, t } }
      fr,  // フード記録
      m,   // メモ
      wr   // また来たい
    }
  },
  pl: { ids: [], opt: false } // プラン（巡回順・最適化フラグ）
}
```

`S`（静的マスタ）— 蔵元データ配列（約52店舗）。各要素は `{ id, n, b, p, la, lo, s[], f[] }` の形式（n=店名, b=蔵元名, p=都道府県, la/lo=座標, s=酒リスト, f=料理リスト）。

### 主要機能

- 蔵元一覧の検索・絞り込み、チェックイン
- 日本酒・フードの評価（星1〜5）と記録
- ルートプラン作成・Googleマップナビ連携
- データエクスポート（TXT / CSV / JSON / KML / GeoJSON）
- グループ共有：URLパラメータで友人記録をインポート、集計表示
- ジオロケーション（近くの蔵元表示）

## 変更時の注意

- CSS変数（`--gld`, `--pl`, `--ac` など）がテーマカラーとして全体で使われている。個別色のハードコードを避けること。
- `S` 配列の蔵元データを変更する場合、`id` はURLパラメータや LocalStorage キーとして使われるため変更禁止。
- LocalStorage のキー名や `D` の構造を変えると既存ユーザーデータが壊れるため慎重に。
