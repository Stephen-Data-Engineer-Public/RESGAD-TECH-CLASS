```python
import csv

class DataIngestor:
    base_path = "/data/jobs"  # shared across all jobs

    def __init__(self, source_path, target_table):
        self.source_path = source_path
        self.target_table = target_table

    # Instance Method: main pipeline
    def ingest(self):
        data = self._read_file()
        cleaned = self._transform(data)
        self._write_to_db(cleaned)

    # Instance Method: file reader
    def _read_file(self):
        print(f"Reading from: {self.source_path}")
        try:
            with open(self.source_path, newline='') as csvfile:
                reader = csv.reader(csvfile)
                return [row for row in reader]
        except FileNotFoundError:
            print("File not found.")
            return []

    # Instance Method: data transformer
    def _transform(self, data):
        print("Cleaning data...")
        return [row for row in data if self.validate_row(row)]

    # Instance Method: mock DB writer
    def _write_to_db(self, data):
        print(f"Writing {len(data)} rows to {self.target_table}")
        for row in data:
            print(" > ", row)

    # Class Method: factory method to create standard jobs
    @classmethod
    def from_job_type(cls, job_type):
        if job_type == "users":
            return cls(f"{cls.base_path}/users.csv", "user_table")
        elif job_type == "sales":
            return cls(f"{cls.base_path}/sales.csv", "sales_table")
        else:
            raise ValueError("Unknown job type.")

    # Static Method: validate a row
    @staticmethod
    def validate_row(row):
        # Ensure row isn't empty and all fields are non-empty
        return row and all(field.strip() for field in row)


# Create an ingestion job manually
job1 = DataIngestor("sample_data/users.csv", "user_table")
job1.ingest()

# Create using the class method
job2 = DataIngestor.from_job_type("users")
job2.ingest() # This will create the same object as:

DataIngestor("/data/jobs/users.csv", "user_table")


```