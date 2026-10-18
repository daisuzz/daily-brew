# Daily Brew

ハンドドリップの注湯タイミングを案内するタイマーと、豆と味の記録帳。GitHub Pages で動く PWA。

- 抽出方法: 透過式 4:6 / 浸漬式 / ハイブリッド（比率 1:16）
- 記録はブラウザの localStorage に保存。「記録」タブからJSONで書き出し・読み込みできる
- ホーム画面に追加するとアプリとして起動し、オフラインでも使える

ビルド不要。`index.html` をそのまま配信する。

## データの保存

記録はサーバーには送らず、ブラウザの [localStorage](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage) に JSON 文字列として保存する（`index.html` の `persist()`）。

- キー: `dailybrew.brews.v1`
- 値: 記録の配列（新しい順）。容量の目安は 5MB 程度、1件あたり約 300 バイト
- 起動時に `navigator.storage.persist()` を要求し、ブラウザに消されにくくしている（保証はされない）

### 1件の形式

```json
{
  "id": "m1x9k2ab3cd",
  "at": "2026-10-18T06:00:00.000Z",
  "method": "46",
  "bean": "エチオピア",
  "roast": "中煎り",
  "temp": 92,
  "grind": "中細挽き",
  "beanG": 20,
  "waterG": 320,
  "ratio": 16,
  "taste": "std",
  "strength": 3,
  "rating": 4,
  "memo": "明るい酸味"
}
```

| フィールド | 内容 |
| --- | --- |
| `method` | `46`（透過式 4:6）/ `immersion`（浸漬式）/ `hybrid`（ハイブリッド） |
| `taste` | 4:6 のみ。`sweet` / `std` / `bright` |
| `strength` | 4:6 のみ。後半の注湯回数 `2` / `3` / `4` |
| `steep` | 浸漬式のみ。浸漬時間（秒） |
| `rating` | 0〜5（0 は未評価） |

### バックアップ

「記録」タブの「書き出す」で、全件を次の形の JSON ファイルにして保存・共有できる。「読み込む」で同じファイルを取り込むと、`id` が重複しない記録だけ追加される。

```json
{ "app": "daily-brew", "version": 1, "exportedAt": "2026-10-18T06:00:00.000Z", "brews": [ ... ] }
```

### 注意点

- iPhone ではホーム画面に追加したアプリと Safari で保存場所が別になる
- localStorage はオリジン（`https://daisuzz.github.io`）単位なので、同じドメインの他のページと領域を共有する。キー名で区別している
- ブラウザのサイトデータ削除やアプリの削除で消えるので、定期的に書き出しておく
