# API Integration Guide

## Overview
This guide explains how to integrate the Infra Control Center frontend with your backend APIs.

## Architecture

```
┌─────────────────────────────────────────┐
│         Frontend (HTML/CSS/JS)          │
│  - Dashboard UI                         │
│  - AI Assistant Interface               │
└─────────────────┬───────────────────────┘
                  │
                  │ REST/WebSocket
                  │
┌─────────────────▼───────────────────────┐
│         Backend API Layer               │
│  - Authentication                       │
│  - Network Device Management            │
│  - AI Processing                        │
│  - Data Analytics                       │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         Network Infrastructure          │
│  - Routers / Switches                   │
│  - Wireless Access Points               │
│  - Monitoring Systems                   │
└─────────────────────────────────────────┘
```

## API Endpoints

### Authentication

#### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password123"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "123",
    "username": "admin",
    "role": "network_admin"
  }
}
```

#### Logout
```http
POST /api/auth/logout
Authorization: Bearer {token}
```

### Network Tools

#### Get Operational Metrics
```http
GET /api/metrics/operational
Authorization: Bearer {token}

Response:
{
  "uptime": "99.9%",
  "devices_online": 150,
  "alerts_active": 3,
  "bandwidth_usage": {
    "in": "2.5 Gbps",
    "out": "1.8 Gbps"
  }
}
```

#### ACL Live Check
```http
POST /api/tools/acl-check
Authorization: Bearer {token}
Content-Type: application/json

{
  "device_id": "router-01",
  "acl_name": "BLOCK_TRAFFIC",
  "source_ip": "192.168.1.100",
  "dest_ip": "10.0.0.50",
  "protocol": "tcp",
  "port": 443
}

Response:
{
  "result": "allowed",
  "matching_rule": "rule-5",
  "explanation": "Traffic is permitted by rule-5: allow tcp any 10.0.0.0/24 443"
}
```

#### MAC Address Lookup
```http
GET /api/tools/mac-address/{mac_address}
Authorization: Bearer {token}

Response:
{
  "mac": "00:1A:2B:3C:4D:5E",
  "device_name": "laptop-john",
  "ip_address": "192.168.1.105",
  "vlan": 10,
  "switch_port": "GigabitEthernet1/0/5",
  "last_seen": "2026-02-19T10:30:00Z"
}
```

#### Config Validation
```http
POST /api/tools/validate-config
Authorization: Bearer {token}
Content-Type: application/json

{
  "device_id": "switch-03",
  "config_text": "interface GigabitEthernet1/0/1\n switchport mode access\n switchport access vlan 10"
}

Response:
{
  "valid": true,
  "warnings": ["VLAN 10 not defined in database"],
  "errors": [],
  "compliance_score": 95
}
```

### AI Assistant

#### Send Message
```http
POST /api/ai/chat
Authorization: Bearer {token}
Content-Type: application/json

{
  "message": "Analyze all devices at site HQ-01",
  "conversation_id": "abc-123",
  "context": {
    "site": "HQ-01"
  }
}

Response:
{
  "response": "I've analyzed 45 devices at HQ-01. All routers are operational...",
  "conversation_id": "abc-123",
  "metadata": {
    "devices_checked": 45,
    "issues_found": 2
  }
}
```

#### Stream AI Response (SSE)
```http
GET /api/ai/chat/stream
Authorization: Bearer {token}
Connection: keep-alive

Event stream:
data: {"type": "token", "content": "Analyzing"}
data: {"type": "token", "content": " devices"}
data: {"type": "complete"}
```

## Frontend Integration

### Configuration
Create `config.js`:

```javascript
const API_CONFIG = {
    baseURL: process.env.API_BASE_URL || 'https://api.yourdomain.com',
    timeout: 30000,
    headers: {
        'Content-Type': 'application/json'
    }
};
```

### Authentication Service

```javascript
class AuthService {
    constructor() {
        this.token = localStorage.getItem('auth_token');
    }

    async login(username, password) {
        try {
            const response = await fetch(`${API_CONFIG.baseURL}/api/auth/login`, {
                method: 'POST',
                headers: API_CONFIG.headers,
                body: JSON.stringify({ username, password })
            });

            if (!response.ok) {
                throw new Error('Login failed');
            }

            const data = await response.json();
            this.token = data.token;
            localStorage.setItem('auth_token', data.token);
            localStorage.setItem('user', JSON.stringify(data.user));
            
            return data;
        } catch (error) {
            console.error('Login error:', error);
            throw error;
        }
    }

    logout() {
        this.token = null;
        localStorage.removeItem('auth_token');
        localStorage.removeItem('user');
        window.location.href = '/login.html';
    }

    getToken() {
        return this.token;
    }

    isAuthenticated() {
        return !!this.token;
    }

    getAuthHeaders() {
        return {
            ...API_CONFIG.headers,
            'Authorization': `Bearer ${this.token}`
        };
    }
}

const authService = new AuthService();
```

### API Client

```javascript
class APIClient {
    constructor() {
        this.baseURL = API_CONFIG.baseURL;
    }

    async get(endpoint) {
        try {
            const response = await fetch(`${this.baseURL}${endpoint}`, {
                method: 'GET',
                headers: authService.getAuthHeaders()
            });

            if (!response.ok) {
                if (response.status === 401) {
                    authService.logout();
                }
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }

            return await response.json();
        } catch (error) {
            console.error(`GET ${endpoint} failed:`, error);
            throw error;
        }
    }

    async post(endpoint, data) {
        try {
            const response = await fetch(`${this.baseURL}${endpoint}`, {
                method: 'POST',
                headers: authService.getAuthHeaders(),
                body: JSON.stringify(data)
            });

            if (!response.ok) {
                if (response.status === 401) {
                    authService.logout();
                }
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }

            return await response.json();
        } catch (error) {
            console.error(`POST ${endpoint} failed:`, error);
            throw error;
        }
    }
}

const apiClient = new APIClient();
```

### Example: Opening Applications

Update the `openApp` function in `index.html`:

```javascript
async function openApp(appName) {
    try {
        // Show loading state
        const button = event.target;
        const originalText = button.innerHTML;
        button.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Loading...';
        button.disabled = true;

        // Call API to launch app
        const response = await apiClient.post(`/api/apps/${appName}/launch`, {
            timestamp: new Date().toISOString()
        });

        // Redirect to app URL
        if (response.url) {
            window.location.href = response.url;
        } else {
            alert(`${appName} application opened successfully!`);
        }

    } catch (error) {
        alert(`Failed to open ${appName}: ${error.message}`);
        button.innerHTML = originalText;
        button.disabled = false;
    }
}
```

### Example: AI Chat Integration

Update `ai-assistant.html`:

```javascript
async function sendMessage() {
    const chatInput = document.getElementById('chatInput');
    const message = chatInput.value.trim();
    
    if (!message) return;

    try {
        // Add user message to UI
        addMessageToChat('user', message);
        chatInput.value = '';

        // Show typing indicator
        showTypingIndicator();

        // Send to API
        const response = await apiClient.post('/api/ai/chat', {
            message: message,
            conversation_id: getConversationId()
        });

        // Remove typing indicator
        hideTypingIndicator();

        // Add AI response to UI
        addMessageToChat('assistant', response.response);

    } catch (error) {
        hideTypingIndicator();
        addMessageToChat('error', 'Failed to get response. Please try again.');
        console.error('Chat error:', error);
    }
}

function addMessageToChat(role, content) {
    const chatContainer = document.getElementById('chatMessages');
    const messageDiv = document.createElement('div');
    messageDiv.className = `message ${role}`;
    messageDiv.innerHTML = `
        <div class="message-avatar">
            <i class="fas fa-${role === 'user' ? 'user' : 'robot'}"></i>
        </div>
        <div class="message-content">${content}</div>
    `;
    chatContainer.appendChild(messageDiv);
    chatContainer.scrollTop = chatContainer.scrollHeight;
}
```

### Example: Streaming AI Responses

```javascript
async function sendMessageWithStreaming() {
    const message = chatInput.value.trim();
    if (!message) return;

    const conversationId = getConversationId();
    const eventSource = new EventSource(
        `${API_CONFIG.baseURL}/api/ai/chat/stream?message=${encodeURIComponent(message)}&conversation_id=${conversationId}`,
        {
            headers: authService.getAuthHeaders()
        }
    );

    let responseText = '';
    const messageDiv = createMessageElement('assistant');

    eventSource.onmessage = (event) => {
        const data = JSON.parse(event.data);
        
        if (data.type === 'token') {
            responseText += data.content;
            updateMessageContent(messageDiv, responseText);
        } else if (data.type === 'complete') {
            eventSource.close();
        }
    };

    eventSource.onerror = () => {
        eventSource.close();
        addMessageToChat('error', 'Connection lost. Please try again.');
    };
}
```

## WebSocket Integration

For real-time updates:

```javascript
class WebSocketClient {
    constructor() {
        this.ws = null;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
    }

    connect() {
        const token = authService.getToken();
        this.ws = new WebSocket(`wss://api.yourdomain.com/ws?token=${token}`);

        this.ws.onopen = () => {
            console.log('WebSocket connected');
            this.reconnectAttempts = 0;
        };

        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.handleMessage(data);
        };

        this.ws.onerror = (error) => {
            console.error('WebSocket error:', error);
        };

        this.ws.onclose = () => {
            console.log('WebSocket disconnected');
            this.reconnect();
        };
    }

    handleMessage(data) {
        switch (data.type) {
            case 'device_status':
                updateDeviceStatus(data.device_id, data.status);
                break;
            case 'alert':
                showAlert(data.message);
                break;
            case 'metrics_update':
                updateMetrics(data.metrics);
                break;
        }
    }

    send(data) {
        if (this.ws && this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(data));
        }
    }

    reconnect() {
        if (this.reconnectAttempts < this.maxReconnectAttempts) {
            this.reconnectAttempts++;
            setTimeout(() => this.connect(), 3000 * this.reconnectAttempts);
        }
    }
}

const wsClient = new WebSocketClient();
wsClient.connect();
```

## Error Handling

```javascript
class APIError extends Error {
    constructor(message, status, response) {
        super(message);
        this.name = 'APIError';
        this.status = status;
        this.response = response;
    }
}

function showErrorNotification(error) {
    const notification = document.createElement('div');
    notification.className = 'error-notification';
    notification.innerHTML = `
        <i class="fas fa-exclamation-circle"></i>
        <span>${error.message}</span>
        <button onclick="this.parentElement.remove()">
            <i class="fas fa-times"></i>
        </button>
    `;
    document.body.appendChild(notification);

    setTimeout(() => notification.remove(), 5000);
}
```

## Rate Limiting

```javascript
class RateLimiter {
    constructor(maxRequests, timeWindow) {
        this.maxRequests = maxRequests;
        this.timeWindow = timeWindow;
        this.requests = [];
    }

    async checkLimit() {
        const now = Date.now();
        this.requests = this.requests.filter(time => now - time < this.timeWindow);

        if (this.requests.length >= this.maxRequests) {
            const waitTime = this.timeWindow - (now - this.requests[0]);
            throw new Error(`Rate limit exceeded. Try again in ${Math.ceil(waitTime / 1000)}s`);
        }

        this.requests.push(now);
    }
}

const rateLimiter = new RateLimiter(10, 60000); // 10 requests per minute
```

## Testing APIs

```javascript
// Test authentication
async function testAuth() {
    try {
        const result = await authService.login('testuser', 'password');
        console.log('Login successful:', result);
    } catch (error) {
        console.error('Login failed:', error);
    }
}

// Test API endpoint
async function testEndpoint(endpoint) {
    try {
        const data = await apiClient.get(endpoint);
        console.log(`${endpoint} response:`, data);
    } catch (error) {
        console.error(`${endpoint} failed:`, error);
    }
}
```

---

For more information, see the main [README.md](../README.md).
