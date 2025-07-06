# n8n Webhook Integration Guide

## Overview
This guide shows you how to integrate your contact form with n8n using webhooks. The form will send data to n8n, which can then process it (send emails, save to databases, integrate with CRM systems, etc.).

## Step 1: Set up n8n Webhook

### 1.1 Create a New Workflow in n8n
1. Log into your n8n instance
2. Create a new workflow
3. Add a **Webhook** node as the trigger

### 1.2 Configure the Webhook Node
1. Click on the Webhook node
2. Set the following parameters:
   - **HTTP Method**: `POST`
   - **Path**: Choose a unique path (e.g., `/contact-form` or `/phaelix-contact`)
   - **Authentication**: None (or set up authentication if needed)
   - **Response Code**: `200`
   - **Response Data**: `First Entry JSON`

3. Copy the webhook URL (it will look like: `https://your-n8n-instance.com/webhook/your-webhook-id`)

### 1.3 Test the Webhook
1. Click "Listen for calls" in the webhook node
2. You can test with curl:
```bash
curl -X POST https://your-n8n-instance.com/webhook/your-webhook-id \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Test","lastName":"User","email":"test@example.com","message":"Test message"}'
```

## Step 2: Update Your Website

### 2.1 Update the Webhook URL
In your `index.html` file, find this line:
```javascript
const N8N_WEBHOOK_URL = 'https://your-n8n-instance.com/webhook/your-webhook-id';
```

Replace it with your actual webhook URL from step 1.2.

### 2.2 The Form Data Structure
Your form sends this data structure to n8n:
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "phone": "+1234567890",
  "company": "Example Corp",
  "projectType": "automation",
  "message": "I need help with automation",
  "timestamp": "2025-01-02T10:00:00.000Z",
  "source": "phaelix-ai-website"
}
```

## Step 3: Process the Data in n8n

### 3.1 Basic Email Notification
Add these nodes after your webhook:

1. **Gmail** node (or any email service):
   - **Operation**: Send Email
   - **To**: `your-email@example.com`
   - **Subject**: `New Contact Form Submission from {{$node["Webhook"].json["firstName"]}} {{$node["Webhook"].json["lastName"]}}`
   - **Email Type**: HTML
   - **Message**: 
   ```html
   <h2>New Contact Form Submission</h2>
   <p><strong>Name:</strong> {{$node["Webhook"].json["firstName"]}} {{$node["Webhook"].json["lastName"]}}</p>
   <p><strong>Email:</strong> {{$node["Webhook"].json["email"]}}</p>
   <p><strong>Phone:</strong> {{$node["Webhook"].json["phone"]}}</p>
   <p><strong>Company:</strong> {{$node["Webhook"].json["company"]}}</p>
   <p><strong>Project Type:</strong> {{$node["Webhook"].json["projectType"]}}</p>
   <p><strong>Message:</strong></p>
   <p>{{$node["Webhook"].json["message"]}}</p>
   <p><strong>Submitted:</strong> {{$node["Webhook"].json["timestamp"]}}</p>
   ```

### 3.2 Save to Google Sheets (Optional)
Add a **Google Sheets** node:
- **Operation**: Append
- **Document ID**: Your Google Sheets ID
- **Sheet**: Sheet1
- **Columns**: A1:I1 (adjust based on your sheet)
- **Values**: Map the webhook data to columns

### 3.3 Send Auto-Reply Email (Optional)
Add another **Gmail** node:
- **To**: `{{$node["Webhook"].json["email"]}}`
- **Subject**: `Thank you for contacting Phaelix AI`
- **Message**: Your auto-reply template

### 3.4 Example Workflow Structure
```
Webhook → Gmail (Notification) → Google Sheets → Gmail (Auto-reply)
```

## Step 4: Advanced Features

### 4.1 Data Validation
Add a **Function** node after webhook to validate data:
```javascript
// Validate required fields
const requiredFields = ['firstName', 'lastName', 'email', 'message'];
const missingFields = [];

for (const field of requiredFields) {
  if (!$input.item.json[field] || $input.item.json[field].trim() === '') {
    missingFields.push(field);
  }
}

if (missingFields.length > 0) {
  throw new Error(`Missing required fields: ${missingFields.join(', ')}`);
}

// Validate email format
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test($input.item.json.email)) {
  throw new Error('Invalid email format');
}

return $input.all();
```

### 4.2 Conditional Logic
Add **IF** nodes to route different project types to different processes:
- Content & Media → Creative team
- Automation → Technical team
- etc.

### 4.3 CRM Integration
Add nodes for your CRM system (HubSpot, Salesforce, etc.) to automatically create leads.

## Step 5: Testing and Deployment

### 5.1 Test the Complete Flow
1. Fill out the form on your website
2. Check that:
   - You receive the notification email
   - Data is saved to your sheet (if configured)
   - Auto-reply is sent (if configured)
   - No errors appear in n8n

### 5.2 Error Handling
Add error handling nodes:
- **Error Trigger** node to catch failures
- **Gmail** node to send error notifications
- **Function** node to log errors

### 5.3 Monitoring
- Set up n8n execution monitoring
- Monitor webhook response times
- Track form submission success rates

## Step 6: Security Considerations

### 6.1 Rate Limiting
Consider adding rate limiting to prevent spam:
```javascript
// In a Function node
const submissions = $node["Webhook"].json;
const userKey = submissions.email || submissions.phone;
// Implement your rate limiting logic here
```

### 6.2 Data Validation
Always validate and sanitize incoming data before processing.

### 6.3 HTTPS
Ensure your webhook URL uses HTTPS for secure data transmission.

## Sample n8n Workflow JSON
```json
{
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "contact-form",
        "responseMode": "responseNode",
        "options": {}
      },
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [240, 300]
    },
    {
      "parameters": {
        "sendTo": "your-email@example.com",
        "subject": "New Contact: {{$node[\"Webhook\"].json[\"firstName\"]}} {{$node[\"Webhook\"].json[\"lastName\"]}}",
        "emailType": "html",
        "message": "<h2>New Contact Form Submission</h2><p><strong>Name:</strong> {{$node[\"Webhook\"].json[\"firstName\"]}} {{$node[\"Webhook\"].json[\"lastName\"]}}</p><p><strong>Email:</strong> {{$node[\"Webhook\"].json[\"email\"]}}</p><p><strong>Phone:</strong> {{$node[\"Webhook\"].json[\"phone\"]}}</p><p><strong>Company:</strong> {{$node[\"Webhook\"].json[\"company\"]}}</p><p><strong>Project Type:</strong> {{$node[\"Webhook\"].json[\"projectType\"]}}</p><p><strong>Message:</strong></p><p>{{$node[\"Webhook\"].json[\"message\"]}}</p>"
      },
      "name": "Gmail",
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 1,
      "position": [460, 300]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={\"status\": \"success\", \"message\": \"Thank you for your message!\"}"
      },
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [680, 300]
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "Gmail",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Gmail": {
      "main": [
        [
          {
            "node": "Respond to Webhook",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

## Troubleshooting

### Common Issues:
1. **CORS Errors**: Ensure your n8n instance allows requests from your domain
2. **Webhook Not Triggered**: Check the webhook URL is correct and accessible
3. **Email Not Sending**: Verify email service credentials and settings
4. **Form Not Submitting**: Check browser console for JavaScript errors

### Testing Tips:
- Use n8n's test/debug mode
- Check browser Network tab for failed requests
- Verify webhook URL is accessible via curl/Postman
- Test with minimal data first, then add complexity

## Next Steps
Once your basic integration is working, consider:
- Adding more sophisticated email templates
- Integrating with your CRM system
- Setting up automated follow-up sequences
- Creating different workflows for different form types
- Adding analytics and reporting