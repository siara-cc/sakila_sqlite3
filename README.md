# Sakila sample database

This database is a port of the [Sakila sample database](https://dev.mysql.com/doc/sakila/en/) provided by MySql for testing its functions and features.  It represents an online DVD store.  The history of the database can be found in the above link.

## License

The database contents are provided under BSD License as described [here](https://dev.mysql.com/doc/sakila/en/sakila-license.html).

## Usage

Download `sakila.db` from this repository and open it with the `sqlite3` command line tool, or any other database manipulation tool.

## Schema

```sql
CREATE TABLE actor (
  actor_id INTEGER PRIMARY KEY AUTOINCREMENT,
  first_name VARCHAR(45) NOT NULL,
  last_name VARCHAR(45) NOT NULL,
  last_update TIMESTAMP NOT NULL
);

CREATE TABLE address (
  address_id INTEGER PRIMARY KEY AUTOINCREMENT,
  address VARCHAR(50) NOT NULL,
  address2 VARCHAR(50) DEFAULT NULL,
  district VARCHAR(20) NOT NULL,
  city_id SMALLINT UNSIGNED NOT NULL,
  postal_code VARCHAR(10) DEFAULT NULL,
  phone VARCHAR(20) NOT NULL,
  last_update TIMESTAMP NOT NULL ,
  CONSTRAINT `fk_address_city` FOREIGN KEY (city_id) 
    REFERENCES city (city_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE category (
  category_id INTEGER PRIMARY KEY AUTOINCREMENT,
  name VARCHAR(25) NOT NULL,
  last_update TIMESTAMP NOT NULL
);

CREATE TABLE city (
  city_id INTEGER PRIMARY KEY AUTOINCREMENT,
  city VARCHAR(50) NOT NULL,
  country_id SMALLINT UNSIGNED NOT NULL,
  last_update TIMESTAMP NOT NULL,
  CONSTRAINT `fk_city_country` FOREIGN KEY (country_id)
    REFERENCES country (country_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE country (
  country_id INTEGER PRIMARY KEY AUTOINCREMENT,
  country VARCHAR(50) NOT NULL,
  last_update TIMESTAMP NOT NULL
);

CREATE TABLE customer (
  customer_id INTEGER PRIMARY KEY AUTOINCREMENT,
  store_id TINYINT UNSIGNED NOT NULL,
  first_name VARCHAR(45) NOT NULL,
  last_name VARCHAR(45) NOT NULL,
  email VARCHAR(50) DEFAULT NULL,
  address_id SMALLINT UNSIGNED NOT NULL,
  active BOOLEAN NOT NULL DEFAULT TRUE,
  create_date DATETIME NOT NULL,
  last_update TIMESTAMP,
  CONSTRAINT fk_customer_address FOREIGN KEY (address_id)
    REFERENCES address (address_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_customer_store FOREIGN KEY (store_id)
    REFERENCES store (store_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE film (
  film_id INTEGER PRIMARY KEY AUTOINCREMENT,
  title VARCHAR(255) NOT NULL,
  description TEXT DEFAULT NULL,
  release_year YEAR DEFAULT NULL,
  language_id TINYINT UNSIGNED NOT NULL,
  original_language_id TINYINT UNSIGNED DEFAULT NULL,
  rental_duration TINYINT UNSIGNED NOT NULL DEFAULT 3,
  rental_rate DECIMAL(4,2) NOT NULL DEFAULT 4.99,
  length SMALLINT UNSIGNED DEFAULT NULL,
  replacement_cost DECIMAL(5,2) NOT NULL DEFAULT 19.99,
  rating ENUM  DEFAULT 'G',
  special_features  DEFAULT NULL,
  last_update TIMESTAMP NOT NULL,
  CONSTRAINT fk_film_language FOREIGN KEY (language_id)
    REFERENCES language (language_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_film_language_original FOREIGN KEY (original_language_id)
    REFERENCES language (language_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE film_actor (
  actor_id INTEGER,
  film_id INTEGER,
  last_update TIMESTAMP NOT NULL,
  PRIMARY KEY  (actor_id,film_id),
  CONSTRAINT fk_film_actor_actor FOREIGN KEY (actor_id)
    REFERENCES actor (actor_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_film_actor_film FOREIGN KEY (film_id)
    REFERENCES film (film_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE film_category (
  film_id INTEGER NOT NULL,
  category_id INTEGER NOT NULL,
  last_update TIMESTAMP NOT NULL,
  PRIMARY KEY (film_id, category_id),
  CONSTRAINT fk_film_category_film FOREIGN KEY (film_id)
    REFERENCES film (film_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_film_category_category FOREIGN KEY (category_id)
    REFERENCES category (category_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE film_text (
  film_id INTEGER NOT NULL,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  PRIMARY KEY  (film_id)
);

CREATE TRIGGER `ins_film` AFTER INSERT ON `film` FOR EACH ROW BEGIN
    INSERT INTO film_text (film_id, title, description)
        VALUES (new.film_id, new.title, new.description);
  END;

CREATE TRIGGER `upd_film` AFTER UPDATE ON `film` FOR EACH ROW
  WHEN (old.title != new.title) OR (old.description != new.description) OR (old.film_id != new.film_id)
  BEGIN
    UPDATE film_text
       SET title=new.title,
           description=new.description,
           film_id=new.film_id
     WHERE film_id=old.film_id;
  END;

CREATE TRIGGER `del_film` AFTER DELETE ON `film` FOR EACH ROW BEGIN
    DELETE FROM film_text WHERE film_id = old.film_id;
  END;
CREATE TABLE inventory (
  inventory_id INTEGER PRIMARY KEY AUTOINCREMENT,
  film_id SMALLINT UNSIGNED NOT NULL,
  store_id TINYINT UNSIGNED NOT NULL,
  last_update TIMESTAMP NOT NULL,
  CONSTRAINT fk_inventory_store FOREIGN KEY (store_id)
    REFERENCES store (store_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_inventory_film FOREIGN KEY (film_id)
    REFERENCES film (film_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE language (
  language_id INTEGER PRIMARY KEY AUTOINCREMENT,
  name CHAR(20) NOT NULL,
  last_update TIMESTAMP NOT NULL
);

CREATE TABLE payment (
  payment_id INTEGER PRIMARY KEY AUTOINCREMENT,
  customer_id SMALLINT UNSIGNED NOT NULL,
  staff_id TINYINT UNSIGNED NOT NULL,
  rental_id INT DEFAULT NULL,
  amount DECIMAL(5,2) NOT NULL,
  payment_date DATETIME NOT NULL,
  last_update TIMESTAMP,
  CONSTRAINT fk_payment_rental FOREIGN KEY (rental_id)
    REFERENCES rental (rental_id) ON DELETE SET NULL ON UPDATE CASCADE,
  CONSTRAINT fk_payment_customer FOREIGN KEY (customer_id)
    REFERENCES customer (customer_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_payment_staff FOREIGN KEY (staff_id)
    REFERENCES staff (staff_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE rental (
  rental_id INTEGER PRIMARY KEY AUTOINCREMENT,
  rental_date DATETIME NOT NULL,
  inventory_id MEDIUMINT UNSIGNED NOT NULL,
  customer_id SMALLINT UNSIGNED NOT NULL,
  return_date DATETIME DEFAULT NULL,
  staff_id TINYINT UNSIGNED NOT NULL,
  last_update TIMESTAMP NOT NULL,
  CONSTRAINT fk_rental_staff FOREIGN KEY (staff_id)
    REFERENCES staff (staff_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_rental_inventory FOREIGN KEY (inventory_id)
    REFERENCES inventory (inventory_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_rental_customer FOREIGN KEY (customer_id)
    REFERENCES customer (customer_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE staff (
  staff_id INTEGER PRIMARY KEY AUTOINCREMENT,
  first_name VARCHAR(45) NOT NULL,
  last_name VARCHAR(45) NOT NULL,
  address_id SMALLINT UNSIGNED NOT NULL,
  picture BLOB DEFAULT NULL,
  email VARCHAR(50) DEFAULT NULL,
  store_id TINYINT UNSIGNED NOT NULL,
  active BOOLEAN NOT NULL DEFAULT TRUE,
  username VARCHAR(16) NOT NULL,
  password VARCHAR(64) DEFAULT NULL,
  last_update TIMESTAMP NOT NULL,
  CONSTRAINT fk_staff_store FOREIGN KEY (store_id)
    REFERENCES store (store_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_staff_address FOREIGN KEY (address_id)
    REFERENCES address (address_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE store (
  store_id INTEGER PRIMARY KEY AUTOINCREMENT,
  manager_staff_id TINYINT UNSIGNED NOT NULL,
  address_id SMALLINT UNSIGNED NOT NULL,
  last_update TIMESTAMP NOT NULL,
  CONSTRAINT fk_store_staff FOREIGN KEY (manager_staff_id)
    REFERENCES staff (staff_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT fk_store_address FOREIGN KEY (address_id)
    REFERENCES address (address_id) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE VIEW customer_list AS 
  SELECT cu.customer_id AS ID, CONCAT(cu.first_name, ' ', cu.last_name) AS name,
    a.address AS address, a.postal_code AS `zip code`,
	  a.phone AS phone, city.city AS city, country.country AS country, 
    IF(cu.active, 'active','') AS notes, cu.store_id AS SID
  FROM customer AS cu JOIN address AS a ON cu.address_id = a.address_id JOIN city ON a.city_id = city.city_id
	  JOIN country ON city.country_id = country.country_id;

CREATE VIEW film_list AS
  SELECT film.film_id AS FID, film.title AS title, film.description AS description, 
    category.name AS category, film.rental_rate AS price,
	  film.length AS length, film.rating AS rating, 
    GROUP_CONCAT(CONCAT(actor.first_name, ' ', actor.last_name, ', ')) AS actors
FROM category LEFT JOIN film_category ON category.category_id = film_category.category_id
              LEFT JOIN film ON film_category.film_id = film.film_id
              JOIN film_actor ON film.film_id = film_actor.film_id
	            JOIN actor ON film_actor.actor_id = actor.actor_id
GROUP BY film.film_id, category.name;

CREATE VIEW nicer_but_slower_film_list AS
  SELECT film.film_id AS FID, film.title AS title,
    film.description AS description, category.name AS category, 
    film.rental_rate AS price, film.length AS length, film.rating AS rating, 
    GROUP_CONCAT(CONCAT(CONCAT(UCASE(SUBSTR(actor.first_name,1,1)),
	    LCASE(SUBSTR(actor.first_name,2,LENGTH(actor.first_name))), 
      ' ',CONCAT(UCASE(SUBSTR(actor.last_name,1,1)),
	    LCASE(SUBSTR(actor.last_name,2,LENGTH(actor.last_name)))))), ', ') AS actors
  FROM category
    LEFT JOIN film_category ON category.category_id = film_category.category_id
    LEFT JOIN film ON film_category.film_id = film.film_id
    JOIN film_actor ON film.film_id = film_actor.film_id
    JOIN actor ON film_actor.actor_id = actor.actor_id
  GROUP BY film.film_id, category.name;

CREATE VIEW staff_list AS
SELECT s.staff_id AS ID, CONCAT(s.first_name, ' ', s.last_name) AS name,
       a.address AS address, a.postal_code AS `zip code`, a.phone AS phone,
       city.city AS city, country.country AS country, s.store_id AS SID
  FROM staff AS s JOIN address AS a ON s.address_id = a.address_id JOIN city ON a.city_id = city.city_id
	JOIN country ON city.country_id = country.country_id;

CREATE VIEW sales_by_store AS
SELECT CONCAT(c.city, ',', cy.country) AS store, 
       CONCAT(m.first_name, ' ', m.last_name) AS manager, SUM(p.amount) AS total_sales
  FROM payment AS p
       INNER JOIN rental AS r ON p.rental_id = r.rental_id
       INNER JOIN inventory AS i ON r.inventory_id = i.inventory_id
       INNER JOIN store AS s ON i.store_id = s.store_id
       INNER JOIN address AS a ON s.address_id = a.address_id
       INNER JOIN city AS c ON a.city_id = c.city_id
       INNER JOIN country AS cy ON c.country_id = cy.country_id
       INNER JOIN staff AS m ON s.manager_staff_id = m.staff_id
 GROUP BY s.store_id
 ORDER BY cy.country, c.city;

CREATE VIEW sales_by_film_category AS
SELECT c.name AS category, SUM(p.amount) AS total_sales
  FROM payment AS p
       INNER JOIN rental AS r ON p.rental_id = r.rental_id
       INNER JOIN inventory AS i ON r.inventory_id = i.inventory_id
       INNER JOIN film AS f ON i.film_id = f.film_id
       INNER JOIN film_category AS fc ON f.film_id = fc.film_id
       INNER JOIN category AS c ON fc.category_id = c.category_id
 GROUP BY c.name
 ORDER BY total_sales DESC

/* sales_by_film_category(category,total_sales) */;
CREATE VIEW actor_info AS
SELECT a.actor_id, a.first_name, a.last_name,
       GROUP_CONCAT(DISTINCT CONCAT(c.name, ': ',
		     (SELECT GROUP_CONCAT(f.title,', ')
                    FROM film f
                    INNER JOIN film_category fc
                      ON f.film_id = fc.film_id
                    INNER JOIN film_actor fa
                      ON f.film_id = fa.film_id
                    WHERE fc.category_id = c.category_id
                    AND fa.actor_id = a.actor_id
                 )
             ), '; ') AS film_info
 FROM actor a
      LEFT JOIN film_actor fa ON a.actor_id = fa.actor_id
      LEFT JOIN film_category fc ON fa.film_id = fc.film_id
      LEFT JOIN category c ON fc.category_id = c.category_id
GROUP BY a.actor_id, a.first_name, a.last_name;
```
