# Calculator

## Task description:
Create a dataflow allowing to count sum and product of 2 numbers provided in the dataset. Dataset is a JSON list shown at the bottom of this file


## Main Flow

### Short description:

- Generating events containing two numbers: A, B
    - Flow should calculate: sum, product
- The results should be sent to Kafka and Redis and should contain the numbers: A, B, sum(A, B), product(A, B)

### Detailed description

#### Main Process

1. **Generate Main FF**:
   - The dataflow starts with the generation of the main FlowFile.
   - Processor name: ```GenerateFlowFile```

2. **Split Events into Separate FFs**:
   - Then  main FlowFile is divided into separate FFs to handle individual events independently.
   - Processor name: ```SpitJson```

3. **Extract Data from FF Content**:
   - Relevant data is extracted from the content of each FlowFile. This step is crucial for performing calculations and transformations in subsequent stages.
   - Processor name: ```EvaluateJsonPath```

4. **Count Sum and Product**:
   - The extracted data is used to calculate both the sum and the product of the given values.
   - Processor name: ```UpdateAttribute```

5. **Data Transformation**:
   - The results from the calculations are transformed and adjusted to ensure they meet the required format we want to sent to Kafka and Redis
   - Processor name: ```JoltTransformJson```

6. **Send to Kafka**:
   - The transformed data are sent to Kafka.
   - Processor name: 
      - for NiFi 1.x:  ```PublishKafka_2_6```
      - for NiFi 2.x:  ```PublishKafka```

7. **Sent to Redis**:
   -  The transformed data are sent to Redis
   - Processor name: ```PutDistributedMapCache```

## Data flow model:

```mermaid
flowchart TD
	subgraph Main
		A((Generate main FF)) --> B[Split events into separete FFs]
		B --> C[Extract data from FF content]
		C --> D[Count sum and product ]
		D --> E[Data transformation adjustment to proper format]
		E --> F((Send to Kafka))
		E --> G((Sent to Redis))
	end 
```
> FF - flowfile / event

## Data

```JSON
[
{
"A" : "10",
"B" : "20"
},
{
"A" : "15",
"B" : "15"
},
{
"A" : "5",
"B" : "10"
},
{
"A" : "8",
"B" : "15"
},
{
"A" : "25",
"B" : "25"
}
]
```

## END Event

```JSON
{
"A" : "<String>",
"B" : "<String>",
"SUM" : "A + B",
"PRODUCT" : "A * B"
}
```



