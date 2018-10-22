
# Table Relationships Review


## Introduction

In this lesson, we'll review the various kinds of table relationships, and work through real-world examples of each kind. 

## Objectives

You will learn how to:

* Understand and Identify one-to-one relationships
* Understand and Identify one-to-many relationships
* Understand and Identify many-to-many relationships
* Understand and explain primary keys and foreign keys

### Normalization in Databases

You've probably noticed by now that tables in databases are essentially just spreadsheets.  However, there's a reason that companies spend millions of dollars on building and maintaining performant relational databases, instead of just keeping a massive spreadsheet in Excel.  The main advantage that databases provide are a robust way to organize our tables--these organization strategies are referred to as **_normalization_**. There are different levels of normalizations, ranging from _0th normal form_ to _3rd normal form_ (there are more normalized versions than 3rd normal form, but they're rare enough that you likely won't have to worry about them). 

#### What is Database Normalization?

Database normalization refers to the practice of storing data across one or more tables based on the information that data contains. You might have noticed that although we typically jam everything into a single DataFrame when exploring or modeling our data, things like a customer's name, address, and order history are typically stored in separate tables, and that each of those corresponding tables only contains data pertaining to a specific topic (e.g. all addresses stored in the "addresses" table, orders stored in the "orders" table, etc).  The information is linked to other relevant records by the use of keys.

#### Benefits of Database Normalization

You've probably wondered at some point--**_why bother_**?  Storing data in separate tables and then joining with foreign keys when that information is needed can be a bit time consuming.  However, Database Normalization provides some great benefits:

#### 1. Minimize Duplicate Data

By storing data in separate tables, we can avoid the need to store duplicate data in our database.  Duplicating records is a waste of space, and space used to be quite expensive!  By storing a customer's information in a "Customer" table, we can just reference the appropriate key for that customer in an "Orders" table every time that customer places an order. In a denormalized database to track orders (technically called 0th normal form), every row in the spreadsheet would be an order, and if a customer has placed 20 orders, then you'll have that customer's name, shipping address, and other information repeated across 20 different rows! With a normalized database, we can save on memory by just pointing to that customer in the customer table every time they make an order. 

#### 2.  Minimize Data Modification Issues

Another benefit of a normalized database is that it makes it much easier to avoid issues that arise from modifying information.  Consider the example we used above of a single spreadsheet storing information about orders and the customers that placed them.  Let's assume that our customer changes their address.  The simplest solution here would be to just put the new address in only on new orders, and leave the old address alone in the old orders.  But what happens when you try to query for that customer's address? That query will return two different addresses for that customer, with no obvious way for you to tell which address is current.  If we decide to change all instances of that customer's address to match the current one, then that leads to a performance problem--making that one change of address means changing it for every single row in our spreadsheet, would slow down our database. 

By normalizing a database, we can plan ahead for issues like this.  With an "Address" table, perhaps we can add datestamps to each address to tell when it is changed, or a column allowing the customer to name the different addresses they ship items to.  Best of all, if we decide to change a value, we only need to change it in one place--whether that customer has placed one order or one million, we're only changing a single cell in a single table--no performance hit!

#### 3. Simplifying Queries

This was alluded to in the previous paragraphs, but one of the main benefits of database normalization is that it simplifies the structure of our queries when we need to get information. 

For instance, let's assume we have a spreadsheet containing information on sales associates in our company. Each row represents a different member of the sales team. Each customer that the associate has dealt with is stored as a different column in the speadsheet under headings such as "Customer_1", "Customer_2", etc.  Some companies have been with us a long time, and have dealt with multiple sales associates.  What if we wanted to query our data to get all the sales associates that have ever sold anything to IBM?

Our query would be horrible, and would look something like:

```sql
SELECT SalesAssociate FROM SalesTeam
WHERE Customer_1 = 'IBM' OR
Customer_2 = 'IBM' OR
Customer_3 = 'IBM' OR... // continues on like this for every customer column :-(
```

This becomes much, much simpler when we use a normalized database.  We can just store all of our sales associate data in one table, all of our customer information in another table, and link them together with a join table (since this is a many-to-many relationship).  This greatly simplifies our query. 

#### Types of Normal Forms

Although there are more strict types of normalization such as 4th and 5th normal form, in practice, you'll rarely ever run into database stored in versions other than 1st, 2nd, or 3rd normal form.  Since you're a data scientist, not a database administrator, you don't need to spend too much time worrying about the differences between the 3--however, you should have a basic understanding of what each means. 

**_1st Normal Form_**: All rows have the same number of columns. No column names are repeated. 

| Sales_Associate |  Name    | Team | Office_Location | Office_Address |
|-----------------|----------|------|-----------------|----------------|

**_2nd Normal Form_**: Meets the specifications of 1st normal form, plus all column data depends on the entire primary key, and not just part (remember, primary keys can be a composite of 2 or more columns in a table!)

Our example above meets specs for 1st normal form, but not 2nd normal form.  This is because the Column `Office_Address` contains information about the office, not the sales associate themselves.  To meet 2nd normal form, it would look like this:

Table 1

| Sales_Associate |  Name    | Team | Office_Location |
|-----------------|----------|------|-----------------|

Table 2

| Office_Location | Office_Address |
|-----------------|----------------|

**_3rd Normal Form_**: Meets the specifications of 2nd normal form, plus no column depends on other columns. Each column in the table depends on the primary key, the whole primary key, and nothing but the primary key. 

Table 1

| Sales_Associate | Name |
|-----------------|------|

Table 2 (Note: Team member is foreign key to the Sales_Associate table, they just have different names)

| Team | Team_Member | Office_Location | 
|------|-------------|-----------------|

Table 3 

| Office_Location | Office_Address |
|-----------------|----------------|

Note that if we needed to get the office address of a particular associate, we can do this because all the tables are linked by foreign keys. 

Often, you'll need to get data out of databases where it is stored in 3rd normal format, and then denormalize the data you need so that it fits in a single DataFrame we can use for all our data science-y purposes. 


## Table Relationships

Recall that entities stored in our database tables can be related to one or more entities in other tables. Before we move onto reading Entity-Relationship diagrams in the next lesson, we'll quickly review the different types of relationships, and provide an example of each. 


### One-to-One Relationships

In one-to-one relationships, an entity in a table is connected to exactly one entity in a corresponding table through a foreign key. 

**_Example_**: Employee and Compensation. Each employee will only have one row in the compensation table related to them. 

### One-to-Many Relationships

In one-to-many relationships, an entity in a table can be connected to one or more entities in a corresponding table through a foreign key. 

**_Example_**: City to Zip Code. A city can contain multiple zip codes, but each zip code is only in one city. 


### Many-to-Many Relationships

In many-to-many relationships, an multiple entities in a table can be connected to one or more of the same entities in a corresponding table.  These connections are queried through an intermediate table called a **_Join Table_** (more on this in a future lesson!)

**_Example:_** Sales Associates and Customers.  In our previous normalization example, each sales associate deals with multiple companies, and each company deals with multiple sales associates. 
