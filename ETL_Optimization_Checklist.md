BigQuery ETL Optimization Checklist for 100M+ Rows
--------------------------------------------------

### I. Data Modeling and Storage Optimization

1.  **Partitioning:**
    
    *   **Always partition large tables:** Partition by ingestion time (\_PARTITIONTIME), DATE, TIMESTAMP, or INTEGER\_RANGE columns. For time-series data (common in e-commerce), DATE or TIMESTAMP partitioning is usually ideal.
        
    *   **Granularity:** Choose the appropriate granularity (DAY, HOUR, MONTH, YEAR) based on your most common query patterns. Aim for partitions with at least 10GB of data. Too many small partitions can increase metadata overhead.
        
    *   **Filter on partition columns:** Ensure all queries filter on the partition column(s) in the WHERE clause (e.g., WHERE \_PARTITIONTIME BETWEEN '2025-01-01' AND '2025-01-31'). This is the single most impactful optimization for reducing scanned bytes.
        
    *   **Avoid wildcard tables where specific partitions can be identified:** While useful, my\_table\_\* can be less efficient than explicitly selecting partitions if you know the exact range (e.g., my\_table\_20250101, my\_table\_20250102). If using wildcards, make sure your \_TABLE\_SUFFIX or \_PARTITIONTIME filters are highly selective.
        
2.  **Clustering:**
    
    *   **Implement clustering on frequently filtered/joined columns:** After partitioning, cluster on columns that are often used in WHERE, JOIN, or GROUP BY clauses. You can specify up to four clustering columns.
        
    *   **Order matters:** The order of clustered columns matters for pruning. Place the most frequently used and highly selective columns first. BigQuery prunes based on the order.
        
    *   **Benefit for unpartitioned/large partitions:** Clustering is beneficial for unpartitioned tables larger than 64 MB and for individual partitions larger than 64 MB.
        
3.  **Data Types:**
    
    *   **Use the smallest appropriate data types:** For example, use INT64 instead of NUMERIC if only integers are needed, or DATE instead of TIMESTAMP if time isn't relevant. Smaller data types consume less storage and lead to faster scans.
        
    *   **Prefer INT64 for join keys:** Integer keys generally perform better than string keys for joins.
        
    *   **Avoid STRING for very large fields if possible:** Consider BYTES for raw data if string operations are not consistently needed.
        
4.  **Table Design:**
    
    *   **Denormalize where appropriate:** While BigQuery handles joins well, excessive joins on very large tables can still be a bottleneck. Consider denormalizing common lookup data into your main fact tables if it significantly reduces the number of joins or the amount of data shuffled.
        
    *   **Use nested and repeated fields (STRUCTs and ARRAYs):** BigQuery's columnar storage benefits from nested data, as it can keep related data together, reducing data shuffling for queries that access these structures.
        

### II. Query Optimization Techniques

1.  **Reduce Data Scanned:**
    
    *   **Avoid SELECT \*:** Explicitly select only the columns you need. This is crucial for cost and performance as BigQuery charges based on bytes processed in selected columns.
        
    *   **Filter early and aggressively:** Apply WHERE clauses as early as possible in your query to reduce the amount of data processed in subsequent stages.
        
    *   **Order filter conditions:** Place the most restrictive filters (those that eliminate the most data) first in your WHERE clause.
        
    *   **Use LIMIT effectively:** LIMIT does NOT reduce the amount of data scanned in BigQuery for non-clustered tables. Use LIMIT with an ORDER BY clause only when you need a sample or top N results, and be aware it can still scan the entire table to perform the sort. To genuinely limit scanned bytes for exploration, use the "Preview" feature or Maximum bytes billed setting.
        
2.  **Optimize Joins:**
    
    *   **Join order matters (for Broadcast Joins):** For broadcast joins (where one table is small enough to be sent to all workers), BigQuery generally performs better if the largest table is on the left side of the join, followed by the smallest.
        
    *   **Join on partitioned/clustered fields:** If possible, join on columns that are partitioned and/or clustered in both tables. This enables BigQuery to use "block pruning" and reduce the data scanned.
        
    *   **Use appropriate join types:** INNER JOIN is generally faster than LEFT JOIN, which is faster than FULL OUTER JOIN or CROSS JOIN. Avoid CROSS JOIN unless absolutely necessary, as they can lead to massive result sets.
        
    *   **Pre-filter joined tables:** Filter tables before joining them to reduce the amount of data that needs to be joined.
        
3.  **Optimize Aggregations and Grouping:**
    
    *   **Aggregate as late as possible:** Unless an early aggregation significantly reduces the data size for subsequent computations, push aggregations towards the end of the query.
        
    *   **Use APPROX\_COUNT\_DISTINCT for high cardinality unique counts:** For very large datasets where exact counts aren't strictly necessary, APPROX\_COUNT\_DISTINCT is significantly faster and cheaper.
        
    *   **Prefer COUNT(DISTINCT column) over GROUP BY column then COUNT(\*):** For simple distinct counts.
        
    *   **Minimize GROUP BY on high cardinality columns:** Grouping on columns with many unique values can lead to significant data shuffling.
        
4.  **Window Functions vs. Self-Joins:**
    
    *   **Prefer window functions:** For calculations involving previous/next rows or ranking within groups, window functions (OVER (PARTITION BY ... ORDER BY ...)) are generally more efficient than self-joins, as they reduce data duplication and shuffling.
        
    *   **Limit data before window functions:** If possible, limit the dataset before applying complex window functions to reduce the amount of data the function processes.
        
5.  **Subqueries and CTEs (Common Table Expressions):**
    
    *   **Use CTEs for readability and reusability:** CTEs improve code readability, but BigQuery doesn't inherently optimize them to avoid recomputing if the same CTE is referenced multiple times.
        
    *   **Materialize complex subqueries or CTEs:** If a complex subquery or CTE is used multiple times or processes a very large amount of data, consider materializing its result into a temporary table or a permanent table if it's a recurring pattern. This can prevent redundant computations.
        
6.  **Wildcard Tables:**
    
    *   **Use \_TABLE\_SUFFIX or \_PARTITIONTIME for filtering:** Always apply highly selective filters on these pseudo-columns when querying wildcard tables to limit the number of tables scanned.
        
    *   **Longer prefixes perform better:** If using wildcard tables, a more specific prefix (e.g., my\_table\_2025\*) will generally perform better than a shorter or empty prefix (my\_table\_\*).
        

### III. ETL Pipeline Specific Optimizations

1.  **Incremental Loads:**
    
    *   **Design for incremental processing:** Instead of reprocessing all historical data, load only new or changed data. This dramatically reduces scan volumes.
        
    *   **Use MERGE statements effectively:** MERGE can be highly efficient for upserting data incrementally into existing tables.
        
    *   **Leverage change data capture (CDC):** If your source system supports CDC, integrate it to process only changes.
        
2.  **Materialized Views:**
    
    *   **Identify common, expensive queries:** If you have frequently run, complex queries (especially for dashboards or BI tools) that involve aggregations or joins, create materialized views.
        
    *   **Automatic refresh:** BigQuery automatically refreshes materialized views incrementally, ensuring fresh data without manual intervention.
        
    *   **Smart tuning:** BigQuery's optimizer can automatically rewrite queries to use materialized views even if the original query targets the base tables.
        
    *   **Consider limitations:** Materialized views have some limitations (e.g., restricted SQL, cannot be nested on other materialized views, cannot query external/wildcard tables).
        
3.  **Scheduled Queries:**
    
    *   **Automate repetitive ETL jobs:** Use BigQuery's scheduled queries to automate recurring data transformations and aggregations.
        
    *   **Optimize schedule:** Stagger the execution of large, resource-intensive queries to avoid contention if you are on a flat-rate pricing model.
        
4.  **External Data Sources:**
    
    *   **Avoid if performance is critical:** While BigQuery supports external data sources (e.g., GCS, Bigtable), querying data within BigQuery's managed storage is typically faster. Ingest data into BigQuery tables if query performance is a top priority.
        
    *   **Use for specific use cases:** External tables are good for staging data before ingestion, infrequent ad-hoc analysis, or frequently changing data where direct loading isn't feasible.
        

### IV. Monitoring and Troubleshooting

1.  **Analyze Query Plans and Execution Details:**
    
    *   **Use the BigQuery UI (Execution Graph) or jobs.get API:** Understand how BigQuery executes your queries. Look for:
        
        *   **Bytes Shuffled:** High shuffling indicates inefficient joins or aggregations, often due to data skew.
            
        *   **Bytes Scanned:** The primary cost driver. Aim to reduce this.
            
        *   **Slot Time Consumed:** Indicates the total computational work performed.
            
        *   **Stages with high duration/slot time:** Pinpoint bottlenecks.
            
        *   **REPARTITION stages:** Can indicate data skew or inefficient distribution.
            
    *   **Identify performance insights:** BigQuery provides automatic insights on potential bottlenecks (e.g., "High cardinality join").
        
2.  **Monitor Slot Usage and Contention:**
    
    *   **BigQuery Monitoring (Cloud Monitoring):** Track slot utilization, queued slot time, and job duration.
        
    *   **INFORMATION\_SCHEMA.JOBS\_BY\_PROJECT:** Query this view to get detailed metadata about past jobs, including slot usage and execution times.
        
    *   **Address slot contention:**
        
        *   If using flat-rate pricing, consider increasing slot capacity or creating separate reservations for different workloads (e.g., ETL vs. ad-hoc analytics).
            
        *   Optimize unoptimized queries that consume excessive slots.
            
        *   Stagger job schedules to avoid peak-time overlaps for heavy jobs.
            
3.  **Cost Monitoring:**
    
    *   **Set up billing alerts and budgets:** Proactively monitor and control costs.
        
    *   **Use DRY RUN:** Before running a large query, perform a dry run to estimate the bytes processed and associated cost.
        
    *   **Set Maximum bytes billed:** For ad-hoc or exploratory queries, set a maximum byte limit to prevent runaway costs.
        

### V. Advanced Considerations

1.  **BI Engine:** For interactive dashboards and BI tools, enable BigQuery BI Engine. It's a fast, in-memory analysis service that can significantly accelerate many SQL queries without query modifications.
    
2.  **Search Indexes:** For tables with frequent point lookups or highly selective filters on specific columns (especially strings), consider creating a search index. This can provide significant performance gains for those specific query patterns.
    
3.  **Understand BigQuery's Architecture:**
    
    *   **Columnar Storage:** Remember BigQuery stores data in columns, so selecting fewer columns means scanning less data.
        
    *   **Distributed Processing:** BigQuery is highly parallelized. Optimizations should aim to distribute work evenly and minimize data shuffling between nodes.
        

By diligently applying these practices, you can significantly improve the performance and cost-efficiency of your BigQuery ETL pipelines, even when dealing with hundreds of millions of rows. Regular monitoring and iterative optimization based on query execution insights are key to long-term success.
