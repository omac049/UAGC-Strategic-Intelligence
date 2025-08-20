# UTM Framework Troubleshooting Guide

## Common Issues & Solutions

### 1. UTM Parameter Extraction Issues

#### Issue: UTM Parameters Not Being Captured
**Symptoms**:
- UTM parameters visible in URL but not stored in cookies
- Analytics showing no UTM data
- Form submissions missing attribution data

**Possible Causes**:
- JavaScript errors preventing UTM extraction
- UTM tracking script not loaded
- URL encoding issues
- Browser blocking cookies

**Solutions**:
```javascript
// Debug UTM parameter extraction
function debugUTMExtraction() {
    console.log('Current URL:', window.location.href);
    console.log('Search params:', window.location.search);
    
    const urlParams = new URLSearchParams(window.location.search);
    const utmParams = {};
    
    ['utm_source', 'utm_medium', 'utm_campaign', 'utm_content', 'utm_term'].forEach(param => {
        const value = urlParams.get(param);
        if (value) {
            utmParams[param] = value;
            console.log(`Found ${param}:`, value);
        } else {
            console.warn(`Missing ${param}`);
        }
    });
    
    return utmParams;
}

// Run debug function
debugUTMExtraction();
```

**Validation Steps**:
1. Check browser console for JavaScript errors
2. Verify UTM tracking script is loaded before other scripts
3. Test with URL encoding/decoding
4. Check browser cookie settings
5. Validate UTM parameter format

#### Issue: Malformed UTM Parameters
**Symptoms**:
- UTM parameters contain special characters
- Parameters not following naming conventions
- Data quality issues in analytics

**Solutions**:
```javascript
// UTM parameter sanitization
function sanitizeUTMParameters(utmParams) {
    const sanitized = {};
    
    Object.entries(utmParams).forEach(([key, value]) => {
        // Remove special characters and convert to lowercase
        sanitized[key] = value
            .toLowerCase()
            .replace(/[^a-z0-9_+]/g, '_')
            .replace(/_+/g, '_')
            .replace(/^_|_$/g, '');
    });
    
    return sanitized;
}

// Validate UTM format
function validateUTMFormat(utmParams) {
    const patterns = {
        source: /^[a-z0-9_]+$/,
        medium: /^[a-z0-9_]+$/,
        campaign: /^[a-z0-9_]+$/,
        content: /^[a-z0-9_]+$/,
        term: /^[a-z0-9+_]+$/
    };
    
    const errors = [];
    
    Object.entries(utmParams).forEach(([param, value]) => {
        if (patterns[param] && !patterns[param].test(value)) {
            errors.push(`Invalid ${param} format: ${value}`);
        }
    });
    
    return errors;
}
```

### 2. Cookie Storage Issues

#### Issue: Cookies Not Being Set
**Symptoms**:
- UTM data not persisting across pages
- Attribution data missing in forms
- First-touch attribution not working

**Possible Causes**:
- Browser cookie restrictions
- Incorrect cookie domain/path settings
- Cookie size limitations
- SameSite policy issues

**Solutions**:
```javascript
// Enhanced cookie management
class CookieManager {
    constructor() {
        this.domain = this.getDomain();
        this.testCookieSupport();
    }
    
    getDomain() {
        const hostname = window.location.hostname;
        // Use subdomain for UAGC domains
        if (hostname.includes('uagc.edu')) {
            return '.uagc.edu';
        }
        return hostname;
    }
    
    testCookieSupport() {
        const testCookie = 'utm_test';
        this.setCookie(testCookie, 'test', 1);
        
        if (!this.getCookie(testCookie)) {
            console.error('Cookie support not available');
            this.fallbackToSessionStorage();
        } else {
            this.deleteCookie(testCookie);
            console.log('Cookie support confirmed');
        }
    }
    
    setCookie(name, value, days) {
        try {
            const expires = new Date(Date.now() + days * 24 * 60 * 60 * 1000).toUTCString();
            const cookieString = `${name}=${encodeURIComponent(value)}; expires=${expires}; path=/; domain=${this.domain}; secure; samesite=lax`;
            
            document.cookie = cookieString;
            
            // Verify cookie was set
            if (!this.getCookie(name)) {
                throw new Error(`Failed to set cookie: ${name}`);
            }
        } catch (error) {
            console.error('Cookie setting failed:', error);
            this.fallbackToSessionStorage(name, value);
        }
    }
    
    getCookie(name) {
        const value = `; ${document.cookie}`;
        const parts = value.split(`; ${name}=`);
        if (parts.length === 2) {
            return decodeURIComponent(parts.pop().split(';').shift());
        }
        return null;
    }
    
    deleteCookie(name) {
        document.cookie = `${name}=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/; domain=${this.domain};`;
    }
    
    fallbackToSessionStorage(name, value) {
        if (name && value) {
            sessionStorage.setItem(`utm_${name}`, value);
        }
        console.warn('Using sessionStorage fallback for UTM data');
    }
}
```

#### Issue: Cookie Size Limitations
**Symptoms**:
- Large touchpoint data not being stored
- Cookie data truncated
- Attribution data incomplete

**Solutions**:
```javascript
// Optimize cookie storage
class OptimizedUTMStorage {
    constructor() {
        this.maxCookieSize = 4000; // bytes
        this.compressionMap = {
            'utm_source': 'us',
            'utm_medium': 'um',
            'utm_campaign': 'uc',
            'utm_content': 'uco',
            'utm_term': 'ut'
        };
    }
    
    compressData(data) {
        let compressed = JSON.stringify(data);
        
        // Apply compression mapping
        Object.entries(this.compressionMap).forEach(([full, short]) => {
            compressed = compressed.replace(new RegExp(`"${full}"`, 'g'), `"${short}"`);
        });
        
        return compressed;
    }
    
    decompressData(compressed) {
        let decompressed = compressed;
        
        // Reverse compression mapping
        Object.entries(this.compressionMap).forEach(([full, short]) => {
            decompressed = decompressed.replace(new RegExp(`"${short}"`, 'g'), `"${full}"`);
        });
        
        return JSON.parse(decompressed);
    }
    
    storeTouchpoints(touchpoints) {
        // Limit touchpoints to prevent cookie overflow
        const limitedTouchpoints = touchpoints.slice(-5); // Keep last 5
        const compressed = this.compressData(limitedTouchpoints);
        
        if (compressed.length > this.maxCookieSize) {
            // Further reduce data
            const minimalTouchpoints = limitedTouchpoints.map(tp => ({
                us: tp.utm_source,
                um: tp.utm_medium,
                uc: tp.utm_campaign,
                ts: tp.timestamp
            }));
            
            const minimalCompressed = JSON.stringify(minimalTouchpoints);
            this.setCookie('touchpoints', minimalCompressed, 90);
        } else {
            this.setCookie('touchpoints', compressed, 90);
        }
    }
}
```

### 3. Attribution Model Issues

#### Issue: Incorrect Attribution Assignment
**Symptoms**:
- First-touch attribution overwriting existing data
- Last-touch attribution not updating
- Multi-touch attribution missing touchpoints

**Solutions**:
```javascript
// Robust attribution logic
class AttributionManager {
    constructor() {
        this.cookieManager = new CookieManager();
    }
    
    handleFirstTouch(utmParams) {
        // Only set first-touch if no existing first-touch data
        const existingFirstTouch = this.cookieManager.getCookie('first_touch_source');
        
        if (!existingFirstTouch) {
            Object.entries(utmParams).forEach(([key, value]) => {
                this.cookieManager.setCookie(`first_touch_${key}`, value, 365);
            });
            this.cookieManager.setCookie('first_touch_timestamp', Date.now(), 365);
            console.log('First-touch attribution set:', utmParams);
        } else {
            console.log('First-touch attribution already exists, skipping');
        }
    }
    
    handleLastTouch(utmParams) {
        // Always update last-touch attribution
        Object.entries(utmParams).forEach(([key, value]) => {
            this.cookieManager.setCookie(`last_touch_${key}`, value, 30);
        });
        this.cookieManager.setCookie('last_touch_timestamp', Date.now(), 30);
        console.log('Last-touch attribution updated:', utmParams);
    }
    
    handleMultiTouch(utmParams) {
        let touchpoints = [];
        
        try {
            const existingTouchpoints = this.cookieManager.getCookie('touchpoints');
            if (existingTouchpoints) {
                touchpoints = JSON.parse(existingTouchpoints);
            }
        } catch (error) {
            console.error('Error parsing existing touchpoints:', error);
            touchpoints = [];
        }
        
        // Add new touchpoint
        touchpoints.push({
            ...utmParams,
            timestamp: Date.now(),
            page: window.location.pathname,
            referrer: document.referrer
        });
        
        // Remove duplicates within short time window (5 minutes)
        touchpoints = this.deduplicateTouchpoints(touchpoints);
        
        // Store updated touchpoints
        const optimizedStorage = new OptimizedUTMStorage();
        optimizedStorage.storeTouchpoints(touchpoints);
    }
    
    deduplicateTouchpoints(touchpoints) {
        const deduplicated = [];
        const timeWindow = 5 * 60 * 1000; // 5 minutes
        
        touchpoints.forEach(current => {
            const isDuplicate = deduplicated.some(existing => {
                const timeDiff = Math.abs(current.timestamp - existing.timestamp);
                const sameSource = current.utm_source === existing.utm_source;
                const sameMedium = current.utm_medium === existing.utm_medium;
                const sameCampaign = current.utm_campaign === existing.utm_campaign;
                
                return timeDiff < timeWindow && sameSource && sameMedium && sameCampaign;
            });
            
            if (!isDuplicate) {
                deduplicated.push(current);
            }
        });
        
        return deduplicated;
    }
}
```

### 4. Cross-Domain Tracking Issues

#### Issue: UTM Parameters Lost During Cross-Domain Navigation
**Symptoms**:
- Attribution breaks when moving between UAGC domains
- UTM parameters not passed to subdomains
- Session continuity lost

**Solutions**:
```javascript
// Cross-domain UTM preservation
class CrossDomainUTMManager {
    constructor() {
        this.uagcDomains = [
            'uagc.edu',
            'www.uagc.edu',
            'cloud.mail.uagc.edu',
            'portal.uagc.edu'
        ];
    }
    
    preserveUTMParameters() {
        // Intercept all outbound links
        document.addEventListener('click', (event) => {
            const link = event.target.closest('a');
            if (link && this.isUAGCDomain(link.hostname)) {
                this.appendUTMParameters(link);
            }
        });
    }
    
    isUAGCDomain(hostname) {
        return this.uagcDomains.some(domain => 
            hostname === domain || hostname.endsWith('.' + domain)
        );
    }
    
    appendUTMParameters(link) {
        const currentUTM = this.getCurrentUTMParameters();
        if (Object.keys(currentUTM).length > 0) {
            const url = new URL(link.href);
            
            // Only append if no existing UTM parameters
            if (!url.searchParams.has('utm_source')) {
                Object.entries(currentUTM).forEach(([key, value]) => {
                    url.searchParams.set(`utm_${key}`, value);
                });
                
                link.href = url.toString();
                console.log('UTM parameters appended to cross-domain link:', link.href);
            }
        }
    }
    
    getCurrentUTMParameters() {
        const cookieManager = new CookieManager();
        const utmParams = {};
        
        ['source', 'medium', 'campaign', 'content', 'term'].forEach(param => {
            const value = cookieManager.getCookie(`last_touch_${param}`);
            if (value) {
                utmParams[param] = value;
            }
        });
        
        return utmParams;
    }
}

// Initialize cross-domain tracking
document.addEventListener('DOMContentLoaded', () => {
    const crossDomainManager = new CrossDomainUTMManager();
    crossDomainManager.preserveUTMParameters();
});
```

### 5. Analytics Integration Issues

#### Issue: GA4 Not Receiving UTM Data
**Symptoms**:
- Custom dimensions empty in GA4
- UTM events not firing
- Attribution reports showing no data

**Solutions**:
```javascript
// Enhanced GA4 debugging
class GA4UTMDebugger {
    constructor(measurementId) {
        this.measurementId = measurementId;
        this.debugMode = window.location.hostname.includes('dev') || window.location.search.includes('debug=true');
    }
    
    enableDebugMode() {
        if (this.debugMode) {
            gtag('config', this.measurementId, {
                'debug_mode': true,
                'send_page_view': false
            });
            
            // Listen for gtag events
            window.dataLayer = window.dataLayer || [];
            const originalPush = window.dataLayer.push;
            
            window.dataLayer.push = function(...args) {
                console.log('GA4 Event:', args);
                return originalPush.apply(this, args);
            };
        }
    }
    
    validateCustomDimensions() {
        const utmTracker = new UTMTracker();
        const utmParams = utmTracker.utmParams;
        
        if (Object.keys(utmParams).length === 0) {
            console.warn('No UTM parameters found for GA4 tracking');
            return false;
        }
        
        // Test custom dimension mapping
        gtag('event', 'utm_debug_test', {
            'custom_parameter_1': utmParams.source || 'not_set',
            'custom_parameter_2': utmParams.medium || 'not_set',
            'custom_parameter_3': utmParams.campaign || 'not_set'
        });
        
        console.log('GA4 UTM debug event sent:', utmParams);
        return true;
    }
    
    monitorEventFiring() {
        // Monitor gtag calls
        const originalGtag = window.gtag;
        
        window.gtag = function(...args) {
            if (args[0] === 'event') {
                console.log('GA4 Event Fired:', args[1], args[2]);
            }
            return originalGtag.apply(this, args);
        };
    }
}

// Initialize GA4 debugging
if (window.location.hostname.includes('dev') || window.location.search.includes('debug=true')) {
    const ga4Debugger = new GA4UTMDebugger('GA_MEASUREMENT_ID');
    ga4Debugger.enableDebugMode();
    ga4Debugger.validateCustomDimensions();
    ga4Debugger.monitorEventFiring();
}
```

### 6. Form Integration Issues

#### Issue: UTM Data Not Populating in Forms
**Symptoms**:
- Hidden form fields remain empty
- Form submissions missing attribution data
- CRM receiving incomplete lead data

**Solutions**:
```javascript
// Enhanced form integration debugging
class FormUTMDebugger {
    constructor(formSelector) {
        this.form = document.querySelector(formSelector);
        this.debugMode = window.location.search.includes('debug=true');
    }
    
    debugFormIntegration() {
        if (!this.form) {
            console.error('Form not found:', formSelector);
            return;
        }
        
        console.log('Form found:', this.form);
        
        // Check for UTM hidden fields
        const utmFields = this.form.querySelectorAll('input[name^="utm_"], input[name^="first_touch_"], input[name^="last_touch_"]');
        console.log('UTM fields found:', utmFields.length);
        
        utmFields.forEach(field => {
            console.log(`Field ${field.name}: ${field.value || 'empty'}`);
        });
        
        // Check cookie data
        const cookieManager = new CookieManager();
        ['utm_source', 'utm_medium', 'utm_campaign', 'first_touch_source', 'last_touch_source'].forEach(cookieName => {
            const value = cookieManager.getCookie(cookieName);
            console.log(`Cookie ${cookieName}: ${value || 'not set'}`);
        });
    }
    
    validateFormData() {
        const formData = new FormData(this.form);
        const utmData = {};
        
        for (const [key, value] of formData.entries()) {
            if (key.startsWith('utm_') || key.startsWith('first_touch_') || key.startsWith('last_touch_')) {
                utmData[key] = value;
            }
        }
        
        console.log('Form UTM data:', utmData);
        
        // Validate required fields
        const requiredFields = ['utm_source', 'utm_medium', 'utm_campaign'];
        const missingFields = requiredFields.filter(field => !utmData[field]);
        
        if (missingFields.length > 0) {
            console.error('Missing required UTM fields:', missingFields);
            return false;
        }
        
        return true;
    }
}

// Debug form integration
document.addEventListener('DOMContentLoaded', () => {
    if (window.location.search.includes('debug=true')) {
        const formDebugger = new FormUTMDebugger('#rfi-form');
        formDebugger.debugFormIntegration();
        
        // Add validation on form submit
        const form = document.querySelector('#rfi-form');
        if (form) {
            form.addEventListener('submit', (event) => {
                if (!formDebugger.validateFormData()) {
                    console.error('Form validation failed');
                    // Don't prevent submission in production
                }
            });
        }
    }
});
```

## Performance Troubleshooting

### Issue: UTM Tracking Impacting Page Load Speed
**Symptoms**:
- Increased page load times
- JavaScript execution delays
- Poor user experience

**Solutions**:
```javascript
// Optimized UTM tracking
class OptimizedUTMTracker {
    constructor() {
        this.initialized = false;
        this.queue = [];
        this.init();
    }
    
    init() {
        // Use requestIdleCallback for non-critical UTM processing
        if ('requestIdleCallback' in window) {
            requestIdleCallback(() => this.processUTMData());
        } else {
            // Fallback for browsers without requestIdleCallback
            setTimeout(() => this.processUTMData(), 100);
        }
    }
    
    processUTMData() {
        const utmParams = this.extractUTMParameters();
        
        if (Object.keys(utmParams).length > 0) {
            // Batch operations to minimize DOM manipulation
            this.batchCookieOperations(utmParams);
            this.queueAnalyticsEvents(utmParams);
        }
        
        this.initialized = true;
        this.processQueue();
    }
    
    batchCookieOperations(utmParams) {
        const operations = [];
        
        // Prepare all cookie operations
        if (!this.getCookie('first_touch_source')) {
            Object.entries(utmParams).forEach(([key, value]) => {
                operations.push({ action: 'set', name: `first_touch_${key}`, value, days: 365 });
            });
        }
        
        Object.entries(utmParams).forEach(([key, value]) => {
            operations.push({ action: 'set', name: `last_touch_${key}`, value, days: 30 });
        });
        
        // Execute all operations at once
        operations.forEach(op => {
            if (op.action === 'set') {
                this.setCookie(op.name, op.value, op.days);
            }
        });
    }
    
    queueAnalyticsEvents(utmParams) {
        // Queue analytics events to prevent blocking
        this.queue.push(() => {
            if (typeof gtag !== 'undefined') {
                gtag('event', 'utm_parameters_captured', {
                    'event_category': 'UTM Tracking',
                    'non_interaction': true,
                    'custom_parameter_1': utmParams.source,
                    'custom_parameter_2': utmParams.medium,
                    'custom_parameter_3': utmParams.campaign
                });
            }
        });
    }
    
    processQueue() {
        while (this.queue.length > 0) {
            const task = this.queue.shift();
            try {
                task();
            } catch (error) {
                console.error('Error processing queued task:', error);
            }
        }
    }
}
```

## Monitoring & Alerting

### UTM Data Quality Monitoring
```javascript
// UTM quality monitoring
class UTMQualityMonitor {
    constructor() {
        this.metrics = {
            total_pageviews: 0,
            utm_pageviews: 0,
            invalid_utm_formats: 0,
            missing_required_params: 0,
            cookie_failures: 0
        };
        
        this.init();
    }
    
    init() {
        this.trackPageview();
        this.monitorUTMQuality();
        this.scheduleReporting();
    }
    
    trackPageview() {
        this.metrics.total_pageviews++;
        
        const utmParams = this.extractUTMParameters();
        if (Object.keys(utmParams).length > 0) {
            this.metrics.utm_pageviews++;
            this.validateUTMQuality(utmParams);
        }
    }
    
    validateUTMQuality(utmParams) {
        // Check for required parameters
        const required = ['source', 'medium', 'campaign'];
        const missing = required.filter(param => !utmParams[param]);
        
        if (missing.length > 0) {
            this.metrics.missing_required_params++;
            console.warn('Missing required UTM parameters:', missing);
        }
        
        // Check format validity
        const patterns = {
            source: /^[a-z0-9_]+$/,
            medium: /^[a-z0-9_]+$/,
            campaign: /^[a-z0-9_]+$/
        };
        
        Object.entries(utmParams).forEach(([param, value]) => {
            if (patterns[param] && !patterns[param].test(value)) {
                this.metrics.invalid_utm_formats++;
                console.warn(`Invalid UTM ${param} format:`, value);
            }
        });
    }
    
    scheduleReporting() {
        // Report metrics every 5 minutes
        setInterval(() => {
            this.reportMetrics();
        }, 5 * 60 * 1000);
    }
    
    reportMetrics() {
        const qualityScore = this.calculateQualityScore();
        
        // Send to analytics
        if (typeof gtag !== 'undefined') {
            gtag('event', 'utm_quality_metrics', {
                'event_category': 'Data Quality',
                'utm_coverage': (this.metrics.utm_pageviews / this.metrics.total_pageviews * 100).toFixed(2),
                'quality_score': qualityScore,
                'non_interaction': true
            });
        }
        
        // Log to console in debug mode
        if (window.location.search.includes('debug=true')) {
            console.log('UTM Quality Metrics:', {
                ...this.metrics,
                quality_score: qualityScore,
                utm_coverage: (this.metrics.utm_pageviews / this.metrics.total_pageviews * 100).toFixed(2) + '%'
            });
        }
    }
    
    calculateQualityScore() {
        if (this.metrics.utm_pageviews === 0) return 100;
        
        const errorRate = (this.metrics.invalid_utm_formats + this.metrics.missing_required_params) / this.metrics.utm_pageviews;
        return Math.max(0, (1 - errorRate) * 100).toFixed(2);
    }
}

// Initialize quality monitoring
document.addEventListener('DOMContentLoaded', () => {
    new UTMQualityMonitor();
});
```

## Support & Escalation

### When to Escalate Issues

1. **Critical Issues** (Escalate Immediately):
   - Complete UTM tracking failure across all pages
   - Data loss affecting attribution reporting
   - Security vulnerabilities in cookie handling

2. **High Priority Issues** (Escalate within 4 hours):
   - GA4 integration failures
   - CRM data sync issues
   - Cross-domain tracking problems

3. **Medium Priority Issues** (Escalate within 24 hours):
   - Form integration problems
   - Attribution model discrepancies
   - Performance impact issues

4. **Low Priority Issues** (Escalate within 72 hours):
   - Minor data quality issues
   - Non-critical format validation errors
   - Enhancement requests

### Contact Information
- **Technical Lead**: [Technical Team Contact]
- **Analytics Team**: [Analytics Team Contact]
- **Marketing Operations**: [Marketing Ops Contact]
- **Emergency Escalation**: [Emergency Contact]

---

**Related Documentation**:
- [UTM Framework Overview](./overview.md)
- [Implementation Guide](./implementation-guide.md)
- [Section 3: Technical Implementation](../03-technical-implementation/_index.md) 