# peertubeRandomSortTrick
- peertube has not (yet) a random sort feature for the local videos list.
- this trick adds local video list random sort, using originallyPublishedAt field/column.
- modification is only in the postgresql database.
- uses pg_cron to randomize order in the video list regularly.

## Create a function in the database for random timestamps

```
su <peertube_user>

psql <peertube_base>

<peertube_base>=# CREATE OR REPLACE FUNCTION get_unsorted() RETURNS TIMESTAMP as $$ SELECT random() * ('2020-02-01 00:00:00'::timestamp - '1980-01-01 00:00:00'::timestamp) + '1980-01-01 00:00:00'::timestamp $$ language sql; 

<peertube_base>=# GRANT EXECUTE ON FUNCTION get_unsorted TO postgres;
<peertube_base>=# GRANT EXECUTE ON FUNCTION get_unsorted TO postgres;

<peertube_base>=# UPDATE video SET "originallyPublishedAt" = get_unsorted()::timestamp;
```

So now sorting by 'original published date' will display local videos randomly.  
With the peertube plugin 'sort-originally-published-at', this random sort can be by default.  

## Reorder randomly every 10 minutes

```
sudo apt-get -y install postgresql-11-cron 

su <peertube_user>

psql <peertube_base>

<peertube_base>=# CREATE EXTENSION pg_cron;

<peertube_base>=# SELECT cron.schedule('*/10 * * * *', $$update video set "originallyPublishedAt" = get_unsorted()::timestamp$$);
```

There may be credential issues. Look in /var/log/postgresql/ if the job is currently running good.  
If not, try:  
- to replace 'peer' by 'trust' in your pg_hba.conf  
- UPDATE cron.job SET nodename = ''; -- in psql as postgres or <peertube_user> user.  




