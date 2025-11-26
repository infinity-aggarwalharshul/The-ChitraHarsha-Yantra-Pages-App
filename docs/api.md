# API Documentation

## Overview

The ChitraHarsha VPK Yantra platform provides a comprehensive RESTful API for integrating with our 10T SQL cloud ecosystem, AI innovation search, and quantum-encrypted data services.

## Base URL

```
Production: https://api.chitraharsha.com/v1
Staging: https://staging-api.chitraharsha.com/v1
```

## Authentication

### API Key Authentication
```http
Authorization: Bearer YOUR_API_KEY
```

### JWT Token Authentication
```http
Authorization: Bearer YOUR_JWT_TOKEN
```

### Request Headers
```http
Content-Type: application/json
Accept: application/json
X-API-Version: 1.0
```

## Rate Limiting

| Tier | Requests/Hour | Burst Limit |
|------|---------------|-------------|
| Free | 1,000 | 100 |
| Professional | 10,000 | 500 |
| Enterprise | Unlimited | 1,000 |

## Endpoints

### Authentication

#### POST /auth/login
Authenticate user and receive JWT token.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "secure_password",
  "biometric_data": "optional_biometric_hash"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "refresh_token_here",
    "expires_in": 3600,
    "user": {
      "id": "user_id",
      "email": "user@example.com",
      "role": "admin",
      "permissions": ["read", "write", "admin"]
    }
  }
}
```

#### POST /auth/refresh
Refresh JWT token using refresh token.

**Request:**
```json
{
  "refresh_token": "refresh_token_here"
}
```

### Innovation Search

#### POST /innovation/search
Search for patents and analyze innovation novelty.

**Request:**
```json
{
  "query": "AI-powered drone delivery system",
  "filters": {
    "date_range": {
      "start": "2020-01-01",
      "end": "2024-12-31"
    },
    "regions": ["US", "EU", "IN"],
    "categories": ["technology", "transportation"]
  },
  "analysis_depth": "comprehensive"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "novelty_score": 89,
    "patent_matches": {
      "direct": 0,
      "similar": 3,
      "related": 15
    },
    "market_analysis": {
      "demand_score": 92,
      "competition_level": "medium",
      "market_size": "$2.5B"
    },
    "recommendations": [
      "Focus on rural delivery applications",
      "Consider weather-resistant design",
      "Explore autonomous navigation patents"
    ],
    "similar_patents": [
      {
        "id": "US10123456",
        "title": "Autonomous Delivery Vehicle",
        "similarity": 0.75,
        "status": "granted"
      }
    ]
  }
}
```

### Cloud Management

#### GET /cloud/resources
Get current cloud resource usage and statistics.

**Response:**
```json
{
  "success": true,
  "data": {
    "sql_queries": {
      "current_month": 2500000000,
      "limit": 10000000000000,
      "percentage_used": 0.025
    },
    "storage": {
      "used_gb": 1500,
      "compressed_gb": 225,
      "compression_ratio": 0.85
    },
    "bandwidth": {
      "used_gb": 500,
      "limit": "unlimited"
    },
    "edge_nodes": {
      "active": 330,
      "regions": 45,
      "avg_latency_ms": 47
    }
  }
}
```

#### POST /cloud/deploy
Deploy application to global edge network.

**Request:**
```json
{
  "app_name": "my-application",
  "source": {
    "type": "git",
    "url": "https://github.com/user/repo.git",
    "branch": "main"
  },
  "config": {
    "regions": ["us-east", "eu-west", "asia-south"],
    "auto_scale": true,
    "min_instances": 1,
    "max_instances": 100
  },
  "environment": {
    "NODE_ENV": "production",
    "API_URL": "https://api.example.com"
  }
}
```

### Data Analytics

#### POST /analytics/query
Execute high-performance SQL queries on 10T dataset.

**Request:**
```json
{
  "query": "SELECT COUNT(*) FROM user_events WHERE timestamp > '2024-01-01'",
  "parameters": {
    "timeout": 30,
    "cache": true,
    "explain": false
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "results": [
      {"count": 1500000000}
    ],
    "execution_time_ms": 245,
    "rows_scanned": 1500000000,
    "bytes_processed": "2.5TB",
    "cache_hit": false
  }
}
```

### Intelligence Hub

#### GET /intelligence/articles
Get SEO-optimized research articles and insights.

**Query Parameters:**
- `category`: Filter by category (technology, ai, cloud, etc.)
- `limit`: Number of articles (default: 10, max: 100)
- `offset`: Pagination offset
- `sort`: Sort order (newest, popular, trending)

**Response:**
```json
{
  "success": true,
  "data": {
    "articles": [
      {
        "id": "article_123",
        "title": "The Future of Quantum Computing in Cloud Infrastructure",
        "excerpt": "Exploring how quantum computing will revolutionize...",
        "content": "Full article content here...",
        "author": "Dr. Research Team",
        "published_at": "2024-01-15T10:00:00Z",
        "read_time_minutes": 12,
        "seo_score": 98,
        "tags": ["quantum", "cloud", "future-tech"],
        "image_url": "https://cdn.chitraharsha.com/articles/quantum-cloud.jpg"
      }
    ],
    "pagination": {
      "total": 1402,
      "page": 1,
      "per_page": 10,
      "total_pages": 141
    }
  }
}
```

## Error Handling

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request parameters are invalid",
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    },
    "request_id": "req_123456789"
  }
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_REQUEST` | 400 | Request parameters are invalid |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMITED` | 429 | Rate limit exceeded |
| `INTERNAL_ERROR` | 500 | Internal server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |

## SDKs and Libraries

### JavaScript/Node.js
```bash
npm install @chitraharsha/yantra-sdk
```

```javascript
import { ChitraHarshaClient } from '@chitraharsha/yantra-sdk';

const client = new ChitraHarshaClient({
  apiKey: 'your-api-key',
  environment: 'production'
});

// Search for innovations
const results = await client.innovation.search({
  query: 'AI-powered healthcare diagnostics'
});

// Execute SQL query
const data = await client.analytics.query({
  query: 'SELECT * FROM user_analytics LIMIT 100'
});
```

### Python
```bash
pip install chitraharsha-yantra
```

```python
from chitraharsha import YantraClient

client = YantraClient(
    api_key='your-api-key',
    environment='production'
)

# Search for innovations
results = client.innovation.search(
    query='AI-powered healthcare diagnostics'
)

# Execute SQL query
data = client.analytics.query(
    query='SELECT * FROM user_analytics LIMIT 100'
)
```

### cURL Examples

#### Authentication
```bash
curl -X POST https://api.chitraharsha.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "secure_password"
  }'
```

#### Innovation Search
```bash
curl -X POST https://api.chitraharsha.com/v1/innovation/search \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "AI-powered drone delivery system",
    "analysis_depth": "comprehensive"
  }'
```

## Webhooks

### Webhook Events
- `deployment.completed` - Application deployment finished
- `query.completed` - Long-running query finished
- `security.alert` - Security event detected
- `usage.threshold` - Usage limit threshold reached

### Webhook Payload
```json
{
  "event": "deployment.completed",
  "timestamp": "2024-01-15T10:00:00Z",
  "data": {
    "deployment_id": "deploy_123",
    "app_name": "my-application",
    "status": "success",
    "regions": ["us-east", "eu-west"],
    "url": "https://my-application.chitraharsha.app"
  },
  "signature": "webhook_signature_here"
}
```

## Testing

### Sandbox Environment
Use the sandbox environment for testing:
```
Sandbox: https://sandbox-api.chitraharsha.com/v1
```

### Test API Keys
Test API keys are available in the developer dashboard and are prefixed with `test_`.

## Support

### API Support
- **Documentation**: https://docs.chitraharsha.com
- **Status Page**: https://status.chitraharsha.com
- **Support Email**: api-support@chitraharsha.com
- **Developer Forum**: https://community.chitraharsha.com

### SLA Guarantees
- **Uptime**: 99.99% for Enterprise plans
- **Response Time**: <100ms for 95% of requests
- **Support Response**: <24 hours for Enterprise customers

---

For more detailed examples and advanced usage, visit our [Developer Portal](https://developers.chitraharsha.com).