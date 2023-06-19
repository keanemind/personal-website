---
title: Using Version 5 UUIDs for Idempotency
date: 2023-06-19T07:09:24.160Z
draft: false
---
I recently discovered version 5 UUIDs and have been using them all over the place whenever I find that I want deterministically-generated ids. Version 5 allows me to hash a string and get a UUID out, which I can provide to Postgres as the primary key of some row that I'm inserting. This means I can make code, like scripts which generate and insert data into the database, idempotent by simply relying on the primary key's unique constraint.

For example, let's say I have a CSV of email addresses and I'm writing a script to insert new rows into a users table from that CSV. The users table has an email column, but it doesn't have a unique constraint on it, for some extraneous reason related to the business requirements of the rest of the application. In that script, I'll pass `${scriptName}-${csvRow.email}` as the name argument to the version 5 UUID function, and then use the resulting UUID as the primary key of each row that I insert. Now the script is automatically idempotent; if I get a new CSV, or my existing one got updated, and I run the script again, duplicate user rows won't be made. 

Another use-case I've found is for setting up test data. I prefer writing tests that actually make queries to a local database rather than mocking the database layer. When inserting rows into the database to setup for tests, I can easily assign foreign keys by using the same hash input instead of copy-pasting hard-coded UUIDs or having to look them up. And as an extra bonus, my test code becomes more assertive because I get what is effectively a virtual unique constraint.

In one of the projects I'm working on right now, every test works with completely different rows than all the other tests; each test inserts its own rows, and the primary keys for those rows are version 5 UUIDs where the test name itself is part of the hash input. Because tests don't share data, I don't do any teardown or resetting of the database state between tests. And after tests finish running, I can open a SQL client to inspect the database state resulting from a particular test. Of course, this won't work for every project, depending on the kinds of queries being used.

I only recently started using version 5 UUIDs this way, so there may be flaws in these approaches that I simply haven't encountered yet. I'll keep at it for the time being, and if the day comes that I find good reasons to stop, then you can expect a new entry here.