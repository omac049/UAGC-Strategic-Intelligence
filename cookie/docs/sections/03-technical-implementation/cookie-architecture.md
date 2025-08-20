# Cookie Architecture & Management System

## Purpose
This document provides the complete technical specification for UAGC's cookie architecture, implementing a GDPR/CCPA compliant system with granular consent management and cross-domain synchronization.

## Cookie Framework Overview

### Four-Category Compliance System
Based on privacy regulations and user experience requirements, UAGC implements a four-category cookie system:

```javascript
const COOKIE_CATEGORIES = {
    ESSENTIAL: {
        id: 'essential',
        consent_required: false,
        description: 'Security, authentication, and core functionality',
        expiration: 'session_or_1_year',
        cross_domain: true
    },
    ANALYTICS: {
        id: 'analytics', 
        consent_required: true,
        description: 'Journey tracking, engagement metrics, and optimization',
        expiration: '2_years',
        cross_domain: true
    },
    PERSONALIZATION: {
        id: 'personalization',
        consent_required: true, 
        description: 'Tailored content, program recommendations, and preferences',
        expiration: '90_days',
        cross_domain: true
    },
    MARKETING: {
        id: 'marketing',
        consent_required: true,
        description: 'Campaign tracking, conversion attribution, and retargeting',
        expiration: '30_days',
        cross_domain: 'limited'
    }
};
```

## Essential Cookies (No Consent Required)

### Core Security & Functionality
Essential cookies are automatically active and cannot be disabled as they're required for basic site functionality and security.

```javascript
// Essential cookie implementation
const essentialCookies = {
    // Session management
    'PHPSESSID': {
        purpose: 'Server session management',
        expiration: 'session',
        domain: '.uagc.edu',
        secure: true,
        httpOnly: true
    },
    
    // Security tokens
    'csrf_token': {
        purpose: 'CSRF protection',
        expiration: '24_hours',
        domain: '.uagc.edu', 
        secure: true,
        httpOnly: true
    },
    
    // Authentication state
    'auth_state': {
        purpose: 'User authentication status',
        expiration: '1_year',
        domain: '.uagc.edu',
        secure: true,
        httpOnly: false // Needed for client-side auth checks
    },
    
    // Cookie consent preferences
    'cookie_consent': {
        purpose: 'User consent preferences storage',
        expiration: '1_year',
        domain: '.uagc.edu',
        secure: true,
        httpOnly: false
    }
};
```

### Cross-Domain Essential Synchronization
```javascript
// Essential data sync across UAGC domains
function syncEssentialData(targetDomain) {
    const essentialData = {
        auth_state: getCookie('auth_state'),
        consent_preferences: getCookie('cookie_consent'),
        session_id: generateSecureSessionId()
    };
    
    // Secure postMessage to target domain
    window.postMessage({
        type: 'ESSENTIAL_SYNC',
        data: essentialData,
        timestamp: Date.now()
    }, targetDomain);
}
```

## Analytics & Performance Cookies

### Journey Tracking Implementation
Analytics cookies enable comprehensive user journey tracking and performance optimization.

```javascript
// Analytics cookie configuration
const analyticsCookies = {
    // Google Analytics 4
    '_ga': {
        purpose: 'Google Analytics client ID',
        expiration: '2_years',
        provider: 'Google',
        data_processor: 'Google LLC'
    },
    
    '_ga_MEASUREMENT_ID': {
        purpose: 'GA4 session and campaign data',
        expiration: '2_years', 
        provider: 'Google',
        data_processor: 'Google LLC'
    },
    
    // UAGC journey tracking
    'uagc_journey_stage': {
        purpose: 'Current user journey stage tracking',
        expiration: '90_days',
        values: ['awareness', 'consideration', 'engagement', 'application'],
        domain: '.uagc.edu'
    },
    
    'uagc_visit_history': {
        purpose: 'Page visit history and engagement patterns',
        expiration: '30_days',
        max_entries: 50,
        domain: '.uagc.edu'
    },
    
    // Performance monitoring
    'performance_metrics': {
        purpose: 'Core Web Vitals and user experience data',
        expiration: '7_days',
        data_points: ['FCP', 'LCP', 'CLS', 'FID'],
        domain: '.uagc.edu'
    }
};
```

### GA4 Enhanced Conversions Setup
```javascript
// GA4 configuration with enhanced conversions
gtag('config', 'GA_MEASUREMENT_ID', {
    // Enhanced conversions for leads
    enhanced_conversions: true,
    
    // Custom parameters for UAGC tracking
    custom_map: {
        'custom_parameter_1': 'journey_stage',
        'custom_parameter_2': 'program_interest',
        'custom_parameter_3': 'lead_score'
    },
    
    // Cross-domain tracking
    linker: {
        domains: ['uagc.edu', 'cloud.mail.uagc.edu', 'portal.uagc.edu']
    }
});
```

## Personalization & Functionality Cookies

### Content Customization System
Personalization cookies enable tailored user experiences and content recommendations.

```javascript
// Personalization cookie implementation
const personalizationCookies = {
    // User preferences
    'user_preferences': {
        purpose: 'Site preferences and customization settings',
        expiration: '1_year',
        data_structure: {
            theme: 'light|dark|auto',
            language: 'en|es',
            accessibility_settings: {},
            communication_preferences: {}
        },
        domain: '.uagc.edu'
    },
    
    // Program interest tracking
    'program_interests': {
        purpose: 'Tracked program interests and exploration history',
        expiration: '90_days',
        max_programs: 10,
        data_structure: {
            viewed_programs: [],
            saved_programs: [],
            comparison_history: [],
            last_updated: 'timestamp'
        },
        domain: '.uagc.edu'
    },
    
    // Personalization profile
    'personalization_profile': {
        purpose: 'User behavior profile for content personalization',
        expiration: '90_days',
        data_structure: {
            visitor_type: 'new|returning|military|rfi|applicant|stealth',
            engagement_score: 'number',
            content_preferences: {},
            optimal_contact_time: 'string'
        },
        domain: '.uagc.edu'
    },
    
    // A/B testing assignments
    'ab_test_assignments': {
        purpose: 'A/B test variant assignments for consistent experience',
        expiration: '30_days',
        data_structure: {
            test_id: 'variant_assignment',
            // Multiple test assignments
        },
        domain: '.uagc.edu'
    }
};
```

### Dynamic Content Personalization
```javascript
// Personalization engine implementation
class UAGCPersonalizationEngine {
    constructor() {
        this.profile = this.loadPersonalizationProfile();
        this.testAssignments = this.loadABTestAssignments();
    }
    
    personalizeContent(pageContext) {
        const profile = this.profile;
        const personalizations = {};
        
        // CTA personalization based on journey stage
        if (profile.visitor_type === 'rfi') {
            personalizations.primary_cta = 'Apply Now';
            personalizations.secondary_cta = 'Schedule Advisor Call';
        } else if (profile.visitor_type === 'new') {
            personalizations.primary_cta = 'Request Information';
            personalizations.secondary_cta = 'Explore Programs';
        }
        
        // Content recommendations
        if (profile.program_interests.length > 0) {
            personalizations.recommended_content = 
                this.getRelatedContent(profile.program_interests);
        }
        
        return personalizations;
    }
    
    trackEngagement(interactionData) {
        // Update engagement score and preferences
        this.updatePersonalizationProfile(interactionData);
        this.saveToCookie();
    }
}
```

## Marketing & Attribution Cookies

### Campaign Tracking System
Marketing cookies enable comprehensive campaign attribution and retargeting capabilities.

```javascript
// Marketing cookie configuration
const marketingCookies = {
    // UTM campaign data
    'utm_campaign_data': {
        purpose: 'Campaign attribution and source tracking',
        expiration: '30_days',
        data_structure: {
            utm_source: 'string',
            utm_medium: 'string', 
            utm_campaign: 'string',
            utm_content: 'string',
            utm_term: 'string',
            first_touch: 'timestamp',
            last_touch: 'timestamp'
        },
        domain: '.uagc.edu'
    },
    
    // Lead scoring data
    'lead_score_data': {
        purpose: 'Algorithmic lead scoring for prioritization',
        expiration: '90_days',
        data_structure: {
            score: 'number',
            factors: {},
            last_calculated: 'timestamp',
            score_history: []
        },
        domain: '.uagc.edu'
    },
    
    // Third-party platform cookies
    '_fbp': {
        purpose: 'Facebook Pixel browser tracking',
        expiration: '90_days',
        provider: 'Facebook',
        data_processor: 'Meta Platforms Inc.'
    },
    
    '_fbc': {
        purpose: 'Facebook click tracking',
        expiration: '90_days',
        provider: 'Facebook', 
        data_processor: 'Meta Platforms Inc.'
    },
    
    // Google Ads attribution
    '_gcl_au': {
        purpose: 'Google Ads attribution',
        expiration: '90_days',
        provider: 'Google',
        data_processor: 'Google LLC'
    },
    
    // LinkedIn conversion tracking
    'li_sugr': {
        purpose: 'LinkedIn conversion tracking',
        expiration: '90_days',
        provider: 'LinkedIn',
        data_processor: 'LinkedIn Corporation'
    }
};
```

### Attribution Logic Implementation
```javascript
// Multi-touch attribution system
class UAGCAttributionEngine {
    constructor() {
        this.attributionModel = 'time_decay'; // time_decay, linear, first_touch, last_touch
        this.touchpointHistory = this.loadTouchpointHistory();
    }
    
    recordTouchpoint(source, medium, campaign, timestamp = Date.now()) {
        const touchpoint = {
            source,
            medium, 
            campaign,
            timestamp,
            page: window.location.pathname,
            referrer: document.referrer
        };
        
        this.touchpointHistory.push(touchpoint);
        this.saveTouchpointHistory();
        
        // Update attribution weights
        this.calculateAttribution();
    }
    
    calculateAttribution() {
        switch (this.attributionModel) {
            case 'time_decay':
                return this.calculateTimeDecayAttribution();
            case 'linear':
                return this.calculateLinearAttribution();
            case 'first_touch':
                return this.calculateFirstTouchAttribution();
            case 'last_touch':
                return this.calculateLastTouchAttribution();
        }
    }
    
    calculateTimeDecayAttribution() {
        const now = Date.now();
        const halfLife = 7 * 24 * 60 * 60 * 1000; // 7 days in milliseconds
        
        return this.touchpointHistory.map(touchpoint => {
            const age = now - touchpoint.timestamp;
            const weight = Math.pow(0.5, age / halfLife);
            return { ...touchpoint, weight };
        });
    }
}
```

## Cross-Domain Synchronization

### Multi-Domain Architecture
UAGC's cookie system operates across three primary domains with secure synchronization:

```javascript
// Cross-domain synchronization configuration
const UAGC_DOMAINS = {
    MARKETING: 'https://uagc.edu',
    STEALTH_APP: 'https://cloud.mail.uagc.edu', 
    APPLICATION: 'https://portal.uagc.edu'
};

// Cross-domain message handling
window.addEventListener('message', (event) => {
    // Validate origin
    if (!Object.values(UAGC_DOMAINS).includes(event.origin)) {
        return; // Reject unauthorized origins
    }
    
    // Process different message types
    switch (event.data.type) {
        case 'COOKIE_SYNC':
            handleCookieSync(event.data);
            break;
        case 'JOURNEY_UPDATE':
            handleJourneyUpdate(event.data);
            break;
        case 'CONSENT_UPDATE':
            handleConsentUpdate(event.data);
            break;
    }
});

// Sync visitor data across domains
function syncVisitorData(targetDomain, data) {
    const iframe = document.createElement('iframe');
    iframe.style.display = 'none';
    iframe.src = `${targetDomain}/sync-endpoint`;
    
    iframe.onload = () => {
        iframe.contentWindow.postMessage({
            type: 'VISITOR_SYNC',
            data: data,
            timestamp: Date.now()
        }, targetDomain);
    };
    
    document.body.appendChild(iframe);
    
    // Clean up after sync
    setTimeout(() => {
        document.body.removeChild(iframe);
    }, 5000);
}
```

### Secure Synchronization Protocol
```javascript
// Secure cross-domain data synchronization
class UAGCCrossDomainSync {
    constructor() {
        this.syncKey = this.generateSyncKey();
        this.pendingSyncs = new Map();
    }
    
    generateSyncKey() {
        // Generate secure key for data integrity
        return btoa(Math.random().toString(36).substring(2, 15) + 
                   Math.random().toString(36).substring(2, 15));
    }
    
    syncData(targetDomain, data, category) {
        const syncId = this.generateSyncId();
        const encryptedData = this.encryptSyncData(data);
        
        const message = {
            type: 'UAGC_SYNC',
            syncId,
            category,
            data: encryptedData,
            timestamp: Date.now(),
            checksum: this.calculateChecksum(encryptedData)
        };
        
        // Store pending sync for verification
        this.pendingSyncs.set(syncId, {
            targetDomain,
            originalData: data,
            timestamp: Date.now()
        });
        
        // Send sync message
        this.postMessageToTarget(targetDomain, message);
    }
    
    handleSyncResponse(response) {
        const pendingSync = this.pendingSyncs.get(response.syncId);
        
        if (pendingSync && response.status === 'success') {
            // Sync confirmed successful
            this.pendingSyncs.delete(response.syncId);
            this.logSyncSuccess(response.syncId);
        } else {
            // Handle sync failure
            this.handleSyncFailure(response.syncId, response.error);
        }
    }
}
```

## OneTrust Enterprise Integration

### Advanced Cookie Scanning & Management
UAGC integrates with OneTrust for enterprise-level cookie compliance and management.

```javascript
// OneTrust integration configuration
const oneTrustConfig = {
    // Data domain ID from OneTrust
    dataDomainScript: "your-data-domain-id",
    
    // Cookie categories mapping
    categoryMapping: {
        'C0001': 'essential',      // Strictly Necessary
        'C0002': 'analytics',      // Performance Cookies  
        'C0003': 'personalization', // Functional Cookies
        'C0004': 'marketing'       // Targeting Cookies
    },
    
    // Geolocation rules
    geolocationRules: {
        'EU': 'gdpr_strict',
        'CA': 'ccpa_compliance', 
        'US': 'ccpa_opt_out',
        'default': 'standard'
    }
};

// OneTrust event handlers
function OneTrustCallback() {
    // Initialize after OneTrust loads
    initializeUAGCCookieManager();
}

// Handle consent changes
window.addEventListener('OneTrustGroupsUpdated', (event) => {
    const activeGroups = OnetrustActiveGroups.split(',');
    
    // Update UAGC cookie permissions
    updateCookiePermissions(activeGroups);
    
    // Trigger appropriate scripts
    if (activeGroups.includes('C0002')) {
        // Analytics consent given
        initializeAnalytics();
    }
    
    if (activeGroups.includes('C0004')) {
        // Marketing consent given  
        initializeMarketingPixels();
    }
});
```

### Granular Consent Management
```javascript
// Advanced consent management
class UAGCConsentManager {
    constructor() {
        this.consentData = this.loadConsentData();
        this.preferences = this.loadUserPreferences();
    }
    
    updateConsent(category, granted, source = 'user') {
        const timestamp = Date.now();
        
        // Update consent record
        this.consentData[category] = {
            granted,
            timestamp,
            source, // 'user', 'default', 'implied'
            version: this.getCurrentPolicyVersion(),
            ip_address: this.getUserIP(),
            user_agent: navigator.userAgent
        };
        
        // Sync across domains
        this.syncConsentAcrossDomains(category, granted);
        
        // Update active scripts
        this.updateActiveScripts(category, granted);
        
        // Save to persistent storage
        this.saveConsentData();
        
        // Trigger consent change events
        this.triggerConsentChangeEvent(category, granted);
    }
    
    getConsentStatus(category) {
        const consent = this.consentData[category];
        
        if (!consent) {
            return this.getDefaultConsent(category);
        }
        
        // Check if consent has expired
        if (this.isConsentExpired(consent)) {
            return this.getDefaultConsent(category);
        }
        
        return consent.granted;
    }
    
    generateConsentRecord() {
        // Generate comprehensive consent record for auditing
        return {
            timestamp: Date.now(),
            categories: this.consentData,
            user_preferences: this.preferences,
            policy_version: this.getCurrentPolicyVersion(),
            browser_info: this.getBrowserInfo(),
            session_id: this.getSessionId()
        };
    }
}
```

## Cookie Security & Encryption

### Security Configuration
```javascript
// Secure cookie configuration
const securityConfig = {
    // Encryption settings
    encryption: {
        algorithm: 'AES-256-GCM',
        keyRotation: '30_days',
        saltLength: 16
    },
    
    // Cookie security flags
    cookieDefaults: {
        secure: true,        // HTTPS only
        sameSite: 'Lax',     // CSRF protection while allowing cross-site navigation
        httpOnly: false,     // Client-side access needed for personalization
        domain: '.uagc.edu', // Cross-subdomain sharing
        path: '/'            // Site-wide availability
    },
    
    // Content Security Policy
    csp: {
        'script-src': ["'self'", "'unsafe-inline'", "*.google-analytics.com", "*.googletagmanager.com"],
        'connect-src': ["'self'", "*.google-analytics.com", "*.salesforce.com"],
        'img-src': ["'self'", "data:", "*.google-analytics.com", "*.facebook.com"]
    }
};

// Encrypt sensitive cookie data
function encryptCookieData(data, category) {
    if (category === 'essential' || category === 'analytics') {
        // Encrypt PII and sensitive data
        return encrypt(JSON.stringify(data), getEncryptionKey());
    }
    
    // Non-sensitive data can be stored as-is
    return JSON.stringify(data);
}

// Decrypt cookie data
function decryptCookieData(encryptedData, category) {
    if (category === 'essential' || category === 'analytics') {
        const decrypted = decrypt(encryptedData, getEncryptionKey());
        return JSON.parse(decrypted);
    }
    
    return JSON.parse(encryptedData);
}
```

## Performance & Monitoring

### Cookie Performance Optimization
```javascript
// Performance monitoring for cookie operations
class CookiePerformanceMonitor {
    constructor() {
        this.metrics = {
            read_times: [],
            write_times: [],
            sync_times: [],
            storage_usage: 0
        };
    }
    
    measureCookieOperation(operation, callback) {
        const start = performance.now();
        const result = callback();
        const end = performance.now();
        
        this.metrics[`${operation}_times`].push(end - start);
        
        // Keep only last 100 measurements
        if (this.metrics[`${operation}_times`].length > 100) {
            this.metrics[`${operation}_times`].shift();
        }
        
        return result;
    }
    
    calculateStorageUsage() {
        let totalSize = 0;
        
        // Calculate cookie storage usage
        document.cookie.split(';').forEach(cookie => {
            totalSize += cookie.length;
        });
        
        // Calculate localStorage usage
        Object.keys(localStorage).forEach(key => {
            totalSize += key.length + localStorage[key].length;
        });
        
        this.metrics.storage_usage = totalSize;
        return totalSize;
    }
    
    getPerformanceReport() {
        return {
            avg_read_time: this.calculateAverage(this.metrics.read_times),
            avg_write_time: this.calculateAverage(this.metrics.write_times),
            avg_sync_time: this.calculateAverage(this.metrics.sync_times),
            storage_usage: this.metrics.storage_usage,
            storage_limit: 4096, // 4KB cookie limit
            usage_percentage: (this.metrics.storage_usage / 4096) * 100
        };
    }
}
```

## Implementation Checklist

### Phase 1: Foundation Setup
- [ ] Implement core cookie categories (Essential, Analytics, Personalization, Marketing)
- [ ] Set up OneTrust integration and cookie scanning
- [ ] Configure secure cookie defaults and encryption
- [ ] Implement basic consent management

### Phase 2: Advanced Features  
- [ ] Cross-domain synchronization implementation
- [ ] GA4 enhanced conversions setup
- [ ] Multi-touch attribution system
- [ ] A/B testing framework integration

### Phase 3: Optimization & Monitoring
- [ ] Performance monitoring and optimization
- [ ] Comprehensive audit logging
- [ ] Automated compliance reporting
- [ ] Cookie cleanup and maintenance procedures

## Related Documentation

- **[Technical Overview](./overview.md)** - Master technical architecture
- **[Cross-Domain Sync](./cross-domain-sync.md)** - Multi-domain implementation details
- **[Event Tracking](./event-tracking.md)** - GA4 and analytics integration
- **[Security Specifications](./security-specifications.md)** - Security protocols and encryption
- **[Troubleshooting](./troubleshooting.md)** - Common cookie issues and solutions

---

*Last Updated: January 27, 2025*  
*Next: [API Reference](./api-reference.md)* 