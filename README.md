# Overview

The original SQL query retrieves job listings along with related job categories, job types, and various affiliations. The query runs in approximately **8 seconds**, which suggests performance bottlenecks due to inefficient joins, lack of proper indexing, and expensive search operations.

However, **the size of the data and the structure design were not provided**, making it difficult to determine the most optimal query. The improvements suggested here are based on general best practices for optimizing SQL queries.

# Key Optimizations
## 1. Use Full-Text Search Instead of LIKE
The original query used multiple LIKE '%キャビンアテンダント%' conditions, which are **very slow** because they prevent index usage and force a full table scan.

**Solution:**

* Use ` FULLTEXT ` indexing and ` MATCH() AGAINST() ` for efficient text-based searches.
* This drastically reduces query execution time.

**Required Index:**

```sql
ALTER TABLE Jobs ADD FULLTEXT(
    name, description, detail, business_skill, knowledge, location, activity, 
    salary_statistic_group, salary_range_remarks, restriction, remarks
);
```  

**Updated Search Condition:**

```sql
MATCH(Jobs.name, Jobs.description, Jobs.detail, Jobs.business_skill, Jobs.knowledge, Jobs.location, Jobs.activity, Jobs.salary_statistic_group, Jobs.salary_range_remarks, Jobs.restriction, Jobs.remarks) 
AGAINST ('キャビンアテンダント' IN NATURAL LANGUAGE MODE)
```

## 2. Remove Unnecessary Columns from ` SELECT `   
The original query retrieves many columns that **may not be necessary for search results.**

✅ **Solution:**

* Select only the **essential** columns to reduce data transfer.
* This improves performance, especially when dealing with large datasets.

## 3. Ensure Proper Indexing for Joins and Filters 
The query has multiple ` LEFT JOIN`s and ` INNER JOIN `s, which can become slow if **indexes are missing**.


✅ **Solution:**

Add indexes on frequently joined columns:

```sql
CREATE INDEX idx_jobs_main ON Jobs(id, job_category_id, job_type_id, publish_status, deleted, sort_order);
CREATE INDEX idx_job_categories ON JobCategories(id, deleted, name);
CREATE INDEX idx_job_types ON JobTypes(id, deleted, name);
CREATE INDEX idx_personalities ON Personalities(id, deleted, name);
CREATE INDEX idx_practical_skills ON PracticalSkills(id, deleted, name);
CREATE INDEX idx_basic_abilities ON BasicAbilities(id, deleted, name);
CREATE INDEX idx_affiliates ON Affiliates(id, deleted, type, name);
```
This allows MySQL to **use indexed lookups instead of full table scans**.

## 4. Remove `GROUP BY Jobs.id`

* `GROUP BY Jobs.id` was unnecessary because Jobs.id is supposed to be unique.
* Removing it will **reduces execution overhead**.

## 5. Optimize Sorting & Pagination

* `ORDER BY Jobs.sort_order DESC, Jobs.id DESC` **should be indexed** to prevent sorting delays.
* Pagination with `LIMIT 50 OFFSET 0` can be optimized using **indexed cursor-based pagination**.
  
# Expected Performance Gains

| Optimization      |         Impact      |
| :---------------- | :------------------ |
| Full-Text Search (MATCH()) | Major improvement (~5-10x faster search than LIKE '%%' search) |
| Removing Unneeded Columns	| Reduces data transfer |
| Adding Indexes	| Faster joins & filtering |
| Removing GROUP BY	| Eliminates unnecessary sorting |
| Sorting & Pagination Optimization	| Faster page loads |

# Final Thoughts

These optimizations will **significantly reduce query execution time**. However, **without information on data size and table structure**, I'm unable to determine the most accurate solution.

I Would recommend:

1. Running `EXPLAIN ANALYZE` on both the original and optimized queries to compare execution plans.
2. Checking table sizes and row counts to further refine indexing strategies.
3. Considering **materialized views or caching** if query execution time remains high.

# Next Steps
1. Apply the suggested indexes.
2. Run `EXPLAIN ANALYZE` to measure improvements.
3. Optimize further based on query performance results.

**With these changes, the query should run significantly faster!**
