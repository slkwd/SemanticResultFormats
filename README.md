# Semantic Result Formats (fork)

Personal fork of [Semantic Result Formats][upstream] (SRF), the MediaWiki extension that provides
additional result formats for [Semantic MediaWiki][smw] queries.

This fork is maintained for use on [The Traditional Tune Archive](https://tunearch.org) (TTA).
Its purpose is a single change: a working `media` result format, without jPlayer
([upstream issue #822](https://github.com/SemanticMediaWiki/SemanticResultFormats/issues/822)).
The change was also proposed upstream as
[pull request #1179](https://github.com/SemanticMediaWiki/SemanticResultFormats/pull/1179).
If it is merged, this fork will no longer be needed.

## Branches

| Branch | SRF base | MediaWiki | Semantic MediaWiki | Use it for |
|---|---|---|---|---|
| [`tta/5.2.0-native-media`](https://github.com/slkwd/SemanticResultFormats/tree/tta/5.2.0-native-media) | 5.2.0 | >= 1.39 | >= 4.2 | Wikis that cannot upgrade. **Installed on the TTA DEV and PROD wikis.** |
| [`media-native-html5`](https://github.com/slkwd/SemanticResultFormats/tree/media-native-html5) | 6.0.2-alpha | >= 1.43 | >= 7.2 | Current versions. Same change as above. Not installed on any wiki yet. |
| `media-native-html5-pr` | 6.0.2-alpha | >= 1.43 | >= 7.2 | Only used for the upstream pull request (the same code, with the original upstream README). |

Both working branches contain the same `media` format. They differ only in the PHP class names
used by `formats/media/MediaPlayer.php` (`Html` and `Title` moved to the `MediaWiki\` namespace
in MediaWiki 1.40) and in the version requirements in `extension.json`.

**Pick the branch by the version of Semantic MediaWiki you run.** For example, MediaWiki 1.43
with SMW 6.0.1 needs `tta/5.2.0-native-media`: the other branches refuse to load because they
require SMW 7.2 or later.

## What changes compared to upstream

The `media` result format used to rely on [jPlayer](https://jplayer.org), which is unmaintained
(last release 2020), uses deprecated jQuery APIs and a Flash fallback, and renders an empty box
on recent MediaWiki versions. In this fork:

* the format renders native HTML5 `<audio>` and `<video>` elements;
* a small dependency-free script (`formats/media/resources/ext.srf.formats.media.js`) adds
  the custom controls (play/pause, stop, seek, volume, repeat, shuffle, full screen) and the
  playlist;
* without JavaScript, the native browser controls and a list of links remain usable;
* the bundled jPlayer, its skins and its ResourceLoader modules and templates are removed;
* more file types are supported (see below) and unplayable or missing files are skipped instead
  of raising PHP warnings;
* the cover art (`poster`) of an audio file is shown as a plain image, because a `<video>`
  element does not display its poster while playing audio;
* a new printout label, `artist`, is shown in brackets next to the title.

All the other result formats are the upstream ones, unchanged.

## Usage

The text after `=` in a printout (`?Property=label`) tells the format what the values mean:

| Label | Meaning |
|---|---|
| `title` | Text shown in the playlist. |
| `artist` | Performer(s). Appended in round brackets to the title. Several values are joined with commas. Any property type is accepted. |
| `poster` | Cover image: a `File:` page or a URL. |
| `source` | Media file: a `File:` page or a URL. |
| *(empty)* | Treated as the title (the behaviour of the jPlayer based format). |

Any other label is read and ignored. If the subject of the query is itself in the `File:`
namespace, it is used as the media source.

### Example

    {{#ask: [[Category:Volume 1]]
     |?Title=title
     |?Has cover art=poster
     |?Has artists=artist
     |?Has audio file=source
     |format=media
    }}

All the results of the query form a single playlist. The player is a `<video>` element if at
least one item is a video, otherwise an `<audio>` element.

### Supported files

| Type | Extensions |
|---|---|
| Audio | `mp3`, `m4a`, `m4p`, `aac`, `oga`, `ogg`, `opus`, `wav`, `flac`, `webma` |
| Video | `mp4`, `m4v`, `ogv`, `webm`, `webmv` |

Whether a file actually plays depends on the browser.

### Parameters

| Parameter | Description |
|---|---|
| `class` | Additional CSS class for the player. |
| `theme` | Kept so that existing queries keep working. It only selects a CSS variant (`blue.monday`, `morning.light`). |
| `inspector` | Kept so that existing queries keep working. It has no effect. |

### Differences that may affect existing queries

* The `theme` and `inspector` parameters no longer have the jPlayer effect (see above).
* The ResourceLoader modules `ext.jquery.jplayer*` and `ext.srf.template.jplayer` no longer
  exist. Custom code or skins that load them must be updated.
* The look of the player is new. A custom stylesheet written for jPlayer will not apply.

## Installation

The fork replaces the whole extension, so remove or rename the existing `SemanticResultFormats`
directory first (keep a backup).

1. In the `extensions` directory of the wiki, clone the branch that fits your versions:

       git clone --branch tta/5.2.0-native-media https://github.com/slkwd/SemanticResultFormats.git
       # or, for Semantic MediaWiki 7.2 and later:
       # git clone --branch media-native-html5 https://github.com/slkwd/SemanticResultFormats.git

2. Keep the loading line you already have in `LocalSettings.php`
   (`wfLoadExtension( 'SemanticResultFormats' );`).
3. If the wiki installs the extension with Composer, make sure Composer does not overwrite the
   directory, or install the PHP dependencies of the extension (`composer install --no-dev`
   inside its directory) if you need formats that use them.
4. Check `"version"` in `extension.json`: `5.2.0` means the `tta` branch, `6.0.2-alpha` means
   `media-native-html5`.
5. Run `php -l formats/media/MediaPlayer.php`, then reload the page.

To update only the media format on a wiki that already runs the matching branch, copy the
changed files (normally `formats/media/MediaPlayer.php` and the files in
`formats/media/resources/`) and reload as described below.

### After installing

* Reload the page with `?action=purge`: the parser cache keeps the old HTML of the query.
* If nothing changes, restart PHP-FPM or Apache to clear the opcache.
* The browser may keep an old copy of the JavaScript and CSS files: reload without cache.

## Tested versions

Tested manually in use on the TTA DEV and PROD wikis:

| | |
|---|---|
| MediaWiki | 1.43.11 |
| Semantic MediaWiki | 6.0.1 |
| Branch | `tta/5.2.0-native-media` |

The `media-native-html5` branch has not been installed on a wiki. Its automated tests (PHPUnit
on MediaWiki 1.43, 1.45 and 1.46, 258 JavaScript tests, Phan, Codecov) pass in the continuous
integration of the upstream pull request.

## Known limits

* PHP is not installed on the maintainer's computer: the PHP code is checked by the continuous
  integration and on the servers, not locally.
* The custom controls need JavaScript. Without it, the native controls of the browser are used.
* The `inspector` parameter and the jPlayer skins are not reproduced.
* The fork is not a replacement for the upstream project and gets no support from it. Report
  problems with the fork [here](https://github.com/slkwd/SemanticResultFormats/issues), not
  upstream.

## Tests

On a machine with PHP, from the extension directory:

    composer phpunit

The JavaScript tests of the media player run with node-qunit (`npm test`).

## Upstream documentation

* [Semantic Result Formats documentation](https://www.semantic-mediawiki.org/wiki/Extension:Semantic_Result_Formats)
* [Release notes](RELEASE-NOTES.md)
* [Upstream repository][upstream]

## Credits

* The Semantic Result Formats extension is the work of its
  [contributors](https://github.com/SemanticMediaWiki/SemanticResultFormats/graphs/contributors).
* The changes in this fork were written with Claude (Anthropic) and tested by Valerio
  Pelliccioni, who maintains the fork for The Traditional Tune Archive.

## License

Published under the [GNU General Public License 2.0 or later](COPYING), together with the licenses
of the bundled third-party plugins. See the upstream project for the list of original contributors.

[upstream]: https://github.com/SemanticMediaWiki/SemanticResultFormats
[smw]: https://github.com/SemanticMediaWiki/SemanticMediaWiki
