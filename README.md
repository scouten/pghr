# Pghr

Quick 'n' dirty test to see how Ecto + Postgres behave in specific microbenchmarks.

## Results

The benchmark results described below were run on my 2015-era MacBook Pro. Tests were always run for 30 seconds with a 2-second warm-up. Varying levels of parallelism were used and noted in the test results.

Update (27 September 2019): In this round of benchmarks, I'm consistently using the following parameters across all experiments:

* 30-second duration
* 40 pgbench threads / 40 Elixir processes
* 40 Postgres connections

I've discarded previous results and am running with these settings across the board.

### Create Item Benchmark

Ecto:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && mix run bench/create_item_fast.exs 
```

Pgbench:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && pgbench -f pgbench/create_item_fast.sql -n -c 40 -j 40 -T 30 pghr
```

notify? | Ecto ips | pg_bench ips | Ecto % of pgbench | Comments
--------|---------:|-------------:|------------------:|-----
 N      |    21138 |        24168 |               87% | Baseline (PR #18)
 Y      |     9445 |         9464 |              100% |   
 N      |    21090 |        24168 |               87% | Warm up and truncate (PR #19)
 Y      |     8885 |         9464 |               94% |   
 N      |    21188 |        24168 |               88% | 1000 iterations per checkout (PR #20)
 Y      |     9141 |         9464 |               97% |   
 N      |    21689 |        24168 |               88% | Remove random number (PR #21)
 Y      |     9266 |         9464 |               98% |   

### Update Item Benchmark (Using Raw SQL Update)

Ecto:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && mix run bench/update_item_sql.exs
```

Pgbench:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && mix run pgbench/seed_items.exs && pgbench -f pgbench/update_item_fast.sql -n -c 40 -j 40 -T 30 pghr
```

notify? | Ecto ips | pg_bench ips | Ecto % of pgbench | Comments
--------|---------:|-------------:|------------------:|-----
 N      |    28579 |        28175 |              101% | Baseline (PR #18)
 Y      |    10142 |         9501 |              107% |   
 N      |    29089 |        28175 |              103% | Warm up and truncate (PR #19)
 Y      |     9935 |         9501 |              105% |   
 N      |    29413 |        28175 |              104% | 1000 iterations per checkout (PR #20)
 Y      |     9959 |         9501 |              105% |   
 N      |    29356 |        28175 |              104% | Remove random number (PR #21)
 Y      |    10057 |         9501 |              106% |   
