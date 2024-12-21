Here's a diagram illustrating the **PowerBI Token Flow** for embedding dashboards:

1. **User**:
   - Requests a PowerBI dashboard from the **Frontend** (ReactJS).

2. **Frontend (ReactJS)**:
   - Passes the user's authentication request to **Azure AD** (if needed) or directly requests a token from the **Backend API**.

3. **Azure AD (Authentication)**:
   - Authenticates the user and returns an access token to the **Backend API**.

4. **Backend API**:
   - Uses the access token to request an **Embed Token** from the **PowerBI Service** for the specified dashboard or report.

5. **PowerBI Service**:
   - Verifies the request and generates an embed token, which it sends back to the **Backend API**.

6. **Backend API**:
   - Sends the embed token to the **Frontend (ReactJS)**.

7. **Frontend (ReactJS)**:
   - Uses the embed token to securely embed the PowerBI dashboard, completing the flow.

This flow ensures secure handling of authentication and tokens, isolating sensitive operations on the backend. Let me know if you'd like further details!