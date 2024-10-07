# Worktime counting system

## Task description:
Create a dataflow which counts employee working time based on events (stop working, start working). Each employee has two events in the main dataset (start and stop) and there is also one enrichment file with personal data for all employee. The common primary key for both datasets is CK (employee ID number). Datasets are stored in the bottom the this file

***

## Main Flow:

### Short description:

- Generating events upon the start and end of work
    - Events for the start of work are pushed to Redis.
    - Events for the end of work are sent to Kafka, enriched with information from Redis (start time of work) and data from S3 regarding the employee's personal information.
- Personal data of the employee is stored in S3.
- Matching should be based on the "CK" field.

### Detailed description

#### Main Process

1. **Generate Main FF (A)**:
   - The workflow begins with the generation of the main FlowFile, which serves as the primary data entity for processing.

2. **Split Events into Separate FFs (L)**:
   - The main FlowFile is split into separate FlowFiles to handle individual events separatelly.

3. **Extract Data from FF Content (N)**:
   - Relevant data is extracted from the content of each FlowFile for further processing.

4. **Divide FF to Groups (B)**:
   - The FlowFiles are categorized into two groups: "Start" and "Stop" events.

5. **Handling "Start" Events (C)**:
   - For FlowFiles identified as "Start," they are stored in Redis with a unique key for future reference.

6. **Handling "Stop" Events (D)**:
   - For "Stop" FlowFiles, the process retrieves the corresponding "Start" FlowFile from Redis using the key.
   - If the "Start" FlowFile is not found, a loop is created (P) to control the rate of retries until the "Start" FlowFile becomes available.

7. **Fetch Data from S3 (F)**:
   - Once the "Start" FlowFile is successfully fetched, additional data is retrieved from S3 to enrich the FlowFile content.

8. **Count Working Time (R)**:
   - The working time is calculated based on the retrieved data.

9. **Extract Data from FF Content (S)**:
   - Relevant information is extracted from the updated FlowFile content.

10. **Count Salary (T)**:
    - The salary is calculated based on the extracted data and the working time.

11. **Data Transformation (G)**:
    - The processed data undergoes transformation to ensure it meets the required format for further processing.

12. **Send to Kafka (H)**:
    - Finally, the enriched and transformed FlowFile is sent to Kafka for further handling or distribution.

13. **Connection (C -.-> D)**:
    - There is a back-reference from the "Start" event stored in Redis to the "Stop" event, allowing the system to access "Start" FlowFiles directly if needed.

#### Enrichment Process

1. **Generate Enrichment FF (I)**:
   - A separate process generates enrichment FlowFiles that contain additional data necessary for enhancing the main FlowFiles.

2. **Split Events into Separate FFs (M)**:
   - Similar to the main process, these enrichment FlowFiles are split into separate entities for processing.

3. **Extract Data from FF Content (O)**:
   - Relevant data is extracted from the content of the enrichment FlowFiles.

4. **Data Transformation (J)**:
   - The extracted data is transformed to ensure it is in the proper format.

5. **Send Data to S3 (K)**:
   - The transformed enrichment data is sent to S3 for storage or future use.

6. **Connection (K -.-> F)**:
   - There is a reverse relationship indicating that the enriched data stored in S3 can later be integrated back into the main FlowFiles during processing.
    
## Data flow model:

```mermaid
flowchart TD
	subgraph Main
		A((Generate \n main FF)) --> L[Split events into\n separete FFs]
		L --> N[Extract data from \nFF content]
		N --> B{Divide FF \n to groups}
		B -- Kind: Start --> C[Put FF to redis DB\n with proper key]
		B -- Kind: Stop --> D[Fetch start FF to an \nattribute from redis \n based on key ]
		D -- Start object not found --> P[Loop created \nwith control rate]
		P --> D
		D --> F[Fetch data from S3 and \n put them to FF content]
		F --> R[Count working time]
		R --> S[Extract data from \nFF content]
		S --> T[Count Salary]
		T --> G[Data transformation\n adjustment to proper format]
		G --> H((Send to\n Kafka))
		C -.-> D
	end 
	subgraph Enrichament
		I((Generate \n enrichment FF)) --> M[Split events into\n separete FFs]
		M --> O[Extract data from \nFF content]
		O --> J[Data transformation \n adjustment to proper format]
		J --> K((Send data\n to S3))
		K -.-> F
	end
```
> FF - flowfile / event
## Data

### Main Event:

```JSON
[{
 "operation" : "START",
 "time" : "1676201442000",
 "CK" : "AA11BB"
},
{
 "operation" : "START",
 "time" : "1676197842000",
 "CK" : "AA22BB"
},
{
 "operation" : "START",
 "time" : "1676212242000",
 "CK" : "AA33BB"
},
{
 "operation" : "START",
 "time" : "1676195742000",
 "CK" : "AA44BB"
},
{
 "operation" : "STOP",
 "time" : "1676201442000",
 "CK" : "AA11BB"
},
{
 "operation" : "STOP",
 "time" : "1676197842000",
 "CK" : "AA22BB"
},
{
 "operation" : "STOP",
 "time" : "1676212242000",
 "CK" : "AA33BB"
},
{
 "operation" : "STOP",
 "time" : "1676195742000",
 "CK" : "AA44BB"
}
]
```

### Enrichment data:

```JSON
[{
  "name" : "Michal",
  "surname" : "Kowalski",
  "position" : "engineer",
  "hourlyRate" : "20",
  "CK" : "AA11BB"
},
{
  "name" : "Kamil",
  "surname" : "Nowak",
  "position" : "trainee",
  "hourlyRate" : "10",
  "CK" : "AA22BB"
},
{
  "name" : "Monika",
  "surname" : "Hamik",
  "position" : "expert",
  "hourlyRate" : "50",
  "CK" : "AA33BB"
},
{
  "name" : "Kasia",
  "surname" : "Nowacka",
  "position" : "expert",
  "hourlyRate" : "50",
  "CK" : "AA44BB"
}]
```

### END event:

```JSON
{
  "Salary" : "WorkingTime * hourlyRate",
  "position" : "position",
  "WorkingTime" : "STOP_time - START_time",
  "surname" : "surname",
  "hourlyRate" : "10",
  "name" : "name",
  "CK" : "AA22BB"
}
```
