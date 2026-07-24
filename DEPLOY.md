# Deploy-Runbook — coachjay.de

**Stand:** 2026-07-25 · **Status:** ✅ LIVE — https://coachjay.de (GitHub Pages, HTTPS erzwungen)
**Muster:** wie `itr.aivantum.com` / `ai-act.aivantum.com` (GitHub Pages + DNS beim Registrar)
**Ein Unterschied, der zählt:** coachjay.de ist eine **Apex-Domain**, keine Subdomain. Apex kann kein CNAME — es braucht **A-Records**. Das ist der einzige Punkt, an dem dieses Runbook von den aivantum-Subdomains abweicht.

Deploy am 25.07. auf Jays Go vollzogen. Repo: `github.com/jaymanuzon/coachjay`. Das Runbook bleibt als Nachvollzug und für den nächsten Deploy stehen.

---

## Vorbereitet (lokal, geprüft)

| Stück | Zustand |
| :-- | :-- |
| `index.html` | Landingpage v2 — dunkle Split-Bühne, randloses Portrait, Vollbreiten-Bänder. 22 KB |
| `impressum.html` · `datenschutz.html` | gegen die **tatsächliche** Funktionalität geschrieben: kein Formular, kein Tracking, kein Cookie, keine KI, Schriften lokal |
| `portrait-jay.jpg` | 1050×1400, 155 KB, aus `IMG_4632.JPG` auf Motiv zugeschnitten |
| `fonts/` | 6 × woff2 (Cormorant Garamond + Hanken Grotesk, latin + latin-ext), 208 KB, SIL OFL 1.1 |
| `CNAME` | `coachjay.de` |
| `.nojekyll` | gesetzt — GitHub Pages liefert die Dateien unverändert aus, kein Jekyll-Durchlauf |
| Favicon | Inline-SVG als Data-URI (keine zusätzliche Datei, keine externe Anfrage) |
| Externe Anfragen | **null** — gemessen: 0 Treffer auf `http(s)://` in allen drei Seiten |
| Render-Gate | `html-render-lint.py` exit 0 (3/3) · PNG-Sicht geprüft |
| Mobile-Gate | `mobile-390-probe.sh` exit 0 · `scrollW=390 bad=[]` auf allen drei Seiten |
| No-JS | Kerninhalt vollständig statisch: **2.119 Zeichen** ohne JavaScript lesbar (die Ursprungsfassung hatte 76) |
| `robots` | **kein** `noindex` mehr — beim Deploy ist das die echte Seite; ein vergessenes `noindex` auf dem Impressum wäre ein stiller Fehler |

## Die Schritte (am 25.07. so gefahren)

### 1. Repo + Push

```bash
gh repo create jaymanuzon/coachjay --public \
  --source="Vantum HQ/Vantum HQ Resources/coachjay-site" --push
```

### 2. GitHub Pages einschalten

Repo → **Settings → Pages** → Source `main` / `/ (root)` → Custom domain `coachjay.de` → speichern.
Danach **Enforce HTTPS** setzen, sobald GitHub das Zertifikat ausgestellt hat (dauert meist wenige Minuten, gelegentlich bis zu einer Stunde).

### 3. DNS — läuft im anderen Fenster, nicht hier

> ⚠️ **Nicht mein Endpunkt.** Am 25.07. bearbeitet eine parallele Session das df.eu-Konto (Weiterleitung + Postfach). Kein Parallelbetrieb an derselben Stelle — die DNS-Werte stehen hier nur zur Vollständigkeit, gesetzt werden sie dort.

Bei **DomainFactory** (`df.eu` → Auftragsverwaltung → Nameserver-Einstellungen → `coachjay.de` → Editieren):

| Typ | Name | Wert |
| :-- | :-- | :-- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| CNAME | `www` | `jaymanuzon.github.io.` |

**Vorher wegräumen:** der aktuelle A-Record zeigt auf `80.67.16.8` (DomainFactory-Server) und liefert eine **302 auf `http://leere.seite`** — diese Weiterleitung muss raus, sonst gewinnt sie gegen Pages.
**MX nicht anfassen** — die steuern das Postfach und haben mit der Website nichts zu tun.

### 4. Abnahme am Endpunkt (nicht am Gefühl)

```bash
dig +short coachjay.de                       # → die vier 185.199.*-Adressen
curl -sSI https://coachjay.de | head -3      # → HTTP 200, kein 302 auf leere.seite
curl -s https://coachjay.de | grep -c "Ruhe im"
bash "00_Resources/mobile-390-probe.sh" <live-kopie>
```

Zusätzlich im Browser: Portrait lädt, Schriften sind Cormorant/Hanken (nicht Georgia), Reveal endet bei voller Deckkraft, Footer-Links auf Impressum und Datenschutz lösen auf.

### 5. Nachziehen

- ~~Fläche ins `flaechen-register.json`~~ — die Datei existiert nicht (mehr), obwohl die Root-CLAUDE.md sie als Quelle führt. Offener Befund.
- Lagezentrum-Faden `coachjay-de-coaching-marke-2026-07-23` nachziehen
- Externer-Endpunkt-Logbuch: was ist am Endpunkt jetzt anders

---

## Offen / bewusst nicht getan

| Punkt | Warum |
| :-- | :-- |
| **Kontaktformular** | Jay will Telefon *und* Formular. Wiederverwendbares Muster liegt: `Productization HQ/…/check-aivantum-worker/`. Braucht Cloudflare-Account + Resend-Key + Worker-Deploy. Bis dahin ist `mailto:` der einzige Weg — auf einer Geld-Seite nur als Zwischenstand vertretbar. |
| **Kontakt-Adresse** | `hallo@coachjay.de` war bei DomainFactory nur kostenpflichtig zu haben. Seit 25.07. steht überall Jays bestehende Adresse **jay@aivantum.com**. |
| **DENIC-Kontaktbestätigung** | Vier df-Mahnungen im Mai 2026 („Aktion erforderlich"). DENIC zeigt aktuell `Status: connect`, also nicht gesperrt — trotzdem prüfen. Der Bestätigungslink ist Jays Identitätsbestätigung, nicht meine. |
| **Anwaltliche Abnahme** | Impressum und Datenschutz sind Haus-Entwurfsstand mit Draft-Banner. |
| **Kein `www`-Redirect erzwungen** | GitHub Pages leitet `www` → Apex automatisch um, sobald der `www`-CNAME steht. Nichts zusätzlich nötig. |
