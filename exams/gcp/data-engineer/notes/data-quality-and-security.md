# Data Quality and Security - GCP Professional Data Engineer

## Overview

Data quality, governance, security, and compliance for data engineering on GCP, covering validation, access control, encryption, and regulatory requirements.

## Key Topics

1. **Data Quality** - Validation, profiling, cleansing, monitoring
2. **Data Security** - Encryption, access control, DLP, audit logging
3. **Data Governance** - Lineage, cataloging, compliance, retention
4. **Compliance** - GDPR, HIPAA, regulatory requirements

## Data Quality

### Data Validation
**At Ingestion**:
- Schema validation
- Data type checking
- Range and constraint validation
- Null value handling
- Duplicate detection

**During Processing**:
- Business rule validation
- Referential integrity
- Data completeness checks
- Consistency validation
- Anomaly detection

**Tools**: Cloud Data Quality (in preview), custom Dataflow validation, BigQuery data quality checks, Great Expectations

### Data Profiling
**Metrics**:
- Row counts and data volumes
- Null/missing value percentages
- Value distributions and cardinality
- Data types and formats
- Statistical summaries (min, max, avg, std dev)

**Implementation**:
```sql
-- BigQuery data profiling
SELECT
  COUNT(*) as total_rows,
  COUNTIF(column IS NULL) as null_count,
  COUNT(DISTINCT column) as distinct_values,
  MIN(column) as min_value,
  MAX(column) as max_value,
  AVG(column) as avg_value
FROM `dataset.table`;
```

### Data Cleansing
**Common Operations**:
- Remove duplicates
- Handle missing values (imputation, removal)
- Standardize formats (dates, addresses, phone numbers)
- Correct data types
- Fix encoding issues
- Normalize values

**Dataflow Example**:
```python
def clean_data(element):
    # Remove duplicates, handle nulls, standardize
    element['email'] = element.get('email', '').lower().strip()
    element['date'] = parse_date(element.get('date'))
    return element

cleaned = (pipeline
    | 'Read' >> beam.io.ReadFromBigQuery(table)
    | 'Clean' >> beam.Map(clean_data)
    | 'Write' >> beam.io.WriteToBigQuery(output_table))
```

### Data Monitoring
**Metrics to Track**:
- Data freshness (last update time)
- Data volume trends
- Schema changes
- Data quality scores
- Processing success rates
- Latency metrics

**Alerting**: Set up Cloud Monitoring alerts for data quality SLOs

## Data Security

### Access Control
**BigQuery**:
- Dataset-level IAM roles
- Table-level permissions
- Column-level security
- Row-level security (policy tags)
- Authorized views
- Authorized routines

**Cloud Storage**:
- Bucket-level IAM
- Object-level ACLs
- Uniform bucket-level access
- Signed URLs for temporary access
- Requester pays buckets

**Example Row-Level Security**:
```sql
CREATE ROW ACCESS POLICY filter_by_region
ON `dataset.table`
GRANT TO ('user:analyst@example.com')
FILTER USING (region = 'US');
```

### Encryption
**Data at Rest**:
- Default encryption (Google-managed keys)
- Customer-managed encryption keys (CMEK) via Cloud KMS
- Customer-supplied encryption keys (CSEK)
- Application-level encryption

**Data in Transit**:
- TLS/HTTPS for all connections
- Private Google Access for internal traffic
- VPN/Interconnect for hybrid

**Key Management**:
- Cloud KMS for key management
- Key rotation policies
- Regional key storage
- Hardware security modules (HSM) support

### Data Loss Prevention (DLP)
**Capabilities**:
- PII detection and classification
- Custom info types
- De-identification (masking, tokenization, redaction)
- Re-identification
- Risk analysis

**Integration**:
```python
from google.cloud import dlp_v2

def inspect_data(project, content):
    dlp = dlp_v2.DlpServiceClient()
    item = {'value': content}
    inspect_config = {
        'info_types': [{'name': 'EMAIL_ADDRESS'}, {'name': 'PHONE_NUMBER'}]
    }
    response = dlp.inspect_content(
        request={'parent': f'projects/{project}', 'inspect_config': inspect_config, 'item': item}
    )
    return response
```

**Use Cases**:
- Scan Cloud Storage for sensitive data
- Redact PII from BigQuery tables
- Tokenize data for analytics
- Classify data sensitivity

### Audit Logging
**Log Types**:
- Admin Activity: Configuration changes
- Data Access: Read/write operations (opt-in for most services)
- System Events: Google-initiated changes
- Policy Denied: Access denied events

**Best Practices**:
- Export audit logs to BigQuery for analysis
- Set up log sinks for long-term retention
- Monitor for suspicious access patterns
- Create alerts for security events
- Regular audit log review

## Data Governance

### Data Cataloging
**Cloud Data Catalog**:
- Automatic metadata discovery
- Technical and business metadata
- Search and discovery
- Tag templates for custom metadata
- Integration with BigQuery, Cloud Storage, Pub/Sub

**Features**:
- Tag sensitive data with policy tags
- Document data assets
- Search across data sources
- Track data lineage
- Organize with taxonomies

### Data Lineage
**Why Important**:
- Understand data flow
- Impact analysis for changes
- Debugging data issues
- Compliance and audit
- Data quality root cause analysis

**Tools**:
- Cloud Data Catalog (limited lineage)
- Custom tracking in metadata
- Third-party lineage tools
- Documentation in data pipelines

### Data Retention
**Policies**:
- BigQuery table expiration
- Cloud Storage lifecycle policies
- Audit log retention
- Backup retention
- Legal hold management

**Implementation**:
```bash
# BigQuery table expiration
bq update --expiration=3600 dataset.table  # Expires in 1 hour

# Cloud Storage lifecycle
cat > lifecycle.json <<EOF
{
  "rule": [{
    "action": {"type": "Delete"},
    "condition": {"age": 365}
  }]
}
EOF
gsutil lifecycle set lifecycle.json gs://bucket
```

## Compliance

### GDPR (General Data Protection Regulation)
**Requirements**:
- Data residency (use regional resources)
- Right to erasure (implement data deletion)
- Data portability (export capabilities)
- Consent management
- Data breach notification
- Privacy by design

**Implementation**:
- Use EU regions for data storage
- Implement user data deletion workflows
- Enable audit logging
- Use DLP for PII protection
- Document data processing activities
- Regular compliance audits

### HIPAA (Healthcare)
**Requirements**:
- PHI protection (encryption, access control)
- Audit logging
- Business Associate Agreement (BAA)
- Breach notification
- Risk assessment

**GCP Services**: Most are HIPAA-eligible with BAA
**Implementation**:
- Enable audit logs
- Use CMEK for sensitive data
- VPC Service Controls for data perimeter
- Regular access reviews
- Incident response plan

### Data Classification
**Levels**:
- Public: No restrictions
- Internal: Company confidential
- Confidential: Restricted access
- Highly Confidential: PII, PHI, financial data

**Implementation**:
- Policy tags in BigQuery
- Labels on Cloud Storage
- DLP for automatic classification
- Access controls based on classification
- Encryption based on sensitivity

## Best Practices

### Data Quality
1. Validate data at ingestion
2. Monitor data quality metrics continuously
3. Implement data profiling
4. Set up automated data quality checks
5. Document data quality rules
6. Alert on quality threshold breaches
7. Track data lineage
8. Regular data quality reviews

### Data Security
1. Use least privilege access
2. Enable audit logging
3. Encrypt sensitive data (CMEK)
4. Use VPC Service Controls
5. Implement DLP scanning
6. Regular security audits
7. Separate production and non-production data
8. Use service accounts for applications

### Data Governance
1. Catalog all data assets
2. Document data lineage
3. Implement data retention policies
4. Tag data with metadata
5. Regular compliance reviews
6. Data classification framework
7. Privacy impact assessments
8. Incident response procedures

## Common Scenarios

**Scenario**: PII protection in data warehouse
**Solution**: DLP for scanning, BigQuery column-level security, row-level security, CMEK encryption, audit logging

**Scenario**: GDPR compliance for EU customer data
**Solution**: EU regions, data deletion workflows, DLP, audit logs, consent tracking in database, privacy policies

**Scenario**: Data quality monitoring pipeline
**Solution**: Dataflow validation, custom metrics to Cloud Monitoring, BigQuery data quality checks, alerting on thresholds

**Scenario**: Multi-tenant data isolation
**Solution**: Separate BigQuery datasets per tenant, row-level security, VPC Service Controls, separate encryption keys

## Study Tips

1. **Data validation techniques**: Know how to implement in Dataflow and BigQuery
2. **Security controls**: IAM, encryption, DLP, audit logs
3. **Compliance requirements**: GDPR, HIPAA implementation on GCP
4. **Data Catalog**: Features and use cases
5. **DLP capabilities**: Detection, de-identification, classification
6. **Access control patterns**: Dataset, table, column, row-level security
7. **Audit logging**: Types and monitoring approaches

## Additional Resources

- [Data Governance on GCP](https://cloud.google.com/architecture/data-governance)
- [DLP Documentation](https://cloud.google.com/dlp/docs)
- [BigQuery Security](https://cloud.google.com/bigquery/docs/access-control)
- [Compliance Resources](https://cloud.google.com/security/compliance)
- [Data Catalog](https://cloud.google.com/data-catalog)
