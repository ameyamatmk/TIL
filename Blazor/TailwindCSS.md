# TailwindCSS

- [Blazor WASMでTailwind CSSを使う #C# - Qiita](https://qiita.com/k-yamamoto/items/a78ab6bea37414cd3308)
- [2024: Blazor のウェブアプリで tailwind.css を利用 (ASP.NET Core / .Net 8.0 対象） #tailwindcss - Qiita](https://qiita.com/Meister619/items/b33b8570ee29aeb4a84a)

## TailwindCSS とは

- ユーティリティ不ファーストアプローチ
  - 一つの特定のスタイルプロパティに対応するユーティリティクラスが多数定義されている
- HTMLマークアップ内で直接スタイルを適用できるため、開発速度の向上が見込める
- `tailwind.config.js` ファイルを通じて、カラーパレット、フォントサイズなどをカスタマイズできる
- 多数のユーティリティクラスを使用するため、HTMLが長くなり可読性が低下する可能性がある
  - 適切にコンポーネントを分割すると保守性も上がってよくなる

[Tailwind CSSとは？初心者でも分かる特徴と使い方ガイド | DeLT WebInsider](https://delt.co.jp/article/510)
[Tailwind CSSは本当にダメなのか](https://zenn.dev/sena21/articles/81290eff275d11)

## BlazorでTailwindCSSを使う

1. Serverレンダーモードでプロジェクトを新規作成
2. `wwwroot/bootstrap/bootstrap.min.css`, `wwwroot/app.css` を削除する。
3. プロジェクトファイルのディレクトリで `npx tailwindcss init` を実行する。 `tailwind.config.js` ファイルがプロジェクトに追加される。
4. `tailwind.config.js` を編集し、`content: ['./**/*.{razor,html}'],` の設定を追加する。
5. `Styles` フォルダを作成し、新規アイテムとして `ASPNET.Core ＞ Web ＞ Content ＞ Style Sheet` を選択して `Styles/app.css` を作成する。
6. `app.css` を初期化する。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

7. `npx tailwindcss -i .\Styles\app.css -o .\wwwroot\css\app.css --watch` でホットリロード込でcssファイルを生成する。
8. App.razor を編集し、 `<link rel="stylesheet" href="css/app.css" />` としてtailwindcssで生成したcssファイルを参照する。

コマンドを毎回打たずに済むようにしたい
