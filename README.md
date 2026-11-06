# test

## GitHub Strings test cases

Each folder is one case: bind one Key Value List (source language English) to it, and set the binding as below. Export rule placeholders: `{lang_code}` (the language's repository code), `{file_name}` and `{file_stem}` (the source file's name, with and without its extension). The existing `locales/` folder belongs to the first test list; it is untouched.

| Folder | Format | Source paths | Export rule (source → target) | Locale codes to set | What to check |
|---|---|---|---|---|---|
| `react-i18next/` | i18next JSON, nested, `{{name}}`, `_one`/`_other` plurals, 2 namespaces | `react-i18next/public/locales/en/*.json` | `react-i18next/public/locales/en/*.json` → `react-i18next/public/locales/{lang_code}/{file_name}` | de-DE → `de`, es-ES → `es` | Nested keys (`nav.home`), plurals become one ICU key, existing `de/common.json` imports as Unreviewed |
| `react-intl/` | react-intl flat ids, ICU plural and select | `react-intl/src/lang/en.json` | `react-intl/src/lang/en.json` → `react-intl/src/lang/{lang_code}.json` | fr-FR → `fr` | Flat ids stay flat on export; existing `fr.json` imports |
| `formatjs/` | formatjs messages (`defaultMessage` + `description`) | `formatjs/lang/en.json` | `formatjs/lang/en.json` → `formatjs/lang/{lang_code}.json` | — | `description` becomes the key's context; `notFormatjs` imports as dotted keys |
| `gettext/` | PO source (`msgstr` = English), `msgctxt`, plural, `#.` comments, obsolete entry | `gettext/locale/en.po` | `gettext/locale/en.po` → `gettext/locale/{lang_code}.po` | de-DE → `de`, pt-BR → `pt_BR` | `menu\|Open` and `status\|Open` are two keys; obsolete entry skipped; existing `de.po` imports |
| `gettext-pot/` | POT template as source (edge) | `gettext-pot/locale/messages.pot` | `gettext-pot/locale/messages.pot` → `gettext-pot/locale/{lang_code}.po` | — | A template export empties every `msgstr`: expect empty translations (worth a question to Kamal) |
| `flutter-arb/` | Flutter ARB with `@name` descriptions, placeholders, ICU plural | `flutter-arb/lib/l10n/app_en.arb` | `flutter-arb/lib/l10n/app_en.arb` → `flutter-arb/lib/l10n/app_{lang_code}.arb` | es-ES → `es`, pt-BR → `pt_BR` | `@@locale` skipped, descriptions as context, existing `app_es.arb` imports |
| `android/` | Android `strings.xml`: `translatable="false"`, `xliff:g`, quotes, `\'`, `@string` reference, `<b>`, string-array, plurals | `android/app/src/main/res/values/strings.xml` | `android/app/src/main/res/values/strings.xml` → `android/app/src/main/res/values-{lang_code}/strings.xml` | de-DE → `de`, pt-BR → `pt-rBR` | `app_name` not translatable, `alias` is no key, `weekdays[0..2]`, `songs` one ICU key; existing `values-de` imports |
| `ios/` | Apple `.strings` (2 files) with comments as context, `%@`, `\n` | `ios/App/en.lproj/*.strings` | `ios/App/en.lproj/*.strings` → `ios/App/{lang_code}.lproj/{file_name}` | fr-FR → `fr`, pt-BR → `pt-BR` | Comments become context; existing `fr.lproj/Localizable.strings` imports |
| `multi-file/` | 3 JSON sources, targets beside them as `name.<lang>.json` | `multi-file/i18n/source/*.json` | `multi-file/i18n/source/*.json` → `multi-file/i18n/{file_stem}.{lang_code}.json` | — | `{file_stem}`: `auth.de.json`, `errors.de.json`, … |
| `edge-cases/unicode/` | Emoji, RTL, CJK, combining accent, NBSP, a file named `café.json` | `edge-cases/unicode/locales/en/*.json` | `edge-cases/unicode/locales/en/*.json` → `edge-cases/unicode/locales/{lang_code}/{file_name}` | — | Text survives the round trip; the accented file name (NFC) syncs and delivers |
| `edge-cases/dotted-keys/` | `Loading...`, `.env`, `a..b`, a dot inside a nested name | `edge-cases/dotted-keys/en.json` | → `edge-cases/dotted-keys/{lang_code}.json` | — | Dots that are not nesting stay in the key name |
| `edge-cases/lenient-json/` | Comments, single quotes, unquoted names, trailing comma; numbers, booleans, null, arrays | `edge-cases/lenient-json/en.json` | → `edge-cases/lenient-json/{lang_code}.json` | — | Accepted; non-string values are skipped |
| `edge-cases/broken/` | JSON that does not parse | `edge-cases/broken/en.json` | → `edge-cases/broken/{lang_code}.json` | — | Sync now warns (unparseable file); the sync refuses that file and names it |
| `edge-cases/reserved-key/` | Key `one_1` (reserved plural form name) | `edge-cases/reserved-key/en.json` | → `edge-cases/reserved-key/{lang_code}.json` | — | The upload is refused naming the key |
| `edge-cases/empty/` | `{}` | `edge-cases/empty/en.json` | → `edge-cases/empty/{lang_code}.json` | — | No keys; nothing delivered |
| `edge-cases/latin1/` | A Latin-1 (not UTF-8) file | `edge-cases/latin1/en.json` | → `edge-cases/latin1/{lang_code}.json` | — | The connector reports "the file is not UTF-8" for that file |
| `edge-cases/duplicate-keys/` | The same key twice | `edge-cases/duplicate-keys/en.json` | → `edge-cases/duplicate-keys/{lang_code}.json` | — | The last value wins |
| `shared-branch/` | Two small files for two lists | List A: `shared-branch/team-a/en.json`; List B: `shared-branch/team-b/en.json` | A → `shared-branch/team-a/{lang_code}.json`; B → `shared-branch/team-b/{lang_code}.json` | — | Two lists on one repository and branch work when their target paths differ. Then point list B's rule at `shared-branch/team-a/{lang_code}.json`: its sync is refused with `TARGET_PATHS_IN_USE` naming list A |

### Several lists on other branches

Target paths are claimed per repository **and branch**, so the same folder bound on two branches never collides. The branch `strings-alt` is `main` plus one extra key in `react-i18next/public/locales/en/common.json` (`"altBranch"`): bind a second list to `react-i18next/` on `strings-alt` with the same rule, and both lists sync and deliver separately (each opens its own PR, against its own branch).
