# LocoCore

LocoCoreは、多言語対応アプリケーションの基盤となるクラスを提供するライブラリです。
これを継承して独自のデータロードロジックを実装することで、JSONファイルやデータベースなど任意のデータソースから翻訳データを柔軟に扱うことができます。
実際に使用できる継承クラスを使いたい場合、[LocoJSON](https://github.com/drago-suzuki58/LocoJSON)・[LocoTOML](https://github.com/drago-suzuki58/LocoTOML)を用いてください。

## このライブラリの使い方

`LocoCore`基底クラスを継承し、`_load_translations`メソッドを自身で実装していただくことで、独自のファイルから翻訳データを取得し、柔軟な実装ができるようになります。

ネスト可能な辞書形式(`Dict[str, Dict[str, Any]]`のような形式)で`self.translations[locale]`(`locale`は指定された言語)にそれぞれをロードできれば、理論上はどのようなファイルでも可能と考えます。

## サンプルコード

以下は、拡張ライブラリの継承クラスライブラリである`LocoJSON`のコードです。

```python
import json
import os
from lococore import LocoCore

class LocoJSON(LocoCore):
    def _load_translations(self, locale: str) -> None:
        if locale in self.cache:
            self.translations[locale] = self.cache[locale]
            return

        locale_file = os.path.join(self.locale_dir, f"{locale}.json")
        if os.path.exists(locale_file):
            try:
                with open(locale_file, "r", encoding="utf-8") as f:
                    self.translations[locale] = json.load(f)
                    self.cache[locale] = self.translations[locale]
                self.logger.info(f"Loaded JSON file for locale {locale}")
            except json.JSONDecodeError as e:
                self.logger.error(f"Failed to load JSON file for locale {locale}: {e}")
```

- キャッシュの確認

  ロード済みのデータが`self.cache`にあれば再ロードを避け、直接利用します。

- JSONファイルの読み込み

  指定された言語に対応するファイルを`self.locale_dir`からロードし、ネスト可能な辞書翻訳データとして登録します。

- エラーハンドリング

  ファイル形式が不正な場合や読み込み失敗時にエラーを記録します。

この機能を全て実装する必要はありませんが、このような機能が揃っていればより使いやすくなります。

また、このクラスを実際に使う場合は以下のようになります。

```python
loc = LocoJSON("ja")

python(loc.example1())
```

これを応用すればLocoCoreを使用してとても柔軟な翻訳機能を簡単に実装できます。
