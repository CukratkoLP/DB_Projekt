# ETL Proces datasetu Movielens

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
<div align="center"> 
<img src="https://github.com/user-attachments/assets/d8630a63-06b5-401e-8e10-6bcbc1862369" alt="MovieLens_ERD"> 
<p>
Obrázok 1. Entitno-relačná schéma MovieLens
</p>
</div>

## 2. Tvorba dimenzionálneho modelu
Navrhnutý dimenzionálny model poskytuje základ pre analýzu údajov o hodnotení filmov. Faktová tabuľka obsahuje kľúčové metriky a odkazuje na rôzne dimenzie. Pomocou tohto modelu je možné odpovedať na širokú škálu otázok týkajúcich sa preferencií používateľov a popularity filmov
## Faktová tabuľka:  ` Fact_ratings `
### Hlavné metriky:

+ **rating:** Číselné hodnotenie, ktoré používateľ udelil filmu. Táto metrika je kľúčová pre analýzu preferencií používateľov a popularity filmov.
+ **average_rating:** Priemerné hodnotenie filmu všetkými používateľmi. Táto metrika umožňuje porovnať popularitu rôznych filmov.
+ **rating_count:** Počet hodnotení, ktoré film získal. Táto metrika poskytuje informáciu o tom, ako často bol film hodnotený.

### Kľúče v tabuľke `Fact_ratings:`

+ **idDim_time:** Odkaz na konkrétny časový úsek, kedy bolo hodnotenie udelené.
  
+ **idDim_date:** Odkaz na konkrétny dátum, kedy bolo hodnotenie udelené.
  
+ **idDim_tags:** Odkaz na tagy alebo kľúčové slová spojené s hodnotením.
  
+ **idDim_users:** Odkaz na konkrétneho používateľa, ktorý hodnotenie udeli.
  
+ **idDim_movies:** Odkaz na konkrétny film, ktorý bol hodnotený.
### Dimenzionálne tabuľky:

**`Dim_timestamp:`** Časové údaje (sekundy až mesiace) pre analýzu zmien hodnotení v čase. **SCD typ 1.**

**`Dim_users:`** Demografické údaje o používateľoch (vek, pohlavie, atď.) pre analýzu preferencií. **SCD typ 2.**

**`Dim_tags:`** Tagy filmov pre analýzu popularity tém. **SCD typ 2.**

**`Dim_movies:`** Informácie o filmoch (názov, rok, žánre) pre analýzu výkonnosti. **SCD typ 2.**
<div align="center"> 
  <img src="https://github.com/user-attachments/assets/3a785924-6d06-4704-8835-9d2d76e0deb4" alt="MovieLens_ERD"> 
  <p>
    Obrázok 2. Schéma hviezdy pre Movielens
  </p> 
</div>



