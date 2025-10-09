# Amazon QLDB (Quantum Ledger Database)

## Service Overview and Purpose

Amazon QLDB is a fully managed ledger database that provides a transparent, immutable, and cryptographically verifiable transaction log. QLDB maintains a complete and verifiable history of all changes made to your data.

### Key Characteristics
- **Immutable Ledger**: Complete, verifiable history of data changes
- **Cryptographic Verification**: SHA-256 hashing for integrity
- **ACID Transactions**: Full transactional consistency
- **Serverless**: Automatic scaling and management
- **SQL-Like Queries**: PartiQL query language

## Key Features and Capabilities

### Core Features
- **Journal**: Immutable sequence of data changes
- **Cryptographic Verification**: Verify data integrity
- **History Tracking**: Complete audit trail of all changes
- **PartiQL**: SQL-compatible query language
- **ACID Compliance**: Strong consistency guarantees

### Cryptographic Features
- **Hash Chaining**: Each block contains hash of previous block
- **Digital Signatures**: Cryptographic proof of data integrity
- **Merkle Trees**: Efficient verification of data sets
- **Revision History**: Complete change history for each document

## Configuration and Use Cases

### Ledger Creation
```bash
aws qldb create-ledger \
  --name FinancialLedger \
  --permissions-mode STANDARD \
  --kms-key-id alias/qldb-encryption-key \
  --tags Key=Environment,Value=Production
```

### Data Operations
```python
import boto3
from pyqldb.driver.pooled_qldb_driver import PooledQldbDriver

# Connect to QLDB
driver = PooledQldbDriver(
    ledger_name='FinancialLedger',
    region_name='us-west-2'
)

def create_table():
    def execute_lambda(transaction_executor):
        transaction_executor.execute_statement("""
            CREATE TABLE Transactions
        """)

    driver.execute_lambda(execute_lambda)

def insert_transaction(transaction_data):
    def execute_lambda(transaction_executor):
        transaction_executor.execute_statement(
            "INSERT INTO Transactions ?",
            transaction_data
        )

    driver.execute_lambda(execute_lambda)

def query_transactions(account_id):
    def execute_lambda(transaction_executor):
        result = transaction_executor.execute_statement(
            "SELECT * FROM Transactions WHERE accountId = ?",
            account_id
        )
        return [doc for doc in result]

    return driver.execute_lambda(execute_lambda)

# Insert financial transaction
transaction = {
    'transactionId': 'tx001',
    'accountId': 'acc123',
    'amount': 1000.00,
    'type': 'credit',
    'timestamp': '2023-01-01T10:00:00Z'
}
insert_transaction(transaction)
```

### Verification
```python
def verify_ledger_integrity():
    qldb = boto3.client('qldb')

    # Get digest
    response = qldb.get_digest(Name='FinancialLedger')
    digest = response['Digest']

    # Get revision
    revision_response = qldb.get_revision(
        Name='FinancialLedger',
        BlockAddress={'IonText': block_address},
        DocumentId=document_id
    )

    # Verify integrity
    proof = revision_response['Proof']
    # Verification logic here
```

## Use Cases and Scenarios

### Primary Use Cases
1. **Financial Systems**: Banking transactions and accounting
2. **Supply Chain**: Product provenance and tracking
3. **Healthcare**: Medical records and audit trails
4. **Legal Documents**: Contract management and compliance
5. **Identity Management**: User access and permissions audit

### Common Exam Scenarios
- **Regulatory Compliance**: Industries requiring audit trails
- **Financial Applications**: Banking and payment systems
- **Supply Chain Tracking**: Product authenticity verification
- **Legal and Compliance**: Document integrity verification

## Exam Tips
- **Know immutable ledger concepts** and cryptographic verification
- **Understand when to use QLDB** vs traditional databases
- **Remember PartiQL** as the query language
- **Know verification capabilities** for data integrity
- **Understand compliance** and audit use cases