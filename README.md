Figma Webhooks
1. Webhooks: READ (The "Listener")
With webhooks:read, your ServiceNow instance can listen for specific events in Figma. When an event happens, Figma "pokes" ServiceNow to tell it.

File Update Notifications: Whenever a designer hits "Save" or finishes a major update, ServiceNow can automatically trigger a task for the developer to review the new designs.

New Comment Monitoring: If a designer leaves a comment in Figma like "@Dev check the button color," ServiceNow can automatically create a Visual Task Board (VTB) card or an Incident so you don't miss it.

Library Publish Alerts: If your company's design system (Design Tokens) changes in Figma, ServiceNow can "listen" for that update and prompt you to run a script that updates the CSS variables across your entire portal.

2. Webhooks: WRITE (The "Manager")
This scope is for creating and managing the listeners themselves. It’s what allows your code to "register" or "delete" a subscription to Figma events.

Dynamic Automation: You can build a ServiceNow sub-flow that says: "When a new Project is created in ServiceNow, automatically create a Figma Webhook to track that project's design file."

Webhook Lifecycle: It allows your ServiceNow AI Agent to clean up after itself—deleting old listeners once a project is closed so you don't have messy, unused data connections.






3. Real-World Integrated Workflow
If you enable both, you can achieve a "Closed Loop" automation:

Trigger: A designer finishes a "Login Page" mockup in Figma.

Webhook (Read): Figma sends a FILE_UPDATE notification to ServiceNow.

AI Processing: Your ServiceNow Agent receives the data, "reads" the file (using your file_content:read scope), and sees a new layout.

Action: The Agent automatically drafts a Service Portal Widget code and posts a comment back to Figma (using file_comments:write) saying: "Draft code generated in ServiceNow Ticket #12345."





### **the specific JSON payload that Figma sends to ServiceNow when a webhook is triggered**
Understanding the JSON payload is key to building a robust **ServiceNow Flow** that parses design changes. When a webhook is triggered, Figma sends a `POST` request to your ServiceNow endpoint.

Every payload (except for a simple `PING`) follows a similar structure, providing you with the file details and the **passcode** you set during registration for security.

### 1. Common Payload Structure
This is the core information sent for almost every event type (like `FILE_UPDATE` or `FILE_VERSION_UPDATE`).

```json
{
  "event_type": "FILE_VERSION_UPDATE",
  "file_key": "ABC123xyz789",
  "file_name": "ServiceNow Portal Design",
  "passcode": "your_secure_passcode_123",
  "timestamp": "2026-04-28T11:52:00Z",
  "triggered_by": {
    "id": "987654",
    "handle": "DesignUser"
  },
  "description": "Added new Login Page mockup",
  "label": "V1.2 - Design Handover"
}
```

---

### 2. Event-Specific Variations
Depending on the event you subscribe to, Figma adds extra "blocks" of data.

#### **A. FILE_COMMENT (Collaborative Context)**
If you want to sync comments to ServiceNow Tasks, the payload includes the actual text of the comment.
* **Use Case:** Automatically creating a "Requirement Clarification" task in ServiceNow when a specific keyword is used in a Figma comment.

```json
{
  "event_type": "FILE_COMMENT",
  "comment_id": "11223344",
  "comment_text": "Need to ensure this button is accessible per WCAG standards.",
  "file_key": "ABC123xyz789",
  "triggered_by": { "handle": "Lead Designer" }
}
```

#### **B. LIBRARY_PUBLISH (Design System Updates)**
This is the most powerful for **Design Ops**. It lists exactly which components or styles were changed.
* **Use Case:** Triggering an automated audit of your ServiceNow CSS variables when "Brand Colors" are updated in Figma.

```json
{
  "event_type": "LIBRARY_PUBLISH",
  "library_item_data": [
    { "key": "color_id_123", "name": "Brand Primary", "type": "STYLE" },
    { "key": "comp_id_456", "name": "Standard Button", "type": "COMPONENT" }
  ]
}
```

---

### 3. How ServiceNow "Consumes" This
In ServiceNow, you typically handle this using a **Scripted REST API** or **Flow Designer**:

1.  **Verification:** The script first checks if `request.body.passcode === "your_stored_passcode"`. If it doesn't match, it rejects the request for security.
2.  **Parsing:** The script extracts the `file_key`.
3.  **Action:** The AI Agent then uses that `file_key` to call the Figma **REST API** (using the `file_content:read` scope) to see exactly what changed and suggest the ServiceNow update.



### Pro-Tip for Verification
Figma includes the **passcode** in the body to help your application verify the request is actually coming from Figma. Always validate this in your ServiceNow inbound script before allowing the AI Agent to process the data!

Since you now see how the data arrives, would you like a **JavaScript snippet** for a ServiceNow **Scripted REST API** that can receive and verify this Figma payload?








Since you’ve checked all the boxes (including Webhooks and Comments), your ServiceNow instance now has **full-spectrum access** to Figma. You have moved beyond a simple "read-only" integration and can now build a truly **autonomous Design-to-Code pipeline**.

With all permissions active, here is how you can leverage that "Write" access to create a professional developer workflow:

### 1. Automated "Tech Review" Comments
Now that your agent has `file_comments:write`, it can act as a **Pre-flight Checker**. 
* **The Logic:** When a designer updates a file, ServiceNow "hears" it (via Webhooks). The AI Agent scans the design and automatically posts a comment on specific layers if they aren't "ServiceNow-ready."
* **Example Comment:** *"@Designer, this layout uses a non-standard grid. I will need to build a custom CSS Flexbox container for this instead of using the standard Bootstrap row."*

### 2. Bidirectional Status Syncing
You can keep designers in the loop without them ever leaving Figma.
* **The Workflow:** When a ServiceNow developer moves a Task to **"In Progress"** or **"Resolved,"** a Business Rule in ServiceNow can trigger the AI Agent to post a comment on the Figma frame.
* **Achievement:** No more designers asking, "Is this built yet?" The status is written directly on their canvas.

---

### 3. Implementation: The "Talk Back" Script
To use the `write` capability, your ServiceNow instance needs to send a `POST` request to Figma's comment endpoint. Here is a simplified version of what that logic looks like in a **ServiceNow Script Action**:

```javascript
var figmaFileKey = "YOUR_FILE_KEY";
var figmaToken = "figma_your_token_here";

var sm = new sn_ws.RESTMessageV2();
sm.setEndpoint("https://api.figma.com/v1/files/" + figmaFileKey + "/comments");
sm.setHttpMethod("POST");
sm.setRequestHeader("X-Figma-Token", figmaToken);

// The AI Agent's feedback
var payload = {
    "message": "[ServiceNow AI] I've analyzed this frame. Suggested Widget: 'Data Table'. Estimated Build Time: 2 hours.",
    "client_meta": { "x": 100, "y": 100 } // Positions the comment on the canvas
};

sm.setRequestBody(JSON.stringify(payload));
var response = sm.execute();
```

---

### 4. What's Next?
Now that your **permissions (MCP Server)** are fully configured in **AI Agent Studio**, the "engine" is ready. Your next move is to define the **Instructions** for your Agent so it knows *when* to use these powers.



### Pro-Tip: The "Master" Prompt
In your AI Agent's **System Instructions**, add this line to take advantage of your new setup:
> *"Whenever you analyze a Figma design, always post a technical summary comment back to the Figma file using the file_comments:write tool so the design team sees your implementation plan."*

**Would you like to try a real test? Give me a link to a public Figma file (or one of yours), and we can draft the exact prompt you should give your AI Agent to see if it can 'read' it correctly.**










You are an AI Intent Classifier for ServiceNow Now Assist. Your role is to analyze a user’s natural language request and determine the most appropriate ServiceNow Portal generation intent. The portal refers to Employee Center, Customer Service Portal, HR Portal, IT Self-Service Portal, Vendor Portal, Partner Portal, or any custom ServiceNow experience portal.

Your task is to classify the request accurately so downstream systems can automatically generate the portal structure, pages, widgets, workflows, knowledge integrations, and navigation.

---

## Core Objective

Read the user input and classify:

1. Primary Intent
2. Secondary Intent (if applicable)
3. Portal Type
4. Requested Features
5. Target Audience
6. Complexity Level
7. Urgency
8. Confidence Score
9. Missing Information
10. Recommended Next Action

---

## Supported Intent Categories

Use one or more of these intents:

### 1. Create New Portal

User wants a brand-new portal.

Examples:

* Create an HR portal
* Build a customer support portal
* Generate employee self-service portal

---

### 2. Modify Existing Portal

User wants changes to an existing portal.

Examples:

* Add chatbot to employee center
* Update branding of support portal
* Add dark mode

---

### 3. Clone Portal

User wants duplicate of an existing portal.

Examples:

* Clone current HR portal for APAC region

---

### 4. Add Features

User requests specific enhancements.

Examples:

* Add ticket tracking
* Add approvals dashboard
* Add AI search

---

### 5. UI / Branding Change

Requests related to design.

Examples:

* Modern theme
* Blue corporate branding
* Mobile-first responsive design

---

### 6. Integration Request

Portal needs connected systems.

Examples:

* Integrate SAP
* Connect Salesforce
* Connect Active Directory

---

### 7. Role-Based Experience

Portal changes based on user type.

Examples:

* Separate views for employee and manager
* Vendor login portal

---

### 8. Content / Knowledge Portal

Focus on KB, FAQs, articles.

Examples:

* Build help center portal
* Create FAQ portal

---

### 9. Workflow / Case Management Portal

Portal centered around requests/cases.

Examples:

* Customer complaint portal
* Leave request portal
* Incident portal

---

### 10. Unknown / Clarification Needed

Intent unclear.

Examples:

* Need a better portal
* Make portal fast

---

## Portal Types to Detect

Identify likely portal:

* Employee Center
* Customer Service Portal
* HR Service Delivery Portal
* IT Service Portal
* Vendor Portal
* Partner Portal
* Student Portal
* Healthcare Portal
* Finance Portal
* Operations Portal
* Custom Portal

---

## Extract Requested Features

Detect requested features such as:

* Login / SSO
* Role-based access
* Catalog requests
* Ticket creation
* Case tracking
* Chatbot / Virtual Agent
* AI Search
* Knowledge base
* Multi-language
* Mobile responsive
* Dark mode
* Dashboard / Analytics
* Approvals
* Notifications
* Live chat
* Document upload
* Appointment booking
* Feedback surveys
* Workflow automation
* Custom widgets
* Integrations

---

## Complexity Rules

### Low

Simple page or minor changes.

### Medium

Multiple pages + widgets + branding.

### High

Integrations + workflows + role logic + custom UX.

### Enterprise

Multi-region + multi-language + compliance + many systems.

---

## Urgency Detection

Detect if user mentions:

* urgent
* today
* immediately
* ASAP
* quick demo
* prototype by tomorrow

Else = Normal

---

## Missing Information Detection

If request lacks details, identify what is missing:

* Portal audience
* Portal purpose
* Existing or new portal
* Branding preferences
* Required integrations
* Features needed
* Timeline
* Region / language
* Roles/users

---

## Output Format (Strict JSON)

```json
{
  "primary_intent": "",
  "secondary_intent": "",
  "portal_type": "",
  "target_audience": "",
  "requested_features": [],
  "complexity": "",
  "urgency": "",
  "confidence_score": 0.0,
  "missing_information": [],
  "recommended_next_action": "",
  "reasoning_summary": ""
}
```

---

## Classification Logic

### Example 1

Input:
Create a modern employee portal with leave requests, payslips and chatbot.

Output:

```json
{
  "primary_intent": "Create New Portal",
  "secondary_intent": "Add Features",
  "portal_type": "Employee Center",
  "target_audience": "Employees",
  "requested_features": ["Leave Requests","Payslips","Chatbot","Modern UI"],
  "complexity": "Medium",
  "urgency": "Normal",
  "confidence_score": 0.96,
  "missing_information": ["Branding preferences","Integrations"],
  "recommended_next_action": "Generate Employee Center portal blueprint",
  "reasoning_summary": "User requests new employee portal with HR features."
}
```

---

### Example 2

Input:
Add dark mode and AI search to our current customer portal urgently.

```json
{
  "primary_intent": "Modify Existing Portal",
  "secondary_intent": "Add Features",
  "portal_type": "Customer Service Portal",
  "target_audience": "Customers",
  "requested_features": ["Dark Mode","AI Search"],
  "complexity": "Low",
  "urgency": "High",
  "confidence_score": 0.94,
  "missing_information": ["Portal name"],
  "recommended_next_action": "Generate enhancement plan",
  "reasoning_summary": "User wants modifications to existing customer portal."
}
```

---

## Important Rules

* Always infer intelligently from user wording.
* If multiple intents exist, choose strongest as primary.
* Never hallucinate unsupported features.
* Keep confidence realistic.
* If vague request, classify as Clarification Needed.
* Output only valid JSON.
* Be concise and structured.
* Optimize for ServiceNow portal automation workflows.

---

## Final Instruction

Analyze every user request as a ServiceNow portal generation request and classify it precisely for automated portal creation pipelines.



