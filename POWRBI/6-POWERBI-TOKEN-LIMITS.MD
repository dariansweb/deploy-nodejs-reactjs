PowerBI’s **embed token limits** govern how and when you can use **embed tokens** to securely display PowerBI content (e.g., dashboards, reports) in applications. These limits ensure secure and efficient usage while preventing misuse or overloading the PowerBI service.

---

## **1. Types of Embed Tokens**
There are two primary types of embed tokens:
1. **User Owns Data**:
   - Used when the user viewing the content is authenticated with their own PowerBI credentials.
   - Requires the user to have appropriate PowerBI licenses and permissions.

2. **App Owns Data**:
   - Used when the application owns the data and provides a seamless experience for users.
   - Requires a PowerBI Pro or PowerBI Premium license and a **Service Principal** to generate tokens.

---

## **2. Key Limits on Embed Tokens**

### **Lifetime**
- **Maximum Token Lifetime**: 
  - Embed tokens are valid for a **maximum of one hour (60 minutes)**.
  - You can specify a shorter duration when generating the token.
- **Recommendation**:
  - For long-lived sessions, implement token renewal logic in your backend.

---

### **Token Scope**
- Embed tokens are **scoped** to specific PowerBI artifacts:
  - A single report, dashboard, or tile.
  - A workspace containing multiple artifacts.

- **Limitation**:
  - Tokens cannot grant access beyond the scope of the artifact(s) specified during token generation.

---

### **Rate Limits**
- **API Rate Limit for Generating Tokens**:
  - The PowerBI REST API enforces rate limits for generating embed tokens to prevent abuse.
  - **Typical Limits**:
    - Approximately **200 requests per minute** for non-Premium capacities.
    - Premium capacities have higher thresholds, depending on the SKU.

- **Recommendation**:
  - Cache embed tokens where appropriate to minimize unnecessary API calls.

---

### **Licensing Requirements**
- **PowerBI Pro**:
  - Both "User Owns Data" and "App Owns Data" embedding require all users or the service principal to have PowerBI Pro licenses.
- **PowerBI Premium**:
  - App embedding is supported without requiring individual users to have PowerBI Pro licenses.
  - Recommended for enterprise-scale embedding due to higher limits and dedicated resources.

---

### **Concurrent Embeds**
- Embed tokens allow embedding multiple reports or dashboards simultaneously, but there are concurrency limits:
  - **Non-Premium Workspaces**: Lower concurrency thresholds (e.g., shared capacity may throttle if multiple users access content).
  - **Premium Workspaces**: Higher concurrency limits, as dedicated resources handle requests.

---

### **Access Control**
- Tokens adhere to the permissions assigned to the user or service principal that generates them:
  - **Row-Level Security (RLS)**: Tokens respect RLS rules, ensuring users see only the data they are authorized to access.
  - **Workspace Permissions**: Tokens cannot override the access restrictions of a PowerBI workspace.

---

## **3. Strategies to Work Within Embed Token Limits**

### **Token Renewal**
- For sessions exceeding one hour, implement token renewal in your backend to regenerate embed tokens as needed.
- Example Flow:
  1. Detect token expiration (or preemptively check before expiration).
  2. Send a request to the backend to generate a new token.
  3. Update the frontend with the new token.

---

### **Token Caching**
- Cache tokens for a short duration in your backend to avoid hitting rate limits when generating tokens for frequent requests.
- Ensure caching respects the token’s expiration time to prevent using invalid tokens.

---

### **Optimize API Usage**
- Batch embed requests where possible to reduce API calls.
- Use "workspace scope" tokens if multiple artifacts are being embedded from the same workspace.

---

### **Leverage PowerBI Premium**
- If embedding is needed at scale (e.g., many users accessing reports simultaneously), use PowerBI Premium:
  - Dedicated resources for embedding content.
  - Higher rate limits for token generation.
  - No per-user licensing requirements.

---

## **4. Example Embed Token Generation**

Here’s an example of generating an embed token using PowerBI’s REST API:

```javascript
const axios = require('axios');

const generateEmbedToken = async (workspaceId, reportId) => {
  const url = `https://api.powerbi.com/v1.0/myorg/groups/${workspaceId}/reports/${reportId}/GenerateToken`;

  const response = await axios.post(
    url,
    {
      accessLevel: "view",
      identities: [
        {
          username: "user@example.com",
          roles: ["RoleName"],
          datasets: ["datasetId"],
        },
      ],
    },
    {
      headers: {
        Authorization: `Bearer YOUR_ACCESS_TOKEN`,
      },
    }
  );

  return response.data.token;
};
```

---

### **Use Case: Token Expiration Handling in ReactJS**
The frontend should gracefully handle token expiration by requesting a new token from the backend.

#### Example Workflow:
1. Detect token expiration in the frontend.
2. Fetch a new token from the backend.
3. Update the PowerBI embed configuration dynamically.

```javascript
const refreshToken = async () => {
  const response = await fetch('/api/getEmbedToken');
  const { embedToken } = await response.json();

  report.updateSettings({
    accessToken: embedToken,
  });
};
```

---

## **5. Conclusion**

Understanding and respecting PowerBI’s embed token limits ensures:
- **Secure Access**: Tokens enforce granular access control.
- **Scalability**: Premium capacities support enterprise-scale usage.
- **Smooth User Experience**: Proper token management avoids interruptions caused by expiration or rate limits.

With strategies like caching, token renewal, and leveraging PowerBI Premium, your integration will remain robust and efficient. Let me know if you’d like help setting up these workflows!