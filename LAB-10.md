## KVM

```bash
virsh shutdown DB1
virsh shutdown DB2
```

```bash
virsh snapshot-create-as --domain DB1 --name clean
virsh snapshot-create-as --domain DB2 --name clean
```

```bash
virsh snapshot-list --domain DB1
virsh snapshot-list --domain DB2
```

### DB1

```bash
dnf install -y @postgresql:13
postgresql-setup initdb
```

```bash
su - postgres
psql -c 'ALTER ROLE postgres PASSWORD Pa$$w0rd'
```

```bash
vim /var/lib/pgsql/data/pg_hba.conf
```

```
host    replication     postgres        <REPLICA_IP>/32          trust
````

```bash
vim /var/lib/pgsql/data/postgresql.conf
```

```ini
listen_addresses = '*'
wal_level = hot_standby
archive_mode = on
archive_command = 'cd .'
max_wal_senders = 8
hot_standby = on
```

```bash
systemctl emanle --now postgresql
```

### DB2

```bash
su - postgres
pg_basebackup -h 10.20.20.11 -U postgres -p 5432 -D /var/lib/pgsql/data -Fp -Xs -P -R
exit
```

```bash
systemctl enable --now postgresql
```

### DB1

```bash
su - postgres
psql -c "CREATE TABLE test (id INT, name TEXT);"
psql -c "INSERT INTO test (id, name) VALUES (1, 'test 1');"
psql -c "INSERT INTO test (id, name) VALUES (2, 'test 2');"
psql -c "INSERT INTO test (id, name) VALUES (3, 'test 3');"
```

### DB2

```bash
su - postgres
psql -c "SELECT * FROM test;"
```

## KVM

```bash
virsh shutdown DB1
virsh shutdown DB2
```

```bash
virsh snapshot-revert --domain DB1 --snapshotname clean
virsh snapshot-revert --domain DB2 --snapshotname clean
```

```bash
virsh snapshot-delete --domain DB1 --snapshotname clean
virsh snapshot-delete --domain DB2 --snapshotname clean
```

```bash
virsh start DB1
virsh start DB2
```