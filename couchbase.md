# Couchbase Database Indexing

### So you can “over-index” in couchbase.

https://forums.couchbase.com/t/very-slow-performance-on-query-without-index-on-simple-documents/6410

http://localhost:8093/query/service?statement=select * from default where a2=4703626&max_parallelism=4

You can use the max_parallelism to utilize the full machine and get your scan to parallelize. We don’t do this auto-magically yet. Unfortunately trying this will require a different tool. cbq tool is not designed for high performance query executions so does not allow this. You need to do this through curl or another REST API capable tool like chrome browser. I like to use a tool like postman. Here is the sample REST API call;


https://docs.couchbase.com/server/current/learn/services-and-indexes/indexes/indexing-and-query-perf.html

## Composite Secondary Index:
This is commonly used for performance optimization.

## Partial Index:
> CREATE INDEX travel_eat ON `travel-sample`.inventory.landmark(name, id, address) WHERE activity='eat';


* https://blog.couchbase.com/database-indexing-best-practices/

> Indexes are generally memory-intensive and queries are CPU-intensive. You can have different hardware for these different services

* https://blog.couchbase.com/database-indexing-best-practices/


## Index by Predicate
The WHERE clause in a query is called as the Predicate and the fields/attributes selected in the SELECT clause is called as Projection. Indexes should always be created with the Predicate clause in mind. This is because Index Selection happens based on the leading key of the index present in the Predicate.


## Use a Leading Key to Force Index Selection

An index is not automatically selected for a query if the predicate used in the query does not match the leading key of the index. If you happen to see that the EXPLAIN PLAN does not force any index to be selected then use ‘IS NOT MISSING’ or ‘IS NOT NULL’ clause to force the index to be selected.

To choose only an already created INDEX, use the USE INDEX directive as part of the N1QL query. This is helpful in cases wherein you know that the index mentioned in USE INDEX has better selectivity than the one chosen by N1QL Rule Based Optimizer:

select count(1) from `travel-sample` USE INDEX (idx_ts_type_iata) where type="airline";


## Use Defer builds
CREATE INDEX `idx_ts_type_iata` ON `travel-sample`(`type`,`iata`) WITH { "defer_build":true }; 
BUILD INDEX ON `travel-sample`(`idx_ts_type_iata`);

* https://www.youtube.com/watch?v=OrC2gkm2OFA
* https://blog.couchbase.com/create-right-index-get-right-performance/

> Every secondary index in Couchbase should have a WHERE clause, with at least a condition on the document type. This isn’t enforced by the system, but it’s good design.

```
CREATE INDEX def_route_src_dst 
 ON `travel-sample` (`sourceairport`, `destinationairport`) 
 WHERE (`type` = "route");
 ```

## Rule #4: INDEX BY WORKLOAD, NOT BY BUCKET/KEYSPACE

## Performance optimization:
> Cluster configuration, tuning, SDK configuration, use of prepared statements all play a significant role.

* https://dzone.com/articles/create-the-right-index-get-the-right-performance-p
* https://www.youtube.com/watch?v=jMIp-aK4TSs
* https://index-advisor.couchbase.com/indexadvisor/#1


## BEFORE IT TOOKS: 4.8s

> CREATE INDEX adv_ledger_idate_serial ON chaldal.erp.ledger_transaction(`ledger`,`idate`,`serial`);

> CREATE INDEX adv_ledger_serial_idate_baltype_debit_credit_balance ON `ledger_transaction`(`ledger`,`serial`,`idate`,`baltype`,`debit`,`credit`,`balance`);

```
SELECT serial,ledger,debit,credit,balance,baltype FROM chaldal.erp.ledger_transaction where ledger="110004" AND idate BETWEEN "2021-12-01" AND "2021-12-31" AND serial IS NOT MISSING ORDER BY serial

SELECT serial,ledger,debit,credit,balance,baltype FROM chaldal.erp.ledger_transaction where ledger="110004" AND idate BETWEEN "2021-12-01" AND "2021-12-31"

SELECT ADVISOR((SELECT RAW statement FROM system:completed_requests));
```

* https://blog.couchbase.com/create-right-index-get-right-performance/
* https://developers.refinitiv.com/en/article-catalog/article/how-to-test-http-rest-api-easily-with-visual-studio-code---thund
* https://dev.to/gholami1313/saving-log-messages-to-a-custom-log-file-in-golang-ce5

> go test -timeout 30s -run ^TestPostMethod$ service
