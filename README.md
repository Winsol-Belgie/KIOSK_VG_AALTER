# VG_Balie_Aalter — Meeting Room Display

Kiosk-weergave voor de vergaderruimte **Balie Aalter (16P)** in Aalter.  
Toont de dagplanning van de ruimte live op een iPad die permanent in de meeting room hangt.

---

## Hoe werkt het?

```
Office 365 (Exchange)
       │
       ▼
Power Automate (elke 10 min)
  → Graph API: GET calendarView van VG_Balie_Aalter@winsol.eu
  → Schrijft JSON naar GitHub Gist
       │
       ▼
GitHub Gist (publieke JSON)
  https://gist.githubusercontent.com/GwennVanthournout/.../raw/events.json
       │
       ▼
GitHub Pages (index.html)
  https://winsol-belgie.github.io/VG_Balie_Aalter/
       │
       ▼
iPad (Safari, kiosk-modus via Guided Access)
```

De HTML-pagina heeft **geen login nodig** en werkt volledig standalone. De iPad hoeft alleen wifi te hebben.

---

## Componenten

| Onderdeel | Details |
|---|---|
| **Power Automate flow** | "Roomdisplay Balie Aalter" — eigenaar: gwenn.vanthournout@winsol.eu |
| **Graph API endpoint** | `GET /v1.0/users/VG_Balie_Aalter@winsol.eu/calendarView` |
| **GitHub Gist** | https://gist.github.com/GwennVanthournout/de0ae2431d0ad466bf3dd9a4497313ae |
| **GitHub repository** | https://github.com/Winsol-Belgie/VG_Balie_Aalter |
| **Live pagina** | https://winsol-belgie.github.io/VG_Balie_Aalter/ |
| **iPad** | iOS 12.5.8, Safari, beginscherm-snelkoppeling + Guided Access |

---

## Onderhoud

### Data wordt niet ververst
1. Ga naar Power Automate → "Roomdisplay Balie Aalter"
2. Controleer de **Run history** — zijn er rode (Failed) entries?
3. Meest voorkomende oorzaak: de Office 365-verbinding is verlopen → "Change connection" en opnieuw authenticeren met gwenn.vanthournout@winsol.eu

### Pagina toont verouderde data
- De Gist wordt elke 10 minuten bijgewerkt door Power Automate
- De pagina haalt zelf elke 5 minuten nieuwe data op
- Maximale vertraging: ~15 minuten

### Pagina werkt niet (wit scherm / "Laden...")
- Controleer of de GitHub Pages actief is: Settings → Pages → moet "Your site is live at..." tonen
- Controleer of de Gist bereikbaar is (open de Gist URL in een browser)
- Herlaad de iPad-pagina manueel

---

## Aanpassingen

### Meer/minder dagen vooruit ophalen
In Power Automate → flow → "Send an HTTP request" → URI:  
Verander het getal in `addDays(..., 14)` naar gewenste aantal dagen.

### Refresh-interval aanpassen
In `index.html`, bovenaan in het `CONFIG`-blok:
```javascript
const CONFIG = {
  refreshMinutes: 5,   // hoe vaak de pagina nieuwe data ophaalt
  dayStart: 7,         // eerste uur op de tijdlijn
  dayEnd: 19           // laatste uur op de tijdlijn
};
```

### Inactiviteit-timer (terugkeer naar vandaag)
In `index.html`:
```javascript
const INACTIVITY_MS = 2 * 60 * 1000;  // 2 minuten, aanpassen indien gewenst
```

---

## iPad instellen (Guided Access / kiosk)

1. **Instellingen → Algemeen → Toegankelijkheid → Guided Access** → aan, pincode instellen
2. Open de pagina via het beginscherm-icoontje (niet via Safari rechtstreeks)
3. Druk **3x op de thuisknop** → tik **Start**
4. **Automatisch slot**: Instellingen → Beeldscherm en helderheid → Automatisch slot → **Nooit**

Guided Access verlaten: 3x thuisknop → pincode → Stop.

---

## GitHub token (Power Automate)

De flow schrijft naar de Gist via een GitHub Personal Access Token (scope: `gist`).  
Dit token staat in de Authorization header van de HTTP-actie in Power Automate.  
Token is aangemaakt op het account van gwenn.vanthournout@winsol.eu.  
Als het token verloopt: nieuw token aanmaken op github.com → Settings → Developer settings → Personal access tokens → scope `gist`.

Er zit een token in de export van de flow, maar die is vervallen wegens "exposed" op github. Dit token is dus niet meer de correcte versie.
Check de flow bij Gwenn voor de werkende token.

---

## Power Automate flow (backup)

> 🚫 **De flow-export (.zip) mag NIET in deze publieke repository worden geplaatst.**  
> De zip bevat de volledige configuratie inclusief het GitHub token in de Authorization header. GitHub detecteert dit automatisch en trekt het token in.  
> Bewaar de export lokaal of in een **private** omgeving (bv. SharePoint).

> ⚠️ **Let op bij importeren:** bij een herinstallatie moet je het volgende opnieuw instellen:
> - De **Office 365 Outlook-verbinding** (authenticeren met gwenn.vanthournout@winsol.eu of een ander account met toegang tot VG_Balie_Aalter@winsol.eu)
> - Het **GitHub token** in de Authorization header van de laatste HTTP-actie (zie sectie hierboven)
> - De **Gist ID** in de PUT-URL van de laatste HTTP-actie

Importeren via Power Automate → **My flows** → **Import** → selecteer de `.zip`.
