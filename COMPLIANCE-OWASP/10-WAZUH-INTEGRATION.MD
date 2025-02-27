# **Wazuh Integration for Node.js Applications**

## **Introduction**
Wazuh is a comprehensive security platform that provides real-time monitoring, threat detection, and compliance auditing. It seamlessly integrates with Node.js and ReactJS architectures, enhancing security by aligning with OWASP and NIST standards.

[Wazah Web Site](https://wazuh.com/)

---

## **Key Features of Wazuh**

### **1. Real-Time Threat Detection**
- **File Integrity Monitoring (FIM)**:
  - Detects unauthorized changes to application files.
  - Protects critical Node.js files from tampering.
- **Log Analysis**:
  - Monitors logs generated by applications, systems, and network devices.
  - Detects anomalies using pre-configured rules and machine learning.

### **2. Compliance Monitoring**
- **OWASP Alignment**:
  - Wazuh includes built-in rules to monitor for OWASP Top 10 vulnerabilities like SQL injection and XSS.
  - Example: Alerts for abnormal query patterns to detect SQL injection attempts.
- **NIST Compliance**:
  - Helps maintain compliance with NIST SP 800-53 by auditing user activities, permissions, and system changes.

### **3. Centralized Security Management**
- **Unified Dashboard**:
  - Provides a single pane of glass to manage and monitor security events across all servers (Dev, Prod, Public).
- **Agent-Based Monitoring**:
  - Lightweight agents run on servers to collect and forward security data to the Wazuh Manager.
- **Node.js Log Integration**:
  - Easily integrates with Node.js logging libraries like Winston and Pino to forward application logs.

### **4. Alerts and Notifications**
- **Customizable Alerts**:
  - Configure email, Slack, or webhook alerts for critical events like unauthorized access attempts.
- **Threat Intelligence Feeds**:
  - Leverages real-time threat feeds to detect malicious IPs or suspicious activities.

### **5. Intrusion Detection System (IDS)**
- Monitors network traffic and alerts on potential intrusion attempts.
- Protects REST APIs and backend servers from brute force and DoS attacks.

---

## **OWASP Compliance Examples**

### **1. Injection Attacks (OWASP A01:2021)**
- **Problem**: Vulnerabilities like SQL injection allow attackers to manipulate database queries.
- **How Wazuh Helps**:
  - Monitors application logs for suspicious patterns such as SQL injection attempts.
  - Example: Detects queries with multiple `OR 1=1` conditions or unusual delimiters.
  - Alert Rule:
    ```json
    {
      "rule": "SQL Injection Detected",
      "level": 10,
      "description": "Suspicious SQL query identified in logs",
    }
    ```

### **2. Cross-Site Scripting (XSS) (OWASP A03:2021)**
- **Problem**: XSS attacks allow attackers to execute malicious scripts in a user’s browser.
- **How Wazuh Helps**:
  - Detects injection of `<script>` tags or other malicious payloads in logs or input fields.
  - Integrates with Content Security Policies (CSP) monitoring to block unsafe resources.

### **3. Broken Access Control (OWASP A01:2021)**
- **Problem**: Weak access control policies can expose sensitive data.
- **How Wazuh Helps**:
  - Tracks unauthorized access attempts and excessive privilege escalation requests.
  - Example: Alerts on repeated failed authentication or role escalation requests.

### **4. Security Misconfiguration (OWASP A05:2021)**
- **Problem**: Default configurations and mismanaged settings leave systems vulnerable.
- **How Wazuh Helps**:
  - Monitors for default credentials, open ports, and insecure configurations.
  - Example: Alerts when the application is running in development mode on production.

### **5. Sensitive Data Exposure (OWASP A02:2021)**
- **Problem**: Exposing sensitive data like API keys, passwords, or PII can lead to breaches.
- **How Wazuh Helps**:
  - Monitors logs for sensitive data patterns such as unmasked credit card numbers or tokens.
  - Example: Configures alerts for unencrypted data transmissions.

---

## **Integration with Node.js Architecture**

### **1. Application Security**
- Monitors Node.js applications for suspicious behavior, such as:
  - Repeated login failures.
  - Anomalous API call patterns.
  - Unauthorized file modifications.

### **2. Logging Integration**
- Enhances the existing Node.js logging strategy by centralizing logs and analyzing them for security threats.
- Example: Forwarding Pino logs to Wazuh for anomaly detection.

### **3. Compliance Reporting**
- Automates security compliance reporting for Dev, Prod, and Public servers.
- Provides dashboards for tracking adherence to OWASP and NIST standards.

### **4. Scalability and Flexibility**
- Supports multi-server deployments, making it ideal for complex architectures with Dev, Prod, and Public environments.
- Scales seamlessly with the application as new servers or instances are added.

---

## **Wazuh Setup for Node.js**

### **1. Installing the Wazuh Agent**
Install the Wazuh agent on all servers (Dev, Prod, Public):
```bash
curl -s https://packages.wazuh.com/4.x/wazuh-install.sh | bash
```

### **2. Configuring Node.js Logging**
Update your Node.js logging library (e.g., Winston) to send logs to Wazuh:
```javascript
const { createLogger, transports } = require('winston');
const logger = createLogger({
  transports: [
    new transports.File({ filename: '/var/ossec/logs/app.log' }),
  ],
});
logger.info('Wazuh integration successful');
```

### **3. Setting Up Alerts**
- Configure alert rules in the Wazuh Manager:
  - Detect repeated failed login attempts:
    ```json
    {
      "rule": "Failed login attempts",
      "level": 10,
      "description": "Multiple login failures detected",
    }
    ```
- Configure email or webhook notifications for critical events.

### **4. Centralized Management**
- Access the Wazuh dashboard to view alerts, monitor compliance, and generate reports.

---

## **Benefits of Using Wazuh**
1. **Enhanced Security**:
   - Real-time monitoring and threat detection for all servers.
2. **Simplified Compliance**:
   - Automated compliance with OWASP and NIST standards.
3. **Cost Efficiency**:
   - Open-source platform with enterprise-grade features.
4. **Seamless Integration**:
   - Fits naturally into the Node.js and ReactJS ecosystem.

---

Here's an update for the **Wazuh Integration** markdown page with diagrams to visually represent its integration into the Node.js architecture and its security workflow. These diagrams will help illustrate the concepts and make the document more engaging.

---

## **Wazuh Integration Architecture**

### **Diagram: Integration with Node.js and ReactJS**

```plaintext
                   +-----------------------+                      
                   |    Wazuh Dashboard    |                  
                   |  (Visualization via   |                    
                   |      Kibana)          |                    
                   +-----------------------+                     
                               ▲                                     
                               |                                    
           +-------------------|-------------------+               
           |                   |                   |                
+-----------------+   +-----------------+   +-----------------+     
|  Public Server  |   |  Prod Server    |   |   Dev Server    |     
|  (React Static) |   |  (Node.js API)  |   | (Node.js + FIM) |     
|  - Logs         |   |  - Logs         |   | - Logs          |     
+-----------------+   +-----------------+   +-----------------+     
           |                   |                   |               
           +-------------------|-------------------+               
                               |                                    
                               ▼                                     
                   +-----------------------+                        
                   |   Wazuh Manager       |                        
                   |  (Security Analysis   |                        
                   |   and Alerts)         |                        
                   +-----------------------+                        
```

### **Explanation**:
- **Wazuh Manager**:
  - Collects data from Wazuh agents installed on each server.
  - Analyzes logs and detects security events.
- **Servers**:
  - Public Server logs frontend interactions.
  - Prod Server monitors backend API activity.
  - Dev Server integrates file integrity monitoring (FIM) and security logs.
- **Wazuh Dashboard**:
  - Provides a visual interface for monitoring and compliance reporting.

---

## **OWASP Compliance Workflow**

### **Diagram: Addressing OWASP Top 10 with Wazuh**

```plaintext
          +---------------------+       
          |     OWASP Top 10    |       
          +---------------------+       
                    ▲                  
                    |                  
                    |                  
      +-------------|-------------+   
      |                           |   
+----------------+       +------------------+
| Wazuh Manager  |       | Wazuh Agents     |
| - Log Analysis |       | - File Integrity |
| - Alert Rules  |       | - API Monitoring |
+----------------+       +------------------+
      ▲                           ▲     
      |                           |     
      | Logs/Alerts               | Logs
+-------------+          +----------------+
| Node.js API |          | React Frontend |
+-------------+          +----------------+
```

---

## **Benefits of Diagrams**
These diagrams help visually demonstrate:
1. How **Wazuh** integrates into your Node.js and ReactJS architecture.
2. How it addresses specific OWASP vulnerabilities (e.g., XSS, Injection, Access Control).
3. The centralized role of the **Wazuh Manager** in managing security and compliance across servers.


## **Conclusion**
Wazuh is a powerful addition to any Node.js application, providing unmatched security, monitoring, and compliance capabilities. By integrating Wazuh into your architecture, you can proactively protect your applications and ensure adherence to the highest security standards.

--- 