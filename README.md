# MongoDB Data Modeling

## Use Case Scenario:

We consider a relational database that holds the data of a chain of DVD stores; the database name is [Sakila](https://dev.mysql.com/doc/sakila/en/sakila-preface.html).

The Sakila database is serving an increasing number of queries from staff and customers around the world. A single monolithic database is not sufficient anymore to serve all the requests and the company is thinking about distributing the database across several servers (**horizontal scalability**). However, a relational database does not handle horizontal scalability very well, due to the fact that the data is scattered across numerous tables, as the result of the normalization process. Hence, the Sakila team is turning to you to help them migrate the database from PostgreSQL to MongoDB.

For the migration to happen, it is necessary to conceive a suitable data model. From the first discussions with the Sakila management, you quickly understand that one of the main use of the database is to manage (add, update and read) rental information.

## Task Flows: 

1. Evaluate initial relational model. 
2. Denormalization in MongoDB and Considerations for a new model.
3. Create a complete model of the Sakila database.

## Initial Relational Model:

The existing data model:
<p align="center">
  <img title="Relational Model" alt="Alt text" src="/assets/sakila_logical_schema.png" width="600" height="349">

Description of the tables can be found in [data dictionary](data/data-dictionary.md)

## New Model Considerations:

A MongoDB document is represented as a JSON record. However, internally MongoDB serializes the JSON record into a BSON record. In practice, a BSON record is a binary representation of a JSON record.

Denormalization in MongoDB is strongly encouraged to read and write a record relative to an entity in one single operation. 

Suppose we have in our database: 
- 512 customers.
- 16384 rentals.
- On average, each customer has around 32 rentals.
  
Therefore, we consider 2 following options: 
1. A collection `customer`, where each document contains information about a customer and an embedded list of all rentals made by the customer.
   
    - Fields from relational schema:
        - 4 integers: `rental_id`, `inventory_id`, `customer_id`, `staff_id`
        - 2 dates: `rental_date`, `return_date`
    - Assumed Sizes:
        - Integer: ~ 12 bytes
        - Date: ~ 8 bytes
        - Field name overhead: ~ 5–10 bytes per field
        - SON document overhead: ~ 16 bytes
    - Estimated Size per Document: ~ 154 bytes

    With each document:
    - Contains one customer (**203 bytes**).
    - Contains a list of **32 embedded rentals** (154 bytes each).
    
    → **Document size** = 203 + 32 x 154 = **5131 bytes** 
   
    → **Collection size** = 512 x 5131 = **2,626,272 bytes** ~ **2.63 MB**  

3. A collection `rental`, where each document contains information about a rental and an embedded document about the customer that made the rental.
   
    - Fields from relational schema:
        - 3 integers: `store_id`, `address_id`, `customer_id`
        - 3 strings: `first_name`, `last_name`, `email`
        - 1 boolean: `active`
        - 1 date: `create_date`
    - Assumed Sizes:
        - First name: 6 characters → 6 × 1.5 = 9 bytes
        - Last name: 8 characters → 12 bytes
        - Email: 20 characters → 30 bytes
        - String field overhead: ~10 bytes per field
        - Integer: 12 bytes
        - Boolean: 1 byte + field overhead
        - Date: 8 bytes
        - SON document overhead: ~ 16 bytes
    - Estimated Size per Document: ~ 203 bytes

    With each document:
    - Contains one rental (154 bytes).
    - Contains one customer (**203 bytes**).
    
    → **Document size** = 154 + 203 = **357 bytes**

    → **Collection size** = 16,384 x 357 = **5,849,088 bytes** ~ **5.85 MB**

**_Comparison of 2 Options_**: 
- `customer` collection with embedded rentals:
  
  ✅ Lower total size with fewer documents will guarantee efficient retrival.
  
  ❌ Document size limitation (MongoDB’s 16 MB limit), Update complexity (new rental requires updating the entire customer document), Harder partial updates.
- `rental` collection with embedded customer:

  ✅ Scalability and flexibility in querying rentals by different dimensions with simple writes. 

  ❌ Data redundancy (repeated rental doc repeats customer info), inconsistencies in updating data.

Based on the [logical schema](assets/sakila_logical_schema), a `rental` document does not only need to include information on the customer, but also `staff`, `inventory`, `payment`.

→ Using `rental` collection as the main collection is more beneficial than `customer`: 
  - It scales better with a large number of rentals.
  - Embedding related info like `staff`, `inventory`, and `payment` is more natural at the `rental` level (they are tightly coupled with the rental event).
  - Avoids giant customer documents that would be hard to update or query.

## Data Model in MongoDB:

A **complete schema** of a `rental` document is defined [here](rental_schema.json), specifically: 
- We embed the `customer` object with full detail (as decided earlier).
- For `staff`, `inventory`, and `payment`, we **only use identifiers** to avoid storage bloat and duplication. Here's why:
  - A `staff` document is **~65 KB** (too large to embed repeatedly).
  - Inventory and payment documents are smaller, but embedding their full data can still lead to redundancy and consistency issues if reused elsewhere.

Another point of consideration is to calculate the **total storage size** of the database using the chosen schema for the `rental` collection: 
- Rental base fields ~ 20 bytes
- Customer info ~ 72 bytes 
- Staff member info ~ 64 KiB (65,536 bytes), because we store a profile picture
- Inventory item info = 12 bytes
- Payment info = 20 bytes 

→ Total estimated size per `rental` document = **65,660 bytes**

Including **full embedded documents** (especially staff with profile pictures) causes storage to **grow dramatically**. Therefore, full denormalization consumes a lot of space, especially when the same data is repeated in many documents. 

To save space in this adopted solution, we can: 
- Reference large or shared documents using **IDs** (applied to staff/inventory/payment info).
- Embed only **essential or frequently-accessed data** (store info in a separate collection)
- Using a **hybrid approach** to balance space and performance. 



## Note: 

The challenge's credit belongs to [Gianluca Quercini](https://gquercini.github.io/) 
