# SonicPlay language packs

Translations live here rather than in the app, so **adding a language is publishing a file** — no
new build, no store release, nothing for existing users to update.

## Files

| File | What it is |
|---|---|
| `index.json` | The catalogue. The app fetches this first to know what exists. |
| `<tag>.json` | One language, keyed by BCP-47 tag (`fr`, `pt-BR`, …). |

## Publishing

1. Host this folder somewhere the app can read over plain HTTPS with no credentials — a **public**
   repo (raw.githubusercontent.com) or a GitHub Release asset. A private repo will not work:
   raw URLs from one require a token, and the app sends none.
2. Set every `url` in `index.json` to the real address of that pack. They are currently
   `REPLACE_WITH_RAW_URL/<tag>.json`.
3. Point the app's catalogue at the hosted `index.json`.

## Adding or updating a language

- **New language** — add `<tag>.json`, add its entry to `index.json`. Nothing else.
- **Improved translation** — edit the pack and **raise its `version`**. The app compares that
  against the installed copy, so a version that does not move means no update is offered.
- Regenerate sizes and hashes after any edit: `python tools/generate_langpacks.py`.

## Pack format

```json
{
  "schemaVersion": 1,
  "tag": "fr",
  "version": 1,
  "strings": { "dac_remove": "Supprimer" }
}
```

Keys are **Android string resource names** — the `name` attribute in `app/src/main/res/values/strings.xml`.
That is what makes packs safe to leave alone as the app grows:

- A key a pack does not translate falls back to the English built into the app. A pack covering
  half the app renders half translated, rather than being rejected for being incomplete.
- A string added to the app in a later release simply appears in English until someone translates
  it. Old packs keep working.

## Index format

```json
{
  "schemaVersion": 1,
  "languages": [{
    "tag": "fr",
    "nativeName": "Français",
    "englishName": "French",
    "version": 1,
    "url": "https://…/fr.json",
    "sha256": "…",
    "sizeBytes": 1136,
    "minAppVersion": 0,
    "completeness": 100
  }]
}
```

- `nativeName` is the **endonym** — the language's own name for itself. It leads the row in the app,
  because that is what its speakers scan for; `englishName` sits underneath for everyone else.
- `sha256` is over the exact bytes of the pack file, so a truncated or tampered download is caught
  before it is installed.
- `minAppVersion` is the lowest app `versionCode` the pack targets. A pack using keys an older build
  does not have is harmless; one written for a newer app is likely missing keys that build needs, so
  the app says "update the app" instead of half-translating itself. `0` means any version.
- `completeness` (0–100) is shown to the user, never enforced. It answers "why is some of this still
  English" before they have to ask.

## Format arguments

A string like `Remove %1$s?` carries a positional argument. A translation may **reorder** them —
that is what positional specifiers are for, and word order differs between languages — but it must
not invent one the original does not supply. The app checks this before displaying a translation
and falls back to English if it does not hold, because the alternative is a crash on a user's
device in a language nobody here reads.
