# Uncovering Process Insights: Applying Graph Algorithms to Object-Centric Event Data
 
This repository accompanies the master's thesis *Uncovering Process Insights – Applying Graph Algorithms to Object-Centric Event Data* (Lars Herbold, Department of Computer and Systems Sciences, Stockholm University, 2026). It contains the implementation of the two algorithms developed in the thesis and the notebook used to produce the BPIC 2019 results reported in Chapter 4.
 
The algorithms identify **synchronization bottlenecks** in Event Knowledge Graphs (EKGs): activities at which related objects of different types (for example, the items of one purchase order) reach the same activity far apart in time.
 
- **Algorithm 1 – `find_bottleneck_activities`** computes, for a pair of object types (a *coarse* type such as `PO` and a *fine* type such as `POItem`), the *spread* per coarse object and activity: the time between the first and the last arrival of its fine objects. It aggregates the spreads per activity (count, sum, mean, median, maximum) and ranks activities by median spread.
- **Algorithm 2 – `find_bottleneck_objects`** drills down into one activity and lists, per coarse object, the number of fine objects, their first and last arrival, and the resulting spread.
The spread is a proxy for synchronization waiting time. It equals the waiting time only where the activity is a joint execution of the related objects; see Section 5.3 of the thesis.
 
## Repository Contents
 
1. **Notebook** – [`calculate_bottlenecks.ipynb`](calculate_bottlenecks.ipynb) contains:
   - the connection helper `run_cypher`,
   - the Cypher implementations of Algorithm 1 and Algorithm 2,
   - the run of Algorithm 1 over all ordered pairs of object types (the 3D overview "cube"),
   - the BPIC 2019 slice for the pair (`PO`, `POItem`) with the bottleneck score and heatmap (Figure 4.1, Table 4.2),
   - the drill-down with Algorithm 2 for the activity *Record Goods Receipt* and the distribution plots (Table 4.3, Figures 4.2–4.4).
   If you only want to see the results, open the notebook on GitHub; the outputs are stored in it.
2. **Data** – The EKGs are not included in this repository. They are the published BPI Challenge event graphs by Esser and Fahland, available from 4TU.ResearchData (see [Datasets](#datasets)).
## Setup
 
### 1. Neo4j
 
1. Install [Neo4j](https://neo4j.com/download/) (Neo4j Desktop or Server, version 5.x) and create a new database.
2. Install the [APOC](https://neo4j.com/docs/apoc/current/installation/) plugin. It is used only to import the GraphML files; the algorithms are plain Cypher and do not need the Graph Data Science library.
3. Increase the memory limits in `conf/neo4j.conf`. The larger graphs fail to import with the defaults:
```properties
   server.memory.heap.initial_size=2g
   server.memory.heap.max_size=4g
   dbms.memory.transaction.total.max=4g
```
   Note that `dbms.memory.transaction.total.max` is commented out by default; remove the leading `#`.
4. Create a file `conf/apoc.conf` with the following line to allow APOC to read files from the import folder:
```properties
   apoc.import.file.enabled=true
```
5. Restart the database.
### 2. Import an Event Knowledge Graph
 
1. Download the `.graphml.zip` file of the graph you want to analyse (see [Datasets](#datasets)) and unzip it.
2. Copy it into the `import` folder of your Neo4j database (in Neo4j Desktop: *… → Open folder → Import*).
3. Run the import in the Neo4j Browser, replacing the file name:
```cypher
   CALL apoc.import.graphml("filename.graphml", {
     batchSize: 500,
     readLabels: true,
     storeNodeIds: true
   })
```
 
Import one graph per database. If you want to analyse several graphs, create one database per graph and pass its name via the `database` parameter of the notebook functions.
 
To see which object types a graph contains (needed to choose the coarse and fine types), run:
 
```cypher
MATCH (n:Entity) RETURN n.EntityType AS type, count(*) AS objects ORDER BY objects DESC
```
 
### 3. Python environment
 
The notebook was developed with Python 3.13.
 
```bash
pip install jupyterlab neo4j pandas numpy matplotlib
jupyter lab
```
 
Open `calculate_bottlenecks.ipynb` and set `URI` and `AUTH` in the second cell to your database connection. The defaults are `neo4j://127.0.0.1:7687` with user `neo4j` and password `password`.
 
## Running the Analysis
 
Run the notebook from top to bottom. The parameters used in the thesis are:
 
| Parameter | Value | Where |
| --- | --- | --- |
| Object types | `PO`, `POItem`, `Vendor` (resources excluded) | `ENTITY_TYPES` |
| Cutoff τ | `2017-01-01T00:00+01:00` | `CUTOFF` / `cutoff_date` |
| Object type pair | (`PO`, `POItem`) | Algorithm 1 slice and Algorithm 2 |
| Drill-down activity | `Record Goods Receipt` | Algorithm 2 |
 
The cutoff excludes events with timestamps before 1 January 2017, which in the BPIC 2019 data are outliers outside the recorded period.
 
To apply the method to another graph, set `ENTITY_TYPES` to the object types of that graph and choose a cutoff that fits its time range.
 
## Datasets
 
| Graph | Source |
| --- | --- |
| BPI Challenge 2014 | Fahland, D., Esser, S. (2021). *Event Graph of BPI Challenge 2014*. [doi:10.4121/14169494.v1](https://doi.org/10.4121/14169494.v1) |
| BPI Challenge 2015 | Fahland, D., Esser, S. (2021). *Event Graph of BPI Challenge 2015*. [doi:10.4121/14169569.v1](https://doi.org/10.4121/14169569.v1) |
| BPI Challenge 2016 | Fahland, D., Esser, S. (2021). *Event Graph of BPI Challenge 2016*. [doi:10.4121/14164220.v1](https://doi.org/10.4121/14164220.v1) |
| BPI Challenge 2017 | Fahland, D., Esser, S. (2021). *Event Graph of BPI Challenge 2017*. [doi:10.4121/14169584.v1](https://doi.org/10.4121/14169584.v1) |
| BPI Challenge 2019 | Fahland, D. (2021). *Event Graph of BPI Challenge 2019*. [doi:10.4121/14169614.v1](https://doi.org/10.4121/14169614.v1) |
 
The graphs were built with the procedure described in Esser, S., Fahland, D. (2021). Multi-dimensional event data in graph databases. *Journal on Data Semantics*, 10(1), 109–141. [doi:10.1007/s13740-021-00122-1](https://doi.org/10.1007/s13740-021-00122-1)