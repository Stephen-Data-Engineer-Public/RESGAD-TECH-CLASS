# Batch Processing using Python!
This example demonstrates a basic ETL (Extract, Transform, Load) process using batch ingestion in Python.
It generates random sample data, processes it in chunks (batches), applies a simple transformation to each record, and loads the results into a Pandas DataFrame for display.

![image](https://github.com/user-attachments/assets/abf8f61f-beca-4662-8d8f-a4f0ea093cae)

### Below is a high-level overview of the key functions used in this ETL pipeline:

- **`generate_sample_data(num_records)`**  
  Creates a list of mock records, each with a random `id` and `value`. This simulates incoming raw data.

- **`process_in_batches(data, batch_size)`**  
  Splits the full dataset into smaller chunks (batches) of size `batch_size` using a generator.

- **`transform_data(batch)`**  
  Transforms each record in a batch by adding a new field `transformed_value` (`value × 1.1`).

- **`load_data(batch)`**  
  Simulates loading the transformed records into a database by printing them to the console.

- **`main()`**  
  Orchestrates the ETL flow: generates data, processes it in batches, transforms each batch, and "loads" it.

------
### **Generate Sample Data Function**

```python
import random
import uuid

def generate_sample_data():
  data = []
  for i in range(100):
    record = {
        "id": str(uuid.uuid4()),
        "value": random.random()            
    }
    data.append(record)
  return data
```
**Design Considerations for Sample Data Generator Function:**
When building a function to generate sample data, it’s important to consider structure, flexibility, and practical processing needs. Here's a breakdown of the design choices we made:

**1. Data Structure Choice**
Each data record is represented as a Python dictionary, storing key-value pairs. This mirrors tabular data with named columns, making records self-describing and easy to manipulate or convert.

**2. Flexible Record Count**
The function accepts a `num_records` parameter, allowing you to specify how many records to generate. We enforce a cap of 200 records to keep resource usage reasonable.

**3. Using a List as a Container**
All generated records are stored in a list, which maintains order and facilitates iteration, slicing, and batch processing — crucial for downstream tasks like transformation or loading.

**4. Record Generation Loop**
A simple `for` loop runs for the requested number of records, appending newly created dictionaries to the list, progressively building the dataset.

**5. Function Purpose**
This function acts as a lightweight simulated data source, useful for testing data pipelines, practicing batch processing, and experimenting with transformation logic without relying on external databases.

------
### **Process Data in Batches**

```python
import time

def process_data(data, batch_size):
  for i in range(0, len(data), batch_size):
    yield data[i:i + batch_size]
    time.sleep(2)
```
**Design Considerations for Process Data in Batches Function :** 

**1. Batch Processing Logic**
The function splits the input data into smaller chunks (batches) of size batch_size using slicing within a `for` loop. This approach enables handling large datasets in manageable pieces, improving memory efficiency and allowing incremental processing.

**2. Use of a Generator (yield)**
Instead of returning all batches at once, the function yields one batch at a time. This lazy evaluation means the caller processes data batch-by-batch, which is ideal for data pipelines or situations where full dataset loading isn’t practical.

**3. Introducing Delays with time.sleep(2)**
After yielding each batch, the function pauses for 2 seconds using `time.sleep(2)`. This simulates realistic batch processing scenarios where data might be received or processed at intervals.

------
### **Transform Data Function**

```python
def transform_data(batch):
    transformed_batch = []
    for record in batch:
        transformed_record = {
            'id': record['id'],
            'value': record['value'],
            'transformed_value': record['value'] * 1.1
        }
        transformed_batch.append(transformed_record)
    return transformed_batch
```

**Design Considerations for transform data Function:**

**1. Batch-Based Transformation**
The function accepts a batch (a list of records) and processes each record individually. This design supports incremental data processing, making it suitable for use with streamed or chunked data.

**2. Record-Level Processing**
Each record is represented as a dictionary. The function creates a new transformed dictionary for each record to keep the original data immutable and avoid side effects.

**3. Calculated Field Addition**
The transformation adds a new field, 'transformed_value', which is computed as the original 'value' multiplied by 1.1. This illustrates how derived data or feature engineering can be integrated into the pipeline.

------
### **Load Data Function**

```python
import pandas as pd

def load_data(transformed_records):
    df = pd.DataFrame(transformed_records)
    # Example: Save to CSV 
    df.to_csv("transformed_data.csv", index=False)
    return df
```

**Design Considerations for load data Function:**

**1. Structured Output with Pandas DataFrame**
The function converts the list of transformed records into a pandas.DataFrame, creating a structured, tabular representation of the data. This format is widely used in data workflows for its flexibility and powerful analysis capabilities.

**2. Data Persistence via CSV Export**
The function saves the DataFrame to a CSV file using df.to_csv(). This makes the data portable and easy to inspect, share, or feed into other tools and systems. Using index=False avoids writing the DataFrame's row index as a column in the CSV, keeping the file clean.

**3. Function Return Value**
After saving, the function returns the DataFrame, allowing further use in our process (e.g., visualization, validation, or further transformation).

------
### **The main Function**

```python
def main():
    whole_data = []
    for batch in process_data(data, batch_size):
        result = transform_data(batch)
        whole_data.extend(result)
    df = load_data(whole_data)
    return df

df = main()
display(df)
```

**Design Considerations for main Function:**

**1. Pipeline Coordination**
The main() function serves as the central controller for the ETL (Extract → Transform → Load) process. It organizes the execution flow, making the script modular, readable, and maintainable.

**2. Batch-Oriented Processing**
Data is processed in batches via process_data(), which helps simulate real-world scenarios where data arrives or is processed incrementally. This design improves scalability and mirrors production workflows.

**3. Incremental Aggregation**
Instead of processing everything at once, transformed results are accumulated in whole_data using .extend(). This keeps the transformation step efficient and memory-aware.

**4. Final Loading Step**
Once all data is transformed, whole_data is passed to load_data() for conversion to a structured format (DataFrame) and optional persistence (e.g., to CSV). Returning the DataFrame allows flexibility for downstream tasks like visualization or further processing.




**Final Consideration:**
By delegating each step to its own function (generate_sample_data, process_data, transform_data, load_data), main() promotes a clean and modular design. This makes the codebase easier to test, debug, and scale.

