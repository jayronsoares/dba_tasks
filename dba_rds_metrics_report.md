
### [AWS RDS Report of the main metrics](https://aws.amazon.com/blogs/database/optimizing-costs-in-amazon-rds/?sc_channel=sm&sc_campaign=Support&sc_publisher=REDDIT&sc_country=global&sc_geo=GLOBAL&sc_outcome=AWS%20Support&sc_content=Support&trk=Support&linkId=410997030)

### 1. **Cost Optimization**

- **Read Replica Management**: Identifies read replica instances with low CPU utilization and I/O throughput. If these metrics are below 30%, the script suggests shutting down or downsizing these replicas to save costs.
- **Under-Utilized Instances**: Detects instances that have no connections for over a month and very low CPU and I/O usage. Alerts are generated, and in non-production environments, it takes further action by suggesting or taking snapshots and stopping the instances if no corrective measures are taken.
- **Right-Sizing**: Flags instances where CPU and I/O metrics are consistently low, suggesting potential downsizing to reduce costs while maintaining sufficient performance.

### Detailed Breakdown of Script Actions

#### **Production Environment Script**

- **Identify Read Replicas**:
  - Criteria: CPU utilization < 30% and I/O throughput < 30%.
  - Action: Suggests considering shutting down or downsizing the read replica to save costs.
  
- **Under-Utilized Instances**:
  - Criteria: No connections for 1 month, CPU utilization < 5%, and I/O throughput < 5%.
  - Action: Alerts the owner to take action. If no action is taken, further escalation might be needed.

- **Right-Sizing Instances**:
  - Criteria: CPU utilization < 30% and I/O throughput < 30%.
  - Action: Suggests right-sizing to a smaller instance type to save on costs while maintaining performance.

#### **Non-Production Environment Script**

- **Identify Read Replicas**:
  - Same criteria and action as in the production environment.

- **Under-Utilized Instances**:
  - Criteria: No connections for 1 month, CPU utilization < 5%, and I/O throughput < 5%.
  - Action: Alerts the owner and suggests taking a snapshot and stopping the instance if no action is taken. This helps in minimizing costs in non-production environments.

- **Right-Sizing Instances**:
  - Criteria: CPU utilization < 50% and I/O throughput < 50%.
  - Action: Suggests right-sizing to a more cost-effective instance type.

### How the Script Helps in Cost Optimization

1. **Efficient Resource Usage**: By identifying instances that are under-utilized or oversized, the script helps ensure that resources are used efficiently, avoiding unnecessary expenditure.
2. **Automated Cost Control**: The script automates the process of monitoring and controlling costs, reducing the need for manual intervention and helping maintain a lean cloud infrastructure.
3. **Scalable Monitoring**: The use of pagination and efficient data retrieval ensures that the script can handle large numbers of instances without incurring additional costs, making it suitable for large-scale environments.

#### `production_rds_cost_optimization.py`

```python
import boto3
import itertools
import datetime
import pandas as pd

# AWS credentials
aws_access_key_id = 'YOUR_AWS_ACCESS_KEY'
aws_secret_access_key = 'YOUR_AWS_SECRET_KEY'
aws_session_token = 'YOUR_AWS_SESSION_TOKEN'
aws_region = 'YOUR_AWS_REGION'  # e.g., 'us-west-2'

# Initialize Boto3 clients
client = boto3.client(
    'rds',
    aws_access_key_id=aws_access_key_id,
    aws_secret_access_key=aws_secret_access_key,
    aws_session_token=aws_session_token,
    region_name=aws_region
)

# Utility function to get paginated results
def paginate(method, **kwargs):
    paginator = client.get_paginator(method)
    for page in paginator.paginate(**kwargs):
        yield from page['DBInstances']

# Mock function to get CPU and I/O metrics
def get_metrics(instance_id):
    # Replace with actual AWS CloudWatch logic to fetch metrics
    return {
        'CPUUtilization': 10,  # Mock value
        'IOThroughput': 10,    # Mock value
        'LastConnection': datetime.datetime.now() - datetime.timedelta(days=35)  # Mock value
    }

# Function to monitor and optimize RDS instances
def monitor_production_rds():
    today = datetime.datetime.now()
    threshold_date = today - datetime.timedelta(days=30)

    metrics = []

    for instance in itertools.chain(paginate('describe_db_instances')):
        instance_id = instance['DBInstanceIdentifier']
        instance_class = instance['DBInstanceClass']
        is_read_replica = 'ReadReplicaSourceDBInstanceIdentifier' in instance
        environment = instance.get('Environment', 'production')

        # Fetch CPU and I/O metrics
        instance_metrics = get_metrics(instance_id)
        cpu_utilization = instance_metrics['CPUUtilization']
        io_throughput = instance_metrics['IOThroughput']
        last_connection = instance_metrics['LastConnection']
        
        # Check for read replica instances
        if is_read_replica and cpu_utilization < 30 and io_throughput < 30:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': environment,
                'Reason': 'Read Replica, CPU < 30%, I/O < 30%',
                'Action': 'Consider shutting down or downsizing'
            })
        
        # Check for under-utilized instances
        if last_connection < threshold_date and cpu_utilization < 5 and io_throughput < 5:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': environment,
                'Reason': 'Under-utilized, no connections for 1 month, CPU < 5%, I/O < 5%',
                'Action': 'Alert owner'
            })
        
        # Check for instances to be right-sized
        if cpu_utilization < 30 and io_throughput < 30:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': environment,
                'Reason': 'Right-size, CPU < 30%, I/O < 30%',
                'Action': 'Consider right-sizing'
            })

    df = pd.DataFrame(metrics)
    df.to_excel('production_rds_metrics.xlsx', index=False)

if __name__ == '__main__':
    monitor_production_rds()
```

### Updated Python Script for Non-Production Environment
#### `non_production_rds_cost_optimization.py`

```python
import boto3
import itertools
import datetime
import pandas as pd

# AWS credentials
aws_access_key_id = 'YOUR_AWS_ACCESS_KEY'
aws_secret_access_key = 'YOUR_AWS_SECRET_KEY'
aws_session_token = 'YOUR_AWS_SESSION_TOKEN'
aws_region = 'YOUR_AWS_REGION'  # e.g., 'us-west-2'

# Initialize Boto3 clients
client = boto3.client(
    'rds',
    aws_access_key_id=aws_access_key_id,
    aws_secret_access_key=aws_secret_access_key,
    aws_session_token=aws_session_token,
    region_name=aws_region
)

# Utility function to get paginated results
def paginate(method, **kwargs):
    paginator = client.get_paginator(method)
    for page in paginator.paginate(**kwargs):
        yield from page['DBInstances']

# Mock function to get CPU and I/O metrics
def get_metrics(instance_id):
    # Replace with actual AWS CloudWatch logic to fetch metrics
    return {
        'CPUUtilization': 10,  # Mock value
        'IOThroughput': 10,    # Mock value
        'LastConnection': datetime.datetime.now() - datetime.timedelta(days=35)  # Mock value
    }

# Function to monitor and optimize RDS instances
def monitor_non_production_rds():
    today = datetime.datetime.now()
    threshold_date = today - datetime.timedelta(days=30)

    metrics = []

    for instance in itertools.chain(paginate('describe_db_instances')):
        instance_id = instance['DBInstanceIdentifier']
        instance_class = instance['DBInstanceClass']
        is_read_replica = 'ReadReplicaSourceDBInstanceIdentifier' in instance
        environment = instance.get('Environment', 'non-production')

        # Fetch CPU and I/O metrics
        instance_metrics = get_metrics(instance_id)
        cpu_utilization = instance_metrics['CPUUtilization']
        io_throughput = instance_metrics['IOThroughput']
        last_connection = instance_metrics['LastConnection']
        
        # Check for read replica instances
        if is_read_replica and cpu_utilization < 30 and io_throughput < 30:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': environment,
                'Reason': 'Read Replica, CPU < 30%, I/O < 30%',
                'Action': 'Consider shutting down or downsizing'
            })
        
        # Check for under-utilized instances
        if last_connection < threshold_date and cpu_utilization < 5 and io_throughput < 5:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': environment,
                'Reason': 'Under-utilized, no connections for 1 month, CPU < 5%, I/O < 5%',
                'Action': 'Alert owner, take snapshot, and stop instance'
            })
            # Normally, the next two lines would perform actions. Uncomment to activate.
            # take_snapshot(instance_id)
            # stop_instance(instance_id)
        
        # Check for instances to be right-sized
        if cpu_utilization < 50 and io_throughput < 50:
            metrics.append({
                'Instance ID': instance_id,
                'Instance Class': instance_class,
                'Environment': environment,
                'Reason': 'Right-size, CPU < 50%, I/O < 50%',
                'Action': 'Consider right-sizing'
            })

    df = pd.DataFrame(metrics)
    df.to_excel('non_production_rds_metrics.xlsx', index=False)

if __name__ == '__main__':
    monitor_non_production_rds()
```
