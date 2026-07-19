# Written Assessment

## README: Approach, Assumptions and Trade-offs

### Approach

I approached this task by following a simple medallion architecture pattern.

First, I ingested the raw CSV files into the Bronze layer without making changes to the source data. This helps preserve the original data for auditing and future debugging.

In the Silver layer, I focused on preparing the data for analysis. This included handling schema differences between files, standardizing column names and data types, validating required fields, removing duplicate records, and identifying invalid records.

Finally, in the Gold layer, I created reporting-ready data by applying aggregations required for analysis.

### Assumptions

- I considered event_id as the unique identifier for an event.
- Records with missing important fields such as event_id or spend were treated as invalid.
- Duplicate records were removed to avoid incorrect reporting.
- Invalid records were separated instead of deleting them permanently, so they can be reviewed later.

### Trade-offs

- Additional validation improves data quality but increases processing time.
- Keeping invalid records separately requires extra storage but helps with troubleshooting.
- Delta Lake provides reliability and data versioning, but it needs more storage compared to plain files.

---

## Client Communication

Hi Team,

We identified an issue with the vendor file received tonight. The file appears to be corrupted, and we are currently unable to complete the data processing successfully.

Due to this issue, the 9 AM dashboard refresh may be delayed. We are working on identifying the root cause and coordinating for a corrected file.

We will provide an update as soon as we have more information and share the revised timeline.

Thanks,
Anitha

---

## Code Review

The main issues I identified in the code are:

1. The schema is not defined while reading the CSV file. Relying on automatic schema inference can create data type issues and may be slower for large files.

2. `dropDuplicates()` is applied without specifying the key column. For large datasets, this can be expensive. Deduplication should usually be based on a business key like event_id.

3. The null check is incorrect. In Spark, null values should be handled using functions like `isNull()` or `isNotNull()`.

4. `collect()` should be avoided because it brings all data to the driver. With large datasets, this can cause memory issues.

5. Processing rows one by one using a loop is not suitable for Spark. Spark works best with distributed DataFrame transformations.

6. Filtering inside the loop creates multiple Spark operations and will reduce performance.

7. Caching the dataframe without knowing whether it will be reused may unnecessarily consume memory.

---

## Scale-up Considerations

If this feed becomes 10 times larger and arrives hourly, I would first focus on improving the ingestion and processing approach.

The changes I would consider are:

- Process only new incoming files instead of reprocessing all historical data every hour.
- Use Delta Lake features for better reliability and incremental processing.
- Add partitioning based on date columns to reduce data scanning.
- Optimize duplicate handling instead of performing expensive full-data comparisons.
- Add automated data quality checks for schema changes, missing values and invalid records.
- Monitor pipeline execution time, failures and record counts.
- Optimize storage by managing small files generated from frequent loads.
