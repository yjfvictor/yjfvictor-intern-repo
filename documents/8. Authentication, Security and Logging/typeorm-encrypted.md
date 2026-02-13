# Reflection: Using typeorm-encrypted for Data Encryption

## Project Summary

The project [test-repo-for-using-typeorm-encrypted-for-data-encryption](https://github.com/yjfvictor/test-repo-for-using-typeorm-encrypted-for-data-encryption) is a small NestJS application that demonstrates field-level encryption for sensitive data in PostgreSQL using the **typeorm-encrypted** package. It provides a `SensitiveData` entity whose `secret` column is encrypted on write and decrypted on read via a TypeORM column transformer. Keys are read from the environment. The application loads a `.env` file at startup with `dotenv`, and the test setup uses the same keys from `.env` when present so that tests and the app share one configuration. When `NODE_ENV` is not `production`, TypeORM creates the `sensitive_data` table on startup so no migrations are required for local use. The project includes unit tests for the encryption config, the transformer round-trip, and the service (including validation of request body and UUID, and handling of decryption failures), an e2e test against PostgreSQL (e.g. in Docker), a `docker-compose.yml` for PostgreSQL 16, and a README with flowcharts and instructions for running and testing.

---

## Why Focus Bear Double Encrypts Sensitive Data

Focus Bear double encrypts sensitive data instead of relying only on database encryption so that exposure is limited even if one layer is compromised. Database encryption at rest protects the whole database (files, backups) from someone who gets disk or backup access but not database credentials. It does not protect against anyone who can run queries (e.g. an attacker with DB credentials, or a compromised application). Field-level encryption with typeorm-encrypted means the application encrypts specific columns before they are written. The database only ever sees ciphertext for those columns. So even with full database access, an attacker cannot read the plaintext without the application-level encryption key. In short, database encryption protects the storage layer, and field-level encryption protects the data itself for chosen columns.

---

## How typeorm-encrypted Integrates with TypeORM Entities

typeorm-encrypted plugs into TypeORM via the **column transformer** API. On an entity you define a column and pass a `transformer` option to `@Column()`. The transformer is an object with two methods: `to(value)` (called when writing the entity to the database) and `from(value)` (called when reading a row into an entity). The package provides `EncryptionTransformer`, which in `to()` encrypts the plaintext (e.g. with AES-256-CBC) and in `from()` decrypts the stored string back to plaintext. You configure it with a key, algorithm, IV or IV length, and optionally a fixed IV. In the test project, the entity imports a config helper that reads `ENCRYPTION_KEY` (and optionally `ENCRYPTION_IV`) from the environment and passes that config into `new EncryptionTransformer(...)`. No changes are needed in repositories or services: they work with plaintext, and TypeORM calls the transformer when persisting or loading the entity.

---

## Best Practices for Securely Managing Encryption Keys

- **Do not store keys in code or in the repo.** Use environment variables or a secret manager (e.g. HashiCorp Vault, cloud provider secrets) so that keys are injected at deploy time.
- **Use strong, random keys.** For AES-256 the key must be 32 bytes (64 hex characters). Generate with a cryptographically secure source (e.g. `crypto.randomBytes(32).toString('hex')` in Node).
- **Restrict access.** Only the application (and deployment/secrets pipeline) should have access to the encryption key. Limit who can read production env or secret manager entries.
- **Rotate keys when needed.** Key rotation requires reading data with the old key and re-writing with the new key, then updating the app to use the new key. Plan migrations for that.
- **Separate test keys from production.** In the test repo, Jest setup sets fixed test keys so tests do not depend on production env. Production must use a different key from a secure store.

---

## Trade-offs Between Database-Level and Application-Level Encryption

**Database-level (e.g. TDE, encryption at rest):**  
Pros: transparent to the application, one place to configure, protects all data on disk and in backups. Cons: anyone with valid DB access (or a dump) can read all data in plaintext. Key is often managed by the DB or infrastructure team.

**Application-level (e.g. typeorm-encrypted):**  
Pros: only the application has the key, so DB dumps or stolen DB credentials do not reveal plaintext for encrypted columns. You can encrypt only the most sensitive fields. Cons: the app must manage keys and handle rotation, you cannot run SQL that compares or searches on encrypted column values (e.g. `WHERE secret = 'x'`), and performance is slightly higher per read/write because of encrypt/decrypt in the app.

In practice, using both (as Focus Bear does) gives defence in depth: encryption at rest for the whole DB and field-level encryption for the most sensitive columns.
