WHY DOES ALTER TABLE TAKE SO MUCH TIME:->

mysql> ALTER TABLE student_info DROP PRIMARY KEY, ADD PRIMARY KEY (id);
Query OK, 0 rows affected (6.42 sec)

Metadata Changes: Dropping and adding a primary key involves significant changes to the table's metadata within the MySQL data dictionary. This includes updating information about constraints, indexes, and the overall structure of the table.

Index Operations (Even on an Empty Table):

Dropping the Primary Key: When you drop a primary key, MySQL needs to remove the associated index structure. Even if the table is empty, the system still has to perform the operations to dismantle this structure.
Adding the Primary Key: When you add a primary key, MySQL needs to enforce the constraints of a primary key (not NULL and unique). Even on an empty table, it prepares the infrastructure for this constraint and builds a new index to enforce it. This index creation process takes time, as the database needs to allocate space and set up the index structure.

So if we want to ALTER a large table with millions of rows, it is better to:
Create a temp table
Creates triggers on the first table (for inserts, updates, deletes) so that they are replicated to the temp table
In small batches, migrate data
When done, rename table to new table, and drop the other table

Ex:
CREATE TABLE main_table_new LIKE main_table;
ALTER TABLE main_table_new ADD COLUMN location VARCHAR(256);
INSERT INTO main_table_new SELECT *, NULL FROM main_table;
RENAME TABLE main_table TO main_table_old, main_table_new TO main_table;
DROP TABLE main_table_old;

You can also speed up the process by temporarily turning off unique checks and foreign key checks.

SET unique_checks = 0;
SET foreign_key_checks = 0;

The statement SET unique_checks = 0; in MySQL is used to disable checks for unique constraints 
(including primary keys and other UNIQUE indexes) during certain operations
Disables Uniqueness Checks: When you set unique_checks to 0, MySQL will not verify that new rows being inserted or imported violate any unique constraints defined on the table. This means you could potentially insert duplicate values into columns that are supposed to be unique

