# Rock Gym Liquibase Database Migrations

This project manages database schema changes for the Rock Gym application using Liquibase.

## Prerequisites

- Java 17 or higher
- Maven 3.6+
- PostgreSQL database running on localhost:5432
- Database name: `rockgym`
- Database user: `postgres` / password: `root`

## Project Structure

```
rock-gym-liquibase/
├── pom.xml
├── src/
│   └── main/
│       └── resources/
│           ├── liquibase.properties
│           └── db/
│               └── changelog/
│                   ├── db.changelog-master.xml
│                   └── changes/
│                       └── (individual changelog files)
```

## Common Commands

### 1. Generate Changelog from Existing Database

Generate a complete changelog from your current database schema:

```bash
mvn liquibase:generateChangeLog
```

This will create `generated-changelog.xml` with all existing tables, columns, constraints, etc.

### 2. Update Database

Apply pending changelogs to the database:

```bash
mvn liquibase:update
```

### 3. Rollback Changes

Rollback the last changeset:

```bash
mvn liquibase:rollback -Dliquibase.rollbackCount=1
```

Rollback to a specific tag:

```bash
mvn liquibase:rollback -Dliquibase.rollbackTag=version-1.0
```

### 4. Generate Diff Changelog

Compare current database with another and generate diff:

```bash
mvn liquibase:diff
```

### 5. Validate Changelog

Validate changelog syntax:

```bash
mvn liquibase:validate
```

### 6. Check Database Status

See which changesets have been applied:

```bash
mvn liquibase:status
```

### 7. Clear Checksums

Clear all checksums (useful after manual edits):

```bash
mvn liquibase:clearCheckSums
```

## Workflow

### Initial Setup (First Time)

1. **Generate baseline from existing database:**
   ```bash
   mvn liquibase:generateChangeLog
   ```

2. **Review the generated changelog:**
   - Check `src/main/resources/db/changelog/generated-changelog.xml`
   - Split into logical files if needed

3. **Include in master changelog:**
   - Edit `db.changelog-master.xml`
   - Add include statement for generated changelog

4. **Mark as applied (since database already exists):**
   ```bash
   mvn liquibase:changelogSync
   ```

### Adding New Changes

1. **Create a new changelog file:**
   ```bash
   # Example: src/main/resources/db/changelog/changes/002-add-new-table.xml
   ```

2. **Add changeset:**
   ```xml
   <changeSet id="002-add-new-table" author="developer">
       <createTable tableName="new_table">
           <column name="id" type="bigint" autoIncrement="true">
               <constraints primaryKey="true" nullable="false"/>
           </column>
           <column name="name" type="varchar(255)">
               <constraints nullable="false"/>
           </column>
       </createTable>
   </changeSet>
   ```

3. **Include in master changelog:**
   ```xml
   <include file="db/changelog/changes/002-add-new-table.xml"/>
   ```

4. **Apply changes:**
   ```bash
   mvn liquibase:update
   ```

## Best Practices

1. **Never modify applied changesets** - Always create new changesets for changes
2. **Use meaningful IDs** - Format: `{number}-{description}` (e.g., `001-initial-schema`)
3. **One logical change per changeset** - Makes rollback easier
4. **Always include rollback** - Define rollback for each changeset when possible
5. **Test in dev first** - Always test migrations in development before production
6. **Use contexts** - Tag changesets with contexts (dev, test, prod) for environment-specific changes
7. **Version control** - Commit all changelog files to Git

## Changelog File Naming Convention

```
001-initial-schema.xml
002-add-user-table.xml
003-add-indexes.xml
004-add-constraints.xml
```

## Example Changeset

```xml
<changeSet id="005-add-email-column" author="john.doe">
    <addColumn tableName="employee_detail">
        <column name="email" type="varchar(255)">
            <constraints nullable="true"/>
        </column>
    </addColumn>
    <rollback>
        <dropColumn tableName="employee_detail" columnName="email"/>
    </rollback>
</changeSet>
```

## Troubleshooting

### Issue: Checksum validation failed

**Solution:**
```bash
mvn liquibase:clearCheckSums
```

### Issue: Database is not up to date

**Solution:**
```bash
mvn liquibase:status
mvn liquibase:update
```

### Issue: Need to mark changelog as applied without running

**Solution:**
```bash
mvn liquibase:changelogSync
```

## Configuration

Edit `src/main/resources/liquibase.properties` to change:
- Database connection details
- Schema names
- Output file locations
- Logging levels

## Integration with Main Application

Once Liquibase is managing your schema:

1. **In rock-gym application:**
   - Change `spring.jpa.hibernate.ddl-auto=update` to `none`
   - Add Liquibase dependency
   - Configure to use these changelogs

2. **Or keep separate:**
   - Run Liquibase migrations manually before deploying application
   - Keep schema management independent

## Resources

- [Liquibase Documentation](https://docs.liquibase.com/)
- [Liquibase Best Practices](https://www.liquibase.org/get-started/best-practices)
- [Liquibase Maven Plugin](https://docs.liquibase.com/tools-integrations/maven/home.html)
