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

## 3. ETL Proces v snowflake
ETL je skratka pre Extract, Transform, Load (extrahovať, transformovať, načítať). Je to proces, ktorý sa používa na presun dát z rôznych zdrojov (napr. databázy, súbory CSV) do jedného centrálneho úložiska, ako je Snowflake, kde môžu byť tieto dáta analyzované.

### Extract (Extrahovanie dát)

CSV súbory boli nahraté do Snowflake prostredníctvom interného stage úložiska, ktoré sa nazýva my_stage.
```sql
CREATE OR REPLACE STAGE my_stage;
```

Príkaz na vytvorenie my_stage úložiska slúži na dočasné uloženie súborov, ako napríklad CSV súbory, pred ich následným spracovaním.
```sql
COPY INTO occupations_staging
FROM @my_stage/occupations.csv
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);
```
### Transform (Transformácia dát)
Transformácia je proces, počas ktorého sa surové dáta očistia, upravia a premenia na štruktúrovaný formát vhodný pre analýzu. V mojom prípade dáta zo staging tabuliek na dimenzie a faktovú tabuľku.

+ `Dim_movies`
  Táto tabuľka ukladá informácie o filmoch v dátovom sklade.
```sql
CREATE OR REPLACE TABLE Dim_movies AS
SELECT DISTINCT
    m.id AS idDim_movies,
    m.title,
    m.release_year,
    LISTAGG(g.name, ', ') WITHIN GROUP (ORDER BY g.name) AS genres
FROM movies_staging m
LEFT JOIN movies_genres_staging mg ON m.id = mg.movie_id
LEFT JOIN genres_staging g ON mg.genre_id = g.id
GROUP BY m.id, m.title, m.release_year;

```
+ `Dim_users`
  Táto tabuľka ukladá informácie o používateľoch, ktorí hodnotili filmy.
```sql
CREATE OR REPLACE TABLE Dim_users AS
SELECT DISTINCT
    u.id AS idDim_users,
    u.age,
    CASE 
        WHEN u.age BETWEEN 0 AND 18 THEN '0-18'
        WHEN u.age BETWEEN 19 AND 35 THEN '19-35'
        WHEN u.age BETWEEN 36 AND 50 THEN '36-50'
        ELSE '51+'
    END AS age_group,
    u.gender,
    o.name AS occupation,
    u.zip_code
FROM users_staging u
LEFT JOIN occupations_staging o ON u.occupation_id = o.id;
```
+ `Dim_tags`
  Táto tabuľka ukladá informácie o značkách, ktoré používatelia priradili filmom.
```sql
CREATE OR REPLACE TABLE Dim_tags AS
SELECT DISTINCT
    t.id AS idDim_tags,
    t.tags,
    t.created_at AS creation
FROM tags_staging t;
```
+ `Dim_timestamp`
  Táto tabuľka ukladá časové pečiatky extrahované z údajov o hodnoteniach v samostatnej tabuľke pre dimenzionálne modelovanie.
```sql
CREATE OR REPLACE TABLE Dim_timestamp AS
SELECT DISTINCT
    EXTRACT(EPOCH_SECOND FROM r.rated_at) AS idDim_time,
    EXTRACT(SECOND FROM r.rated_at) AS second,
    EXTRACT(MINUTE FROM r.rated_at) AS minute,
    EXTRACT(HOUR FROM r.rated_at) AS hour,
    EXTRACT(DAY FROM r.rated_at) AS day,
    EXTRACT(WEEK FROM r.rated_at) AS week,
    EXTRACT(MONTH FROM r.rated_at) AS month
FROM ratings_staging r;
```
+ `Fact_ratings`
  Táto tabuľka ukladá faktické údaje o hodnoteniach filmov.
```sql
CREATE OR REPLACE TABLE Fact_ratings AS
SELECT 
    r.id AS Fact_ratingscol,
    r.rating,
    AVG(r.rating) OVER (PARTITION BY r.movie_id) AS average_rating,
    COUNT(r.rating) OVER (PARTITION BY r.movie_id) AS rating_count,
    t.id AS idDim_tags,
    u.id AS idDim_users,
    m.id AS idDim_movies,
    EXTRACT(EPOCH_SECOND FROM r.rated_at) AS idDim_time
FROM ratings_staging r
LEFT JOIN movies_staging m ON r.movie_id = m.id
LEFT JOIN users_staging u ON r.user_id = u.id
LEFT JOIN tags_staging t ON r.movie_id = t.movie_id AND r.user_id = t.user_id;
```

### Load (Načítanie dát)
Po úspešnom vytvorení dimenzií a faktovej tabuľky boli údaje premiestnené do finálnej štruktúry. Nakoniec boli staging tabuľky odstránené, aby sa optimalizovalo využitie úložného priestoru.

```sql
DROP TABLE IF EXISTS ratings_staging;
DROP TABLE IF EXISTS occupations_staging;
DROP TABLE IF EXISTS tags_staging;
DROP TABLE IF EXISTS movies_genres_staging;
DROP TABLE IF EXISTS users_staging;
DROP TABLE IF EXISTS age_group_staging;
DROP TABLE IF EXISTS movies_genres_staging;
DROP TABLE IF EXISTS movies_staging;
```

## 4. Vizualizácia dát

1. Top 10 najvyššie hodnotených filmov:
```SQL
SELECT 
    dm.title, 
    AVG(fr.rating) AS avg_rating 
FROM 
    Fact_ratings fr 
JOIN 
    Dim_movies dm ON fr.idDim_movies = dm.idDim_movies 
GROUP BY 
    dm.title 
ORDER BY 
    avg_rating DESC 
LIMIT 10;
```
Tento dotaz vyberie 10 filmov s najvyšším priemerným hodnotením. Spojí tabuľku hodnotení (Fact_ratings) s tabuľkou filmov (Dim_movies), vypočíta priemerné hodnotenie pre každý film a potom zoradí výsledky zostupne podľa priemerného hodnotenia a vyberie len prvých 10 riadkov.

2. Priemerné hodnotenie podľa vekovej skupiny:
```SQL
SELECT 
    du.age_group, 
    AVG(fr.rating) AS avg_rating 
FROM 
    Fact_ratings fr 
JOIN 
    Dim_users du ON fr.idDim_users = du.idDim_users 
GROUP BY 
    du.age_group 
ORDER BY 
    avg_rating DESC;
```
Tento dotaz vypočíta priemerné hodnotenie pre každú vekovú skupinu používateľov. Spojí tabuľku hodnotení (Fact_ratings) s tabuľkou používateľov (Dim_users), vypočíta priemerné hodnotenie pre každú vekovú skupinu a potom zoradí výsledky zostupne podľa priemerného hodnotenia.

3. Top 10 najčastejšie hodnotených filmov:
```SQL
SELECT 
    dm.title, 
    COUNT(*) AS pocet_hodnoteni
FROM 
    Fact_ratings fr
JOIN 
    Dim_movies dm ON fr.idDim_movies = dm.idDim_movies
GROUP BY 
    dm.title
ORDER BY 
    pocet_hodnoteni DESC
LIMIT 10;
```
Tento dotaz vyberie 10 filmov s najväčším počtom hodnotení. Spojí tabuľku hodnotení (Fact_ratings) s tabuľkou filmov (Dim_movies), spočíta počet hodnotení pre každý film a potom zoradí výsledky zostupne podľa počtu hodnotení a vyberie len prvých 10 riadkov.

4. Rozdelenie hodnotení:
```SQL
SELECT rating, COUNT(*) AS pocet_hodnoteni
FROM Fact_ratings
GROUP BY rating;
```
Tento dotaz vypočíta počet hodnotení pre každú možnú hodnotu (1-5). Zoskupí záznamy podľa hodnotenia a spočíta počet záznamov pre každú hodnotu.

5. Top 10 najhoršie hodnotených filmov:
```SQL
SELECT 
    dm.title, 
    AVG(fr.rating) AS avg_rating 
FROM 
    Fact_ratings fr 
JOIN 
    Dim_movies dm ON fr.idDim_movies = dm.idDim_movies 
GROUP BY 
    dm.title 
ORDER BY 
    avg_rating ASC
LIMIT 10;
```
Popis: Tento dotaz vyberie 10 filmov s najnižším priemerným hodnotením. Spojí tabuľku hodnotení (Fact_ratings) s tabuľkou filmov (Dim_movies), vypočíta priemerné hodnotenie pre každý film a potom zoradí výsledky vzostupne podľa priemerného hodnotenia a vyberie len prvých 10 riadkov.
<div align="center"> 
  <img src="https://github.com/user-attachments/assets/1076ed4b-2863-42bc-9a1e-0f43aee709d6" alt="MovieLens_ERD"> 
  <p>
    Obrázok 3. Dashboard
  </p> 
</div>

<p>
    Autor: Lukáš Hošek
  </p> 



