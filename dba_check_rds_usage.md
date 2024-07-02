## Check AWS RDS Report 

```python
import boto3
import itertools
import datetime
import pandas as pd

# AWS credentials and region
aws_access_key_id = 'YOUR_AWS_ACCESS_KEY'
aws_secret_access_key = 'YOUR_AWS_SECRET_KEY'
aws_session_token = 'YOUR_AWS_SESSION_TOKEN'
aws_region = 'YOUR_AWS_REGION'

# Initialize Boto3 clients
rds_client = boto3.client(
    'rds',
    aws_access_key_id=aws_access_key_id,
    aws_secret_access_key=aws_secret_access_key,
    aws_session_token=aws_session_token,
    region_name=aws_region
)

cloudwatch_client = boto3.client(
    'cloudwatch',
    aws_access_key_id=aws_access_key_id,
    aws_secret_access_key=aws_secret_access_key,
    aws_session_token=aws_session_token,
    region_name=aws_region
)

# Utility function to get paginated results
def paginate(method, **kwargs):
    paginator = rds_client.get_paginator(method)
    for page in paginator.paginate(**kwargs):
        yield from page['DBInstances']

# Function to get CloudWatch metrics for an RDS instance
def get_cloudwatch_metrics(instance_id):
    end_time = datetime.datetime.now()
    start_time = end_time - datetime.timedelta(days=30)  # 30 days ago

    metrics = {
        'CPUUtilization': None,
        'IOThroughput': None,
        'ReadLatency': None,
        'WriteLatency': None,
        'FreeStorageSpace': None,
        'DatabaseConnections': None
    }

    # Fetch CPUUtilization metric
    cpu_response = cloudwatch_client.get_metric_statistics(
        Namespace='AWS/RDS',
        MetricName='CPUUtilization',
        Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=86400,  # Daily data points
        Statistics=['Average']
    )
    if cpu_response['Datapoints']:
        metrics['CPUUtilization'] = cpu_response['Datapoints'][-1]['Average']

    # Fetch IOThroughput metric (example, adjust as per actual metrics needed)
    io_response = cloudwatch_client.get_metric_statistics(
        Namespace='AWS/RDS',
        MetricName='WriteThroughput',  # Example metric, change as needed
        Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=86400,  # Daily data points
        Statistics=['Average']
    )
    if io_response['Datapoints']:
        metrics['IOThroughput'] = io_response['Datapoints'][-1]['Average']

    # Fetch ReadLatency metric
    read_latency_response = cloudwatch_client.get_metric_statistics(
        Namespace='AWS/RDS',
        MetricName='ReadLatency',
        Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=86400,  # Daily data points
        Statistics=['Average']
    )
    if read_latency_response['Datapoints']:
        metrics['ReadLatency'] = read_latency_response['Datapoints'][-1]['Average']

    # Fetch WriteLatency metric
    write_latency_response = cloudwatch_client.get_metric_statistics(
        Namespace='AWS/RDS',
        MetricName='WriteLatency',
        Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=86400,  # Daily data points
        Statistics=['Average']
    )
    if write_latency_response['Datapoints']:
        metrics['WriteLatency'] = write_latency_response['Datapoints'][-1]['Average']

    # Fetch FreeStorageSpace metric
    storage_response = rds_client.describe_db_instances(DBInstanceIdentifier=instance_id)
    metrics['FreeStorageSpace'] = storage_response['DBInstances'][0]['FreeStorageSpace'] / (1024 ** 3)  # Convert bytes to GB

    # Fetch DatabaseConnections metric
    db_connections_response = cloudwatch_client.get_metric_statistics(
        Namespace='AWS/RDS',
        MetricName='DatabaseConnections',
        Dimensions=[{'Name': 'DBInstanceIdentifier', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=86400,  # Daily data points
        Statistics=['Average']
    )
    if db_connections_response['Datapoints']:
        metrics['DatabaseConnections'] = db_connections_response['Datapoints'][-1]['Average']

    return metrics

# Function to monitor and optimize RDS instances
def monitor_rds_instances():
    today = datetime.datetime.now()
    threshold_date = today - datetime.timedelta(days=30)

    metrics = []

    for instance in itertools.chain(paginate('describe_db_instances')):
        instance_id = instance['DBInstanceIdentifier']
        instance_class = instance['DBInstanceClass']
        is_read_replica = 'ReadReplicaSourceDBInstanceIdentifier' in instance

        # Fetch metrics from CloudWatch
        instance_metrics = get_cloudwatch_metrics(instance_id)
        cpu_utilization = instance_metrics['CPUUtilization']
        io_throughput = instance_metrics['IOThroughput']
        read_latency = instance_metrics['ReadLatency']
        write_latency = instance_metrics['WriteLatency']
        free_storage_space = instance_metrics['FreeStorageSpace']
        db_connections = instance_metrics['DatabaseConnections']

        # Check for read replica instances
        if is_read_replica and cpu_utilization < 30 and io_throughput < 30:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': 'Read Replica',
                'CPU Utilization (%)': cpu_utilization,
                'I/O Throughput (IOPS)': io_throughput,
                'Read Latency (ms)': read_latency,
                'Write Latency (ms)': write_latency,
                'Free Storage Space (GB)': free_storage_space,
                'Database Connections': db_connections,
                'Action': 'Transfer load to primary and shut down or downsize'
            })

        # Check for under-utilized instances
        if cpu_utilization < 5 and io_throughput < 5 and not instance['DBInstanceStatus'].startswith('stopped'):
            instance_created_date = instance['InstanceCreateTime']
            if instance_created_date < threshold_date:
                if instance.get('DBInstanceClass', '').startswith('db.t'):
                    environment = 'Non-production'
                    action = 'Alert owner, escalate, take a snapshot, and shut down if no action is taken within a given time window'
                else:
                    environment = 'Production'
                    action = 'Alert owner and escalate if no action is taken within a given time window'
                
                metrics.append({
                    'Instance ID': instance_id,
                    'Instance Class': instance_class,
                    'Environment': environment,
                    'CPU Utilization (%)': cpu_utilization,
                    'I/O Throughput (IOPS)': io_throughput,
                    'Read Latency (ms)': read_latency,
                    'Write Latency (ms)': write_latency,
                    'Free Storage Space (GB)': free_storage_space,
                    'Database Connections': db_connections,
                    'Action': action
                })

        # Check for right-sizing instances
        if cpu_utilization < 30 and io_throughput < 30:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': 'Right-size (CPU < 30% and I/O < 30%)',
                'CPU Utilization (%)': cpu_utilization,
                'I/O Throughput (IOPS)': io_throughput,
                'Read Latency (ms)': read_latency,
                'Write Latency (ms)': write_latency,
                'Free Storage Space (GB)': free_storage_space,
                'Database Connections': db_connections,
                'Action': 'Alert owner and escalate if no action is taken within a given time window'
            })
        elif cpu_utilization < 50 and io_throughput < 50:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': 'Right-size (CPU < 50% and I/O < 50%)',
                'CPU Utilization (%)': cpu_utilization,
                'I/O Throughput (IOPS)': io_throughput,
                'Read Latency (ms)': read_latency,
                'Write Latency (ms)': write_latency,
                'Free Storage Space (GB)': free_storage_space,
                'Database Connections': db_connections,
                'Action': 'Alert owner, take a snapshot, and downsize if no action is taken within the given time window'
            })

    # Create a DataFrame and save to Excel
    df = pd.DataFrame(metrics)
    df.to_excel('rds_metrics_report.xlsx', index=False)

# Entry point for the script
if __name__ == "__main__":
    monitor_rds_instances()
    print("RDS instance monitoring and optimization script executed successfully.")
```
