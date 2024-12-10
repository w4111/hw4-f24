# HW4 Solutions

## HW4 Question 1 Cost Estimation Solution
Given data
* 400,000 records
* Page size = 4000 bytes
* tuple size = 100 bytes
* directory entry size = 10 bytes

**(Q1.1)** **5000**

		A query with equality select on a key would be a unique result. 
		Thus using the formula as in the slides on average we would find a equality search result within half the number of pages
		
		Tuple per page: 4000 / 100 = 40
	
		Total number of pages with 40 records each = 400,000/40 = 10000
	
		We have the average result as 10000/2 = 5000

**(Q1.2)** **3**

		Number of leaf pages - 400,000/(4000/100) = 10000
	
		Entry Size - 10B
	
	  	Fan out - 4000/10 = 400
	
		Height of the tree - ceil[log400(10000)] = 2
	
		Cardinality of ouput - 1
	
		Primary index implies data in the tuple.
	
		The result is the traversing the height of the tree + 1

**(Q1.3)** **4**	

		Number of leaf pages - 400,000/(4000/10) = 1000
	
		Fan out: 400
	
		Height of the tree: ceil[log400(1000)] = 2
	
		The result is traversing height of the tree + 1 + 1 (given the secondary index)

**(Q1.4) 3**

		Number of leaf pages - 2 * 400,000/(4000/100) = 20000
	
		Fan out = 200
	
		Height of tree = ceil[log200(20000)] = 2
	
		The result is cost of traversal of height of tree + 1(given primary index)

**(Q1.5) 4**

```
	Number of leaf pages - 2 * 400,000/(4000/10) = 2000

	Fan out = 200

	Hence result is height of the tree-ceil[log200(2000)] + 1 + 1(given secondary index)	
```

**(Q1.6)** **2**

		Given no overflow pages, a hash index would have
		1. Cost of one bucket access for the ssn value in the query
		2. Cost of one page access to fetch the record(given that it is secondary)


## Aggregation Queries

**(Q1.7)** **10000**

		Scan everything that is cost is scanning all pages. 

**(Q1.8)**  **3** 

		Since the index is built on salary we have to traverse to rightmost leaf node to fetch the max salary 
	
		The result is cost traversing height of tree(which is same as Q1.3) + 1(given secondary index)
		
		Alternatively, there are partial marks if you have the result - traversing height of tree + #leaves

**(Q1.9) 1000**

		The operation is 
		- Cost of scanning all the buckets for the hash index which is 400000/(4000/10) 
		since the index is built on salary and returning the max salary

**Question 2 Jupyter Notebook**

**Q1**:\
This query plan first runs an index scan on the BTree index `iowa_zip_btree`, with condition `zipcode = 10027`.
The results are then fed into the heap scan, which subsequently sorts the tuples by the data pages they reside in, 
and reads the tuples from those data pages. 

Both B+ tree and hash index can handle equality conditions. Postgres prefer tree index rather than hash on equality comparisons could be many reasons. Bucket chains could be one. Typically databases will use trees since they are heavily optimized as compared to random accesses for hash indexes.

 **Q2**:\
This is because the estimated number is only an educated guess based on the table's metadata. 

The estimate is usually `NCARD * selectivity`, where `NCARD` is the number of rows in the table and `selectivity` varies for different values and ranges. As an example, one possible value for selectivity is `1/ICARD`, where `ICARD` is the number of distinct values of the `zipcde` row. This can differ a lot from actual number of rows in the result.

Estimated costs and row counts might vary slightly because they are random samples rather than exact, and because costs are inherently somewhat platform-dependent.

**Q3**:\
The query plan uses the hash index. This is an equality constraint, and we only require one row as output. 
Typically databases will use trees since they are heavily optimized as compared to random accesses for hash indexes.
However, when the query result has very low `LIMIT` number (e.g. `LIMIT 1`) will hash index win over btree. Since there are so few that the extra gaim of sorting the row locations is not worth it.

**Q4**:\
The difference lies in the selectivity of these two queries. Q4A has higher selectivity, Btree index access path will help in
eliminating unnecessary disk IOs.   
However, Q4B covers basically the whole zipcode range, which suggests the output would be nearly all the tuples in the tables, so it is better to do sequential scan for reducing disk IO, since Btree requires 1 IO for accessing 1 records, other than 1 page with Sequential Scan.  
Same thing could be told from with cost statistics yield by `EXPLAIN` query.

**Q5**:\
Q5A uses Bitmap Index Scan on iowa_store_tree and Bitmap Heap Scan on iowa. 
We can see Q5A has selectivity = 145523 /1000000 = 14.6%, and Q5B Q5B has selectivity = 9267 / 1000000 = 0.009%, thus disk IO can be saved greatly compared with sequential scan. Btree allows Bitmap Index Scan, which will save a lot unnecessary random IOs.

Bitmap Scans are very different from plain Index Scans. An Index Scan simply scans through an index and fetches tuples that satisfy a condition one by one. Thus each tuple would incur a separate access to a data page. On the other hand, a Bitmap Scan collects all pointers to tuples matching a condition, sorts them by the data pages they are stored in, and then fetches those tuples in one go. This improves the locality of page accesses. 

**Q6**:\
It will take more time for records update/insertion, since indexes force some ordered data structure, which is costly when update/insertion.
