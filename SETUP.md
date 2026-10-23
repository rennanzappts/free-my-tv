# Samsung Tizen setup (Windows worker)

Workspace: `C:\Users\renna\Projects\samsungTV-samsung`

| Remote | URL | Role |
|---|---|---|
| `origin` | https://github.com/rennanzappts/free-my-tv | this fork (push here) |
| `upstream` | https://github.com/simbasjungles/free-my-tv | upstream (fetch only; push URL is disabled) |

TV requirement: Samsung 2017 or newer (Tizen 3+), same Wi-Fi as this PC. No rooting. Developer Mode plus TizenBrew or Samsung's own install path only.

Python on this PC: 3.12.4 (`py -3` or `C:\Program Files\Python312\python.exe`). `tizen/build.py` uses the standard library only. `python3` is the Windows Store alias and does not run the script. Java and Tizen Studio are not installed. `TIZEN_STUDIO` is unset.

## a. Install with TizenBrew Installer

TizenBrew on the TV is not the installer. The installer is the PC app.

1. On the TV, turn Developer Mode on (section b) and restart the TV.
2. On this PC, download [TizenBrew Installer](https://github.com/reisxd/TizenBrewInstaller/releases/latest), open it, and connect to the TV.
3. **Install from GitHub:** `simbasjungles/free-my-tv`.

That upstream repo has GitHub Releases. Latest is [v1.1.0](https://github.com/simbasjungles/free-my-tv/releases/tag/v1.1.0) (`Free My TV 1.1: LG and Samsung, more TVs`), asset `FreeMyTV-tizen.wgt`. TizenBrew Installer takes the first `.wgt` on the latest release. That package is the generic signed build: no TMDB key (picks stay off) and the city, currency, and region come from the TV time zone. With the TV set to `America/Sao_Paulo`, that is São Paulo, `BR`, and `BRL`.

On 2023 and newer TVs the installer asks for a Samsung account sign-in once, to sign the app for that TV.

This fork (`rennanzappts/free-my-tv`) has the git tags `v1.0.0` and `v1.1.0` and **no GitHub Releases**, so Install from GitHub on the fork finds no `.wgt`. Use `simbasjungles/free-my-tv` until a release with `FreeMyTV-tizen.wgt` is published on the fork.

Optional, after it is installed: TV settings → Start Screen Options → **Autorun Last App**, so the TV turns on into Free My TV.

## b. Developer Mode (Apps → 12345)

On the TV remote:

1. Open **Apps**.
2. Type `1 2 3 4 5`.
3. Turn **Developer mode** on.
4. Set **Host PC IP** to this computer's address on the TV's Wi-Fi.
5. Restart the TV.

The PC then reaches the TV on port **26101** (SDB). `tizen/install.py` checks that port. Host PC IP must be this PC, not the TV.

## c. Local `.wgt` (built on this worker)

`config.properties` is on this machine only (gitignored). It starts from `config.example.properties` with the project's `America/Sao_Paulo` row from `web/zones.js` (`BR,-23.53,-46.62,São Paulo`):

| Key | Value |
|---|---|
| `tmdb_key` | empty — **TODO:** paste a TMDB API Read Access Token or API Key from https://www.themoviedb.org/settings/api |
| `tmdb_region` | `BR` |
| `latitude` / `longitude` | `-23.53` / `-46.62` |
| `city` | `São Paulo` |
| `currency` | `BRL` (same mapping the generic build uses for country `BR`) |
| `language` | `auto` |
| `watchmode_key` | empty |

Do not commit `config.properties` or put a key in git or chat.

Unsigned package build (succeeded on this worker, 2026-10-23):

```text
py -3 tizen/build.py
Built FreeMyTV-tizen.wgt (FreeMyTVio.FreeMyTV 1.1.0, 713 KB, not signed)
No TMDB key: the picks stay off (everything else works).
```

File: `C:\Users\renna\Projects\samsungTV-samsung\FreeMyTV-tizen.wgt` (730139 bytes). It contains `config.xml`, `icon.png`, and `index.html`. It does **not** contain `author-signature.xml` or `signature1.xml`. `config.js` inside it has `tmdbRegion` `BR`, coordinates `-23.53,-46.62`, city São Paulo, currency `BRL`, and an empty `tmdbKey`.

The TV only installs a signed package. `tizen/install.py` exits if those signature files are missing. Signing was attempted and stopped here:

```text
py -3 tizen/build.py --sign FreeMyTV
Signing needs Samsung's Tizen tools (Tizen Studio, or Samsung's tizen-sdk-skills). Not found: set TIZEN_STUDIO to where they are installed.
```

Tizen Studio was not installed. The TizenBrew path in section a does not need it.

After a certificate profile exists (Tizen Studio Certificate Manager, or Samsung's `tizen` CLI), from this folder:

```text
py -3 tizen/build.py --sign <profile>
py -3 tizen/install.py <TV_IP>
```

`<profile>` is the security profile name. `<TV_IP>` is the TV on the same Wi-Fi, with Developer Mode already on (section b). The install command pushes the `.wgt` over SDB and runs the TV's `vd_appinstall`. It needs only Python. Updates must use the same certificate; a different one is refused until Free My TV is removed on the TV (Apps → Settings → Free My TV → Delete).

`FreeMyTV-tizen.wgt` is gitignored. Rebuild after any `config.properties` change, then sign again before installing.
