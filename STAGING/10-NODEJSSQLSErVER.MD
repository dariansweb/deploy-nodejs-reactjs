### **Secure Data Retrieval from SQL Server Using Node.js, Sequelize, and Read-Only Views**

---

## **1. Overview**
This document provides a step-by-step guide to securely configure the Dev Server for accessing the SQL Server using **Node.js**, **Sequelize**, and **Read-Only Views**. The setup ensures:
- **Minimal Permissions**: The Dev Server has `SELECT` access only.
- **Encrypted Communication**: Data in transit is secured with TLS.
- **Active Directory Authentication**: Secure authentication using Azure Active Directory credentials.

---

## **2. Prerequisites**
1. **SQL Server Configuration**:
   - A valid TLS certificate installed on the SQL Server.
   - A **read-only view** created for Dev Server queries.
2. **Dev Server Configuration**:
   - Node.js installed.
   - Required Node.js packages: `sequelize`, `tedious`, `@azure/keyvault-secrets`, and `@azure/identity`.
3. **Active Directory Account**:
   - A dedicated AD account for the Dev Server with `SELECT` permissions on the view.

---

## **3. SQL Server Setup**

### **3.1 Enable TLS on SQL Server**
1. Install a valid TLS certificate issued by your internal Certificate Authority.
2. Configure SQL Server to use the certificate:
   - Open **SQL Server Configuration Manager**.
   - Navigate to **Protocols for SQL Server** > **Properties**.
   - Enable **Force Encryption**.

### **3.2 Create a Read-Only View**
1. Define the view to expose only the required data:
   ```sql
   CREATE VIEW DevServer_ProviderData AS
   SELECT 
       ProviderID, 
       Name, 
       Address, 
       Services, 
       PhoneNumber 
   FROM Providers
   WHERE IsActive = 1;
   ```

2. Grant `SELECT` permissions to the Dev Server's AD account:
   ```sql
   GRANT SELECT ON DevServer_ProviderData TO [Your_DevServer_AD_Account];
   ```

---

## **4. Node.js Setup**

### **4.1 Install Required Packages**
Run the following commands on the Dev Server:
```bash
npm install sequelize tedious @azure/keyvault-secrets @azure/identity
```

### **4.2 Configure Sequelize for Secure Connection**
1. Fetch AD credentials securely using Azure Key Vault:
   ```javascript
   const { DefaultAzureCredential } = require('@azure/identity');
   const { SecretClient } = require('@azure/keyvault-secrets');

   const vaultUrl = "https://<your-keyvault-name>.vault.azure.net/";
   const client = new SecretClient(vaultUrl, new DefaultAzureCredential());

   async function getPassword(secretName) {
       const secret = await client.getSecret(secretName);
       return secret.value;
   }

   const password = await getPassword('sql-ad-password');
   ```

2. Configure Sequelize to connect with Active Directory authentication:
   ```javascript
   const { Sequelize } = require('sequelize');

   const sequelize = new Sequelize('database_name', null, null, {
       dialect: 'mssql',
       host: 'sqlserver.prod.local',
       port: 1433,
       dialectOptions: {
           authentication: {
               type: 'ntlm',
               options: {
                   domain: 'YOUR_AD_DOMAIN',
                   userName: 'your_ad_username',
                   password: password, // Fetched from Azure Key Vault
               },
           },
           encrypt: true, // Enforce TLS
       },
   });

   sequelize.authenticate()
       .then(() => console.log('Secure connection established!'))
       .catch(err => console.error('Error connecting:', err));
   ```

---

## **5. Fetch Data Securely**

### **5.1 Query the Read-Only View**
Use Sequelize to fetch data from the SQL Server:
```javascript
async function fetchProviderData() {
    const [results] = await sequelize.query('SELECT * FROM DevServer_ProviderData');
    console.log(results);
}

fetchProviderData();
```

### **5.2 Encrypt Data Before Storing**
Immediately encrypt the data before storing or transmitting it:
```javascript
const crypto = require('crypto');

const algorithm = 'aes-256-gcm';
const key = crypto.randomBytes(32); // Store securely in Azure Key Vault
const iv = crypto.randomBytes(16);

function encryptData(data) {
    const cipher = crypto.createCipheriv(algorithm, key, iv);
    let encrypted = cipher.update(JSON.stringify(data), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    const authTag = cipher.getAuthTag();
    return { encrypted, iv: iv.toString('hex'), authTag: authTag.toString('hex') };
}

const secureData = encryptData(results);
console.log('Encrypted Data:', secureData);
```

---

## **6. Security Best Practices**
1. **TLS Encryption**:
   - Confirm TLS is active by connecting with tools like Azure Data Studio or `openssl`.

2. **Minimal Permissions**:
   - The Dev Server’s AD account should only have `SELECT` access to the view.

3. **Azure Key Vault**:
   - Store sensitive credentials like the AD account password and encryption keys in Azure Key Vault.

4. **Audit and Monitor**:
   - Enable SQL Server Audit to track queries and connections.
   - Use Azure Monitor for logging and alerting.

---

## **7. Next Steps**
1. Automate data pulls and encryption processes using a scheduled task or cron job.
2. Implement logging for all access and data transfers.
3. Perform regular reviews of permissions and security configurations.

---