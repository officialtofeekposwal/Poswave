# Lyrics translation

## Overview

Poswave can translate the currently displayed lyrics into the language selected
inside the app. The translation keeps the original lyric timestamps, scroll
position, line activation and playback-driven highlighting. A compact button in
the full lyrics view switches between the original and translated versions.

This contribution is intentionally limited to lyrics translation. It does not
change Automix, playback transitions, source replacement or audio processing.

## User flow

1. Open the full lyrics view.
2. Tap the translation button beside the source credit.
3. Poswave resolves the active application locale and requests a translation.
4. If the detected source language matches the app language, the original lyrics
   remain visible and Poswave explains that no translation is needed.
5. Otherwise, the translated text appears with a short particle transition.
6. Tap the same button again to return immediately to the original lyrics.

The original lyrics are always retained in memory and are never overwritten.
While the translated version is selected, the compact current-lyric strip above
the playback scrubber uses that version too. Closing the full lyrics view does
not switch the strip back to the source language.

## Destination language

The destination comes from
`AppCompatDelegate.getApplicationLocales()`, which is the language explicitly
selected in Poswave's language dialog. When the application locale list is
empty, Poswave follows Android's effective configuration locale, matching the
app's normal “follow system” behaviour.

The translation therefore follows the language shown by Poswave, not an
unrelated device default. Language tags are normalized to their base language,
including legacy aliases such as `iw` → `he` and `in` → `id`.

## Translation pipeline

`LyricsTranslation` performs translation on demand:

- Blank instrumental lines are preserved and never sent.
- Section labels and background vocals are translated as independent text slots.
- Text slots are batched up to 3,500 characters using private-use delimiters,
  which preserves the exact mapping between the response and lyric lines.
- At most two short requests run concurrently, avoiding contention with the
  audio stream while keeping long lyrics responsive.
- The source language is automatically detected by the translation response.
- Requests have a 12-second call deadline and are cancelled if the song, lyrics
  or application language changes.
- A partial or malformed response is discarded; original lyrics stay on screen.

The current implementation uses Google's lightweight web translation endpoint
over HTTPS with `sl=auto`. It does not install an on-device translation model or
require an API key.

## Timing and lyric animation

Every translated `LyricLine` retains:

- `timeMs`
- `sungUntilMs`
- background-vocal placement
- instrumental gaps
- the original list position

For word-synced lyrics, Poswave projects the original character progress onto the translated text. The sweep
therefore follows the original holds, pauses and pace changes instead of running
uniformly from start to end. Bloom also uses the source vocal envelope.

## Visual response

Switching versions keeps the existing lyrics list and playback clock mounted.
For 540 ms, small particles drift from actual glyph positions in the active lyric
and its immediate neighbours (up to 18 per voice).

Enabling Poswave's **Reduce animations** preference turns it into an immediate
text swap.

## Storage and performance

No language packs or ML translation models are downloaded.

Translated text uses a two-level cache:

- 12 entries in memory for instant toggling and recently played songs;
- GZIP-compressed JSON files in Android's reclaimable `cacheDir`;
- a hard 2 MiB disk limit, trimmed least-recently-used first;
- SHA-256 keys derived from track, destination language and source lyrics;
- a cache format version so incompatible entries are ignored safely.

## Files

- `app/src/main/java/com/poswal/poswave/data/lyrics/LyricsTranslation.kt`
- `app/src/main/java/com/poswal/poswave/ui/player/NowPlayingScreen.kt`
- `app/src/main/res/values-*/strings.xml`
