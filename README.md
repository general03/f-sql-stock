Utilisations des procédures, function et trigger SQL avec PostgreSql

# 1. Utiliser elephantsql.com

# 2. Importer le fichier CSV via dbeaver

## 2.1 DDL

```
CREATE TABLE public."data" (
	country varchar NOT NULL,
	population int4 NOT NULL,
	density float4 NOT NULL,
	area float4 NOT NULL,
	id int4 NOT NULL GENERATED ALWAYS AS IDENTITY,
	updated timestamp NULL
);
```

# 3. Fonction qui retourne le pays qui correspond au critère passé en paramètres

```
CREATE OR REPLACE function get_country(c varchar)
    RETURNS table (
        country varchar
    ) 
    LANGUAGE plpgsql
    AS 
$$
BEGIN
    return query select data.country from data where data.country = c ;
END;
$$;

select get_country('test');
```

# 4. Procédure qui insert un nvx pays avec des données random (on précise uniquement le pays)

```
CREATE OR REPLACE PROCEDURE add_data(country varchar)
    LANGUAGE SQL
AS
$$
    INSERT INTO data VALUES (country, random() * 10000, floor(random() * 10), random() * 1000);
$$;

CALL add_data('test');
```

# 5. Configurer un trigger qui va mettre à jour la colonne de la table data correspondant à la date de l'insertion

```
CREATE OR REPLACE FUNCTION update_date()
  RETURNS TRIGGER 
  LANGUAGE PLPGSQL  
  AS
$$
BEGIN
	update data set updated=now() where id = NEW.id;

	RETURN NEW;
END;
$$


CREATE TRIGGER country_trigger AFTER INSERT ON data
for each row 
EXECUTE PROCEDURE update_date();
```

# 6. Faire une requête pour obtenir la densité pour 1 pays

```
CREATE OR REPLACE function get_stats(c varchar)
    RETURNS TABLE(dens int )
    LANGUAGE plpgsql
    AS 
$$
BEGIN
      return query 
      SELECT 
        CASE WHEN data.density < 10 THEN 1 
            WHEN data.density < 100 THEN 2 
            WHEN data.density < 500 THEN 3 
            WHEN data.density >= 500 THEN 4 
            ELSE 0
        END as dens 
        FROM data 
        WHERE data.country = c 
        GROUP BY data.country, data.density;
END;
     
$$;
select get_stats('test');
```
