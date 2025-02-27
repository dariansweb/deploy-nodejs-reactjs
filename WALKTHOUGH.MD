# **Division of Youth Services Solution Outline**
## **Key Talking Points**
1. **Empowering DYS and Stakeholders**:
   - The Provider Search tool provides accessible, up-to-date service information to court personnel, improving efficiency and resource discovery.
   - The Juvenile Case Progress Dashboard enhances transparency and reduces the workload on DYS staff by enabling stakeholders to monitor juvenile progress directly.

2. **Prioritizing Security and Compliance**:
   - Adheres to DHS protocols, including robust encryption (AES-256-GCM) and secure authentication mechanisms.
   - Follows industry standards like **OWASP** and **NIST** for secure development and data protection.

3. **Modular and Scalable Design**:
   - Built with ReactJS and NodeJS, ensuring flexibility to adapt to future requirements.
   - Integrates seamlessly with existing DHS infrastructure (e.g., SQL Server, Azure Key Vault) while supporting modern development workflows.

4. **Demonstrated Performance and Integration**:
   - Proof-of-concept pages show how NodeJS replaces or works alongside IIS for hosting and API handling.
   - Charts and analytics demonstrate why ReactJS provides more flexibility and customization compared to PowerBI.

5. **Comprehensive Security Testing**:
   - Includes middleware defenses against attacks like XSS and CSRF.
   - Simulates potential threats and provides mitigation strategies to protect sensitive resources.

---
## **Apps Functionalities**
### 1. **Publicly Accessible Provider Search**
   **Prototype URL:** [HHS Providers Demo](https://www.edoracases.com/hhsproviders)

   **Purpose:** 
   - A public resource for judges, parole officers, and court workers to quickly locate youth service providers.
   - Reduces friction in finding support for juveniles by providing detailed, searchable provider information.

   **Features to Highlight:**
   - **Search and Filters**: 
     - Ability to search by keywords or filter providers by service type, location, or eligibility criteria.
     - Example: Filtering providers who offer counseling or residential services.
   - **Detailed Provider Cards**:
     - Each card includes the provider’s name, address, phone number, email, and specific services offered.
     - Ensures quick access to actionable contact information without unnecessary navigation.
   - **Public Accessibility**:
     - Designed to be mobile-friendly and accessible from any browser.
     - Adheres to DHS security guidelines for publicly available resources.

   **Security Considerations:**
   - The page will:
     - Use HTTPS to encrypt all data.
     - Ensure no sensitive or internal DHS data is exposed.
     - Implements rate-limiting and CAPTCHA to mitigate abuse, ensuring system availability.

---

### 2. **Juvenile Case Progress Dashboard**
   **Prototype URL:** [Client Page Grid Demo](https://www.edoracases.com/demos/clientpagegrid)

   **Purpose:** 
   - A private resource for judges, parole officers, and court workers to track juveniles’ progress post-commitment.
   - Reduces reliance on DYS staff to relay updates via phone or email.

   **Features to Highlight:**
   - **Individualized Dashboards**:
     - Each court personnel will have a personalized view of the juveniles they’ve committed.
     - Displays key information such as current placements, progress updates, and uploaded documents.
   - **Document Management**:
     - Allows court staff to view and download important documents directly.
     - Saves time for both DYS employees and court personnel.
   - **Progress Updates**:
     - Tracks milestones, compliance with court orders, and service engagement.
     - Helps court staff monitor outcomes and identify potential concerns proactively.

   **Security Considerations:**
   - This page will:
     - Be secured behind a login system integrated with DHS’s Active Directory or other authentication services.
     - Use role-based access controls (RBAC) to ensure court personnel can only view the juveniles they are authorized to follow.
     - Audit user actions to maintain accountability and prevent data misuse.

---

## **Development Server Setup**
   - **Purpose**: Build and refine both pages in a secure, isolated environment before public deployment.
       - [View Development Server Overview](STAGING/1-DEVSERVER.MD)   
       - [View Dev/IIS Server Setup](STAGING/0-NODEJS-WINDOWS-SETUP.MD)
       - [IIS Setup](DIAGRAMS/server-iis-setup.png)
       - [No IIS Setup](DIAGRAMS/server-express-setup.png)
       - ![IIS Setup](DIAGRAMS/server-iis-setup.png)
       - ![No IIS Setup](DIAGRAMS/server-express-setup.png)

   - **Security Features**:
     - HTTPS enabled for all internal environments.
     - Firewall rules to restrict access to the development server.
     - Regular vulnerability scans using tools like `npm audit` or Snyk.

   - **Workflow**:
     1. Build features on the development server.
     2. Conduct internal reviews and testing.
     3. Push to a staging server for final approval.
     4. Deploy to the public server for live access.

---

## **ReactJS Charts vs. PowerBI: Proof Points**
1. **Modularity**:
   - ReactJS charting libraries allow you to:
     - Combine multiple chart types on a single page.
     - Add custom interactions (tooltips, zoom, annotations).
     - Style charts to match your application’s theme seamlessly.
     - [Modularity Documentation](MODULARITY/1-MODULARITY.MD)
   - PowerBI, while powerful, is less modular. Embedding a single chart often requires embedding an entire dashboard, which may not match the rest of the webpage.
2. **Web Development Workflow**:
   - ReactJS charts are integrated into the development process as reusable components. This allows:
     - Version control with the rest of the codebase.
     - Custom data sources without platform constraints.
     - [Flexibility Documenation](MODULARITY/2-REACTJS-FLEXIBILITY.MD)
   - PowerBI workflows operate independently from application code, requiring additional integration steps and reliance on external platforms.
3. **Performance**:
   - ReactJS charts are optimized for the web, loading directly as part of the frontend.
   - PowerBI dashboards often require significant data pre-processing and API requests, leading to slower load times.
   - [PowerBI React Client](https://github.com/microsoft/powerbi-client-react)
4. **Platform Independence**:
   - ReactJS works on any platform with a browser.
   - PowerBI ties you to Microsoft’s ecosystem, creating dependencies and potential licensing costs.

---

## **Prove the Security and Integration of NodeJS**
1. **Demo HTTPS Setup**:
   - Show how to configure HTTPS certificates in NodeJS.
   - Use DHS’s internal Certificate Authority to demonstrate compatibility.
   - [NodeJS Https Setup](STAGING/2-NODEJSHTTPS.MD)
2. **Service Management**:
   - Demonstrate PM2 managing NodeJS processes on Windows Server.
   - Explain clustering for performance and fault tolerance.
   - [NodeJS Service Management PM2](STAGING/4-NODEJS-PM2-TOOL.MD)
   - ![PM2 Setup](DIAGRAMS/PM2-Setup.png)       

---

### 4. Security Summary
The DYS Solution employs a multi-layered security approach:
- **Encryption**: All sensitive data is protected using AES-256-GCM for data at rest and HTTPS for data in transit.
- **Authentication and Access Control**: Secure login mechanisms with Azure Key Vault for managing API keys and secrets.
- **Middleware Protections**: Prevents XSS, CSRF, and injection attacks through robust middleware configurations.
These measures align with DHS standards and ensure a secure environment for sensitive juvenile data.

---

### 5. Next Steps
1. Finalize the deployment pipeline:
   - Automate key rotations and secure deployments using CI/CD pipelines.
2. Conduct full security and compliance testing:
   - Simulate additional attack vectors and perform penetration testing.
3. Continue stakeholder engagement:
   - Incorporate feedback from DYS leadership and DHS Security teams.

---

### Closing Remarks
The DYS Solution is designed to meet the highest standards of security, scalability, and functionality. By addressing critical gaps in youth services and empowering stakeholders, this solution represents a significant step forward in operational efficiency and data security for the Department of Human Services.

---