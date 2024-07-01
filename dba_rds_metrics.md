
### [Report of the main metrics](https://aws.amazon.com/blogs/database/optimizing-costs-in-amazon-rds/?sc_channel=sm&sc_campaign=Support&sc_publisher=REDDIT&sc_country=global&sc_geo=GLOBAL&sc_outcome=AWS%20Support&sc_content=Support&trk=Support&linkId=410997030)

```python
import boto3
from datetime import datetime, timedelta
import argparse
import pandas as pd
from itertools import chain

# Argument parser for flexibility
parser = argparse.ArgumentParser(description='Check RDS instance metrics and optimizations')
parser.add_argument('--region', type=str, default='us-west-2', help='AWS region')
parser.add_argument('--days', type=int, default=7, help='Number of days for metrics')
parser.add_argument('--output', type=str, default='output.xlsx', help='Output file name for metrics')
args = parser.parse_args()

# AWS session initialization
session = boto3.Session(region_name=args.region)
rds_client = session.client('rds')
cloudwatch_client = session.client('cloudwatch')

# Define time range
end_time = datetime.utcnow()
start_time = end_time - timedelta(days=args.days)

# Function to fetch metrics for a single instance
def get_instance_metrics(instance_id):
    try:
        # Retrieve CPU Utilization metrics
        cpu_response = cloudwatch_client.get_metric_statistics(
            Namespace='AWS/RDS',
            MetricName='CPUUtilization',
            Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
            StartTime=start_time,
            EndTime=end_time,
            Period=86400,  # One day
            Statistics=['Average']
        )
        cpu_avg = sum(data_point['Average'] for data_point in cpu_response['Datapoints']) / len(cpu_response['Datapoints']) if cpu_response['Datapoints'] else 0

        # Retrieve ReadIOPS and WriteIOPS metrics
        read_iops_response = cloudwatch_client.get_metric_statistics(
            Namespace='AWS/RDS',
            MetricName='ReadIOPS',
            Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
            StartTime=start_time,
            EndTime=end_time,
            Period=86400,
            Statistics=['Average']
        )
        read_iops_avg = sum(data_point['Average'] for data_point in read_iops_response['Datapoints']) / len(read_iops_response['Datapoints']) if read_iops_response['Datapoints'] else 0

        write_iops_response = cloudwatch_client.get_metric_statistics(
            Namespace='AWS/RDS',
            MetricName='WriteIOPS',
            Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
            StartTime=start_time,
            EndTime=end_time,
            Period=86400,
            Statistics=['Average']
        )
        write_iops_avg = sum(data_point['Average'] for data_point in write_iops_response['Datapoints']) / len(write_iops_response['Datapoints']) if write_iops_response['Datapoints'] else 0

        total_iops_avg = read_iops_avg + write_iops_avg

        return {
            'Instance Identifier': instance_id,
            'Average CPU (%)': cpu_avg,
            'Average Read IOPS': read_iops_avg,
            'Average Write IOPS': write_iops_avg,
            'Average Total IOPS': total_iops_avg
        }
    except Exception as e:
        print(f"Error retrieving metrics for {instance_id}: {e}")
        return None

# Function to check if an instance is a Read Replica
def is_read_replica(instance):
    return instance['ReadReplicaSourceDBInstanceIdentifier'] is not None

# Function to check if an instance has no connections for 1 month
def no_connections_for_one_month(instance):
    try:
        # Calculate time delta
        last_connection_time = instance['LatestRestorableTime']
        if last_connection_time:
            return (datetime.utcnow() - last_connection_time).days > 30
        else:
            return False
    except Exception as e:
        print(f"Error checking connection time for {instance['DBInstanceIdentifier']}: {e}")
        return False

# Function to apply optimization checks
def apply_optimization_checks(instance_metrics):
    optimized_instances = []
    for metrics in instance_metrics:
        instance_id = metrics['Instance Identifier']
        is_read_rep = is_read_replica(instance_id)
        cpu_utilization = metrics['Average CPU (%)']
        io_throughput = metrics['Average Total IOPS']

        # Check for Read Replica with CPU utilization < 30% and I/O throughput < 30%
        if is_read_rep and cpu_utilization < 30 and io_throughput < 30:
            metrics['Optimization Type'] = 'Read Replica Optimization'

        # Check for Under-utilized Instances - No connections for 1 month, CPU utilization < 5%, and I/O throughput < 5%
        elif no_connections_for_one_month(instance_id) and cpu_utilization < 5 and io_throughput < 5:
            metrics['Optimization Type'] = 'Under-utilized Instance'

        # Check for Right-size Instances - CPU utilization < 30% and I/O throughput < 30%
        elif cpu_utilization < 30 and io_throughput < 30:
            metrics['Optimization Type'] = 'Right-size Instance'

        optimized_instances.append(metrics)

    return optimized_instances

# Function to fetch all RDS instances with pagination
def paginate(func):
    def pager(*args, **kwargs):
        paginator = func(*args, **kwargs)
        for page in paginator:
            yield from page[args[1]]
    return pager

# Retrieve all RDS instances with pagination
try:
    paginate_rds_instances = paginate(rds_client.get_paginator('describe_db_instances').paginate)
    instances = list(chain.from_iterable(paginate_rds_instances()))
except Exception as e:
    print(f"Error retrieving RDS instances: {e}")
    exit()

# Initialize list to store instance metrics
instance_metrics = []

# Process each instance to check optimization criteria
for instance in instances:
    instance_id = instance['DBInstanceIdentifier']
    print(f"Checking metrics for instance: {instance_id}")

    # Fetch metrics for the instance
    metrics = get_instance_metrics(instance_id)
    if metrics:
        instance_metrics.append(metrics)

# Apply optimization checks
optimized_instances = apply_optimization_checks(instance_metrics)

# Convert data list to DataFrame
df = pd.DataFrame(optimized_instances)

# Save DataFrame to Excel file
output_file = args.output
df.to_excel(output_file, index=False)
print(f"Metrics saved to {output_file}")
```

### Explanation and Enhancements

1. **Optimization Checks Functions**: 
   - **`is_read_replica`**: Checks if an instance is a Read Replica by inspecting `ReadReplicaSourceDBInstanceIdentifier`.
   - **`no_connections_for_one_month`**: Calculates if an instance has had no connections for 1 month using `LatestRestorableTime`.

2. **Apply Optimization Checks**: 
   - `apply_optimization_checks` function iterates through `instance_metrics` and applies the respective checks for Read Replica Optimization, Under-utilized Instances, and Right-size Instances based on CPU utilization and I/O throughput criteria.

3. **Integration with Existing Logic**: 
   - Integrated these functions into the main script flow after fetching instance metrics to evaluate each instance against optimization criteria.

4. **Output**: 
   - Data is processed, optimized instances are categorized, and results are saved into an Excel file (`output.xlsx`) for further analysis and reporting.
