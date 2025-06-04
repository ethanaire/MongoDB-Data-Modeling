| Table         | Description | Columns |
| :-----------: | ----------- | ------- |
| actor         | information for all actors | actor_id, first_name, last_name |
| adress        | address information for customers, staff, stores | address_id, address, address2, district, city_id, postal_code, phone |
| category      | categories that can be assigned to a film | category_id, name |
| city          | a list of cities | city_id, city, country_id                     |
| country       | a list of countries | country_id, country                        |
| customer      | a list of all customers                                          | customer_id, store_id, first_name, last_name, email, address_id, active, create_date |
| film          | a list of all films potentially in stock in the stores           | film_id, title, description, release_year, language_id, original_language_id, rental_duration, rental_rate, length |
| film_actor    | support a many-to-many relationship between films and actors     | film_id, actor_id |
| film_category | support a many-to-many relationship between films and categories | film_id, category_id |
| inventory     | copy of a given film in a given store                            | inventory_id, film_id, store_id |
| language      | possible languages that films follow                             | language_id, name |
| payment       | payment made by a customer                                       | payment_id, customer_id, staff_id, rental_id, amount, payment_date |
| rental        | each rental of each inventory item                               | rental_id, rental_date, inventory_id, customer_id, return_date, staff_id |
| staff         | lists all staff members                                          | staff_id, first_name, last_name, address_id, picture, email, store_id, active, username, password |
| store         | lists all stores in the system                                   | store_id, manager_staff_id, address_id |
