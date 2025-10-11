# IBM Cloud Site Reliability Engineer (C1000-179) - Reliability & Performance Optimization

## Table of Contents
- [SRE Principles and Practices](#sre-principles-and-practices)
- [Service Level Objectives (SLOs)](#service-level-objectives-slos)
- [Capacity Planning](#capacity-planning)
- [Performance Optimization](#performance-optimization)
- [Chaos Engineering](#chaos-engineering)
- [Load Testing and Benchmarking](#load-testing-and-benchmarking)
- [Database Performance](#database-performance)
- [Network Optimization](#network-optimization)
- [Auto-Scaling Strategies](#auto-scaling-strategies)
- [Cost Optimization](#cost-optimization)

---

## SRE Principles and Practices

### Core SRE Concepts
```yaml
# sre-principles.yaml
sre_philosophy:
  error_budgets:
    description: "Acceptable level of unreliability"
    calculation: "1 - SLO"
    example: "99.9% SLO = 0.1% error budget = 43.2 min/month"

  toil_reduction:
    target: "< 50% of SRE time"
    automation_first: true
    eliminate_manual_tasks: true

  blameless_postmortems:
    focus: "systemic issues, not individuals"
    documentation: "required for all incidents"
    action_items: "tracked and completed"

  gradual_rollouts:
    canary_deployments: true
    feature_flags: true
    progressive_delivery: true

slo_targets:
  tier_1_critical:
    availability: "99.99%"
    latency_p50: "50ms"
    latency_p99: "200ms"
    error_rate: "0.01%"

  tier_2_important:
    availability: "99.9%"
    latency_p50: "100ms"
    latency_p99: "500ms"
    error_rate: "0.1%"

  tier_3_standard:
    availability: "99.5%"
    latency_p50: "200ms"
    latency_p99: "1000ms"
    error_rate: "0.5%"
```

### Error Budget Management
```python
# error_budget.py
from datetime import datetime, timedelta
import json

class ErrorBudgetCalculator:
    def __init__(self, slo_percentage):
        self.slo = slo_percentage / 100.0
        self.error_budget = 1.0 - self.slo

    def calculate_monthly_budget(self):
        """Calculate monthly error budget in minutes"""
        minutes_per_month = 30 * 24 * 60  # ~43,200 minutes
        allowed_downtime = minutes_per_month * self.error_budget
        return {
            'total_minutes': minutes_per_month,
            'allowed_downtime_minutes': round(allowed_downtime, 2),
            'allowed_downtime_hours': round(allowed_downtime / 60, 2),
            'slo_percentage': self.slo * 100,
            'error_budget_percentage': self.error_budget * 100
        }

    def calculate_remaining_budget(self, downtime_minutes):
        """Calculate remaining error budget"""
        budget = self.calculate_monthly_budget()
        remaining = budget['allowed_downtime_minutes'] - downtime_minutes
        remaining_percentage = (remaining / budget['allowed_downtime_minutes']) * 100

        return {
            'total_budget': budget['allowed_downtime_minutes'],
            'consumed': downtime_minutes,
            'remaining': round(remaining, 2),
            'remaining_percentage': round(remaining_percentage, 2),
            'is_exhausted': remaining <= 0
        }

    def should_halt_releases(self, current_downtime):
        """Determine if releases should be halted"""
        remaining = self.calculate_remaining_budget(current_downtime)
        threshold = 20  # Halt if less than 20% budget remaining

        return {
            'halt_releases': remaining['remaining_percentage'] < threshold,
            'reason': f"Error budget at {remaining['remaining_percentage']}%",
            'recommended_action': 'Focus on reliability' if remaining['remaining_percentage'] < threshold else 'Continue feature development'
        }

# Usage
if __name__ == "__main__":
    # 99.9% SLO
    calculator = ErrorBudgetCalculator(99.9)

    # Monthly budget
    budget = calculator.calculate_monthly_budget()
    print(f"Monthly SLO Budget:")
    print(f"  Total minutes: {budget['total_minutes']}")
    print(f"  Allowed downtime: {budget['allowed_downtime_minutes']} minutes ({budget['allowed_downtime_hours']} hours)")

    # After 30 minutes of downtime this month
    remaining = calculator.calculate_remaining_budget(30)
    print(f"\nRemaining Budget:")
    print(f"  Consumed: {remaining['consumed']} minutes")
    print(f"  Remaining: {remaining['remaining']} minutes ({remaining['remaining_percentage']}%)")

    # Should we halt releases?
    decision = calculator.should_halt_releases(30)
    print(f"\nRelease Decision:")
    print(f"  Halt releases: {decision['halt_releases']}")
    print(f"  Action: {decision['recommended_action']}")
```

---

## Service Level Objectives (SLOs)

### Defining and Tracking SLOs
```python
# slo_tracker.py
from datetime import datetime, timedelta
from collections import defaultdict
import statistics

class SLOTracker:
    def __init__(self, slo_config):
        self.config = slo_config
        self.measurements = defaultdict(list)

    def record_request(self, endpoint, latency_ms, is_error=False):
        """Record a single request"""
        self.measurements[endpoint].append({
            'timestamp': datetime.now(),
            'latency_ms': latency_ms,
            'is_error': is_error
        })

    def calculate_availability(self, endpoint, time_window_hours=24):
        """Calculate availability SLI"""
        cutoff = datetime.now() - timedelta(hours=time_window_hours)
        recent = [m for m in self.measurements[endpoint]
                 if m['timestamp'] > cutoff]

        if not recent:
            return None

        total_requests = len(recent)
        successful_requests = sum(1 for m in recent if not m['is_error'])
        availability = (successful_requests / total_requests) * 100

        slo = self.config[endpoint]['availability_slo']

        return {
            'endpoint': endpoint,
            'time_window_hours': time_window_hours,
            'total_requests': total_requests,
            'successful_requests': successful_requests,
            'availability_percentage': round(availability, 3),
            'slo_percentage': slo,
            'meeting_slo': availability >= slo,
            'margin': round(availability - slo, 3)
        }

    def calculate_latency_sli(self, endpoint, time_window_hours=24):
        """Calculate latency SLI"""
        cutoff = datetime.now() - timedelta(hours=time_window_hours)
        recent = [m['latency_ms'] for m in self.measurements[endpoint]
                 if m['timestamp'] > cutoff and not m['is_error']]

        if not recent:
            return None

        recent.sort()
        p50 = recent[len(recent) // 2]
        p95 = recent[int(len(recent) * 0.95)]
        p99 = recent[int(len(recent) * 0.99)]

        config = self.config[endpoint]

        return {
            'endpoint': endpoint,
            'time_window_hours': time_window_hours,
            'sample_count': len(recent),
            'p50_ms': round(p50, 2),
            'p95_ms': round(p95, 2),
            'p99_ms': round(p99, 2),
            'p50_slo': config['latency_p50_ms'],
            'p99_slo': config['latency_p99_ms'],
            'meeting_p50_slo': p50 <= config['latency_p50_ms'],
            'meeting_p99_slo': p99 <= config['latency_p99_ms']
        }

    def generate_slo_report(self, time_window_hours=24):
        """Generate comprehensive SLO report"""
        report = {
            'generated_at': datetime.now().isoformat(),
            'time_window_hours': time_window_hours,
            'endpoints': {}
        }

        for endpoint in self.measurements.keys():
            availability = self.calculate_availability(endpoint, time_window_hours)
            latency = self.calculate_latency_sli(endpoint, time_window_hours)

            report['endpoints'][endpoint] = {
                'availability': availability,
                'latency': latency,
                'overall_status': 'HEALTHY' if (availability['meeting_slo'] and
                                              latency['meeting_p99_slo']) else 'DEGRADED'
            }

        return report

# Configuration
slo_config = {
    '/api/users': {
        'availability_slo': 99.9,
        'latency_p50_ms': 50,
        'latency_p99_ms': 200
    },
    '/api/products': {
        'availability_slo': 99.95,
        'latency_p50_ms': 100,
        'latency_p99_ms': 500
    },
    '/api/orders': {
        'availability_slo': 99.99,
        'latency_p50_ms': 75,
        'latency_p99_ms': 300
    }
}

# Usage
tracker = SLOTracker(slo_config)

# Simulate requests
import random
for _ in range(10000):
    endpoint = random.choice(list(slo_config.keys()))
    latency = random.gauss(50, 20)  # Normal distribution
    is_error = random.random() < 0.001  # 0.1% error rate
    tracker.record_request(endpoint, max(1, latency), is_error)

# Generate report
report = tracker.generate_slo_report(24)
for endpoint, data in report['endpoints'].items():
    print(f"\n{endpoint}:")
    print(f"  Availability: {data['availability']['availability_percentage']}% (SLO: {data['availability']['slo_percentage']}%)")
    print(f"  P99 Latency: {data['latency']['p99_ms']}ms (SLO: {data['latency']['p99_slo']}ms)")
    print(f"  Status: {data['overall_status']}")
```

---

## Capacity Planning

### Resource Capacity Analysis
```python
# capacity_planning.py
from datetime import datetime, timedelta
import numpy as np
from sklearn.linear_model import LinearRegression

class CapacityPlanner:
    def __init__(self):
        self.historical_data = []

    def add_measurement(self, timestamp, cpu_percent, memory_percent,
                       requests_per_second):
        """Add capacity measurement"""
        self.historical_data.append({
            'timestamp': timestamp,
            'cpu_percent': cpu_percent,
            'memory_percent': memory_percent,
            'requests_per_second': requests_per_second
        })

    def predict_capacity_needs(self, days_ahead=30, growth_rate=0.1):
        """Predict future capacity requirements"""
        if len(self.historical_data) < 7:
            return None

        # Prepare data for linear regression
        timestamps = [(d['timestamp'] - self.historical_data[0]['timestamp']).days
                     for d in self.historical_data]
        cpu_values = [d['cpu_percent'] for d in self.historical_data]
        memory_values = [d['memory_percent'] for d in self.historical_data]
        rps_values = [d['requests_per_second'] for d in self.historical_data]

        # Fit models
        X = np.array(timestamps).reshape(-1, 1)

        cpu_model = LinearRegression().fit(X, cpu_values)
        memory_model = LinearRegression().fit(X, memory_values)
        rps_model = LinearRegression().fit(X, rps_values)

        # Predict
        future_day = timestamps[-1] + days_ahead
        future_X = np.array([[future_day]])

        predicted_cpu = cpu_model.predict(future_X)[0]
        predicted_memory = memory_model.predict(future_X)[0]
        predicted_rps = rps_model.predict(future_X)[0]

        # Current capacity
        current = self.historical_data[-1]

        return {
            'current': {
                'cpu_percent': current['cpu_percent'],
                'memory_percent': current['memory_percent'],
                'requests_per_second': current['requests_per_second']
            },
            'predicted': {
                'cpu_percent': round(predicted_cpu, 2),
                'memory_percent': round(predicted_memory, 2),
                'requests_per_second': round(predicted_rps, 2)
            },
            'days_ahead': days_ahead,
            'recommendations': self._generate_recommendations(
                predicted_cpu, predicted_memory, predicted_rps
            )
        }

    def _generate_recommendations(self, cpu, memory, rps):
        """Generate scaling recommendations"""
        recommendations = []

        # CPU threshold: 70%
        if cpu > 70:
            scale_factor = cpu / 70
            recommendations.append({
                'resource': 'CPU',
                'action': 'SCALE_UP',
                'current_utilization': f"{cpu}%",
                'recommended_scale_factor': round(scale_factor, 2),
                'urgency': 'HIGH' if cpu > 85 else 'MEDIUM'
            })

        # Memory threshold: 75%
        if memory > 75:
            scale_factor = memory / 75
            recommendations.append({
                'resource': 'Memory',
                'action': 'SCALE_UP',
                'current_utilization': f"{memory}%",
                'recommended_scale_factor': round(scale_factor, 2),
                'urgency': 'HIGH' if memory > 90 else 'MEDIUM'
            })

        # RPS capacity
        current_max_rps = self.historical_data[-1]['requests_per_second']
        if rps > current_max_rps * 0.8:
            recommendations.append({
                'resource': 'Application Instances',
                'action': 'SCALE_OUT',
                'predicted_rps': round(rps, 2),
                'current_capacity': current_max_rps,
                'recommended_additional_instances': max(1, int((rps / current_max_rps) - 1)),
                'urgency': 'MEDIUM'
            })

        if not recommendations:
            recommendations.append({
                'action': 'NO_ACTION_NEEDED',
                'message': 'Current capacity is sufficient'
            })

        return recommendations

# Terraform for auto-scaling based on capacity
capacity_tf = '''
# capacity-based-autoscaling.tf
resource "ibm_is_instance_group" "app_servers" {
  name              = "app-server-group"
  instance_template = ibm_is_instance_template.app_template.id
  instance_count    = 3
  subnets           = [ibm_is_subnet.subnet1.id]
  load_balancer     = ibm_is_lb.app_lb.id
  load_balancer_pool = ibm_is_lb_pool.app_pool.id

  application_port = 8080
}

resource "ibm_is_instance_group_manager" "app_manager" {
  name                 = "app-autoscale-manager"
  instance_group       = ibm_is_instance_group.app_servers.id
  manager_type         = "autoscale"
  aggregation_window   = 120
  cooldown             = 300
  max_membership_count = 10
  min_membership_count = 3
}

# CPU-based scaling policy
resource "ibm_is_instance_group_manager_policy" "cpu_policy" {
  instance_group         = ibm_is_instance_group.app_servers.id
  instance_group_manager = ibm_is_instance_group_manager.app_manager.manager_id
  name                   = "cpu-scaling-policy"
  metric_type            = "cpu"
  metric_value           = 70
  policy_type            = "target"
}

# Memory-based scaling policy
resource "ibm_is_instance_group_manager_policy" "memory_policy" {
  instance_group         = ibm_is_instance_group.app_servers.id
  instance_group_manager = ibm_is_instance_group_manager.app_manager.manager_id
  name                   = "memory-scaling-policy"
  metric_type            = "memory"
  metric_value           = 75
  policy_type            = "target"
}

# Network-based scaling policy
resource "ibm_is_instance_group_manager_policy" "network_policy" {
  instance_group         = ibm_is_instance_group.app_servers.id
  instance_group_manager = ibm_is_instance_group_manager.app_manager.manager_id
  name                   = "network-scaling-policy"
  metric_type            = "network_in"
  metric_value           = 5000000  # 5 MB/s
  policy_type            = "target"
}
'''

# Usage
planner = CapacityPlanner()

# Add 30 days of historical data
base_date = datetime.now() - timedelta(days=30)
for i in range(30):
    planner.add_measurement(
        timestamp=base_date + timedelta(days=i),
        cpu_percent=40 + i * 0.8 + np.random.normal(0, 5),
        memory_percent=50 + i * 0.5 + np.random.normal(0, 3),
        requests_per_second=1000 + i * 20 + np.random.normal(0, 50)
    )

# Predict 30 days ahead
prediction = planner.predict_capacity_needs(days_ahead=30)
print(f"Capacity Prediction (30 days):")
print(f"  Current CPU: {prediction['current']['cpu_percent']}%")
print(f"  Predicted CPU: {prediction['predicted']['cpu_percent']}%")
print(f"  Current Memory: {prediction['current']['memory_percent']}%")
print(f"  Predicted Memory: {prediction['predicted']['memory_percent']}%")
print(f"\nRecommendations:")
for rec in prediction['recommendations']:
    print(f"  - {rec['action']}: {rec.get('message', rec.get('resource'))}")
```

---

## Performance Optimization

### Application Performance Tuning
```bash
# Performance analysis and optimization

# 1. Analyze application performance
ibmcloud ce application get my-app --output json | jq '.status'

# 2. Check resource utilization
ibmcloud ce application events my-app

# 3. Optimize Code Engine configuration
ibmcloud ce application update my-app \
  --cpu 2 \
  --memory 4G \
  --min-scale 2 \
  --max-scale 20 \
  --concurrency 100 \
  --concurrency-target 80 \
  --timeout 300

# 4. Enable HTTP/2
ibmcloud ce application update my-app \
  --cluster-local false \
  --port 8080

# 5. Configure health checks
ibmcloud ce application update my-app \
  --probe-live /healthz \
  --probe-ready /ready \
  --probe-live-timeout 5 \
  --probe-ready-timeout 3
```

### CDN and Caching Optimization
```bash
# Cloud Internet Services caching

# Create caching rules
ibmcloud cis page-rule-create $DOMAIN_ID \
  --targets '[{"target":"url","constraint":{"operator":"matches","value":"*.example.com/static/*"}}]' \
  --actions '[{
    "id":"cache_level",
    "value":"aggressive"
  },{
    "id":"edge_cache_ttl",
    "value":86400
  },{
    "id":"browser_cache_ttl",
    "value":14400
  }]'

# Configure cache purge
ibmcloud cis cache-purge $DOMAIN_ID \
  --purge-all false \
  --files '["https://example.com/static/old-file.js"]'

# Set up custom caching
ibmcloud cis cache-settings-update $DOMAIN_ID \
  --caching-level aggressive \
  --browser-expiration 14400 \
  --development-mode off
```

### Terraform for Performance Optimization
```hcl
# performance-optimization.tf

# CDN with edge caching
resource "ibm_cis_page_rule" "cache_static_assets" {
  cis_id    = ibm_cis.instance.id
  domain_id = ibm_cis_domain.domain.id
  priority  = 1
  status    = "active"

  targets {
    target = "url"
    constraint {
      operator = "matches"
      value    = "*.example.com/static/*"
    }
  }

  actions {
    id    = "cache_level"
    value = "cache_everything"
  }

  actions {
    id    = "edge_cache_ttl"
    value = 86400  # 24 hours
  }

  actions {
    id    = "browser_cache_ttl"
    value = 14400  # 4 hours
  }

  actions {
    id    = "minify"
    value = {
      css  = "on"
      html = "on"
      js   = "on"
    }
  }
}

# Performance-optimized Code Engine app
resource "ibm_code_engine_app" "optimized_app" {
  project_id      = ibm_code_engine_project.project.project_id
  name            = "optimized-app"
  image_reference = "us.icr.io/namespace/app:latest"

  # Performance settings
  scale_cpu_limit      = "2"
  scale_memory_limit   = "4G"
  scale_min_instances  = 2
  scale_max_instances  = 20
  scale_concurrency    = 100
  scale_concurrency_target = 80
  scale_request_timeout = 300

  # HTTP/2 and connection pooling
  managed_domain_mappings = "local_public"

  # Health checks
  run_env_variables {
    name  = "HEALTH_CHECK_PATH"
    value = "/healthz"
  }

  run_env_variables {
    name  = "READY_CHECK_PATH"
    value = "/ready"
  }

  # Performance monitoring
  run_env_variables {
    name  = "ENABLE_METRICS"
    value = "true"
  }

  run_env_variables {
    name  = "ENABLE_PROFILING"
    value = "true"
  }
}

# Redis cache for session storage
resource "ibm_database" "redis" {
  name              = "redis-cache"
  plan              = "standard"
  location          = "us-south"
  service           = "databases-for-redis"
  resource_group_id = data.ibm_resource_group.default.id

  group {
    group_id = "member"
    memory {
      allocation_mb = 8192
    }
    disk {
      allocation_mb = 20480
    }
    cpu {
      allocation_count = 3
    }
  }

  configuration = jsonencode({
    maxmemory-policy = "allkeys-lru"
    timeout          = 300
    tcp-keepalive    = 60
  })
}
```

---

## Chaos Engineering

### Chaos Testing Framework
```python
# chaos_engineering.py
import random
import time
from datetime import datetime
import subprocess

class ChaosExperiment:
    def __init__(self, name, description):
        self.name = name
        self.description = description
        self.results = []

    def run(self):
        """Override in subclasses"""
        raise NotImplementedError

class NetworkLatencyExperiment(ChaosExperiment):
    def __init__(self, target_service, latency_ms, duration_seconds):
        super().__init__(
            name="Network Latency Injection",
            description=f"Add {latency_ms}ms latency to {target_service}"
        )
        self.target_service = target_service
        self.latency_ms = latency_ms
        self.duration_seconds = duration_seconds

    def run(self):
        """Inject network latency"""
        print(f"Starting chaos experiment: {self.name}")
        print(f"Target: {self.target_service}, Latency: {self.latency_ms}ms")

        start_time = datetime.now()

        # Inject latency using tc (traffic control)
        inject_cmd = f"""
        kubectl exec -it {self.target_service} -- tc qdisc add dev eth0 root netem delay {self.latency_ms}ms
        """

        try:
            # Apply chaos
            subprocess.run(inject_cmd, shell=True, check=True)
            print(f"Latency injected. Waiting {self.duration_seconds} seconds...")

            # Monitor during chaos
            time.sleep(self.duration_seconds)

            # Remove chaos
            remove_cmd = f"kubectl exec -it {self.target_service} -- tc qdisc del dev eth0 root"
            subprocess.run(remove_cmd, shell=True, check=True)

            end_time = datetime.now()
            duration = (end_time - start_time).total_seconds()

            result = {
                'experiment': self.name,
                'status': 'SUCCESS',
                'start_time': start_time.isoformat(),
                'end_time': end_time.isoformat(),
                'duration_seconds': duration
            }

            self.results.append(result)
            print(f"Experiment completed successfully")
            return result

        except Exception as e:
            print(f"Experiment failed: {str(e)}")
            return {
                'experiment': self.name,
                'status': 'FAILED',
                'error': str(e)
            }

class PodFailureExperiment(ChaosExperiment):
    def __init__(self, deployment_name, namespace, num_pods=1):
        super().__init__(
            name="Pod Failure Injection",
            description=f"Kill {num_pods} pods in {deployment_name}"
        )
        self.deployment_name = deployment_name
        self.namespace = namespace
        self.num_pods = num_pods

    def run(self):
        """Kill random pods"""
        print(f"Starting chaos experiment: {self.name}")

        # Get pods
        get_pods_cmd = f"kubectl get pods -n {self.namespace} -l app={self.deployment_name} -o name"
        result = subprocess.run(get_pods_cmd, shell=True, capture_output=True, text=True)
        pods = result.stdout.strip().split('\n')

        if len(pods) < self.num_pods:
            print(f"Not enough pods to kill. Found {len(pods)}, need {self.num_pods}")
            return {'status': 'SKIPPED'}

        # Kill random pods
        pods_to_kill = random.sample(pods, self.num_pods)

        for pod in pods_to_kill:
            print(f"Killing pod: {pod}")
            kill_cmd = f"kubectl delete {pod} -n {self.namespace}"
            subprocess.run(kill_cmd, shell=True)

        # Wait for recovery
        print("Waiting for pods to recover...")
        time.sleep(30)

        # Check if deployment recovered
        check_cmd = f"kubectl get deployment {self.deployment_name} -n {self.namespace} -o jsonpath='{{.status.readyReplicas}}'"
        result = subprocess.run(check_cmd, shell=True, capture_output=True, text=True)
        ready_replicas = int(result.stdout.strip())

        return {
            'experiment': self.name,
            'status': 'SUCCESS' if ready_replicas > 0 else 'FAILED',
            'pods_killed': len(pods_to_kill),
            'ready_replicas': ready_replicas
        }

class ResourceStressExperiment(ChaosExperiment):
    def __init__(self, pod_name, namespace, stress_type='cpu', duration_seconds=60):
        super().__init__(
            name=f"{stress_type.upper()} Stress Test",
            description=f"Stress {stress_type} on {pod_name}"
        )
        self.pod_name = pod_name
        self.namespace = namespace
        self.stress_type = stress_type
        self.duration_seconds = duration_seconds

    def run(self):
        """Apply resource stress"""
        print(f"Starting chaos experiment: {self.name}")

        if self.stress_type == 'cpu':
            stress_cmd = f"kubectl exec {self.pod_name} -n {self.namespace} -- stress --cpu 2 --timeout {self.duration_seconds}s"
        elif self.stress_type == 'memory':
            stress_cmd = f"kubectl exec {self.pod_name} -n {self.namespace} -- stress --vm 1 --vm-bytes 512M --timeout {self.duration_seconds}s"
        else:
            return {'status': 'INVALID_TYPE'}

        try:
            subprocess.run(stress_cmd, shell=True, check=True)
            return {
                'experiment': self.name,
                'status': 'SUCCESS',
                'stress_type': self.stress_type,
                'duration': self.duration_seconds
            }
        except Exception as e:
            return {
                'experiment': self.name,
                'status': 'FAILED',
                'error': str(e)
            }

# Chaos experiment orchestrator
class ChaosOrchestrator:
    def __init__(self):
        self.experiments = []
        self.results = []

    def add_experiment(self, experiment):
        """Add experiment to queue"""
        self.experiments.append(experiment)

    def run_all(self, interval_seconds=60):
        """Run all experiments with intervals"""
        print(f"Running {len(self.experiments)} chaos experiments...")

        for i, experiment in enumerate(self.experiments):
            print(f"\n[{i+1}/{len(self.experiments)}] Running: {experiment.name}")
            result = experiment.run()
            self.results.append(result)

            if i < len(self.experiments) - 1:
                print(f"Waiting {interval_seconds} seconds before next experiment...")
                time.sleep(interval_seconds)

        print("\nAll experiments completed!")
        self.print_summary()

    def print_summary(self):
        """Print experiment summary"""
        print("\n" + "="*60)
        print("CHAOS ENGINEERING SUMMARY")
        print("="*60)

        total = len(self.results)
        successful = sum(1 for r in self.results if r.get('status') == 'SUCCESS')
        failed = sum(1 for r in self.results if r.get('status') == 'FAILED')

        print(f"Total Experiments: {total}")
        print(f"Successful: {successful}")
        print(f"Failed: {failed}")
        print(f"Success Rate: {(successful/total)*100:.1f}%")

        print("\nDetailed Results:")
        for result in self.results:
            status_symbol = "✓" if result.get('status') == 'SUCCESS' else "✗"
            print(f"  {status_symbol} {result['experiment']}: {result['status']}")

# Usage example
if __name__ == "__main__":
    orchestrator = ChaosOrchestrator()

    # Add experiments
    orchestrator.add_experiment(
        NetworkLatencyExperiment(
            target_service="api-service",
            latency_ms=100,
            duration_seconds=30
        )
    )

    orchestrator.add_experiment(
        PodFailureExperiment(
            deployment_name="web-app",
            namespace="production",
            num_pods=1
        )
    )

    orchestrator.add_experiment(
        ResourceStressExperiment(
            pod_name="api-pod",
            namespace="production",
            stress_type="cpu",
            duration_seconds=60
        )
    )

    # Run all experiments
    orchestrator.run_all(interval_seconds=60)
```

---

## Load Testing and Benchmarking

### Load Testing with Artillery
```yaml
# load-test.yaml
config:
  target: "https://api.example.com"
  phases:
    # Warm-up phase
    - duration: 60
      arrivalRate: 10
      name: "Warm-up"

    # Ramp-up phase
    - duration: 300
      arrivalRate: 10
      rampTo: 100
      name: "Ramp-up"

    # Sustained load
    - duration: 600
      arrivalRate: 100
      name: "Sustained load"

    # Spike test
    - duration: 60
      arrivalRate: 500
      name: "Spike test"

    # Cool-down
    - duration: 120
      arrivalRate: 10
      name: "Cool-down"

  http:
    timeout: 10

  variables:
    user_id:
      - "user-{{ $randomNumber() }}"

scenarios:
  - name: "API Load Test"
    weight: 70
    flow:
      - get:
          url: "/api/users"
          capture:
            - json: "$.users[0].id"
              as: "userId"
      - think: 1
      - get:
          url: "/api/users/{{ userId }}"
      - think: 2
      - post:
          url: "/api/orders"
          json:
            userId: "{{ userId }}"
            items: [{ id: 1, quantity: 2 }]

  - name: "Search Load Test"
    weight: 30
    flow:
      - get:
          url: "/api/search?q=test"
      - think: 1
      - get:
          url: "/api/products/{{ $randomNumber(1, 1000) }}"
```

### Performance Benchmarking Script
```bash
#!/bin/bash
# benchmark.sh - Comprehensive performance benchmark

echo "Starting IBM Cloud Performance Benchmark"
echo "========================================"

API_URL="https://api.example.com"
DURATION=300  # 5 minutes
RESULTS_DIR="./benchmark-results"

mkdir -p $RESULTS_DIR

# 1. Baseline performance test
echo "Running baseline test..."
artillery run load-test.yaml \
  --output "$RESULTS_DIR/baseline.json"

artillery report "$RESULTS_DIR/baseline.json" \
  --output "$RESULTS_DIR/baseline.html"

# 2. Stress test
echo "Running stress test..."
cat > stress-test.yaml <<EOF
config:
  target: "$API_URL"
  phases:
    - duration: 60
      arrivalRate: 1000
scenarios:
  - flow:
      - get:
          url: "/api/health"
EOF

artillery run stress-test.yaml \
  --output "$RESULTS_DIR/stress.json"

# 3. Endurance test
echo "Running endurance test (30 minutes)..."
cat > endurance-test.yaml <<EOF
config:
  target: "$API_URL"
  phases:
    - duration: 1800
      arrivalRate: 50
scenarios:
  - flow:
      - get:
          url: "/api/users"
      - think: 5
EOF

artillery run endurance-test.yaml \
  --output "$RESULTS_DIR/endurance.json"

# 4. Database performance
echo "Testing database performance..."
ibmcloud cdb deployment-scaling-get $DB_DEPLOYMENT_ID
ibmcloud cdb deployment-scaling-set $DB_DEPLOYMENT_ID \
  --members 6 \
  --memory 8192 \
  --disk 20480

# 5. CDN performance
echo "Testing CDN cache hit ratio..."
ibmcloud cis analytics-dashboard $DOMAIN_ID \
  --since -1440 > "$RESULTS_DIR/cdn-analytics.json"

# 6. Generate summary report
echo "Generating summary report..."
python3 <<PYTHON
import json
import sys

def analyze_artillery_results(filename):
    with open(filename) as f:
        data = json.load(f)

    aggregate = data.get('aggregate', {})

    return {
        'scenarios': aggregate.get('scenariosCompleted', 0),
        'requests': aggregate.get('requestsCompleted', 0),
        'rps': aggregate.get('rps', {}).get('mean', 0),
        'latency': {
            'p50': aggregate.get('latency', {}).get('median', 0),
            'p95': aggregate.get('latency', {}).get('p95', 0),
            'p99': aggregate.get('latency', {}).get('p99', 0)
        },
        'errors': aggregate.get('errors', 0)
    }

print("Benchmark Summary")
print("=" * 60)

baseline = analyze_artillery_results('$RESULTS_DIR/baseline.json')
print(f"Baseline Test:")
print(f"  Requests: {baseline['requests']}")
print(f"  RPS: {baseline['rps']:.2f}")
print(f"  P50 Latency: {baseline['latency']['p50']}ms")
print(f"  P99 Latency: {baseline['latency']['p99']}ms")
print(f"  Errors: {baseline['errors']}")

stress = analyze_artillery_results('$RESULTS_DIR/stress.json')
print(f"\nStress Test:")
print(f"  Max RPS: {stress['rps']:.2f}")
print(f"  P99 Latency: {stress['latency']['p99']}ms")
print(f"  Error Rate: {(stress['errors']/stress['requests'])*100:.2f}%")
PYTHON

echo ""
echo "Benchmark complete! Results saved to $RESULTS_DIR"
```

---

## Database Performance

### PostgreSQL Performance Tuning
```bash
# Database performance optimization

# 1. Analyze current performance
ibmcloud cdb deployment-configuration $DEPLOYMENT_ID

# 2. Scale database resources
ibmcloud cdb deployment-scaling-set $DEPLOYMENT_ID \
  --members 6 \
  --memory 16384 \
  --disk 40960 \
  --cpu 6

# 3. Configure connection pooling
ibmcloud cdb deployment-configuration-update $DEPLOYMENT_ID \
  --configuration '{
    "max_connections": 200,
    "shared_buffers": "4GB",
    "effective_cache_size": "12GB",
    "maintenance_work_mem": "1GB",
    "checkpoint_completion_target": 0.9,
    "wal_buffers": "16MB",
    "default_statistics_target": 100,
    "random_page_cost": 1.1,
    "effective_io_concurrency": 200,
    "work_mem": "20MB",
    "min_wal_size": "2GB",
    "max_wal_size": "8GB"
  }'

# 4. Enable query performance insights
ibmcloud cdb deployment-configuration-update $DEPLOYMENT_ID \
  --configuration '{
    "shared_preload_libraries": "pg_stat_statements",
    "pg_stat_statements.track": "all"
  }'

# 5. Create read replicas
ibmcloud resource service-instance-create \
  production-db-replica \
  databases-for-postgresql \
  standard \
  us-south \
  -p '{
    "members_memory_allocation_mb": "8192",
    "members_disk_allocation_mb": "20480",
    "backup_id": "'$BACKUP_ID'",
    "point_in_time_recovery_time": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
  }'
```

### Database Performance Monitoring
```python
# db_performance_monitor.py
import psycopg2
from datetime import datetime
import json

class DatabasePerformanceMonitor:
    def __init__(self, connection_string):
        self.conn = psycopg2.connect(connection_string)

    def get_slow_queries(self, threshold_ms=1000):
        """Identify slow queries"""
        query = """
        SELECT
            queryid,
            query,
            calls,
            total_exec_time / calls as avg_time_ms,
            total_exec_time,
            rows,
            100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS cache_hit_ratio
        FROM pg_stat_statements
        WHERE total_exec_time / calls > %s
        ORDER BY avg_time_ms DESC
        LIMIT 20;
        """

        cursor = self.conn.cursor()
        cursor.execute(query, (threshold_ms,))

        results = []
        for row in cursor.fetchall():
            results.append({
                'query_id': row[0],
                'query': row[1][:200],  # Truncate long queries
                'calls': row[2],
                'avg_time_ms': round(row[3], 2),
                'total_time_ms': round(row[4], 2),
                'rows': row[5],
                'cache_hit_ratio': round(row[6], 2) if row[6] else 0
            })

        return results

    def get_connection_stats(self):
        """Get connection pool statistics"""
        query = """
        SELECT
            count(*) as total_connections,
            count(*) FILTER (WHERE state = 'active') as active,
            count(*) FILTER (WHERE state = 'idle') as idle,
            count(*) FILTER (WHERE state = 'idle in transaction') as idle_in_transaction,
            max(extract(epoch from (now() - state_change))) as max_conn_age_seconds
        FROM pg_stat_activity
        WHERE datname = current_database();
        """

        cursor = self.conn.cursor()
        cursor.execute(query)
        row = cursor.fetchone()

        return {
            'total': row[0],
            'active': row[1],
            'idle': row[2],
            'idle_in_transaction': row[3],
            'max_age_seconds': round(row[4], 2) if row[4] else 0
        }

    def get_table_stats(self):
        """Get table statistics"""
        query = """
        SELECT
            schemaname,
            tablename,
            pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
            seq_scan,
            idx_scan,
            n_tup_ins,
            n_tup_upd,
            n_tup_del
        FROM pg_stat_user_tables
        ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
        LIMIT 10;
        """

        cursor = self.conn.cursor()
        cursor.execute(query)

        results = []
        for row in cursor.fetchall():
            results.append({
                'schema': row[0],
                'table': row[1],
                'size': row[2],
                'seq_scans': row[3],
                'index_scans': row[4],
                'inserts': row[5],
                'updates': row[6],
                'deletes': row[7]
            })

        return results

    def get_index_usage(self):
        """Analyze index usage"""
        query = """
        SELECT
            schemaname,
            tablename,
            indexname,
            idx_scan,
            idx_tup_read,
            idx_tup_fetch,
            pg_size_pretty(pg_relation_size(indexrelid)) as index_size
        FROM pg_stat_user_indexes
        WHERE idx_scan = 0
        ORDER BY pg_relation_size(indexrelid) DESC;
        """

        cursor = self.conn.cursor()
        cursor.execute(query)

        results = []
        for row in cursor.fetchall():
            results.append({
                'schema': row[0],
                'table': row[1],
                'index': row[2],
                'scans': row[3],
                'tuples_read': row[4],
                'tuples_fetched': row[5],
                'size': row[6]
            })

        return results

    def generate_report(self):
        """Generate comprehensive performance report"""
        report = {
            'timestamp': datetime.now().isoformat(),
            'slow_queries': self.get_slow_queries(),
            'connections': self.get_connection_stats(),
            'top_tables': self.get_table_stats(),
            'unused_indexes': self.get_index_usage()
        }

        return report

# Usage
monitor = DatabasePerformanceMonitor(
    "postgresql://user:password@host:port/database"
)

report = monitor.generate_report()

print(f"Database Performance Report - {report['timestamp']}")
print("\nConnection Stats:")
print(f"  Total: {report['connections']['total']}")
print(f"  Active: {report['connections']['active']}")
print(f"  Idle: {report['connections']['idle']}")

print("\nSlow Queries:")
for query in report['slow_queries'][:5]:
    print(f"  - {query['query'][:50]}... ({query['avg_time_ms']}ms avg)")

print("\nUnused Indexes:")
for index in report['unused_indexes'][:5]:
    print(f"  - {index['schema']}.{index['table']}.{index['index']} ({index['size']})")
```

---

## Network Optimization

### Network Performance Analysis
```bash
# Network performance optimization

# 1. Analyze VPC network performance
ibmcloud is flow-logs --vpc $VPC_ID

# Create flow log collector
ibmcloud is flow-log-create \
  vpc-flow-logs \
  --bucket $COS_BUCKET_NAME \
  --target $VPC_ID

# 2. Optimize routing with Transit Gateway
ibmcloud tg gateway-create \
  performance-gateway \
  --location us-south \
  --routing global

# Enable global routing for lower latency
ibmcloud tg gateway-update $TG_ID --routing global

# 3. Configure direct link for dedicated connectivity
ibmcloud dl gateway-create \
  --name production-direct-link \
  --type dedicated \
  --speed 5000 \
  --bgp-asn 64999 \
  --global true \
  --location us-south

# 4. Optimize load balancer configuration
ibmcloud is lb-update $LB_ID \
  --connection-limit 50000 \
  --algorithm round_robin \
  --health-delay 5 \
  --health-retries 2 \
  --health-timeout 2

# 5. Configure DNS with low TTL for faster failover
ibmcloud dns resource-record-create $ZONE_ID \
  --type A \
  --name www \
  --rdata $LB_IP \
  --ttl 300

# 6. Enable HTTP/3 and QUIC
ibmcloud cis settings-update $DOMAIN_ID \
  --http3 on \
  --zero-rtt on
```

### Terraform for Network Optimization
```hcl
# network-optimization.tf

# High-performance VPC with optimized subnets
resource "ibm_is_vpc" "optimized_vpc" {
  name                      = "high-performance-vpc"
  resource_group            = data.ibm_resource_group.default.id
  address_prefix_management = "manual"
  default_network_acl_name  = "allow-all-acl"
  default_security_group_name = "allow-outbound-sg"
}

# Multiple availability zones for redundancy
resource "ibm_is_vpc_address_prefix" "az1" {
  name = "az1-prefix"
  zone = "us-south-1"
  vpc  = ibm_is_vpc.optimized_vpc.id
  cidr = "10.240.0.0/20"
}

resource "ibm_is_vpc_address_prefix" "az2" {
  name = "az2-prefix"
  zone = "us-south-2"
  vpc  = ibm_is_vpc.optimized_vpc.id
  cidr = "10.240.16.0/20"
}

resource "ibm_is_vpc_address_prefix" "az3" {
  name = "az3-prefix"
  zone = "us-south-3"
  vpc  = ibm_is_vpc.optimized_vpc.id
  cidr = "10.240.32.0/20"
}

# Load balancer with optimized settings
resource "ibm_is_lb" "optimized_lb" {
  name            = "optimized-load-balancer"
  subnets         = [
    ibm_is_subnet.az1.id,
    ibm_is_subnet.az2.id,
    ibm_is_subnet.az3.id
  ]
  type            = "public"
  profile         = "network-fixed"
  resource_group  = data.ibm_resource_group.default.id

  # High connection limit
  logging = true
}

resource "ibm_is_lb_pool" "optimized_pool" {
  name                = "optimized-backend-pool"
  lb                  = ibm_is_lb.optimized_lb.id
  algorithm           = "round_robin"
  protocol            = "http"
  health_delay        = 5
  health_retries      = 2
  health_timeout      = 2
  health_type         = "http"
  health_monitor_url  = "/healthz"
  session_persistence_type = "source_ip"
}

# Transit Gateway for low-latency cross-region connectivity
resource "ibm_tg_gateway" "performance_gateway" {
  name     = "performance-transit-gateway"
  location = "us-south"
  global   = true
}

resource "ibm_tg_connection" "vpc1_connection" {
  gateway      = ibm_tg_gateway.performance_gateway.id
  network_type = "vpc"
  name         = "vpc1-connection"
  network_id   = ibm_is_vpc.optimized_vpc.crn
}

# Cloud Internet Services with performance features
resource "ibm_cis_domain_settings" "performance_settings" {
  cis_id    = ibm_cis.instance.id
  domain_id = ibm_cis_domain.domain.id

  # Enable HTTP/3 and QUIC
  http3                     = "on"
  zero_rtt                  = "on"

  # Optimize TLS
  min_tls_version           = "1.2"
  tls_1_3                   = "on"
  automatic_https_rewrites  = "on"
  opportunistic_encryption  = "on"

  # Enable Brotli compression
  brotli                    = "on"

  # WebSockets
  websockets                = "on"

  # Early Hints
  early_hints               = "on"
}

# CDN optimization
resource "ibm_cis_cache_settings" "performance_cache" {
  cis_id          = ibm_cis.instance.id
  domain_id       = ibm_cis_domain.domain.id
  caching_level   = "aggressive"
  browser_expiration = 14400
  development_mode = "off"
  query_string_sort = "on"
  purge_all       = false
}
```

---

## Auto-Scaling Strategies

### Advanced Auto-Scaling Configuration
```bash
# Kubernetes Horizontal Pod Autoscaler (HPA)

# 1. Create HPA based on CPU
kubectl autoscale deployment web-app \
  --cpu-percent=70 \
  --min=3 \
  --max=20

# 2. Custom metrics HPA
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 4
        periodSeconds: 30
      selectPolicy: Max
EOF

# 3. Vertical Pod Autoscaler (VPA)
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
EOF

# 4. Cluster Autoscaler
ibmcloud oc worker-pool create \
  --name autoscale-pool \
  --cluster $CLUSTER_NAME \
  --flavor bx2.4x16 \
  --size-per-zone 1 \
  --label pool=autoscale \
  --enable-autoscale \
  --min-size-per-zone 1 \
  --max-size-per-zone 10
```

### Predictive Auto-Scaling
```python
# predictive_autoscaling.py
from datetime import datetime, timedelta
import numpy as np
from sklearn.ensemble import RandomForestRegressor
import joblib

class PredictiveAutoscaler:
    def __init__(self):
        self.model = RandomForestRegressor(n_estimators=100)
        self.historical_data = []

    def add_historical_data(self, timestamp, load_metrics):
        """Add historical load data"""
        self.historical_data.append({
            'timestamp': timestamp,
            'hour': timestamp.hour,
            'day_of_week': timestamp.weekday(),
            'requests_per_second': load_metrics['requests_per_second'],
            'cpu_percent': load_metrics['cpu_percent'],
            'memory_percent': load_metrics['memory_percent'],
            'active_connections': load_metrics['active_connections']
        })

    def train_model(self):
        """Train prediction model on historical data"""
        if len(self.historical_data) < 100:
            return False

        # Prepare features
        X = []
        y = []

        for data in self.historical_data:
            features = [
                data['hour'],
                data['day_of_week'],
                data['cpu_percent'],
                data['memory_percent'],
                data['active_connections']
            ]
            X.append(features)
            y.append(data['requests_per_second'])

        self.model.fit(X, y)
        return True

    def predict_load(self, future_timestamp):
        """Predict future load"""
        features = [[
            future_timestamp.hour,
            future_timestamp.weekday(),
            0,  # Unknown future CPU
            0,  # Unknown future memory
            0   # Unknown future connections
        ]]

        predicted_rps = self.model.predict(features)[0]
        return predicted_rps

    def recommend_scale(self, current_replicas, predicted_rps,
                       rps_per_replica=100, buffer=0.2):
        """Recommend scaling action"""
        required_replicas = int(np.ceil(predicted_rps / rps_per_replica))
        required_with_buffer = int(required_replicas * (1 + buffer))

        if required_with_buffer > current_replicas:
            return {
                'action': 'SCALE_UP',
                'current_replicas': current_replicas,
                'recommended_replicas': required_with_buffer,
                'predicted_rps': round(predicted_rps, 2),
                'reason': f'Predicted load: {predicted_rps:.0f} RPS'
            }
        elif required_with_buffer < current_replicas - 2:
            return {
                'action': 'SCALE_DOWN',
                'current_replicas': current_replicas,
                'recommended_replicas': required_with_buffer,
                'predicted_rps': round(predicted_rps, 2),
                'reason': 'Predicted load decrease'
            }
        else:
            return {
                'action': 'NO_CHANGE',
                'current_replicas': current_replicas,
                'predicted_rps': round(predicted_rps, 2)
            }

# Usage
autoscaler = PredictiveAutoscaler()

# Add 30 days of historical data
base_date = datetime.now() - timedelta(days=30)
for i in range(30 * 24):  # Hourly data
    timestamp = base_date + timedelta(hours=i)

    # Simulate load patterns (higher during business hours)
    base_rps = 500
    if 9 <= timestamp.hour <= 17:
        base_rps = 2000

    autoscaler.add_historical_data(timestamp, {
        'requests_per_second': base_rps + np.random.normal(0, 100),
        'cpu_percent': 50 + np.random.normal(0, 10),
        'memory_percent': 60 + np.random.normal(0, 5),
        'active_connections': base_rps * 2
    })

# Train model
autoscaler.train_model()

# Predict next hour
next_hour = datetime.now() + timedelta(hours=1)
prediction = autoscaler.predict_load(next_hour)
recommendation = autoscaler.recommend_scale(
    current_replicas=5,
    predicted_rps=prediction
)

print(f"Scaling Recommendation:")
print(f"  Action: {recommendation['action']}")
print(f"  Current Replicas: {recommendation['current_replicas']}")
if 'recommended_replicas' in recommendation:
    print(f"  Recommended Replicas: {recommendation['recommended_replicas']}")
print(f"  Predicted Load: {recommendation['predicted_rps']} RPS")
```

---

## Cost Optimization

### Cost Analysis and Optimization
```bash
# Cost optimization strategies

# 1. Analyze resource usage
ibmcloud billing usage-account --month $(date +%Y-%m)

# 2. Identify idle resources
ibmcloud resource service-instances --output json | \
  jq -r '.[] | select(.last_operation.state == "succeeded") | .name'

# 3. Right-size instances
ibmcloud is instance-update $INSTANCE_ID --profile bx2-2x8

# 4. Use reserved capacity
ibmcloud is reservations --output json

# 5. Enable auto-scaling to reduce waste
ibmcloud ce application update my-app --min-scale 1 --max-scale 10

# 6. Use spot instances for non-critical workloads
ibmcloud oc worker-pool create \
  --name spot-pool \
  --cluster $CLUSTER_NAME \
  --flavor bx2.4x16 \
  --size-per-zone 2 \
  --label pool=spot \
  --secondary-storage 100 \
  --price-per-hour 0.05

# 7. Optimize storage costs
ibmcloud cos bucket-update \
  --bucket archive-bucket \
  --storage-class glacier

# 8. Set up budget alerts
ibmcloud billing budget-create \
  --name production-budget \
  --amount 10000 \
  --currency USD \
  --threshold 80
```

### Cost Optimization Dashboard
```python
# cost_optimization.py
from datetime import datetime, timedelta
import json

class CostOptimizer:
    def __init__(self, billing_data):
        self.billing_data = billing_data

    def identify_waste(self):
        """Identify wasted resources"""
        waste = {
            'idle_instances': [],
            'oversized_instances': [],
            'unused_storage': [],
            'orphaned_resources': []
        }

        # Example: Find idle compute instances
        for resource in self.billing_data.get('resources', []):
            if resource['type'] == 'compute':
                if resource.get('cpu_utilization_avg', 0) < 10:
                    waste['idle_instances'].append({
                        'name': resource['name'],
                        'cost_per_month': resource['cost'],
                        'utilization': resource.get('cpu_utilization_avg', 0)
                    })
                elif resource.get('cpu_utilization_avg', 0) < 30:
                    waste['oversized_instances'].append({
                        'name': resource['name'],
                        'current_size': resource['size'],
                        'recommended_size': self._calculate_right_size(resource),
                        'potential_savings': resource['cost'] * 0.5
                    })

        return waste

    def _calculate_right_size(self, resource):
        """Calculate optimal instance size"""
        util = resource.get('cpu_utilization_avg', 0)
        if util < 30:
            return 'downsize_50_percent'
        elif util < 50:
            return 'downsize_25_percent'
        else:
            return 'current_size_appropriate'

    def generate_recommendations(self):
        """Generate cost optimization recommendations"""
        waste = self.identify_waste()
        recommendations = []

        total_potential_savings = 0

        for instance in waste['idle_instances']:
            savings = instance['cost_per_month']
            total_potential_savings += savings
            recommendations.append({
                'resource': instance['name'],
                'action': 'TERMINATE',
                'reason': f"Idle instance (CPU: {instance['utilization']}%)",
                'savings_per_month': savings,
                'priority': 'HIGH'
            })

        for instance in waste['oversized_instances']:
            savings = instance['potential_savings']
            total_potential_savings += savings
            recommendations.append({
                'resource': instance['name'],
                'action': 'RESIZE',
                'current': instance['current_size'],
                'recommended': instance['recommended_size'],
                'savings_per_month': savings,
                'priority': 'MEDIUM'
            })

        return {
            'total_potential_savings': round(total_potential_savings, 2),
            'recommendations': recommendations
        }

# Example usage
billing_data = {
    'resources': [
        {
            'name': 'dev-server-1',
            'type': 'compute',
            'size': 'bx2-4x16',
            'cost': 150,
            'cpu_utilization_avg': 5
        },
        {
            'name': 'app-server-1',
            'type': 'compute',
            'size': 'bx2-8x32',
            'cost': 300,
            'cpu_utilization_avg': 25
        }
    ]
}

optimizer = CostOptimizer(billing_data)
report = optimizer.generate_recommendations()

print(f"Cost Optimization Report")
print(f"Total Potential Savings: ${report['total_potential_savings']}/month")
print(f"\nRecommendations:")
for rec in report['recommendations']:
    print(f"  - {rec['resource']}: {rec['action']} (${rec['savings_per_month']}/month)")
```

---

## Exam Tips

1. **SRE Principles**: Understand error budgets, SLOs/SLIs, toil reduction
2. **Capacity Planning**: Know predictive scaling, resource optimization
3. **Performance**: Master load testing, benchmarking, optimization techniques
4. **Chaos Engineering**: Understand fault injection, resilience testing
5. **Auto-Scaling**: Know HPA, VPA, cluster autoscaling, predictive scaling
6. **Database Performance**: Understand connection pooling, query optimization, read replicas
7. **Network Optimization**: Know CDN, load balancing, Transit Gateway
8. **Cost Optimization**: Identify waste, right-sizing, reserved capacity

### Common Exam Scenarios

**Scenario 1**: Application not meeting 99.9% SLO
- Calculate error budget (43.2 minutes/month)
- Identify reliability issues in monitoring
- Implement chaos engineering tests
- Set up automated incident response
- Create SLO dashboards

**Scenario 2**: Optimize application for peak traffic
- Implement predictive auto-scaling
- Configure CDN caching
- Set up load testing
- Optimize database queries
- Enable connection pooling

**Scenario 3**: Reduce cloud costs by 30%
- Identify idle resources
- Right-size overprovisioned instances
- Implement auto-scaling
- Use reserved capacity
- Archive old data to glacier storage

**Scenario 4**: Improve database performance
- Analyze slow queries
- Add appropriate indexes
- Configure connection pooling
- Scale database resources
- Create read replicas

**Scenario 5**: Implement chaos engineering
- Start with network latency tests
- Progress to pod failure injection
- Test auto-recovery mechanisms
- Document failure patterns
- Build remediation automation

### Key Metrics to Know

- **SLI (Service Level Indicator)**: Quantitative measure of service level
- **SLO (Service Level Objective)**: Target value for SLI
- **SLA (Service Level Agreement)**: Contract with consequences
- **Error Budget**: Acceptable amount of unreliability (1 - SLO)
- **MTBF (Mean Time Between Failures)**: Reliability measure
- **MTTR (Mean Time To Repair)**: Recovery speed measure
- **P50/P95/P99 Latency**: Percentile latency metrics
- **Availability**: Uptime percentage (99.9% = 43.2 min downtime/month)

### Hands-On Practice

1. Set up comprehensive monitoring with SLO tracking
2. Implement error budget calculations
3. Perform load testing with Artillery
4. Execute chaos engineering experiments
5. Configure predictive auto-scaling
6. Optimize database performance
7. Implement cost optimization strategies
8. Build performance dashboards in Grafana
