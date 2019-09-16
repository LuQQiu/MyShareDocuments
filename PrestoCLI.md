Run `presto-cli` to enter the presto cli

If using standalone metastore:
```
USE hive.default;
```

If using Glue:
```
USE hive;
create schema default;
USE hive.default;
```

```

CREATE TABLE reason (
		r_reason_sk bigint,
		r_reason_id varchar,
		r_reason_desc varchar
) WITH (
    external_location = 'alluxio://<master_rpc_addresses>/reason',
		format = 'PARQUET'
);
```
