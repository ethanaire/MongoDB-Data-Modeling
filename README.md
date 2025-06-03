# MongoDB Data Modeling

## Use Case Scenario 

We consider a relational database that holds the data of a chain of DVD stores; the database name is [Sakila](https://dev.mysql.com/doc/sakila/en/sakila-preface.html).

The Sakila database is serving an increasing number of queries from staff and customers around the world. A single monolithic database is not sufficient anymore to serve all the requests and the company is thinking about distributing the database across several servers (**horizontal scalability**). However, a relational database does not handle horizontal scalability very well, due to the fact that the data is scattered across numerous tables, as the result of the normalization process. Hence, the Sakila team is turning to you to help them migrate the database from PostgreSQL to MongoDB.

For the migration to happen, it is necessary to conceive a suitable data model. From the first discussions with the Sakila management, you quickly understand that one of the main use of the database is to manage (add, update and read) rental information.

## Note: 

The challenge's credit belongs to [Gianluca Quercini](https://gquercini.github.io/) 
