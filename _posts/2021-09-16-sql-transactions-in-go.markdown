---
layout: post
title: SQL Transactions in Go
date: 2021-08-31
---

# SQL Transactions in Go

There are several advantages into using prepared statements and transactions in SQL, in my case mostly with  MariaDB. I will explain some of the advantages, some of the caveats and handy tips to manage them, specifically with Go.

## Useful Resouces

[More information about transactions specifically for MariaDB.](https://mariadb.com/kb/en/transactions/)
[More information about transactions](https://dev.mysql.com/doc/refman/8.0/en/commit.html)
[Official SQL package for working with Go](https://pkg.go.dev/database/sql)

 
## Why SQL Transactions ?

We want our unit of work to be reliable and consistent, even in case of system failure.

We want to provide isolation between programs that access the database concurrently.

When you start an explicit transaction and issue a DML (Data Manipulation Language) statement, the resources being locked by the statement remain locked, and the results of statement are not visible from outside the transaction until you manually commit or rollback it.

This is what you may or may not need.

The InnoDB storage engine, which is my default go-to database storage engine and the one you should be going to for relational data, supports ACID-compliant transactions. SQL transactions are also a great way to avoid data races in the database engines, and to run queries against a cluster of databases, while preserving data consistency and integrity, a performant way to do batch operations, a way to avoid SQL injections into the database, and more.
ACID properties

In order to achieve these 2 goals, a database transaction must satisfy the ACID properties, where:

* A is Atomicity, which means either all operations of the transaction complete successfully, or the whole transaction fails, and everything is rolled back, the database is unchanged.

* C is Consistency, which means the database state should remains valid after the transaction is executed. More precisely, all data written to the database must be valid according to predefined rules, including constraints, cascades, and triggers.

* I is Isolation, meaning all transactions that run concurrently should not affect each other. There are several levels of isolation that defines when the changes made by 1 transaction can be visible to others.

* D is Durability. It basically means that all data written by a successful transaction must stay in a persistent storage and cannot be lost, even in case of system failure.

How to run a SQL DB transaction from a SQL shell or Navicat / DataGrip?

It’s pretty simple:

* We start a transaction with the `BEGIN` statement.

* Then we write a series of normal SQL queries (or operations).

* If all of them are successful, We `COMMIT` the transaction to make it permanent, the database will be changed to a new state.

* Otherwise, if any query fails, we `ROLLBACK` the transaction, thus all changes made by previous queries of the transaction will be gone, and the database stays the same as it was before the transaction.

## Using Transactions with Go

There are several reasons why you should prefer using the `COMMIT`, `ROLLBACK` and others in programmatic and type safe Go. 

This gives you the opportunity to create prepared statements, execute them in a loop and then commit. If you were to not use transactions, and had to do 1000 `INSERT` statements, you would need 1000 database connections opening and closing one after the other, for single `EXEC` queries, while with prepared statements you would leave an open tunnel for communication from your application and the database. This obviously is much faster and brings less load on all services. 

We always recommend you do prepared statements.

We always recommend you use prepared statements in a SQL transaction when executing more than one `INSERT` / `UPDATE` / `DELETE` statement to the database.



## Rules of thumb

* Always remember to defer `stmt.Close()` - assuming your prepared statement is called `stmt` - as soon as possible.

* If you have started a transaction and you need to stop execution or return from the function, **ALWAYS** make sure to `tx.Rollback()` otherwise the resources allocated for the transaction will never be de-allocated.

* Leaving transactions open without committing or rolling back exhausts the database connection and you might get an unresponsive database. This has already occurred and caused several issue. Make sure you always either rollback or commit. 

* Assuming you called your transaction variable that resulted from `db.Begin()` as `tx`, you should only skip issuing a `tx.Rollback()` if you are certain that the `tx.Commit()` has been executed successfully.

* Always make sure to defer `rows.Close()` if working with rows as soon as possible.


## Real life transaction in Go

Here you can see some production transactions that I have slightly altered for this purpose.

```go
// AddressBulkInsertion will perform a transaction to insert addresses in bulk
func AddressBulkInsertion(db *sql.DB, addresses []Address) error{
	tx, err := db.Begin()
	if err != nil {
		_ = tx.Rollback()
		return err
	}

	// here should be your SQL, never mind the naming here
	insertStmt, err := tx.Prepare(`
	INSERT INTO addresses (uid, address_data, created_at, updated_at) 
	VALUES (?, ?, ?, ?) ON DUPLICATE KEY 
	    UPDATE address_data = ?, updated_at = ?;
	`)
	if err != nil {
		_ = tx.Rollback()
		return err
	}

	defer insertStmt.Close()

	for i := range payload {
		now := time.Now().Unix()

		if _, errr := insertStmt.Exec(
			uuid.NewString(),
			address.addressData,
			now,
			now,
		); errr != nil {
			// you might be interested in continuing the loop
			// even when some error occurs, perhaps due to 
			// bad data or inconsistencies and you try to do as good as possible
			log.Println(errr.Error())
			continue

			// otherwise simply rollback and return
			// if you want the entire bulk job to stop
			_ = tx.Rollback()
			return errr
		}
	}

	return tx.Commit()
}
```