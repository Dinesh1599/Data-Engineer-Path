# 16 - Security

#kafka #security #TLS #SASL #ACL #authentication #authorisation

← [[Kafka MOC]]

---

## Three Layers of Kafka Security

1. **Encryption** — data in transit is encrypted (TLS)
2. **Authentication** — who are you? (SASL)
3. **Authorisation** — what are you allowed to do? (ACLs)

---

## 1. Encryption with TLS

By default, Kafka sends data in plain text. Anyone on the network can read your events. TLS encrypts communication between clients (producers/consumers) and brokers.

### Broker Configuration
```properties
# server.properties
listeners=PLAINTEXT://:9092,SSL://:9093
ssl.keystore.location=/var/ssl/private/kafka.keystore.jks
ssl.keystore.password=keystore-password
ssl.key.password=key-password
ssl.truststore.location=/var/ssl/private/kafka.truststore.jks
ssl.truststore.password=truststore-password
```

### Producer/Consumer Configuration
```python
config = {
    "bootstrap.servers": "kafka:9093",
    "security.protocol": "SSL",
    "ssl.ca.location": "/path/to/ca-cert.pem",
    "ssl.certificate.location": "/path/to/client-cert.pem",
    "ssl.key.location": "/path/to/client-key.pem",
}
```

---

## 2. Authentication with SASL

**SASL (Simple Authentication and Security Layer)** verifies the identity of clients connecting to Kafka. Multiple mechanisms are supported:

| Mechanism | How it works | Use Case |
|---|---|---|
| `SASL/PLAIN` | Username + password | Simple, but credentials in plaintext → use with TLS |
| `SASL/SCRAM-SHA-256` | Hashed credentials stored in ZooKeeper/KRaft | Better than PLAIN |
| `SASL/SCRAM-SHA-512` | Same, stronger hash | Recommended for username/password auth |
| `SASL/GSSAPI (Kerberos)` | Enterprise Kerberos | Large enterprises with existing Kerberos |
| `SASL/OAUTHBEARER` | OAuth 2.0 tokens | Cloud-native, integrates with IAM |

### SASL_SSL (most common production config)
Combines SASL authentication + TLS encryption.

```python
config = {
    "bootstrap.servers": "kafka:9093",
    "security.protocol": "SASL_SSL",
    "sasl.mechanism": "SCRAM-SHA-256",
    "sasl.username": "my-service-account",
    "sasl.password": "my-password",
    "ssl.ca.location": "/path/to/ca-cert.pem",
}
```

---

## 3. Authorisation with ACLs

**ACLs (Access Control Lists)** define what an authenticated user is allowed to do on which resources.

### ACL Operations
- `Read` — consume from a topic
- `Write` — produce to a topic
- `Create` — create topics
- `Delete` — delete topics
- `Describe` — see topic metadata
- `Alter` — modify topic configuration

### CLI Examples

```bash
# Allow "order-service" to produce to "orders" topic
kafka-acls.sh --add \
  --allow-principal User:order-service \
  --operation Write \
  --topic orders \
  --bootstrap-server localhost:9092

# Allow "invoice-service" to consume from "orders" topic
kafka-acls.sh --add \
  --allow-principal User:invoice-service \
  --operation Read \
  --topic orders \
  --group invoice-group \
  --bootstrap-server localhost:9092

# List ACLs
kafka-acls.sh --list --bootstrap-server localhost:9092
```

### Enable ACL Authoriser in Broker
```properties
# server.properties
authorizer.class.name=org.apache.kafka.metadata.authorizer.StandardAuthorizer
super.users=User:admin
```

---

## Encryption at Rest

By default, Kafka does **not** encrypt data stored on disk. For encryption at rest:

- Use **encrypted disks** at the OS/cloud level (AWS EBS encryption, GCP disk encryption)
- Or use **Confluent Cloud** which handles this transparently

---

## Security Best Practices

| Practice | Why |
|---|---|
| Always use TLS in production | Prevent eavesdropping |
| Use SASL_SSL (not PLAINTEXT) | Authentication + encryption |
| Least-privilege ACLs | Each service only accesses what it needs |
| Rotate credentials regularly | Reduce blast radius of credential leaks |
| Use service accounts per service | Audit trail per service, easy revocation |
| Enable audit logging | Know who accessed what, when |

---

← [[15 - Deploying Kafka]] | [[17 - Monitoring and Observability]] →
