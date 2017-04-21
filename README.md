# postgres
# Postgres PITR Backup & Restore Demo Configuration

## Postgres WAL Archiving 
1. Create the WAL & backup Directory and add necessary Permisson 

```
bash~ mkdir -p  /var/lib/pgsql/wals/
bash~ mkdir -p  /var/lib/pgsql/bac/
bash~ chown postgres:postgres -R /var/lib/pgsql/backups/`
```

2. Edit the postgresql.conf with postgres user and add the below changes

```
wal_level=archive
archive_mode=on
archive_command = 'test ! -f /var/lib/pgsql/wals/%f && cp %p /var/lib/pgsql/wals/%f'
```

3. Restart the database

`bash~ service postgresql restart`

4. Prepare for the basebackup 

```
su - postgres
psql -c "SELECT pg_start_backup('label');" postgres 
tar -C /var/lib/pgsql/data/ -czvf /var/lib/pgsql/backups/pg_basebackup_backup.tar.gz .
psql -c "SELECT pg_stop_backup();" postgres
```
## Add Data to PostgreSQL
```
bash~  su - postgres
bash~  createdb mydb
bash~  psql -s mydb
```

## Testing Restore 
1. Remove the data folder 

```
bash~ cd /var/lib/pgsql/
bash~ rm -rf data/
```
2. Initilaize the Database
```
bash~ /etc/init.d/postgresql start
```

## Restore from WAL Archiving 
1. Extract the Basebackup...

```
bash~  tar xvf /var/lib/pgsql/backups/pg_basebackup_backup.tar.gz -C /var/lib/pgsql/data
```

2. Create recovery.conf with the below content 
```
bash~ cd /var/lib/pgsql/data
bash~ cat recovery.conf 
restore_command = 'cp /var/lib/pgsql/wals/%f %p'
```

3. Upon completion of the recovery process, the server will rename recovery.conf to recovery.done

```
bash~ cd /var/lib/pgsql/data
bash~ cat recover.done
```


