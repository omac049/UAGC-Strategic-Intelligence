# API Reference & Integration Specifications

## Purpose
This document provides complete API specifications for all UAGC Cookie Personalization system integrations, including Lead API, analytics platforms, CRM systems, and cross-domain communication.

## API Architecture Overview

### Integration Flow
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Lead API      │    │   Salesforce    │
│   (uagc.edu)    │───►│   (Workato)     │───►│   CRM           │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   GA4 Events    │    │   Cookie Sync   │    │   Lead Scoring  │
│   & Analytics   │    │   PostMessage   │    │   & Attribution │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Authentication & Security
All API endpoints require proper authentication and follow security best practices:

```javascript
// Standard API request configuration
const apiConfig = {
    baseURL: 'https://api.uagc.edu',
    timeout: 30000,
    headers: {
        'Content-Type': 'application/json',
        'X-API-Version': '2.0',
        'X-Request-ID': generateRequestId(),
        'Authorization': `Bearer ${getAuthToken()}`
    },
    retryPolicy: {
        maxRetries: 3,
        backoffStrategy: 'exponential'
    }
};
```

## Lead API Integration

### RFI Submission Endpoint
Handles Request for Information form submissions with comprehensive field mapping.

**Endpoint:** `POST /api/v2/leads/rfi`

**Request Format:**
```json
{
    "personal_info": {
        "first_name": "string",
        "last_name": "string", 
        "email": "string",
        "phone": "string",
        "date_of_birth": "YYYY-MM-DD",
        "address": {
            "street": "string",
            "city": "string",
            "state": "string",
            "zip_code": "string",
            "country": "string"
        }
    },
    "academic_info": {
        "program_interest": "string",
        "education_level": "high_school|some_college|associates|bachelors|masters|doctoral",
        "military_status": "active|veteran|spouse|dependent|none",
        "start_date_preference": "YYYY-MM-DD",
        "enrollment_type": "full_time|part_time|flexible"
    },
    "tracking_data": {
        "utm_source": "string",
        "utm_medium": "string", 
        "utm_campaign": "string",
        "utm_content": "string",
        "utm_term": "string",
        "source_id": "string",
        "affiliate_id": "string",
        "click_id": "string",
        "session_id": "string",
        "visitor_id": "string",
        "journey_stage": "awareness|consideration|engagement",
        "page_url": "string",
        "referrer": "string",
        "user_agent": "string",
        "ip_address": "string",
        "timestamp": "ISO-8601"
    },
    "consent_data": {
        "privacy_policy_accepted": true,
        "marketing_consent": "boolean",
        "sms_consent": "boolean", 
        "email_consent": "boolean",
        "consent_timestamp": "ISO-8601",
        "consent_version": "string"
    }
}
```

**Response Format:**
```json
{
    "status": "success|error",
    "lead_id": "string",
    "salesforce_id": "string", 
    "lead_score": "number",
    "next_steps": {
        "advisor_assignment": "string",
        "follow_up_scheduled": "ISO-8601",
        "recommended_actions": ["string"]
    },
    "tracking": {
        "conversion_id": "string",
        "attribution_data": {},
        "journey_update": "engagement"
    },
    "errors": [
        {
            "field": "string",
            "code": "string", 
            "message": "string"
        }
    ]
}
```

**Implementation Example:**
```javascript
// RFI submission with comprehensive error handling
async function submitRFI(formData) {
    try {
        // Validate form data
        const validationResult = validateRFIData(formData);
        if (!validationResult.isValid) {
            throw new ValidationError(validationResult.errors);
        }
        
        // Prepare tracking data
        const trackingData = {
            ...getUTMData(),
            ...getVisitorData(),
            session_id: getSessionId(),
            visitor_id: getVisitorId(),
            journey_stage: getCurrentJourneyStage(),
            timestamp: new Date().toISOString()
        };
        
        // Submit to Lead API
        const response = await fetch('/api/v2/leads/rfi', {
            method: 'POST',
            headers: apiConfig.headers,
            body: JSON.stringify({
                ...formData,
                tracking_data: trackingData,
                consent_data: getConsentData()
            })
        });
        
        if (!response.ok) {
            throw new APIError(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const result = await response.json();
        
        // Handle successful submission
        if (result.status === 'success') {
            // Update journey stage
            updateJourneyStage('engagement');
            
            // Track conversion
            trackConversion('rfi_submission', result.lead_id);
            
            // Update lead scoring
            updateLeadScore(result.lead_score);
            
            // Sync across domains
            syncLeadDataAcrossDomains(result);
        }
        
        return result;
        
    } catch (error) {
        // Comprehensive error handling
        handleAPIError(error);
        throw error;
    }
}
```

### Application Submission Endpoint
Handles application form submissions for the stealth application process.

**Endpoint:** `POST /api/v2/applications`

**Request Format:**
```json
{
    "applicant_info": {
        "lead_id": "string",
        "personal_info": {}, // Same as RFI
        "academic_history": {
            "previous_education": [
                {
                    "institution": "string",
                    "degree_type": "string",
                    "field_of_study": "string",
                    "graduation_date": "YYYY-MM-DD",
                    "gpa": "number",
                    "credits_earned": "number"
                }
            ],
            "transcripts_submitted": "boolean",
            "test_scores": {}
        },
        "program_selection": {
            "primary_program": "string",
            "secondary_program": "string",
            "start_term": "string",
            "enrollment_type": "string"
        }
    },
    "application_metadata": {
        "application_source": "stealth|direct|advisor_assisted",
        "completion_percentage": "number",
        "time_to_complete": "number", // seconds
        "save_points": ["string"], // Auto-save checkpoints
        "referral_source": "string"
    },
    "tracking_data": {}, // Same as RFI
    "financial_aid": {
        "fafsa_completed": "boolean",
        "aid_interest": "boolean",
        "military_benefits": "boolean"
    }
}
```

**Response Format:**
```json
{
    "status": "success|pending|error",
    "application_id": "string",
    "salesforce_application_id": "string",
    "next_steps": {
        "required_documents": ["string"],
        "advisor_contact": {
            "name": "string",
            "email": "string", 
            "phone": "string",
            "calendar_link": "string"
        },
        "application_deadline": "ISO-8601",
        "estimated_decision_date": "ISO-8601"
    },
    "status_tracking": {
        "current_stage": "submitted|under_review|decision_pending|accepted|declined",
        "progress_percentage": "number",
        "last_updated": "ISO-8601"
    }
}
```

## Analytics API Integration

### GA4 Event Tracking
Enhanced Google Analytics 4 integration with custom events and parameters.

**Event Configuration:**
```javascript
// GA4 custom events for UAGC tracking
const ga4Events = {
    // Journey progression events
    journey_stage_update: {
        event_name: 'journey_stage_update',
        parameters: {
            previous_stage: 'string',
            new_stage: 'string', 
            trigger_action: 'string',
            time_in_previous_stage: 'number'
        }
    },
    
    // Program interest tracking
    program_interest: {
        event_name: 'program_interest',
        parameters: {
            program_name: 'string',
            program_category: 'string',
            interaction_type: 'view|save|compare|share',
            page_location: 'string'
        }
    },
    
    // Lead generation events
    lead_generation: {
        event_name: 'generate_lead',
        parameters: {
            lead_type: 'rfi|application|contact',
            program_interest: 'string',
            lead_score: 'number',
            attribution_source: 'string'
        }
    },
    
    // Personalization events
    personalization_display: {
        event_name: 'personalization_display',
        parameters: {
            personalization_type: 'content|cta|recommendation',
            visitor_profile: 'string',
            ab_test_variant: 'string',
            content_id: 'string'
        }
    }
};

// GA4 event tracking implementation
function trackGA4Event(eventName, parameters = {}) {
    // Add standard parameters
    const standardParams = {
        page_title: document.title,
        page_location: window.location.href,
        visitor_id: getVisitorId(),
        session_id: getSessionId(),
        journey_stage: getCurrentJourneyStage(),
        timestamp: Date.now()
    };
    
    // Merge with custom parameters
    const allParams = { ...standardParams, ...parameters };
    
    // Send to GA4
    gtag('event', eventName, allParams);
    
    // Also send to custom analytics if configured
    if (window.customAnalytics) {
        window.customAnalytics.track(eventName, allParams);
    }
}
```

### Enhanced Conversions Setup
```javascript
// GA4 Enhanced Conversions configuration
gtag('config', 'GA_MEASUREMENT_ID', {
    enhanced_conversions: true,
    enhanced_conversion_data: {
        email: 'user_email',
        phone_number: 'user_phone',
        first_name: 'user_first_name',
        last_name: 'user_last_name',
        home_address: {
            street: 'user_street',
            city: 'user_city', 
            region: 'user_state',
            postal_code: 'user_zip',
            country: 'user_country'
        }
    },
    custom_map: {
        'custom_parameter_1': 'journey_stage',
        'custom_parameter_2': 'program_interest',
        'custom_parameter_3': 'lead_score',
        'custom_parameter_4': 'visitor_profile',
        'custom_parameter_5': 'attribution_source'
    }
});

// Enhanced conversion tracking for leads
function trackEnhancedConversion(conversionData, userInfo) {
    gtag('event', 'conversion', {
        send_to: 'GA_MEASUREMENT_ID/CONVERSION_LABEL',
        value: conversionData.value,
        currency: 'USD',
        transaction_id: conversionData.lead_id,
        enhanced_conversion_data: {
            email: userInfo.email,
            phone_number: userInfo.phone,
            first_name: userInfo.first_name,
            last_name: userInfo.last_name
        },
        custom_parameter_1: conversionData.journey_stage,
        custom_parameter_2: conversionData.program_interest,
        custom_parameter_3: conversionData.lead_score
    });
}
```

## Cross-Domain Communication API

### PostMessage Protocol
Secure cross-domain communication between UAGC properties.

**Message Format:**
```javascript
// Standard cross-domain message structure
const crossDomainMessage = {
    type: 'UAGC_MESSAGE',
    action: 'VISITOR_SYNC|JOURNEY_UPDATE|CONSENT_UPDATE|LEAD_DATA',
    payload: {
        visitor_id: 'string',
        session_id: 'string',
        data: {}, // Action-specific data
        timestamp: 'number',
        source_domain: 'string'
    },
    security: {
        checksum: 'string',
        version: '2.0'
    }
};
```

**Implementation:**
```javascript
// Cross-domain communication manager
class UAGCCrossDomainAPI {
    constructor() {
        this.allowedOrigins = [
            'https://uagc.edu',
            'https://cloud.mail.uagc.edu', 
            'https://portal.uagc.edu'
        ];
        this.messageHandlers = new Map();
        this.setupMessageListener();
    }
    
    setupMessageListener() {
        window.addEventListener('message', (event) => {
            // Validate origin
            if (!this.allowedOrigins.includes(event.origin)) {
                console.warn('Rejected message from unauthorized origin:', event.origin);
                return;
            }
            
            // Validate message format
            if (!this.isValidMessage(event.data)) {
                console.warn('Invalid message format received');
                return;
            }
            
            // Handle message
            this.handleMessage(event.data, event.origin);
        });
    }
    
    sendMessage(targetOrigin, action, payload) {
        const message = {
            type: 'UAGC_MESSAGE',
            action,
            payload: {
                visitor_id: getVisitorId(),
                session_id: getSessionId(),
                data: payload,
                timestamp: Date.now(),
                source_domain: window.location.origin
            },
            security: {
                checksum: this.calculateChecksum(payload),
                version: '2.0'
            }
        };
        
        // Send via iframe or direct postMessage
        if (targetOrigin === window.location.origin) {
            // Same origin - direct call
            this.handleMessage(message, window.location.origin);
        } else {
            // Cross-origin - use iframe
            this.sendViaIframe(targetOrigin, message);
        }
    }
    
    registerHandler(action, handler) {
        this.messageHandlers.set(action, handler);
    }
    
    handleMessage(message, origin) {
        const handler = this.messageHandlers.get(message.action);
        if (handler) {
            handler(message.payload, origin);
        }
    }
}

// Usage example
const crossDomainAPI = new UAGCCrossDomainAPI();

// Register handlers
crossDomainAPI.registerHandler('VISITOR_SYNC', (payload, origin) => {
    syncVisitorData(payload.data);
});

crossDomainAPI.registerHandler('JOURNEY_UPDATE', (payload, origin) => {
    updateJourneyStage(payload.data.new_stage);
});

// Send visitor data to application portal
crossDomainAPI.sendMessage(
    'https://portal.uagc.edu',
    'VISITOR_SYNC',
    {
        journey_stage: getCurrentJourneyStage(),
        lead_data: getLeadData(),
        personalization_profile: getPersonalizationProfile()
    }
);
```

## Salesforce CRM Integration

### Lead Creation & Updates
Integration with Salesforce for lead management and scoring.

**Endpoint:** `POST /api/v2/salesforce/leads`

**Request Format:**
```json
{
    "lead_data": {
        "FirstName": "string",
        "LastName": "string",
        "Email": "string",
        "Phone": "string",
        "Company": "Self",
        "LeadSource": "Website",
        "Status": "New|Contacted|Qualified|Unqualified",
        "Program_Interest__c": "string",
        "Journey_Stage__c": "string",
        "Lead_Score__c": "number",
        "UTM_Source__c": "string",
        "UTM_Medium__c": "string",
        "UTM_Campaign__c": "string"
    },
    "custom_fields": {
        "Military_Status__c": "string",
        "Education_Level__c": "string",
        "Start_Date_Preference__c": "date",
        "Enrollment_Type__c": "string",
        "Consent_Marketing__c": "boolean",
        "Consent_SMS__c": "boolean"
    },
    "tracking_data": {
        "Visitor_ID__c": "string",
        "Session_ID__c": "string",
        "Page_URL__c": "string",
        "Referrer__c": "string",
        "Attribution_Data__c": "text"
    }
}
```

**Lead Scoring API:**
```javascript
// Automated lead scoring integration
async function updateLeadScore(leadId, behaviorData) {
    const scoreFactors = {
        // Demographic scoring
        military_status: behaviorData.military_status === 'veteran' ? 15 : 0,
        education_level: getEducationScore(behaviorData.education_level),
        
        // Behavioral scoring  
        pages_viewed: Math.min(behaviorData.pages_viewed * 2, 20),
        time_on_site: Math.min(Math.floor(behaviorData.time_on_site / 60), 15),
        return_visits: Math.min(behaviorData.return_visits * 5, 25),
        
        // Engagement scoring
        rfi_submitted: behaviorData.rfi_submitted ? 30 : 0,
        advisor_contact: behaviorData.advisor_contact ? 20 : 0,
        application_started: behaviorData.application_started ? 40 : 0,
        
        // Program interest depth
        program_research_depth: calculateProgramResearchScore(behaviorData.program_interactions)
    };
    
    const totalScore = Object.values(scoreFactors).reduce((sum, score) => sum + score, 0);
    
    // Update in Salesforce
    const response = await fetch(`/api/v2/salesforce/leads/${leadId}/score`, {
        method: 'PUT',
        headers: apiConfig.headers,
        body: JSON.stringify({
            Lead_Score__c: totalScore,
            Score_Factors__c: JSON.stringify(scoreFactors),
            Last_Score_Update__c: new Date().toISOString()
        })
    });
    
    return response.json();
}
```

## Error Handling & Monitoring

### Standardized Error Responses
```javascript
// Standard API error format
const apiError = {
    error: {
        code: 'string', // ERROR_CODE_CONSTANT
        message: 'string', // Human-readable message
        details: {}, // Additional error context
        timestamp: 'ISO-8601',
        request_id: 'string',
        documentation_url: 'string'
    }
};

// Common error codes
const ERROR_CODES = {
    VALIDATION_ERROR: 'VALIDATION_ERROR',
    AUTHENTICATION_ERROR: 'AUTHENTICATION_ERROR',
    RATE_LIMIT_EXCEEDED: 'RATE_LIMIT_EXCEEDED',
    INTERNAL_SERVER_ERROR: 'INTERNAL_SERVER_ERROR',
    SERVICE_UNAVAILABLE: 'SERVICE_UNAVAILABLE',
    LEAD_DUPLICATE: 'LEAD_DUPLICATE',
    INVALID_PROGRAM: 'INVALID_PROGRAM',
    CONSENT_REQUIRED: 'CONSENT_REQUIRED'
};
```

### API Monitoring & Logging
```javascript
// API performance and error monitoring
class APIMonitor {
    constructor() {
        this.metrics = {
            requests: 0,
            errors: 0,
            response_times: [],
            error_rates: new Map()
        };
    }
    
    logRequest(endpoint, method, responseTime, status) {
        this.metrics.requests++;
        this.metrics.response_times.push(responseTime);
        
        if (status >= 400) {
            this.metrics.errors++;
            const errorKey = `${method} ${endpoint}`;
            const currentCount = this.metrics.error_rates.get(errorKey) || 0;
            this.metrics.error_rates.set(errorKey, currentCount + 1);
        }
        
        // Send to monitoring service
        this.sendToMonitoring({
            endpoint,
            method,
            response_time: responseTime,
            status,
            timestamp: Date.now()
        });
    }
    
    getHealthMetrics() {
        const avgResponseTime = this.metrics.response_times.reduce((a, b) => a + b, 0) / this.metrics.response_times.length;
        const errorRate = (this.metrics.errors / this.metrics.requests) * 100;
        
        return {
            total_requests: this.metrics.requests,
            error_rate: errorRate,
            avg_response_time: avgResponseTime,
            health_status: errorRate < 5 ? 'healthy' : 'degraded'
        };
    }
}
```

## Rate Limiting & Throttling

### Request Throttling
```javascript
// Rate limiting implementation
class RateLimiter {
    constructor(maxRequests = 100, windowMs = 60000) {
        this.maxRequests = maxRequests;
        this.windowMs = windowMs;
        this.requests = new Map();
    }
    
    checkLimit(clientId) {
        const now = Date.now();
        const windowStart = now - this.windowMs;
        
        // Clean old requests
        if (this.requests.has(clientId)) {
            this.requests.set(
                clientId,
                this.requests.get(clientId).filter(time => time > windowStart)
            );
        } else {
            this.requests.set(clientId, []);
        }
        
        const requestCount = this.requests.get(clientId).length;
        
        if (requestCount >= this.maxRequests) {
            return {
                allowed: false,
                retryAfter: this.windowMs - (now - Math.min(...this.requests.get(clientId)))
            };
        }
        
        // Add current request
        this.requests.get(clientId).push(now);
        
        return {
            allowed: true,
            remaining: this.maxRequests - requestCount - 1
        };
    }
}
```

## Testing & Validation

### API Testing Framework
```javascript
// Comprehensive API testing
class APITester {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
        this.testResults = [];
    }
    
    async testEndpoint(endpoint, method, payload, expectedStatus = 200) {
        const startTime = Date.now();
        
        try {
            const response = await fetch(`${this.baseUrl}${endpoint}`, {
                method,
                headers: apiConfig.headers,
                body: payload ? JSON.stringify(payload) : undefined
            });
            
            const responseTime = Date.now() - startTime;
            const responseData = await response.json();
            
            const testResult = {
                endpoint,
                method,
                status: response.status,
                expected_status: expectedStatus,
                response_time: responseTime,
                success: response.status === expectedStatus,
                response_data: responseData,
                timestamp: new Date().toISOString()
            };
            
            this.testResults.push(testResult);
            return testResult;
            
        } catch (error) {
            const testResult = {
                endpoint,
                method,
                status: 'ERROR',
                expected_status: expectedStatus,
                success: false,
                error: error.message,
                timestamp: new Date().toISOString()
            };
            
            this.testResults.push(testResult);
            return testResult;
        }
    }
    
    generateTestReport() {
        const totalTests = this.testResults.length;
        const passedTests = this.testResults.filter(test => test.success).length;
        const avgResponseTime = this.testResults
            .filter(test => test.response_time)
            .reduce((sum, test) => sum + test.response_time, 0) / totalTests;
        
        return {
            total_tests: totalTests,
            passed: passedTests,
            failed: totalTests - passedTests,
            success_rate: (passedTests / totalTests) * 100,
            avg_response_time: avgResponseTime,
            test_results: this.testResults
        };
    }
}
```

## Related Documentation

- **[Technical Overview](./overview.md)** - Master technical architecture
- **[Cookie Architecture](./cookie-architecture.md)** - Cookie management system
- **[Event Tracking](./event-tracking.md)** - Analytics and tracking implementation
- **[Cross-Domain Sync](./cross-domain-sync.md)** - Multi-domain communication details
- **[Troubleshooting](./troubleshooting.md)** - API issues and solutions

---

*Last Updated: January 27, 2025*  
*Next: [Cross-Domain Synchronization](./cross-domain-sync.md)* 