# Daily Brew

ハンドドリップの注湯タイミングを案内するタイマーと、豆と味の記録帳。GitHub Pages で動く PWA。

- レシピを選んでからタイマー画面に進む。どのレシピもホット / アイスを切り替えられ、豆の量を変えると湯・氷・各投の量が比例する
- お店・バリスタのレシピ（出典はアプリ内にも表示）
  - PHILOCOFFEA 4:6メソッド（[公式](https://philocoffea.com/pages/how-to-brew)、アイスは [BARISTAI](https://baristai.net/recipes/shops/2)）
  - PHILOCOFFEA ハイブリッドメソッド（[BARISTAI ホット](https://baristai.net/recipes/shops/9) / [アイス](https://baristai.net/recipes/shops/18)）
  - LIGHT UP COFFEE ドリップ（[ホット](https://lightupcoffee.com/en/blogs/brew-guide/drip) / [アイス](https://lightupcoffee.com/en/blogs/brew-guide/iced-coffee)。アイスの氷量は記載がないので目安）
  - GLITCH COFFEE & ROASTERS（[BE-PAL の記事](https://www.bepal.net/archives/257885)。3投目以降の時間は目安）
  - James Hoffmann Ultimate V60（[HARIO USA](https://www.hario-usa.com/blogs/recipes-and-more-from-friends/james-hoffmann-uitimate-v60-technique)。元の半量）
- Daily Brew の定番: 透過式 4:6 / 浸漬式 / ハイブリッド（比率 1:16）
- 公式のアイスレシピがないものは、ホットを急冷式にアレンジする（湯を6割にし、残り4割を氷に）
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
  "recipeId": "philo-46",
  "recipeName": "PHILOCOFFEA 4:6メソッド",
  "bean": "エチオピア",
  "roast": "中煎り",
  "temp": 92,
  "grind": "中細挽き",
  "beanG": 20,
  "waterG": 320,
  "ratio": 16,
  "serve": "hot",
  "rating": 4,
  "memo": "明るい酸味"
}
```

| フィールド | 内容 |
| --- | --- |
| `serve` | `hot` / `iced`（未設定の古い記録は `hot` 扱い） |
| `waterG` | 抽出に使った湯の量（アイスでは氷を含まない） |
| `iceG` | アイスのみ。サーバーに入れる氷の量 |
| `recipeId` | レシピのID（`index.html` の `RECIPES`）。古い記録は `method`（`46` / `immersion` / `hybrid`）で定番レシピに対応づける |
| `recipeName` | 記録時のレシピ名 |
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
