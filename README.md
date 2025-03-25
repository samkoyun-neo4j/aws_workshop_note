# aws_workshop_note

## Data ingestion scripts

### Form13 - One day sample data

* **Managers:** Entities that manage assets.
* **Companies:** Entities that are held as investments.
* **OWNS:** Relationships connecting Managers to Companies, representing ownership.

```cypher
LOAD CSV WITH HEADERS FROM "https://neo4j-dataset.s3.amazonaws.com/hands-on-lab/form13-2023-05-11.csv" AS row
MERGE (m:Manager {managerName:row.managerName})
MERGE (c:Company {companyName:row.companyName, cusip:row.cusip})
MERGE (m)-[r:OWNS {value:toFloat(row.value), shares:toInteger(row.shares), reportCalendarOrQuarter:date(row.reportCalendarOrQuarter)}]->(c);
```

Delete the sample dataset

```cypher
MATCH (n) DETACH DELETE n;
```

### Form13 - Filings over $10M from 2023

Pre-filtered Form 13 data that includes filings with over $10 million in value.

_[Note] Increase the query time window for this part._

Creating constraints

```cypher
CREATE CONSTRAINT unique_company_id IF NOT EXISTS FOR (p:Company) REQUIRE (p.cusip) IS NODE KEY;
CREATE CONSTRAINT unique_manager IF NOT EXISTS FOR (p:Manager) REQUIRE (p.managerName) IS NODE KEY; 
```

Load the companies

```cypher
LOAD CSV WITH HEADERS FROM 'https://neo4j-dataset.s3.amazonaws.com/hands-on-lab/form13-2023.csv' AS row
MERGE (c:Company {cusip:row.cusip})
ON CREATE SET c.companyName=row.companyName;
```

Load the managers

```cypher
LOAD CSV WITH HEADERS FROM 'https://neo4j-dataset.s3.amazonaws.com/hands-on-lab/form13-2023.csv' AS row
MERGE (m:Manager {managerName:row.managerName});
```

Load the OWNS relationships

```cypher
LOAD CSV WITH HEADERS FROM 'https://neo4j-dataset.s3.amazonaws.com/hands-on-lab/form13-2023.csv' AS row
CALL (row) {
     MATCH (m:Manager {managerName:row.managerName})
     MATCH (c:Company {cusip:row.cusip})
     MERGE (m)-[r:OWNS {reportCalendarOrQuarter:date(row.reportCalendarOrQuarter)}]->(c)
     SET r.value = toFloat(row.value), r.shares = toInteger(row.shares)
} IN TRANSACTIONS OF 10000 ROWS
```
