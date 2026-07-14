# Bevinding: dekkingsgat in PHN_20260101 (juli 2026)

Geconstateerd tijdens het geocoderen van 4,36 mln NVM-transacties met BAG-Tools
(`cfg/Geocode`, PHN als hoogtebron). Deze notitie legt vast **wat** we hebben
gemeten, **wat de vermoedelijke oorzaak** is en **hoe het te verifiëren** op de
machine waar de AHN-brondata staat (`%AHNDrive%`).

## Wat we zien

Van de aan de BAG gekoppelde NVM-panden heeft **10,5% geen pandhoogte** uit
`PHN_20260101.mmd`, tegen **6,0%** in de vorige PHN-vintage (2023, via
`PHN_20230101.fss`). De uitval is dus ruim verdubbeld terwijl de nieuwe PHN
landsdekkend oogt.

Belangrijkste kenmerk: **de uitval is sterk regionaal geclusterd**, niet
willekeurig. Aandeel panden zonder hoogte per gemeente (n > 20k transacties):

| gemeente | % zonder hoogte |
|---|---|
| GM0193 (Zwolle) | 42,9 |
| GM0202 (Arnhem) | 42,4 |
| GM0014 (Groningen) | 38,3 |
| GM0106 / GM0150 | 31,4 |
| GM0228 (Ede) | 31,3 |
| GM0200 | 27,8 |
| westelijke gemeenten | ~5–7 |

Naar bouwjaar loopt de uitval op met nieuwbouw (21–26: 40%; 16–20: 18%), maar de
regionale clustering domineert: ook vooroorlogse panden in Zwolle/Arnhem missen
massaal. In 6,7%-punt van de gevallen is het BAG-bouwjaar wél bekend en de
hoogte niet — het is dus geen BAG-koppelprobleem.

Aanvullende observatie (Jip): **de panden zonder hoogte hebben ook geen status**
— ze lijken dus in het geheel geen rij te hebben in de PHN-output, in plaats van
een rij met `hoogte = null` en `betrouwbaar = false`.

## Vermoedelijke oorzaak

De AHN-bron is één gecombineerde tegel-index:
`%AHNDrive%/AHN4_AHN5_AHN6_combination.gpkg` (`cfg/main/sourcedata/AHN.dms`), en
`results.dms` bouwt de eindtabel als `union_data` over **`Tiles_met_panden`** —
alleen tegels die in die index voorkomen. Ontbreekt een gebied in de
combinatie-index (of ontbreken/falen de bijbehorende `DSM/DTM/<tile>_05m.tif`),
dan komen de panden daar **helemaal niet in de output** — geen rij, dus ook geen
`betrouwbaar`-status. Dat past exact op het waargenomen patroon: aaneengesloten
regio's in oost/noord-Nederland (AHN5/AHN6-inwinning nog niet compleet), terwijl
het westen normale dekking heeft.

Alternatieve/aanvullende verklaring voor een deel van de uitval zijn de
kwaliteitsdrempels in `templates.dms` (`betrouwbaar`): `min_pandhoogte` 1 m,
`max_pandhoogte` 250 m, `min_pixels_boven_mv` ≥ 4 pixels. Die geven wél een rij
(met `hoogte = null`), dus die verklaren de "geen status"-gevallen niet, maar ze
kunnen de nieuwbouw-uitval (bouwjaar ≥ 2021: pand bestaat nog niet op de
AHN-opname) mede verklaren.

## Verificatieplan (uit te voeren waar de AHN-data staat)

1. **Tegeldekking**: hoeveel BAG-panden vallen buiten `Tiles_met_panden`?
   Vergelijk `sourcedata/BAG/pand` (alle panden) met de union in
   `results.dms:155` (`PHN_<datum>.mmd`) — het verschil is de "geen rij"-groep.
   Verwachting: aaneengesloten tegels in oost/noord ontbreken in
   `AHN4_AHN5_AHN6_combination.gpkg`.
2. **Tif-dekking per tegel**: bestaan voor élke tegel in de index de bestanden
   `DSM/<tile>_05m.tif` en `DTM/<tile>_05m.tif`? Een ontbrekend/leeg tif geeft
   een tegel zonder bruikbare hoogtes.
3. **Drempel-uitval**: van de panden mét rij: verdeling van `betrouwbaar`,
   `n_pixels`, `n_pixels_boven_mv` en `hoogte_ruw` — welke drempel bijt?
4. **Provenance**: `DSM/<tile>_src.tif` geeft per pixel de AHN-versie (4/5/6);
   daarmee is te tonen of de gaten samenvallen met AHN5/6-inwinvakken.

## Mogelijke oplossingen (in volgorde van voorkeur)

1. **AHN4 als fallback in de combinatie-index**: neem voor tegels zonder
   AHN5/AHN6 de AHN4-tegel op, zodat PHN landsdekkend blijft (met `AHN_inwinjaar`
   /provenance als kwaliteitsindicator voor de gebruiker).
2. **Ontbrekende tegels expliciet loggen** in de PHN-run (nu verdwijnen ze
   stilzwijgend), zodat een dekkingsgat niet pas bij een afnemer opvalt.
3. **Rij behouden voor elk BAG-pand**, ook zonder AHN-dekking, met
   `hoogte = null` + reden — afnemers kunnen dan onderscheiden tussen "geen
   AHN-dekking" en "onbetrouwbare meting".

## Wat er inmiddels aan de afnemerskant is gedaan

In **BAG-Tools** (`cfg/Geocode/BAG.dms`, commit "PHN-fallback…") is een
vangnet ingebouwd: `pand_hoogte := MakeDefined(PHN-<jaar>, PHN-<fallback_jaar>)`
met `pand_hoogte_fallback_jaar := '2023'`. Daarmee daalt de uitval in de
NVM-koppeling van 10,5% naar **5,1%** (de rest is echte nieuwbouw van ná beide
AHN-inwinrondes). Dit is een pleister, geen oplossing: PHN zelf zou landsdekkend
moeten zijn.

Effect op de hedonische prijsanalyse (appartementen, `_Tools/PriceIndices`):
`d_highrise` 0,058 → 0,064 en de missing-indicator `d_hoogte_onbekend` 0,092 →
0,082; overige coëfficiënten stabiel.
