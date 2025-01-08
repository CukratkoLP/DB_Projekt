# Téma projektu

## 1. Úvod a popis zdrojových dát

Téma projektu sa zameriava na analýzu filmových preferencií používateľov prostredníctvom databázy MovieLens. Cieľom je analyzovať údaje o filmoch, hodnoteniach, žánroch a demografických údajoch používateľov, aby sa identifikovali vzorce správania, obľúbené žánre, trendy v hodnoteniach a ďalšie užitočné poznatky. Tento dokument popisuje štruktúru databázy a možnosti jej využitia.

## 1.1  Tabuľky a ich popis 
Databáza MovieLens obsahuje tabuľky, ktoré uchovávajú údaje o používateľoch, filmoch, hodnoteniach, žánroch a ďalších aspektoch. Medzi hlavné tabuľky patria:

+ **users**: Obsahuje demografické údaje používateľov, ako sú vek, pohlavie, PSČ a povolanie.

+ **movies**: Zahŕňa informácie o filmoch vrátane názvu a roku vydania.

+ **ratings**: Obsahuje hodnotenia filmov, ktoré vytvorili používatelia spolu s časovými pečiatkami.

+ **tags**: Zaznamenáva tagy (značky), ktoré používatelia priraďujú filmom.

+ **genres**: Ukladajú informácie o filmových žánroch

+ **genres_movies**: Zaznamenáva vzťahy medzi žánrami a filmami

+ **occupations**: Doplňujúca tabuľka o zamestnaní používateľa

+ **age_group**: Doplňujúca tabuľka o vekovej skupine používateľa
## 1.2 ERD Diagram
ERD (Entitno - relačný diagram) znázorňuje štruktúru databázy MovieLens a vzťahy medzi jednotlivými tabuľkami na nasledujúcom obrázku:

![MovieLens_ERD](https://github.com/user-attachments/assets/d8630a63-06b5-401e-8e10-6bcbc1862369)
<p align="center">
Obrázok 1. Entitno-relačná schéma MovieLens
</p>

## 2. Tvorba dimenzionálneho modelu
