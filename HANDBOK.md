# Garm-handboken 🐺

Familjens egen Linux. Bazzite KDE i grunden, med våra egna inställningar ovanpå.

---

## Del 1 – För alla i familjen

### Första gången på en ny dator
1. Öppna programmenyn och starta **Garm: Kom igång**.
2. Skriv ditt användarnamn och din nyckel (be administratören om den).
3. Svara **Ja** på frågan om automatisk hämtning.
4. Klart! Dina inställningar och appar kommer på plats av sig själva.

> Datorn måste vara ansluten till wifi hemma första gången.

### I vardagen
| Jag vill… | Gör så här |
|---|---|
| spara mina inställningar | Programmenyn → **Garm: Spara mina inställningar** |
| hämta det senaste direkt | Programmenyn → **Garm: Hämta mina inställningar** |
| uppdatera datorn | Ingenting! Den uppdaterar sig själv. Starta om ibland. |

### Om något krånglar
1. **Starta om datorn.** Det löser det mesta.
2. **Fortfarande trasigt efter en uppdatering?** Starta om, och välj den **näst översta** raden i menyn som visas vid start. Då kör du förra versionen, som fungerade.
3. Säg till administratören.

---

## Del 2 – För administratören

### Viktiga adresser
| Vad | Var |
|---|---|
| Garm-receptet (distron) | GitHub → `garm-os` → `recipes/recipe.yml` |
| Byggen | GitHub → `garm-os` → fliken **Actions** |
| Färdig image | `ghcr.io/nullpointernecromancer/garm` |
| Familjens Git (inställningar) | adressen i `files/system/etc/garm/garm.conf` |
| Serveradress för Garm-verktygen | `files/system/etc/garm/garm.conf` i garm-os |

### Installera Garm på en ny dator
1. Ladda ner **Bazzite KDE, Nvidia (GTX 16/RTX)** från bazzite.gg och installera.
2. Byt till Garm:
   ```
   rpm-ostree rebase ostree-unverified-registry:ghcr.io/nullpointernecromancer/garm:latest
   systemctl reboot
   rpm-ostree rebase ostree-image-signed:docker://ghcr.io/nullpointernecromancer/garm:latest
   systemctl reboot
   ```
3. Kör `fastfetch`. Står det GARM i rött är allt rätt.
4. Användaren kör **Garm: Kom igång** (se Del 1).

### Ny familjemedlem i Forgejo
1. Forgejo → **Site Administration** → **User Accounts** → **Create User Account**.
2. Logga in som personen (eller gör det tillsammans) → **Settings** → **Applications** → **Generate new token**.
   Rättigheter: **repository = Read and write**, **user = Read**.
3. Använd namnet och nyckeln i **Garm: Kom igång** på personens dator. Repot `dotfiles` skapas automatiskt.

### Lägga till appar
| För vem | Var |
|---|---|
| **Alla** i familjen | `recipes/recipe.yml` → `default-flatpaks` → `install:` |
| **En person** | Forgejo → personens repo `dotfiles` → `.chezmoidata/flatpaks.yaml` |
| Systemprogram (RPM) | `recipes/recipe.yml` → `dnf` → `packages:` |

App-ID:t hittar du på flathub.org, i adressen till appens sida (till exempel `com.discordapp.Discord`).
Personens app-lista kan redigeras direkt i Forgejos webbgränssnitt. Datorn hämtar ändringen inom ett dygn, eller direkt via **Garm: Hämta mina inställningar**.

### Spara en ny inställningsfil (terminal)
```
chezmoi add ~/.config/sökväg/till/filen
garm-spara
```
Ändringar i filer som redan följer med: bara **Garm: Spara mina inställningar**.

Spara **aldrig**: inloggningar och nycklar (`gh/`, VPN-klientens inställningsmapp, `kdeconnect/`, `~/.git-credentials`), skärminställningar (`kwinoutputconfig.json`) eller panelen (`plasma-org.kde.plasma.desktop-appletsrc`).

### Hålla koll
- **Actions** på GitHub ska ha färska gröna byggen. GitHub pausar schemalagda byggen efter 60 dagar utan aktivitet i repot.
- `rpm-ostree status` visar vilken version en dator kör.
- Loggar från Garm-verktygen: `~/.cache/garm/`

### Ångra en systemuppdatering
- Tillfälligt: välj förra posten i startmenyn.
- Permanent: `rpm-ostree rollback` och starta om.
