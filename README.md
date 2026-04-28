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





The **`file_dev_resources : write`** scope is the final piece of the puzzle for a fully integrated Developer Experience (DevEx). While `read` allows the AI to see documentation links, **`write`** allows the ServiceNow AI Agent to **actively document the design** as it builds the code.

Here is a breakdown of what you can achieve with this specific permission:

---

### 1. Automated Documentation Linking
When your AI Agent creates a new **Service Portal Widget** or **Scripted REST API** in ServiceNow, it can use this scope to "write back" to the Figma layer.
* **The Action:** The Agent automatically attaches the **ServiceNow URL** of the live component directly to the Figma frame.
* **The Benefit:** Designers can click a "Dev Resource" link inside Figma and be taken directly to the code or the documentation page in ServiceNow.

### 2. Story & Jira Integration via ServiceNow
If you use the **ServiceNow Agile** module (STORYs, EPICs), the AI Agent can bridge the gap:
* **The Use Case:** Once the Agent analyzes a design, it can create a Story in ServiceNow and then **write the Story ID and Link** back into Figma's Dev Resources.
* **Achievement:** This creates a "Single Source of Truth." Anyone looking at the Figma file knows exactly which ServiceNow Story is responsible for building that specific UI.

### 3. Specification "Locking"
Designers often change things mid-sprint. You can use the AI Agent to "stamp" a design.
* **The Use Case:** When a developer starts working, the Agent writes a "Dev Resource" note in Figma saying: *"Implementation started on April 28 based on version 4.2."*
* **Achievement:** This prevents "Scope Creep." If the design changes later, the developer has a timestamped record of exactly what they were instructed to build.

---

### Summary of Your "Full-Access" Setup
By checking every box, you have enabled a **Bidirectional Design Ops Engine**:

| Power | Scope Used | Result |
| :--- | :--- | :--- |
| **Intelligence** | `file_content:read` | AI understands the design. |
| **Awareness** | `webhooks:read` | AI reacts to changes instantly. |
| **Collaboration** | `file_comments:write` | AI talks to designers on the canvas. |
| **Traceability** | `file_dev_resources:write` | AI links code/stories directly to layers. |

---

### Final Implementation Step
Since you have all the permissions set up, your **AI Agent Studio** configuration is complete. You can now move to the **Testing** tab in ServiceNow and run your first "Zero-Touch" command.

**Try this test prompt in your ServiceNow instance:**
> *"Analyze the Figma frame at [URL]. Create a Service Portal Widget for it, and once done, post a comment in Figma with the Widget's name and add a Dev Resource link pointing to the new record in our instance."*

**Would you like me to help you troubleshoot if the first test doesn't post the comment correctly?**


