Run `presto-cli` to enter the presto cli

```
USE hive.default;
```

```

CREATE TABLE reason (
		r_reason_sk bigint,
		r_reason_id varchar,
		r_reason_desc varchar
) WITH (
    external_location = 'alluxio://ip-172-31-45-81.us-east-2.compute.internal:19998/reason',
		format = 'PARQUET'
);
```
