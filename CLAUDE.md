# CLAUDE.md – Garm

Den här filen läses automatiskt av Claude Code. Den sammanfattar vad Garm är, vad som är bestämt och varför, och vad som återstår. **Svara användaren på svenska.**

## Om administratören
- Garm byggs och underhålls av en person åt ett hushåll. Administratören är **nybörjare på git/GitHub**: förklara kort vad du gör och varför, och undvik jargong.
- Värderingar: **robust och pålitligt före allt**, inget onödigt ögongodis, lätt att komma igång på en ny dator, och övriga användare ska aldrig behöva en terminal.
- Mer personlig kontext (hårdvara, hemmanätverk, planer) finns inte här med flit, eftersom repot är publikt. Fråga administratören när det behövs.

## Vad Garm är
En egen atomisk Linux-image, byggd med **BlueBuild** ovanpå **Bazzite KDE (Nvidia open)**.
- Recept: `recipes/recipe.yml`. Bas: `ghcr.io/ublue-os/bazzite-nvidia-open:stable`
- Byggs av GitHub Actions (dagligen och vid push) och publiceras som `ghcr.io/nullpointernecromancer/garm` (publik, signerad med cosign).
- Datorer uppdaterar sig själva via Bazzites uppdateringstjänst och kan rullas tillbaka i bootmenyn.
- Rebase-kommandon (GitHub-namnet ska skrivas med **små bokstäver**):
  ```
  rpm-ostree rebase ostree-unverified-registry:ghcr.io/nullpointernecromancer/garm:latest
  rpm-ostree rebase ostree-image-signed:docker://ghcr.io/nullpointernecromancer/garm:latest
  ```

## Beslut (och varför), rör inte utan att fråga
- **Bazzite som bas**, inte ren Fedora eller CachyOS: Nvidia, codecs och spelstack är färdiga, och vi får ett image-upplägg med rollback. Alternativ som utvärderats och valts bort: CachyOS (ingen image eller rollback för flera datorer), AerynOS (alfa), Omarchy (Hyprland och ögongodis), NixOS (brant inlärningskurva).
- **KDE** som standard. Eventuell tiling (Niri eller Sway) blir senare ett tillval för administratören, inte standard.
- **os-release:** bara `NAME` och `PRETTY_NAME` ändras. **`ID` lämnas orört**, eftersom Bazzites verktyg och uppdateringar är beroende av det.
- **Ta inte bort kärnpaket** från Bazzite (drivrutiner, Steam-delar, uppdateringstjänst, KDE-grund). Bara "löv", och testa först. Att ta bort paket krymper inte imagen (lagerprincipen).
- **Namn:** Garm (vakthunden vid Gnipahålan). Repot heter `garm-os`, imagen heter `garm`.
- **Fastfetch-loggan** är ett rött GARM-ordmärke (block-tecken). Försök inte med ASCII-konst av hundar, den har redan underkänts. 😄

## Git-servern för inställningar (inte nåbar från molnet)
- En Forgejo-instans på hemmanätverket. Adressen står i `files/system/etc/garm/garm.conf`. Den nås bara lokalt och ska **inte** exponeras mot internet.
- Nextcloud-klienten finns i Garm, redo för en framtida Nextcloud-server.

## Personliga inställningar: chezmoi + Forgejo
- Varje person har ett privat repo `dotfiles` på Forgejo, och chezmoi applicerar det.
- `.chezmoidata/flatpaks.yaml` innehåller app-listan. `run_onchange_after_install-flatpaks.sh.tmpl` installerar listan (`--user`, Flathub) **bara när `osRelease.name == "Garm"`**.
- Garm-verktygen (`files/system/usr/bin/`):
  - `garm-hem`: kdialog-guide. Frågar efter användarnamn och token, validerar mot Forgejos API, skapar repot om det saknas och fyller det med mallen `/usr/share/garm/dotfiles-mall/`, sparar credentials (`credential.<url>.helper store`), kör `chezmoi init` och `apply --force` och erbjuder daglig synk.
  - `garm-spara`: `chezmoi re-add`, därefter git add/commit/push.
  - `garm-hamta [--auto]`: `chezmoi update --force`. Tyst och avbryter om servern inte nås i `--auto`-läge, varnar vid osparade ändringar i manuellt läge.
  - systemd user-timer `garm-synk.timer` (dagligen) och menyposter i `usr/share/applications/`.
  - Serveradressen ligger i `/etc/garm/garm.conf`.
- Skripten är testade med mockade verktyg, men **inte på riktig hårdvara ännu**.

## Viktiga fällor
- **Körbar-biten försvinner** vid uppladdning via GitHubs webb. Därför gör `script`-modulen `chmod 0755` på garm-verktygen. Nya skript i `usr/bin` måste läggas till där.
- **Dependabot-PR:ar blir röda**, eftersom de inte får repots hemligheter (signeringsnyckeln). Det är normalt: merga, så verifierar bygget på `main`.
- GitHub **pausar schemalagda byggen efter 60 dagar** utan aktivitet i repot.
- Äldre Nvidia (GTX 900/1000) kräver en `-nvidia` (legacy) variant, och AMD/Intel-datorer kräver en egen variant. Det finns inte ännu.
- Spara aldrig hemligheter (tokens, `~/.git-credentials`, `gh/`, VPN-klientens inställningsmapp, `kdeconnect/`) i dotfiles.

## Status (2026-09-25)
- ✅ Garm byggs grönt, och första datorn är rebasad och fungerar.
- ✅ Forgejo är igång, och det första `dotfiles`-repot håller på att sättas upp.
- ⏳ Garm-verktygen är uppladdade men inte testade på riktig hårdvara. Nästa steg är ett grönt bygge och sedan ett test av `Garm: Kom igång` med ett testkonto.

## Att göra (ungefär i den här ordningen)
1. Verifiera garm-verktygen på riktigt och fixa det som strular.
2. Gemensamma KDE-kortkommandon via `/etc/xdg/kglobalshortcutsrc`.
3. Gå igenom Bazzites paket och ta bort onödiga "löv".
4. Varianter: AMD/Intel, eventuellt `garm-lite` för svaga datorer (bas: Universal Blues mindre KDE-image).
5. Installations-ISO för Garm (BlueBuild kan generera en).

## Arbetssätt
- Gör ändringar på en gren och öppna en PR med en kort svensk beskrivning av vad som ändrats och varför. Administratören mergar.
- **Repot är publikt:** skriv aldrig in namn, hårdvara, nätverksdetaljer utöver `garm.conf`, eller annat personligt i filer eller commit-meddelanden.
- Håll receptet kommenterat på svenska. Kommentarerna är administratörens dokumentation.
- Uppdatera `HANDBOK.md` när något ändras som familjen eller administratören behöver veta.
- Validera YAML och kör `bash -n` på skript innan commit.
