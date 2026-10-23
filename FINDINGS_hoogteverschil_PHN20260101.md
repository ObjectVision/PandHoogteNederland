# Bevinding: PHN_20260101 is hoger dan PHN_20210101 (september 2026)

Aanleiding: Hestia (`docs/bag.md`) meldt dat dezelfde panden in PHN 2026
gemiddeld 52 cm hoger zijn dan in PHN 2021 (598 tegen 545 cm). Dat klopt als
gemiddelde, maar Nederland is niet een halve meter hoger geworden.

## Het verschil

10.003.779 panden hebben in beide sets een hoogte. Het gemiddelde gaat van 545,4
naar 597,8 cm (+52,4 cm). De mediaan van het verschil is +13 cm, en 70% van de
panden verschilt minder dan 25 cm.

| groep | panden | bijdrage aan +52,4 cm | mediaan verschil |
|---|---:|---:|---:|
| gebouwd in of na het AHN3-inwinjaar | 5,4% | 21,6 cm | +2,71 m |
| bestaand pand, ≥ 1 m hoger | 4,8% | 20,7 cm | +3,68 m |
| overige bestaande panden | 89,8% | 10,2 cm | +0,12 m |

1. **Nieuwbouw.** AHN3 is ingewonnen in 2014–2019 en mat deze panden als
   bouwterrein of in aanbouw. Dit verschil is echt: PHN 2021 is hier verouderd.
2. **Sprongen bij bestaande panden.** AHN3 (PHN 2021) en AHN4 (PHN 2023) geven
   voor deze panden dezelfde hoogte; alleen AHN5/6 geeft er meters bij. In de
   rasters is gekeken naar 224 van deze panden en naar 229 controlepanden.
   - Het oppervlak binnen het pand is ruw: het mediane verschil tussen
     buurpixels is 0,38 m, tegen 0,07 m bij de controlepanden.
   - Het hoge oppervlak loopt buiten het pand door: bij 57% van de panden is
     ook de rand van 2 m rond het pand hoog, tegen 5% bij de controlepanden.

   Dit zijn boomkruinen boven vooral kleine panden: mediaan 26 m², tegen 56 m²
   bij de overige bestaande panden. Landelijk heeft 5,0% van de bestaande
   panden zo'n sprong. In bosrijke gemeenten is dat 10–12% (Ermelo, Lochem,
   Utrechtse Heuvelrug, Wassenaar, Westerveld, Zeist), in Almere, Lelystad en
   Purmerend 2%. Het is dus een artefact.
3. **Systematisch verschil door AHN5/6.**
   - AHN3 → AHN4 met dezelfde methode: 0 cm (mediaan +1).
   - De huidige code op AHN4-data (0,1% van de panden) geeft precies PHN 2023
     (mediaan 0,0).
   - Op AHN5-data is de mediaan +11 cm, op AHN6-data +15 cm.

   Het verschil komt dus uit de AHN5/6-data, niet uit de methode. Het past bij
   de verlaging naar percentiel 0,6 die EO voor AHN5/6 gebruikte
   (`templates.dms`, nog te valideren).

## Gevolg voor Hestia

Hestia legt de grens tussen meergezins laag/midden en hoog bij 12,6 m (5
verdiepingen van 2,80 m). Met PHN 2026 gaan ongeveer 218 duizend woningen van
laag/midden naar hoog:

| woningen | oorzaak |
|---:|---|
| 125.519 | pand gebouwd na de AHN3-vlucht (PHN 2021 mat mediaan 0,55 m): echt |
| 54.564 | pand zonder hoogte in PHN 2021: echt |
| 26.511 | bestaand pand met een sprong van ≥ 1 m; bij 13.520 daarvan geeft AHN4 ook ≥ 12,6 m |
| 24.990 | bestaand pand dat met < 1 m verschil over de grens gaat (mediaan +31 cm): AHN5/6-verschuiving |
| −7.824 | van hoog naar laag/midden |

De 119 duizend panden die over de 12,6 m gaan, zijn vooral kleine panden met een
boomsprong: 108.805 bestaande panden, mediaan +7,1 m. Daar zitten weinig
woningen in.

## Open

- Waarom AHN5/6 boomkruin meet waar AHN3/4 dat niet deden: inwinseizoen of de
  rastering door EO. Navragen bij EO.
- Percentiel 0,6 tegen 0,7 valideren (3D BAG), en een vegetatiefilter
  overwegen, bijvoorbeeld op de ruwheid binnen het pand of de hoogte van de
  rand eromheen.
