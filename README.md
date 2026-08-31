# VMH Horeca — kaart van alle objecten

Interactieve kaart van alle objecten op vmh-horeca.nl/aanbod/, met dagelijkse
automatische update.

## Hoe het werkt

- **`index.html`** — de kaart zelf. Volledig statisch (design, Leaflet-code),
  laadt de objectenlijst bij het openen via `fetch('./data/objects.json')`.
- **`data/objects.json`** — de actuele lijst met objecten (id, titel, lat/lon,
  makelaar, link). Dit bestand wordt elke nacht automatisch ververst.
- **`scripts/update_data.py`** — het scrape-script dat `data/objects.json`
  opnieuw opbouwt vanaf vmh-horeca.nl. Haalt eerst op wat er *echt* op de
  `/aanbod/`-pagina staat (niet de volledige backend-database, die ook oude
  posts bevat), en leest per object de coördinaten en makelaar uit.
- **`.github/workflows/update-map-data.yml`** — een GitHub Action die het
  script elke nacht om 03:00 UTC automatisch draait en het JSON-bestand
  commit als er iets veranderd is.

## Eenmalige setup

1. **Maak een (privé of publiek) GitHub-repository** aan en push deze
   bestanden erin.
2. **Zet GitHub Pages aan** (Settings → Pages → Deploy from branch → `main`
   / root), of host de map ergens anders waar `index.html` en de `data/`-map
   samen staan.
3. **Test de Action handmatig**: ga naar het tabblad *Actions* in GitHub,
   kies "Update VMH map data" → *Run workflow*. Controleer of
   `data/objects.json` daarna een nieuwe commit heeft gekregen.
4. Vanaf nu draait hij vanzelf elke nacht.

## Op de eigen site plaatsen (WordPress)

Twee opties:

- **Iframe** (makkelijkst, kaart blijft los beheerd):
  ```html
  <iframe src="https://<jouw-gehoste-url>/index.html"
          style="width:100%; height:80vh; border:0;"></iframe>
  ```
- **Custom HTML-blok**: plak de inhoud van `index.html` in een WordPress
  Custom HTML-blok. Pas dan wel `DATA_URL` in het script aan naar het
  volledige pad naar `objects.json` op je hosting (bijv.
  `https://vmh-horeca.nl/wp-content/uploads/vmh-map/data/objects.json`),
  en zorg dat dat JSON-bestand ook via die Action geüpdatet wordt (upload
  het bestand bijvoorbeeld via (S)FTP/een deploy-stap in de workflow, in
  plaats van er alleen een git-commit van te maken).

## Zelf lokaal testen

```bash
pip install -r requirements.txt
python scripts/update_data.py     # ververst data/objects.json
python -m http.server 8000        # open daarna http://localhost:8000
```

## Onderhoud

- Duurt de scrape te lang of faalt hij vaak? Verlaag `max_workers` in
  `update_data.py` (nu 12 gelijktijdige requests).
- Wil je vaker/minder vaak verversen? Pas de `cron`-regel in de workflow
  aan (bijv. `"0 */6 * * *"` voor elke 6 uur).
- Krijg je meldingen dat de Action faalt? Voeg desgewenst een Slack/e-mail
  notificatie toe aan de workflow (GitHub Marketplace heeft hier kant-en-
  klare Actions voor).
