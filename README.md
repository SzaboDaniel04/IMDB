ETL Proces pre IMDb Dataset
Tento repozitár obsahuje implementáciu ETL procesu v Snowflake pre analýzu dát z IMDb datasetu. Projekt je zameraný na spracovanie údajov o filmoch, režiséroch, hercoch a ich hodnoteniach s cieľom identifikovať trendy v kinematografii, najpopulárnejšie filmy, a analyzovať vzťahy medzi filmovými metrikami. Výsledný dátový model umožňuje multidimenzionálnu analýzu a vizualizáciu kľúčových metrik.

1. Úvod a popis zdrojových dát
Cieľom projektu je analyzovať a transformovať údaje týkajúce sa filmov, režisérov, hercov a ich hodnotení, aby bolo možné:

Identifikovať trendy vo filmovej tvorbe, ako sú obľúbené žánre alebo zisky.
Analyzovať hodnotenia filmov a ich súvislosti s tržbami, dĺžkou filmu či certifikátmi.
Preskúmať prácu režisérov a herecké obsadenie s ohľadom na úspešnosť filmov.
Zdrojové dáta pochádzajú z IMDb datasetu, ktorý obsahuje nasledujúce hlavné tabuľky:

movies – Informácie o filmoch, ako názov, dĺžka, rok vydania, zisky a hodnotenia.
directors – Informácie o režiséroch, vrátane ich mena, národnosti a roku narodenia.
actors – Informácie o hercoch, ako sú meno, národnosť a rok narodenia.
genres – Zoznam filmových žánrov priradených k jednotlivým filmom.
regions – Geografické oblasti, kde boli filmy produkované alebo kde dosiahli popularitu.
 
  2.Dimenzionálny model
Pre tento projekt bol navrhnutý hviezdicový model (star schema), ktorý umožňuje efektívnu a prehľadnú analýzu dát. Centrálnym bodom modelu je faktová tabuľka fact_ratings, ktorá obsahuje hlavné metriky a referencie na dimenzie. Táto štruktúra umožňuje jednoduché dotazovanie a rýchly prístup k agregovaným údajom.
  ![Star](https://github.com/user-attachments/assets/4c740841-6068-441b-bf1d-8d3a2484d745)

3. ETL proces v Snowflake
ETL proces pozostával z troch hlavných fáz: extrahovanie (Extract), transformácia (Transform) a načítanie (Load). Tento proces bol implementovaný v Snowflake s cieľom pripraviť zdrojové dáta zo staging vrstvy do viacdimenzionálneho modelu vhodného na analýzu a vizualizáciu.

3.1 Extract (Extrahovanie dát)
Dáta zo zdrojového datasetu (formát .csv) boli najprv nahraté do Snowflake prostredníctvom interného stage úložiska s názvom my_stage. Stage v Snowflake slúži ako dočasné úložisko na import alebo export dát. Vytvorenie stage bolo zabezpečené príkazom:

Príklad kódu:

    CREATE OR REPLACE STAGE my_stage;

Do stage boli následne nahraté súbory obsahujúce údaje o knihách, používateľoch, hodnoteniach, zamestnaniach a úrovniach vzdelania. Dáta boli importované do staging tabuliek pomocou príkazu COPY INTO. Pre každú tabuľku sa použil podobný príkaz:

    COPY INTO occupations_staging
    FROM @my_stage/occupations.csv
    FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1);

3.2 Transformácia dát (Transform)
V ETL procese pre IMDb dataset je transformácia dát kľúčovým krokom, v ktorom sa údaje čistia, štandardizujú, obohacujú a pripravujú do formátu, ktorý podporuje analýzu v dimenzionálnom modeli

Príklad kódu:

    CREATE OR REPLACE TABLE dim_movies AS
    SELECT DISTINCT
        dataID AS movie_key,
        title AS movie_title,
        contentType AS movie_type,
        releaseYear AS release_year,
        length AS movie_length,
        certificate AS movie_certificate,
        description AS movie_description
    FROM movies_staging;

4.Vizualizácia dát
Vizualizácia dát je kľúčovým krokom na interpretáciu výsledkov ETL procesu a na identifikáciu zaujímavých vzorcov alebo trendov.
![Diagramy](https://github.com/user-attachments/assets/b671848f-61da-41f7-b1a9-134b79fdcf06)


Graf 1:Počet filmov podľa žánru
Vizualizácia ukazuje, ktoré žánre majú najviac filmov v databáze IMDb.

    SELECT 
        g.genre AS genre,
        COUNT(f.data_key) AS movie_count
    FROM fact_movies f
    JOIN dim_genre g ON f.genre_key = g.genre_key
    GROUP BY g.genre
    ORDER BY movie_count DESC;


Graf 2:Priemerné hodnotenie filmov podľa žánru
Ukazuje, ktoré žánre majú filmy s najvyššími priemernými hodnoteniami podľa používateľov

    SELECT 
        g.genre AS genre,
        AVG(f.rating) AS avg_rating
    FROM fact_movies f
    JOIN dim_genre g ON f.genre_key = g.genre_key
    GROUP BY g.genre
    ORDER BY avg_rating DESC;


Graf 3:Počet filmov podľa roku vydania
Graf ukazuje, ako sa počet vydaných filmov menil v jednotlivých rokoch.

    SELECT 
        f.release_year AS release_year,
        COUNT(f.data_key) AS movie_count
    FROM fact_movies f
    GROUP BY f.release_year
    ORDER BY f.release_year;

Graf 4:Top 10 režisérov podľa počtu filmov
Môžete zistiť, ktorí režiséri mali najväčší vplyv na filmovú produkciu

    SELECT 
        d.director_name AS director_name,
        COUNT(f.data_key) AS movie_count
    FROM fact_movies f
    JOIN dim_director d ON f.director_key = d.director_key
    GROUP BY d.director_name
    ORDER BY movie_count DESC
    LIMIT 10;

Graf 5:Celkové tržby filmov podľa regiónu
Ukazuje, ktoré regióny generujú najväčšie tržby z filmov. Môže to ukázať, ktorý región je dominantný v oblasti filmového priemyslu.


    SELECT 
    r.region AS region,
    SUM(f.gross) AS total_gross
    FROM fact_movies f
    JOIN dim_region r ON f.region_key = r.region_key
    GROUP BY r.region
    ORDER BY total_gross DESC;

Autor: Daniel Szabó


