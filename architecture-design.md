### **Documentation for DYS Solution Architecture**

---

# **DYS Solution: Architecture and Security Design**

## **1. Overview**

The DYS Solution is designed to provide a secure, modular, and scalable system for handling sensitive youth services data. This architecture separates roles and responsibilities across servers to minimize the attack surface while maintaining compliance with security standards like OWASP and NIST.

---

## **2. Architecture Design**

### **2.1 Components**
1. **SQL Server (Sensitive Data)**:
   - The source of truth for all production data.
   - Restricted access; only the Dev Server can query this server.
2. **Dev Server (Node.js + SQLite)**:
   - Pulls data from SQL Server.
   - Encrypts data into JSON files.
   - Stages and tests updates using SQLite as a lightweight local database.
   - Pushes JSON files to Prod and Public Servers.
3. **JSON Files (Encrypted)**:
   - Temporarily stored on the Dev Server for staging and testing.
   - Used by Prod and Public Servers for serving data.
4. **Prod Server (Node.js Backend)**:
   - Hosts private, secure pages for internal users.
   - Consumes encrypted JSON files for dynamic content.
5. **Public Server (Static Hosting)**:
   - Serves static files and encrypted JSON files for public-facing content.
   - No database access to reduce risk.
6. **Secrets Manager**:
   - Centralized management of API keys and encryption keys.
   - Ensures secure storage and retrieval of sensitive information.

---

### **2.2 Data Flow**
```plaintext
1. SQL Server → Dev Server: Pulls sensitive data securely.
2. Dev Server → JSON Files: Encrypts and stages data for testing.
3. Dev Server → Prod/Public Servers: Pushes JSON files securely.
4. Secrets Manager → All Servers: Provides API keys and encryption keys as needed.
```
## **3. Security Measures**

### **3.1 Encryption**

#### **1. Data at Rest**:
- All JSON files are encrypted using AES-256-GCM.
- Example encryption flow:
  ```javascript
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  const encrypted = cipher.update(data, 'utf8', 'hex') + cipher.final('hex');
  const authTag = cipher.getAuthTag(); // Attach this for validation
  ```

---

#### **2. Data in Transit**:
   - Uses HTTPS or SFTP for secure data transmission.
   - Example: SFTP file transfer from Dev to Public Server:
     ``` PS
      scp .\data.json.enc user@publicserver:/var/www/data/
     ```
---

### **3.2 Authentication and Access Control**
1. **Environment Isolation**:
   - SQL Server is only accessible to the Dev Server.
   - No database access for Prod or Public Server.

2. **Secrets Management**:
   - API keys and encryption keys are stored securely in **Azure Key Vault**.
   - Azure Key Vault ensures centralized management, secure access, and compliance with DHS standards.
   - Example: Fetching a secret from Azure Key Vault in Node.js:
     ```javascript
     const { DefaultAzureCredential } = require('@azure/identity');
     const { SecretClient } = require('@azure/keyvault-secrets');

     const keyVaultName = '<Your-KeyVault-Name>';
     const vaultUri = `https://${keyVaultName}.vault.azure.net`;
     const credential = new DefaultAzureCredential();

     const client = new SecretClient(vaultUri, credential);

     async function getSecret(secretName) {
         try {
             const secret = await client.getSecret(secretName);
             console.log(`Secret Value: ${secret.value}`);
         } catch (err) {
             console.error(`Error fetching secret: ${err.message}`);
         }
     }

     getSecret('MySecretKey');
     ```

3. **Role-Based Access Control (RBAC)**:
   - Access to Azure Key Vault is restricted using **Azure AD Role-Based Access Control**.
   - Managed identities or service principals authenticate the Dev Server to fetch secrets securely.
   - This approach ensures:
     - Minimal privileges for each application.
     - Full control and auditability for DHS IT.

4. **Auditing and Monitoring**:
   - Azure Monitor logs all access to the Key Vault for complete visibility.
   - Alerts can be configured for unauthorized access attempts or suspicious behavior.

---

### **4. Compliance**

#### **4.1 OWASP Standards**
- **Secure Data Handling**:
  - All sensitive data is encrypted both at rest and in transit using AES-256-GCM.
  - Aligns with the OWASP Top 10 recommendations for secure data management.
  - [Learn More: OWASP Top 10](https://owasp.org/www-project-top-ten/)
- **Access Control**:
  - Role-based access ensures minimal privilege for each server and user.
- **Logging and Monitoring**:
  - Logs are maintained for all key access, data transfers, and server access, meeting OWASP guidelines for secure monitoring.

#### **4.2 NIST Compliance**
- **Data Encryption**:
  - AES-256-GCM is used, meeting FIPS 140-2 and NIST SP 800-57 recommendations for encryption standards.
  - [Learn More: NIST SP 800-57](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final)
- **Authentication and Access Control**:
  - Access policies align with NIST SP 800-63 guidelines for secure authentication.
  - [Learn More: NIST SP 800-63](https://www.nist.gov/special-publication-800-63)
- **Key Management and Rotation**:
  - All encryption keys are stored securely in **Azure Key Vault**, ensuring compliance with NIST SP 800-57 key management standards.
  - **Key Rotation**:
    - Keys are rotated periodically to reduce the risk of compromise.
    - Automated rotation is configured in Azure Key Vault, ensuring seamless updates without manual intervention.
    - Rotation intervals align with DHS’s compliance policies, typically every 90–180 days or as per specific security requirements.

--- 

## **5. Additional Questions**

### **5.1 Network and Data Flow**
1. **How are servers connected?**
   - SQL Server and Dev Server communicate over a private VPN or restricted network.
   - JSON files are pushed securely to Prod and Public Servers.

2. **Why does the Public Server not access the database?**
   - This minimizes the attack surface and ensures the public-facing server is lightweight and secure.

---

### **5.2 Security Concerns**
1. **How is sensitive data protected?**
   - Data is encrypted immediately upon generation.
   - Encrypted JSON files are transferred and served securely.

2. **What happens if a server is compromised?**
   - Public Server has no direct access to sensitive data or databases.
   - Dev Server encrypts all data, so no plaintext is exposed.

---

### **5.3 Scalability**
**Note**: By leveraging **Content Delivery Networks (CDNs)** and caching strategies, the system ensures high performance and reliability. CDNs distribute static and frequently accessed content (e.g., JSON files) to servers closer to users, reducing latency and load on the Public Server. Caching further optimizes performance by allowing browsers and edge nodes to store copies of data, minimizing redundant requests and ensuring faster access to critical resources.
#### **1. How does the system handle increased traffic?**
- **Public Server Scalability**:
  - The Public Server is designed to serve **pre-generated static files**, making it lightweight and inherently scalable.
  - **Use of Content Delivery Networks (CDNs)**:
    - Static files, including encrypted JSON files and frontend assets, are distributed through a **CDN** to ensure global availability and low-latency access.
    - Examples: Azure CDN, AWS CloudFront, or Akamai.
    - Benefits:
      - **Load Distribution**: Offloads traffic from the Public Server.
      - **Edge Caching**: Files are cached closer to users for faster access.
  - **Caching Strategies**:
    - Frequently accessed JSON files are cached in the browser or on the CDN edge nodes.
    - **Cache-Control Headers**:
      - Static files use long-lived cache headers for efficiency:
        ```http
        Cache-Control: public, max-age=31536000, immutable
        ```
      - JSON files use shorter cache times to ensure timely updates:
        ```http
        Cache-Control: public, max-age=3600
        ```
    - Stale files are invalidated when updates are pushed from the Dev Server to ensure data freshness.

#### **2. Prod Server Scalability**:
- The Prod Server uses **Node.js clustering** to handle concurrent requests efficiently:
  - A multi-threaded approach ensures even high traffic does not overwhelm the server.
  - Load balancers distribute incoming requests across Node.js instances.

---

## **6. Operational Workflow**

### **6.1 Data Generation**
- Dev Server queries SQL Server daily or on-demand to fetch updated data.

### **6.2 Data Processing**
- Data is encrypted and stored as JSON files.
- Files are tested locally on the Dev Server using SQLite.

### **6.3 Deployment**
- JSON files are securely pushed to Prod and Public Servers.

---

### **6.4 Logging and Monitoring**

#### **1. Centralized Logging for Security and Compliance**
- **Tools Used**:
  - **Azure Monitor**:
    - Integrated with Azure Key Vault for real-time logging of secret access and management events.
    - Provides dashboards and alerts for auditing and proactive monitoring.
  - **ELK Stack** (Elasticsearch, Logstash, Kibana):
    - Centralized log aggregation and analysis for all server activity.
    - Dashboards provide visibility into system operations, including access patterns and anomalies.
  - **Application-Level Logging**:
    - Node.js applications log key operations using libraries like `winston` or `pino`.

#### **2. Example Log Entries**
- **Key Vault Access**:
  - Tracks every request to Azure Key Vault, including:
    - Successful retrieval of encryption keys or secrets.
    - Unauthorized access attempts.
    - Example:
      ```json
      {
        "timestamp": "2024-12-20T14:32:00Z",
        "event": "KeyVaultAccess",
        "status": "Success",
        "secretId": "MySecretKey",
        "clientId": "DevServerApp",
        "sourceIp": "192.168.1.10"
      }
      ```

- **File Transfers**:
  - Logs file encryption, staging, and transfer events for traceability.
  - Example:
      ```json
      {
        "timestamp": "2024-12-20T15:45:00Z",
        "event": "FileTransfer",
        "status": "Success",
        "fileName": "data.json.enc",
        "source": "DevServer",
        "destination": "PublicServer"
      }
      ```

- **Failed Authentication Attempts**:
  - Logs unauthorized attempts to access sensitive resources.
  - Example:
      ```json
      {
        "timestamp": "2024-12-20T16:12:00Z",
        "event": "Authentication",
        "status": "Failed",
        "username": "UnknownUser",
        "sourceIp": "203.0.113.45",
        "reason": "Invalid credentials"
      }
      ```

#### **3. Alerts and Anomaly Detection**
- **Real-Time Alerts**:
  - Configure alerts for unusual activities, such as:
    - Multiple failed authentication attempts.
    - Unauthorized access to Key Vault secrets.
  - Alerts are delivered via email or SMS for immediate response.
- **Anomaly Detection**:
  - Use Azure Monitor or Kibana to detect patterns that deviate from normal behavior, such as unexpected spikes in file access.

#### **4. Retention and Compliance**
- Logs are stored securely for a minimum of **90 days** or as per DHS compliance requirements.
- Retained logs are encrypted and periodically reviewed to ensure adherence to security policies.

---

## **7. Diagram**

The ASCII diagram from earlier can be included here for visual clarity:
```plaintext
                         +---------------------------+
                         |     SQL Server (Prod)     |
                         |  - Sensitive Data Source  |
                         |  - Generates JSON Output  |
                         +---------------------------+
                                     |
                                     | (Pull Data)
                                     v
      +-------------------------------+-------------------------------+
      |              Dev Server (Node.js + SQLite)                   |
      |  - Pulls data from SQL Server                                |
      |  - Encrypts data into JSON files                             |
      |  - Stores JSON in SQLite for staging and testing             |
      |  - Pushes JSON files to Prod and Public Servers              |
      +-------------------------------+-------------------------------+
                      | (Push Encrypted JSON)              | (Push Encrypted JSON)
                      v                                    v
      +-----------------------+                 +-----------------------+
      |     Prod Server       |                 |   Public Server       |
      |  (Node.js Backend)    |                 |  (Static Hosting)     |
      |  - Hosts secure pages |                 |  - Serves static pages|
      |  - Reads JSON files   |                 |  - Reads JSON files   |
      +-----------------------+                 +-----------------------+
                                     |
                                     | (Access Encryption Keys & Secrets)
                                     v
                         +---------------------------+
                         |    Secrets Manager        |
                         |  - Stores API Keys        |
                         |  - Manages Encryption Keys|
                         +---------------------------+
```

## **8. Next Steps**
1. **Finalize CI/CD Pipeline**:
   - Automate JSON file generation and deployment.
2. **Implement Logging**:
   - Use a centralized logging system like ELK Stack.
3. **Document Compliance**:
   - Maintain detailed documentation of encryption standards and access policies.

---