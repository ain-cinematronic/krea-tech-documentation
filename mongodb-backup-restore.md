# MongoDB Backup (from cloud) & Restore (to local)

A lean, explicit workflow for pulling a production (or staging) MongoDB database down to your local environment using separate **URI** and **DB name** variables. Written for PowerShell on Windows.

---

## 0) Define variables once

```powershell
$CloudUri = '<CLOUD_CONNECTION_URI>'
$CloudDb  = '<CLOUD_DB_NAME>'
$LocalUri = 'mongodb://127.0.0.1:27017'
$LocalDb  = 'payload-test'
```

Validate:

```powershell
echo $CloudUri
echo $CloudDb
```

---

## 1) Create timestamped dump folder

```powershell
$ts = Get-Date -Format "yyyyMMdd-HHmmss"
$dumpDir = "dump-$ts"
```

---

## 2) Dump cloud DB to local folder

```powershell
mongodump --uri="$CloudUri" --db="$CloudDb" --numParallelCollections=1 --out="$dumpDir"
```

Expected output includes lines like:

```
done dumping <DB_NAME>.users (N documents)
```

---

## 3) Restore into local Mongo

```powershell
mongorestore --uri="$LocalUri" --db="$LocalDb" --dir="$dumpDir\$CloudDb" --drop
```

Expected output ends with:

```
NNN document(s) restored successfully
```

---

## 4) (Optional) Verify inside running Mongo container

If you run Mongo in Docker and your container is named `payload-test-mongo`:

```powershell
docker exec payload-test-mongo mongosh --quiet --eval "db.getSiblingDB('$LocalDb').getCollectionNames()"
```

Count a collection:

```powershell
docker exec payload-test-mongo mongosh --quiet --eval "db.getSiblingDB('$LocalDb').users.countDocuments()"
```

---

## 5) Point local app at restored DB

```powershell
$env:DATABASE_URI = "$LocalUri/$LocalDb"
```

(Or add to your `.env.local` as `MONGODB_URI=$LocalUri/$LocalDb`.)

---

## 6) Quick sanity queries

List first 5 users (email + created date):

```powershell
docker exec payload-test-mongo mongosh --quiet --eval "db.getSiblingDB('$LocalDb').users.find({}, {email:1, createdAt:1}).limit(5).toArray()"
```

Check courses count (example collection):

```powershell
docker exec payload-test-mongo mongosh --quiet --eval "db.getSiblingDB('$LocalDb').courses.countDocuments()"
```

---

## Rationale / Safety

- Separating URI from DB names avoids accidental truncation of query params.
- `--drop` ensures a clean restore (no stale collections). Omit if you want to merge.
- Explicit collection count checks provide fast validation without Compass.
- `--numParallelCollections=1` avoids noisy throttling on some hosted tiers.

> Adjust naming (`payload-test-mongo`) if your Mongo container differs.

---
