# Reflection: Seeding and Migrations in TypeORM

## Project summary

The [test-repo-for-seeding-and-migrations-in-TypeORM](https://github.com/yjfvictor/test-repo-for-seeding-and-migrations-in-TypeORM) project shows how to manage database schema changes and seed initial data using TypeORM. It includes two migrations (creating the `item` table and adding a `quantity` column), a seed script that uses TypeORM repositories to insert sample items, a minimal NestJS API to read items, and Docker Compose for PostgreSQL 15 Alpine.

## What is the purpose of database migrations in TypeORM?

Database migrations in TypeORM are a way to change the database schema in a controlled, repeatable way. Each migration is a file that defines an `up` method (to apply the change) and a `down` method (to undo it). Migrations let you add or drop tables and columns, change types, and run custom SQL in a order that is tracked by TypeORM. This keeps the schema in sync across environments and avoids relying on `synchronize: true`, which is unsafe when you have real data. In short, migrations give you version-controlled, reversible schema updates that you can run in production safely.

## How do migrations differ from seeding?

Migrations change the **structure** of the database (tables, columns, indexes, constraints). They are run by the TypeORM CLI (for example `migration:run` and `migration:revert`) and are stored as files, usually in a `migrations` folder. Seeding fills the database with **data**, such as default values, reference data, or test data. In this project, seeding is done by a script that uses the TypeORM `Repository<Item>` to create and save sample items. Migrations run first so the tables exist, then you run the seed script to insert data. So migrations define the schema, seeding fills it with content.

## Why is it important to version-control database schema changes?

Version-controlling migrations keeps the schema aligned with the application code and makes changes auditable and reproducible. You can see what changed, when, and in what order. New environments (or other developers) can apply the same migrations in the same order to get the same schema. If something goes wrong, you can trace it back to a specific migration. Without version control, schema changes are ad hoc and hard to repeat, which leads to drift between environments and makes rollback and debugging difficult.

## How can you roll back a migration if an issue occurs?

TypeORM supports rolling back the **last** applied migration with `migration:revert`. That command runs the `down` method of the most recently executed migration (for example dropping a column or table). To undo more than one migration, you run `migration:revert` repeatedly, one time per migration, in reverse order. TypeORM does not support reverting a specific migration by name or out of order. If you need to undo an older change, the usual approach is to add a new migration that fixes the schema going forward rather than reverting past migrations. In this project you run `npm run migration:revert` to revert the latest migration.
