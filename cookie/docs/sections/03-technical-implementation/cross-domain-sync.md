# Cross-Domain Synchronization Implementation

## Purpose
This document provides comprehensive technical specifications for implementing secure cross-domain visitor data synchronization across the UAGC domain ecosystem, enabling seamless user experiences while maintaining data integrity and security.

## UAGC Domain Architecture

### Domain Ecosystem Overview
The UAGC Cookie Personalization system operates across multiple domains with distinct purposes:

```
┌─────────────────────────────────────────────────────────────────┐
│                    UAGC Domain Ecosystem                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  │   uagc.edu      │    │ cloud.mail.     │    │ portal.uagc.edu │
│  │                 │    │ uagc.edu        │    │                 │
│  │ Marketing Site  │◄──►│ Stealth Apps    │◄──►│ Application     │
│  │ Lead Generation │    │ Seamless Apply  │    │ Portal          │
│  │ Program Info    │    │ Reduced Friction│    │ Student Portal  │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘
│           │                       │                       │      │
│           └───────────────────────┼───────────────────────┘      │
│                                   │                              │
│                    ┌─────────────────────────┐                   │
│                    │   Unified Visitor       │                   │
│                    │   State Management      │                   │
│                    │   (Cross-Domain Sync)   │                   │
│                    └─────────────────────────┘                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Domain Responsibilities

#### uagc.edu (Marketing Hub)
- **Primary Purpose:** Lead generation and program marketing
- **Key Functions:**
  - Initial visitor tracking and journey stage detection
  - UTM parameter capture and attribution
  - RFI form submissions and lead qualification
  - Program exploration and content personalization
- **Sync Requirements:** Must share visitor profile and journey stage with other domains

#### cloud.mail.uagc.edu (Stealth Application)
- **Primary Purpose:** Seamless application initiation with reduced friction
- **Key Functions:**
  - Pre-populated application forms using visitor data
  - Streamlined application process for qualified leads
  - Behavioral trigger-based application invitations
  - Application progress tracking and abandonment recovery
- **Sync Requirements:** Receives visitor data from marketing, shares application status

#### portal.uagc.edu (Application Portal)
- **Primary Purpose:** Full application management and student portal
- **Key Functions:**
  - Complete application submission and document management
  - Application status tracking and decision communication
  - Student portal access for enrolled students
  - Academic records and course management
- **Sync Requirements:** Receives application data, provides enrollment status

## Technical Implementation

### PostMessage Security Framework
All cross-domain communication uses the PostMessage API with comprehensive security measures:

```javascript
// Security configuration for cross-domain messaging
const CROSS_DOMAIN_CONFIG = {
    // Allowed origins for message exchange
    allowedOrigins: [
        'https://uagc.edu',
        'https://cloud.mail.uagc.edu', 
        'https://portal.uagc.edu'
    ],
    
    // Message types and their security levels
    messageTypes: {
        VISITOR_SYNC: { encryption: false, authentication: true },
        JOURNEY_UPDATE: { encryption: false, authentication: true },
        LEAD_DATA: { encryption: true, authentication: true },
        APPLICATION_DATA: { encryption: true, authentication: true },
        CONSENT_UPDATE: { encryption: false, authentication: true }
    },
    
    // Security settings
    security: {
        messageTimeout: 30000, // 30 seconds
        maxRetries: 3,
        requireChecksum: true,
        requireTimestamp: true,
        timestampTolerance: 300000 // 5 minutes
    }
};
```

### Message Format Specification
```javascript
// Standard cross-domain message structure
const CrossDomainMessage = {
    // Message identification
    id: 'string', // Unique message ID
    type: 'VISITOR_SYNC|JOURNEY_UPDATE|LEAD_DATA|APPLICATION_DATA|CONSENT_UPDATE',
    version: '2.0',
    
    // Security headers
    security: {
        timestamp: 'number', // Unix timestamp
        checksum: 'string',  // SHA-256 hash for integrity
        signature: 'string', // HMAC signature for authentication
        nonce: 'string'      // Unique nonce to prevent replay attacks
    },
    
    // Source information
    source: {
        domain: 'string',    // Sending domain
        session_id: 'string', // Session identifier
        visitor_id: 'string' // Persistent visitor identifier
    },
    
    // Message payload
    payload: {
        // Type-specific data structure
    },
    
    // Response handling
    response: {
        required: 'boolean',
        callback_id: 'string'
    }
};
```

## Visitor Data Synchronization

### Visitor Profile Structure
```javascript
// Comprehensive visitor profile for cross-domain sync
const VisitorProfile = {
    // Identity
    visitor_id: 'string',        // Persistent across domains
    session_id: 'string',        // Current session
    created_at: 'timestamp',     // First visit timestamp
    last_updated: 'timestamp',   // Last profile update
    
    // Journey tracking
    journey: {
        current_stage: 'awareness|consideration|engagement|application',
        stage_history: [
            {
                stage: 'string',
                entered_at: 'timestamp',
                duration: 'number',
                trigger_event: 'string'
            }
        ],
        progression_score: 'number', // 0-100 likelihood to convert
        next_recommended_action: 'string'
    },
    
    // Behavioral data
    behavior: {
        page_views: 'number',
        session_count: 'number',
        total_time_on_site: 'number',
        bounce_rate: 'number',
        engagement_score: 'number',
        last_active: 'timestamp',
        interaction_history: [
            {
                type: 'page_view|form_interaction|download|video_play',
                details: {},
                timestamp: 'timestamp'
            }
        ]
    },
    
    // Program interests
    programs: {
        viewed_programs: ['string'],
        saved_programs: ['string'],
        comparison_history: ['string'],
        primary_interest: 'string',
        secondary_interests: ['string'],
        interest_strength: 'number' // 0-100
    },
    
    // Attribution data
    attribution: {
        first_touch: {
            utm_source: 'string',
            utm_medium: 'string',
            utm_campaign: 'string',
            utm_content: 'string',
            utm_term: 'string',
            timestamp: 'timestamp',
            referrer: 'string'
        },
        last_touch: {
            // Same structure as first_touch
        },
        touchpoint_history: [
            {
                source: 'string',
                medium: 'string',
                campaign: 'string',
                timestamp: 'timestamp',
                attribution_weight: 'number'
            }
        ]
    },
    
    // Personalization preferences
    personalization: {
        visitor_type: 'new|returning|military|rfi|applicant|stealth',
        content_preferences: {},
        communication_preferences: {
            email_consent: 'boolean',
            sms_consent: 'boolean',
            call_consent: 'boolean',
            preferred_contact_time: 'string'
        },
        ab_test_assignments: {
            'test_id': 'variant_name'
        }
    },
    
    // Lead information (if applicable)
    lead_data: {
        lead_id: 'string',
        salesforce_id: 'string',
        lead_score: 'number',
        qualification_status: 'unqualified|marketing_qualified|sales_qualified',
        contact_info: {
            // Only synced if consent provided
            first_name: 'string',
            last_name: 'string',
            email: 'string',
            phone: 'string'
        },
        academic_info: {
            education_level: 'string',
            military_status: 'string',
            program_interest: 'string',
            start_date_preference: 'string'
        }
    }
};
```

### Synchronization Implementation
```javascript
// Cross-domain visitor synchronization manager
class CrossDomainSyncManager {
    constructor() {
        this.config = CROSS_DOMAIN_CONFIG;
        this.messageQueue = new Map();
        this.callbacks = new Map();
        this.syncKey = this.generateSyncKey();
        
        this.setupMessageListener();
        this.setupHeartbeat();
    }
    
    setupMessageListener() {
        window.addEventListener('message', (event) => {
            this.handleIncomingMessage(event);
        });
    }
    
    async syncVisitorProfile(targetDomain, visitorProfile) {
        try {
            // Validate target domain
            if (!this.config.allowedOrigins.includes(targetDomain)) {
                throw new Error(`Unauthorized target domain: ${targetDomain}`);
            }
            
            // Prepare message
            const message = this.createMessage('VISITOR_SYNC', {
                visitor_profile: visitorProfile,
                sync_timestamp: Date.now(),
                source_domain: window.location.origin
            });
            
            // Send message
            const response = await this.sendMessage(targetDomain, message);
            
            if (response.status === 'success') {
                console.log(`Visitor profile synced successfully to ${targetDomain}`);
                this.logSyncEvent('visitor_sync', targetDomain, 'success');
            } else {
                throw new Error(`Sync failed: ${response.error}`);
            }
            
            return response;
            
        } catch (error) {
            console.error('Cross-domain sync failed:', error);
            this.logSyncEvent('visitor_sync', targetDomain, 'error', error.message);
            throw error;
        }
    }
    
    createMessage(type, payload) {
        const timestamp = Date.now();
        const nonce = this.generateNonce();
        
        const message = {
            id: this.generateMessageId(),
            type,
            version: '2.0',
            security: {
                timestamp,
                nonce,
                checksum: '',
                signature: ''
            },
            source: {
                domain: window.location.origin,
                session_id: this.getSessionId(),
                visitor_id: this.getVisitorId()
            },
            payload,
            response: {
                required: true,
                callback_id: this.generateCallbackId()
            }
        };
        
        // Calculate checksum
        message.security.checksum = this.calculateChecksum(message);
        
        // Generate signature
        message.security.signature = this.generateSignature(message);
        
        return message;
    }
    
    async sendMessage(targetDomain, message) {
        return new Promise((resolve, reject) => {
            // Store callback for response handling
            this.callbacks.set(message.response.callback_id, {
                resolve,
                reject,
                timestamp: Date.now(),
                timeout: setTimeout(() => {
                    this.callbacks.delete(message.response.callback_id);
                    reject(new Error('Message timeout'));
                }, this.config.security.messageTimeout)
            });
            
            // Send via iframe for cross-domain communication
            this.sendViaIframe(targetDomain, message);
        });
    }
    
    sendViaIframe(targetDomain, message) {
        // Create hidden iframe for secure cross-domain messaging
        const iframe = document.createElement('iframe');
        iframe.style.display = 'none';
        iframe.src = `${targetDomain}/sync-receiver`;
        
        iframe.onload = () => {
            // Send message to iframe
            iframe.contentWindow.postMessage(message, targetDomain);
            
            // Clean up iframe after delay
            setTimeout(() => {
                if (iframe.parentNode) {
                    iframe.parentNode.removeChild(iframe);
                }
            }, 10000);
        };
        
        iframe.onerror = () => {
            console.error(`Failed to load sync iframe for ${targetDomain}`);
        };
        
        document.body.appendChild(iframe);
    }
    
    handleIncomingMessage(event) {
        try {
            // Validate origin
            if (!this.config.allowedOrigins.includes(event.origin)) {
                console.warn(`Rejected message from unauthorized origin: ${event.origin}`);
                return;
            }
            
            const message = event.data;
            
            // Validate message format
            if (!this.validateMessage(message)) {
                console.warn('Invalid message format received');
                return;
            }
            
            // Verify security
            if (!this.verifyMessageSecurity(message)) {
                console.warn('Message security verification failed');
                return;
            }
            
            // Handle message based on type
            this.processMessage(message, event.origin);
            
        } catch (error) {
            console.error('Error handling incoming message:', error);
        }
    }
    
    processMessage(message, origin) {
        switch (message.type) {
            case 'VISITOR_SYNC':
                this.handleVisitorSync(message, origin);
                break;
            case 'JOURNEY_UPDATE':
                this.handleJourneyUpdate(message, origin);
                break;
            case 'LEAD_DATA':
                this.handleLeadData(message, origin);
                break;
            case 'APPLICATION_DATA':
                this.handleApplicationData(message, origin);
                break;
            case 'CONSENT_UPDATE':
                this.handleConsentUpdate(message, origin);
                break;
            case 'RESPONSE':
                this.handleResponse(message);
                break;
            default:
                console.warn(`Unknown message type: ${message.type}`);
        }
    }
    
    handleVisitorSync(message, origin) {
        try {
            const visitorProfile = message.payload.visitor_profile;
            
            // Update local visitor profile
            this.updateLocalVisitorProfile(visitorProfile);
            
            // Trigger personalization updates
            if (window.UAGCPersonalizationEngine) {
                window.UAGCPersonalizationEngine.updateProfile(visitorProfile);
            }
            
            // Send success response
            this.sendResponse(message, origin, { status: 'success' });
            
            // Log sync event
            this.logSyncEvent('visitor_sync_received', origin, 'success');
            
        } catch (error) {
            console.error('Error handling visitor sync:', error);
            this.sendResponse(message, origin, { 
                status: 'error', 
                error: error.message 
            });
        }
    }
    
    handleJourneyUpdate(message, origin) {
        try {
            const { new_stage, trigger_event } = message.payload;
            
            // Update journey stage
            this.updateJourneyStage(new_stage, trigger_event);
            
            // Trigger analytics events
            if (window.gtag) {
                gtag('event', 'journey_stage_update', {
                    previous_stage: this.getCurrentJourneyStage(),
                    new_stage,
                    trigger_event,
                    source_domain: origin
                });
            }
            
            // Send success response
            this.sendResponse(message, origin, { status: 'success' });
            
        } catch (error) {
            console.error('Error handling journey update:', error);
            this.sendResponse(message, origin, { 
                status: 'error', 
                error: error.message 
            });
        }
    }
    
    sendResponse(originalMessage, targetOrigin, responseData) {
        const response = {
            id: this.generateMessageId(),
            type: 'RESPONSE',
            version: '2.0',
            security: {
                timestamp: Date.now(),
                nonce: this.generateNonce()
            },
            source: {
                domain: window.location.origin,
                session_id: this.getSessionId(),
                visitor_id: this.getVisitorId()
            },
            payload: {
                original_message_id: originalMessage.id,
                callback_id: originalMessage.response.callback_id,
                ...responseData
            },
            response: {
                required: false
            }
        };
        
        // Send response
        window.parent.postMessage(response, targetOrigin);
    }
    
    // Security helper methods
    validateMessage(message) {
        return message && 
               message.id && 
               message.type && 
               message.version === '2.0' &&
               message.security &&
               message.source &&
               message.payload;
    }
    
    verifyMessageSecurity(message) {
        // Verify timestamp (prevent replay attacks)
        const now = Date.now();
        const messageTime = message.security.timestamp;
        const timeDiff = Math.abs(now - messageTime);
        
        if (timeDiff > this.config.security.timestampTolerance) {
            console.warn('Message timestamp outside tolerance');
            return false;
        }
        
        // Verify checksum
        const expectedChecksum = this.calculateChecksum(message);
        if (message.security.checksum !== expectedChecksum) {
            console.warn('Message checksum verification failed');
            return false;
        }
        
        // Verify signature
        if (!this.verifySignature(message)) {
            console.warn('Message signature verification failed');
            return false;
        }
        
        return true;
    }
    
    calculateChecksum(message) {
        // Create a copy without the checksum field
        const messageForChecksum = { ...message };
        messageForChecksum.security = { ...message.security };
        delete messageForChecksum.security.checksum;
        delete messageForChecksum.security.signature;
        
        // Calculate SHA-256 hash
        const messageString = JSON.stringify(messageForChecksum);
        return this.sha256(messageString);
    }
    
    generateSignature(message) {
        // HMAC-SHA256 signature using secret key
        const messageString = JSON.stringify(message.payload);
        return this.hmacSha256(messageString, this.syncKey);
    }
    
    verifySignature(message) {
        const expectedSignature = this.generateSignature(message);
        return message.security.signature === expectedSignature;
    }
}
```

## Real-Time Synchronization Scenarios

### Scenario 1: Marketing to Stealth Application
When a qualified lead on uagc.edu triggers the stealth application invitation:

```javascript
// Marketing site detects stealth application trigger
async function triggerStealthApplication(visitorProfile) {
    const syncManager = new CrossDomainSyncManager();
    
    // Update journey stage to engagement
    visitorProfile.journey.current_stage = 'engagement';
    visitorProfile.journey.next_recommended_action = 'start_application';
    
    // Sync visitor data to stealth application domain
    await syncManager.syncVisitorProfile(
        'https://cloud.mail.uagc.edu',
        visitorProfile
    );
    
    // Track the transition
    gtag('event', 'stealth_app_trigger', {
        visitor_id: visitorProfile.visitor_id,
        lead_score: visitorProfile.lead_data.lead_score,
        program_interest: visitorProfile.programs.primary_interest
    });
    
    // Redirect to stealth application with pre-populated form
    window.location.href = `https://cloud.mail.uagc.edu/apply?visitor_id=${visitorProfile.visitor_id}`;
}
```

### Scenario 2: Stealth to Full Application Portal
When a user completes the stealth application and needs full application access:

```javascript
// Stealth application completion handler
async function completeStealthApplication(applicationData) {
    const syncManager = new CrossDomainSyncManager();
    
    // Update visitor profile with application data
    const updatedProfile = {
        ...getCurrentVisitorProfile(),
        journey: {
            current_stage: 'application',
            application_started: true,
            application_completion_percentage: 75
        },
        lead_data: {
            ...getCurrentVisitorProfile().lead_data,
            qualification_status: 'sales_qualified'
        }
    };
    
    // Sync to full application portal
    await syncManager.syncVisitorProfile(
        'https://portal.uagc.edu',
        updatedProfile
    );
    
    // Sync application data
    await syncManager.sendMessage(
        'https://portal.uagc.edu',
        syncManager.createMessage('APPLICATION_DATA', {
            application_data: applicationData,
            transfer_type: 'stealth_to_full',
            completion_status: 'partial'
        })
    );
    
    // Redirect to full application portal
    window.location.href = `https://portal.uagc.edu/complete-application?transfer=stealth&id=${applicationData.application_id}`;
}
```

### Scenario 3: Consent Preference Synchronization
When a user updates their consent preferences on any domain:

```javascript
// Universal consent update handler
async function updateConsentAcrossDomains(consentPreferences) {
    const syncManager = new CrossDomainSyncManager();
    const allDomains = [
        'https://uagc.edu',
        'https://cloud.mail.uagc.edu',
        'https://portal.uagc.edu'
    ];
    
    // Remove current domain from sync targets
    const targetDomains = allDomains.filter(domain => 
        domain !== window.location.origin
    );
    
    // Sync consent to all other domains
    const syncPromises = targetDomains.map(domain =>
        syncManager.sendMessage(domain, 
            syncManager.createMessage('CONSENT_UPDATE', {
                consent_preferences: consentPreferences,
                updated_at: Date.now(),
                legal_basis: 'user_consent'
            })
        )
    );
    
    try {
        await Promise.all(syncPromises);
        console.log('Consent preferences synced across all domains');
        
        // Update local cookie consent
        updateLocalConsentPreferences(consentPreferences);
        
    } catch (error) {
        console.error('Failed to sync consent across domains:', error);
        // Handle partial sync failure
    }
}
```

## Performance Optimization

### Efficient Data Transfer
```javascript
// Optimized data synchronization with compression
class OptimizedSyncManager extends CrossDomainSyncManager {
    constructor() {
        super();
        this.compressionEnabled = true;
        this.batchingEnabled = true;
        this.batchQueue = [];
        this.batchTimeout = null;
    }
    
    async syncVisitorProfile(targetDomain, visitorProfile) {
        if (this.batchingEnabled) {
            return this.addToBatch(targetDomain, 'VISITOR_SYNC', { visitor_profile: visitorProfile });
        }
        
        // Compress large payloads
        if (this.compressionEnabled && JSON.stringify(visitorProfile).length > 1024) {
            visitorProfile = await this.compressData(visitorProfile);
        }
        
        return super.syncVisitorProfile(targetDomain, visitorProfile);
    }
    
    addToBatch(targetDomain, messageType, payload) {
        this.batchQueue.push({ targetDomain, messageType, payload });
        
        // Process batch after delay or when batch size limit reached
        if (this.batchQueue.length >= 5) {
            this.processBatch();
        } else if (!this.batchTimeout) {
            this.batchTimeout = setTimeout(() => this.processBatch(), 1000);
        }
    }
    
    async processBatch() {
        if (this.batchQueue.length === 0) return;
        
        clearTimeout(this.batchTimeout);
        this.batchTimeout = null;
        
        const batch = [...this.batchQueue];
        this.batchQueue = [];
        
        // Group by target domain
        const domainGroups = batch.reduce((groups, item) => {
            if (!groups[item.targetDomain]) {
                groups[item.targetDomain] = [];
            }
            groups[item.targetDomain].push(item);
            return groups;
        }, {});
        
        // Send batched messages
        const batchPromises = Object.entries(domainGroups).map(([domain, items]) =>
            this.sendBatchMessage(domain, items)
        );
        
        await Promise.all(batchPromises);
    }
    
    async sendBatchMessage(targetDomain, items) {
        const batchMessage = this.createMessage('BATCH_SYNC', {
            items: items.map(item => ({
                type: item.messageType,
                payload: item.payload
            })),
            batch_id: this.generateBatchId(),
            item_count: items.length
        });
        
        return this.sendMessage(targetDomain, batchMessage);
    }
    
    async compressData(data) {
        // Simple compression for demonstration
        // In production, use actual compression library
        const jsonString = JSON.stringify(data);
        const compressed = btoa(jsonString);
        
        return {
            compressed: true,
            data: compressed,
            original_size: jsonString.length,
            compressed_size: compressed.length
        };
    }
}
```

### Caching and Storage Optimization
```javascript
// Cross-domain sync with intelligent caching
class CachedSyncManager extends OptimizedSyncManager {
    constructor() {
        super();
        this.syncCache = new Map();
        this.cacheTimeout = 300000; // 5 minutes
    }
    
    async syncVisitorProfile(targetDomain, visitorProfile) {
        // Check if we've recently synced this data
        const cacheKey = `${targetDomain}:${visitorProfile.visitor_id}`;
        const cached = this.syncCache.get(cacheKey);
        
        if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
            // Compare data to see if sync is needed
            if (this.isDataEquivalent(cached.data, visitorProfile)) {
                console.log(`Skipping sync to ${targetDomain} - data unchanged`);
                return { status: 'cached', message: 'No changes detected' };
            }
        }
        
        // Perform sync
        const result = await super.syncVisitorProfile(targetDomain, visitorProfile);
        
        // Cache the synced data
        this.syncCache.set(cacheKey, {
            data: visitorProfile,
            timestamp: Date.now(),
            result
        });
        
        return result;
    }
    
    isDataEquivalent(data1, data2) {
        // Deep comparison of relevant fields
        const relevantFields = [
            'journey.current_stage',
            'behavior.engagement_score',
            'programs.primary_interest',
            'lead_data.lead_score',
            'personalization.visitor_type'
        ];
        
        return relevantFields.every(field => {
            const value1 = this.getNestedValue(data1, field);
            const value2 = this.getNestedValue(data2, field);
            return value1 === value2;
        });
    }
    
    getNestedValue(obj, path) {
        return path.split('.').reduce((current, key) => 
            current && current[key] !== undefined ? current[key] : undefined, obj
        );
    }
}
```

## Error Handling and Recovery

### Robust Error Handling
```javascript
// Comprehensive error handling for cross-domain sync
class RobustSyncManager extends CachedSyncManager {
    constructor() {
        super();
        this.retryQueue = new Map();
        this.maxRetries = 3;
        this.retryDelay = 1000; // Start with 1 second
    }
    
    async syncWithRetry(targetDomain, visitorProfile, attempt = 1) {
        try {
            return await this.syncVisitorProfile(targetDomain, visitorProfile);
            
        } catch (error) {
            console.warn(`Sync attempt ${attempt} failed:`, error.message);
            
            if (attempt < this.maxRetries) {
                // Exponential backoff
                const delay = this.retryDelay * Math.pow(2, attempt - 1);
                
                console.log(`Retrying sync to ${targetDomain} in ${delay}ms...`);
                
                return new Promise((resolve, reject) => {
                    setTimeout(async () => {
                        try {
                            const result = await this.syncWithRetry(targetDomain, visitorProfile, attempt + 1);
                            resolve(result);
                        } catch (retryError) {
                            reject(retryError);
                        }
                    }, delay);
                });
            } else {
                // Max retries exceeded - queue for later
                this.queueForRetry(targetDomain, visitorProfile);
                throw new Error(`Sync failed after ${this.maxRetries} attempts: ${error.message}`);
            }
        }
    }
    
    queueForRetry(targetDomain, visitorProfile) {
        const retryKey = `${targetDomain}:${visitorProfile.visitor_id}`;
        
        this.retryQueue.set(retryKey, {
            targetDomain,
            visitorProfile,
            queued_at: Date.now(),
            retry_count: 0
        });
        
        // Process retry queue periodically
        if (this.retryQueue.size === 1) {
            this.startRetryProcessor();
        }
    }
    
    startRetryProcessor() {
        setInterval(() => {
            this.processRetryQueue();
        }, 60000); // Process every minute
    }
    
    async processRetryQueue() {
        if (this.retryQueue.size === 0) return;
        
        const retryPromises = Array.from(this.retryQueue.entries()).map(async ([key, item]) => {
            try {
                await this.syncVisitorProfile(item.targetDomain, item.visitorProfile);
                this.retryQueue.delete(key);
                console.log(`Retry sync successful for ${key}`);
                
            } catch (error) {
                item.retry_count++;
                
                if (item.retry_count >= 5) {
                    // Give up after 5 retry attempts
                    this.retryQueue.delete(key);
                    console.error(`Giving up on sync for ${key} after 5 retry attempts`);
                } else {
                    console.log(`Retry ${item.retry_count} failed for ${key}, will try again later`);
                }
            }
        });
        
        await Promise.allSettled(retryPromises);
    }
}
```

## Monitoring and Analytics

### Sync Performance Monitoring
```javascript
// Cross-domain sync monitoring and analytics
class MonitoredSyncManager extends RobustSyncManager {
    constructor() {
        super();
        this.syncMetrics = {
            total_syncs: 0,
            successful_syncs: 0,
            failed_syncs: 0,
            avg_sync_time: 0,
            sync_times: [],
            domain_performance: new Map()
        };
    }
    
    async syncVisitorProfile(targetDomain, visitorProfile) {
        const startTime = Date.now();
        this.syncMetrics.total_syncs++;
        
        try {
            const result = await super.syncVisitorProfile(targetDomain, visitorProfile);
            
            // Record successful sync
            this.recordSyncMetrics(targetDomain, startTime, true);
            
            return result;
            
        } catch (error) {
            // Record failed sync
            this.recordSyncMetrics(targetDomain, startTime, false, error);
            throw error;
        }
    }
    
    recordSyncMetrics(targetDomain, startTime, success, error = null) {
        const syncTime = Date.now() - startTime;
        
        // Update global metrics
        if (success) {
            this.syncMetrics.successful_syncs++;
        } else {
            this.syncMetrics.failed_syncs++;
        }
        
        this.syncMetrics.sync_times.push(syncTime);
        
        // Keep only last 100 sync times for average calculation
        if (this.syncMetrics.sync_times.length > 100) {
            this.syncMetrics.sync_times.shift();
        }
        
        this.syncMetrics.avg_sync_time = 
            this.syncMetrics.sync_times.reduce((a, b) => a + b, 0) / 
            this.syncMetrics.sync_times.length;
        
        // Update domain-specific metrics
        if (!this.syncMetrics.domain_performance.has(targetDomain)) {
            this.syncMetrics.domain_performance.set(targetDomain, {
                total: 0,
                successful: 0,
                failed: 0,
                avg_time: 0,
                last_sync: null
            });
        }
        
        const domainMetrics = this.syncMetrics.domain_performance.get(targetDomain);
        domainMetrics.total++;
        domainMetrics.last_sync = Date.now();
        
        if (success) {
            domainMetrics.successful++;
        } else {
            domainMetrics.failed++;
        }
        
        // Send metrics to analytics
        this.sendSyncAnalytics(targetDomain, syncTime, success, error);
    }
    
    sendSyncAnalytics(targetDomain, syncTime, success, error) {
        if (window.gtag) {
            gtag('event', 'cross_domain_sync', {
                target_domain: targetDomain,
                sync_time: syncTime,
                success: success,
                error_message: error ? error.message : null,
                custom_parameter_1: 'cross_domain_performance'
            });
        }
        
        // Send to custom analytics if available
        if (window.customAnalytics) {
            window.customAnalytics.track('cross_domain_sync', {
                target_domain: targetDomain,
                sync_time: syncTime,
                success: success,
                error: error ? error.message : null
            });
        }
    }
    
    getSyncHealthReport() {
        const successRate = (this.syncMetrics.successful_syncs / this.syncMetrics.total_syncs) * 100;
        
        return {
            total_syncs: this.syncMetrics.total_syncs,
            success_rate: successRate,
            avg_sync_time: this.syncMetrics.avg_sync_time,
            failed_syncs: this.syncMetrics.failed_syncs,
            retry_queue_size: this.retryQueue.size,
            domain_performance: Object.fromEntries(this.syncMetrics.domain_performance),
            health_status: successRate > 95 ? 'healthy' : successRate > 85 ? 'warning' : 'critical'
        };
    }
}
```

## Implementation Checklist

### Phase 1: Foundation
- [ ] Implement PostMessage security framework
- [ ] Create visitor profile data structure
- [ ] Set up basic cross-domain message handling
- [ ] Implement message validation and security checks

### Phase 2: Core Synchronization
- [ ] Implement visitor profile synchronization
- [ ] Create journey stage update handling
- [ ] Set up lead data synchronization
- [ ] Implement consent preference synchronization

### Phase 3: Optimization & Reliability
- [ ] Add message batching and compression
- [ ] Implement intelligent caching
- [ ] Create retry mechanism with exponential backoff
- [ ] Add comprehensive error handling

### Phase 4: Monitoring & Analytics
- [ ] Implement sync performance monitoring
- [ ] Create health reporting dashboard
- [ ] Set up sync analytics tracking
- [ ] Add automated alerting for sync failures

## Related Documentation

- **[Technical Overview](./overview.md)** - Master technical architecture
- **[Cookie Architecture](./cookie-architecture.md)** - Cookie management and consent
- **[API Reference](./api-reference.md)** - Complete API specifications
- **[Event Tracking](./event-tracking.md)** - Analytics and tracking implementation
- **[Troubleshooting](./troubleshooting.md)** - Cross-domain sync issues and solutions

---

*Last Updated: January 27, 2025*  
*Next: [Event Tracking](./event-tracking.md)* 