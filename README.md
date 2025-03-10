# LocoCore

LocoCore is a library that provides a base class for multilingual applications.
By inheriting this class and implementing your own data loading logic, you can flexibly handle translation data from any data source such as JSON files or databases.
If you want to use an already implemented subclass, please use [LocoJSON](https://github.com/drago-suzuki58/LocoJSON) or [LocoTOML](https://github.com/drago-suzuki58/LocoTOML).

Other Language LEADME (GitHub)
[Japanese](https://github.com/drago-suzuki58/LocoCore/blob/main/README.ja.md)

## How to use this library

By inheriting the `LocoCore` base class and implementing the `_load_translations` method yourself, you can obtain translation data from your own files and achieve flexible implementation.

As long as you can load each translation into `self.translations[locale]` (where `locale` is the specified language) in a nested dictionary format (such as `Dict[str, Dict[str, Any]]`), theoretically any file format is possible.

## Sample code

Below is the code for the `LocoJSON` subclass library, which is an extension library.

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

- Checking the Cache

  If the data is already loaded in self.cache, it avoids reloading and uses it directly.

- Loading JSON Files

  It loads the file corresponding to the specified language from self.locale_dir and registers it as nested dictionary translation data.

- Error Handling

  It logs errors if the file format is invalid or if loading fails.

You do not need to implement all of these features, but having these features will make it more user-friendly.

Also, to actually use this class, you can do the following:

```python
loc = LocoJSON("ja")

python(loc.example1())
```

By applying this, you can easily implement very flexible translation functionality using LocoCore.
