# CLAUDE.md

Notes for working on this fork of Semantic Result Formats (SRF). The fork is used by
The Traditional Tune Archive (tunearch.org). The maintainer is Valerio Pelliccioni; the code
changes in this fork were written with Claude (Anthropic).

## Repository and branches

* Remotes: `origin` is the **upstream** SemanticMediaWiki repository. Never push there.
  `fork` is `github.com/slkwd/SemanticResultFormats`: push there.
* `tta/5.2.0-native-media`: SRF 5.2.0, requires MediaWiki >= 1.39 and SMW >= 4.2.
  **This is the branch installed on DEV (and later PROD).**
* `media-native-html5`: SRF 6.0.2-alpha, requires MediaWiki >= 1.43 and SMW >= 7.2.
  Same work for current versions. Not installed anywhere yet.
* `master` follows upstream.

Every change to the media format must be made on **both** branches.

## What the fork changes

The `media` result format (`formats/media/`) no longer uses jPlayer (unmaintained, broken on
recent MediaWiki). It renders native HTML5 `<audio>` / `<video>` elements plus a small
dependency-free playlist script (`resources/ext.srf.formats.media.js`).

Printout labels understood by `format=media` (the text after `=` in `?Property=label`):

| Label | Meaning |
|---|---|
| `title` | text shown in the playlist |
| `artist` | performer(s), appended in round brackets to the title; several values are joined with commas |
| `poster` | cover image (File: page or URL) |
| `source` | media file (File: page or URL) |
| empty | treated as the title (legacy behaviour) |

Any other label is read and ignored. If the query subject is itself in the `File:` namespace
it is used as the media source.

Example:

    {{#ask: [[Category:Volume 1]]
     |?Title=title
     |?Has cover art=poster
     |?Has artists=artist
     |format=media
    }}

## The two versions of `MediaPlayer.php`

`formats/media/MediaPlayer.php` is the same on both branches except for three things.
Keep these differences when porting a change:

| | `tta/5.2.0-native-media` | `media-native-html5` |
|---|---|---|
| Html class | `use Html;` | `use MediaWiki\Html\Html;` |
| Title class | `use Title;` (after `use SMWOutputs;`) | `use MediaWiki\Title\Title;` |
| `getParamDefinitions` | no return type | `: array` |

`MediaWiki\Html\Html` and `MediaWiki\Title\Title` do not exist before MediaWiki 1.40, so the
`media-native-html5` file will not load on the tta (1.39) installation.

## Working conventions

* Rewrite files in full; do not leave partial patches.
* README, code and commit messages are in English.
* Commits end with the line `Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>`.
* PHP is **not installed on the maintainer's Mac**, so `php -l` and the tests cannot be run
  locally. Run `php -l` on the server before reloading the wiki, or `composer phpunit` where
  PHP is available.
* The folder `SemanticResultFormats-5.2.0-media` (next to this repository, in `/opt/NewSRF`) is
  **not a git repository**. It is a copy of the `tta/5.2.0-native-media` tree, used to copy files
  to DEV. Keep it aligned by hand after changing that branch.

## Testing on DEV

1. Copy the changed file from the tta branch (or from the folder above) to the extension
   directory on the server. Do not copy files from `media-native-html5` to DEV.
2. Reload the page with `?action=purge`: the parser cache keeps the old HTML of the query.
3. If nothing changes, restart php-fpm / Apache to clear opcache.
4. Check `"version"` in the server's `extension.json`: `5.2.0` means the tta branch,
   `6.0.2-alpha` means `media-native-html5`.

## Open items

* Write the full README (differences from upstream, installation on DEV and PROD, usage
  examples, known limits, tested versions, credits) once the DEV and PROD tests are finished.
  The current README is a placeholder.
