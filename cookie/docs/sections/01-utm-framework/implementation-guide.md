# UTM Framework Implementation Guide

## Pre-Implementation Setup

### 1. Environment Preparation

**Required Tools & Access**:
- Google Analytics 4 account with admin access
- Google Tag Manager container access
- Salesforce CRM integration permissions
- Marketing automation platform API access
- Development environment for testing

**Stakeholder Alignment**:
- Marketing team training on UTM standards
- Sales team briefing on lead attribution
- IT team coordination for technical implementation
- Analytics team setup for reporting infrastructure

### 2. UTM Parameter Standards Definition

**Create UTM Parameter Library**:
```javascript
// UTM Parameter Configuration
const UTM_CONFIG = {
    sources: {
        'google': 'Google Search/Display',
        'facebook': 'Facebook/Meta platforms',
        'linkedin': 'LinkedIn advertising',
        'bing': 'Microsoft Bing',
        'email': 'Email campaigns',
        'direct': 'Direct traffic',
        'referral': 'Referral traffic',
        'organic': 'Organic search'
    },
    
    mediums: {
        'cpc': 'Cost-per-click advertising',
        'display': 'Display advertising',
        'email': 'Email marketing',
        'social': 'Social media',
        'organic': 'Organic search',
        'referral': 'Referral traffic',
        'direct': 'Direct traffic',
        'affiliate': 'Affiliate marketing'
    },
    
    audiences: {
        'working_professionals': 'Full-time working adults',
        'traditional_students': '18-24 age demographic',
        'military_veterans': 'Military/veteran community',
        'career_changers': 'Career transition seekers',
        'international': 'International students'
    }
};
```

## Campaign Setup

### 1. Campaign Naming Convention Implementation

**Campaign URL Builder**:
```javascript
class UTMCampaignBuilder {
    constructor(config) {
        this.config = config;
        this.baseUrl = 'https://www.uagc.edu';
    }
    
    buildCampaignURL(params) {
        const {
            source,
            medium,
            campaign,
            content = '',
            term = '',
            landingPage = '/'
        } = params;
        
        // Validate required parameters
        if (!source || !medium || !campaign) {
            throw new Error('Source, medium, and campaign are required');
        }
        
        // Build UTM parameters
        const utmParams = new URLSearchParams({
            utm_source: source,
            utm_medium: medium,
            utm_campaign: campaign
        });
        
        if (content) utmParams.append('utm_content', content);
        if (term) utmParams.append('utm_term', term);
        
        return `${this.baseUrl}${landingPage}?${utmParams.toString()}`;
    }
    
    generateCampaignName(year, quarter, program, audience, objective, variant = 'a') {
        return `${year}_${quarter}_${program}_${audience}_${objective}_${variant}`;
    }
}

// Usage example
const campaignBuilder = new UTMCampaignBuilder(UTM_CONFIG);

const campaignURL = campaignBuilder.buildCampaignURL({
    source: 'google',
    medium: 'cpc',
    campaign: campaignBuilder.generateCampaignName(2025, 'q1', 'mba', 'working_professionals', 'conversion'),
    content: 'hero_banner',
    term: 'mba+online+degree',
    landingPage: '/programs/mba'
});
```

### 2. Cross-Platform Campaign Setup

**Google Ads Campaign Configuration**:
```javascript
// Google Ads UTM template
const googleAdsUTMTemplate = {
    finalUrl: 'https://www.uagc.edu/programs/{program}',
    trackingTemplate: `{lpurl}?utm_source=google&utm_medium=cpc&utm_campaign={campaignid}&utm_content={adgroupid}&utm_term={keyword}`,
    customParameters: {
        'program': '{program}',
        'audience': '{audience}',
        'objective': 'conversion'
    }
};
```

**Facebook Ads Campaign Configuration**:
```javascript
// Facebook Ads UTM template
const facebookAdsUTMTemplate = {
    destinationUrl: 'https://www.uagc.edu/programs/{program}',
    urlParameters: 'utm_source=facebook&utm_medium=social&utm_campaign={{campaign.name}}&utm_content={{adset.name}}&utm_term={{ad.name}}'
};
```

## Technical Implementation

### 1. UTM Parameter Capture

**Client-Side UTM Extraction**:
```javascript
class UTMTracker {
    constructor() {
        this.utmParams = this.extractUTMParameters();
        this.init();
    }
    
    extractUTMParameters() {
        const urlParams = new URLSearchParams(window.location.search);
        const utmParams = {};
        
        // Standard UTM parameters
        const utmKeys = ['utm_source', 'utm_medium', 'utm_campaign', 'utm_content', 'utm_term'];
        
        utmKeys.forEach(key => {
            const value = urlParams.get(key);
            if (value) {
                utmParams[key.replace('utm_', '')] = value;
            }
        });
        
        // Extended parameters
        const extendedKeys = ['utm_audience', 'utm_placement', 'utm_creative'];
        extendedKeys.forEach(key => {
            const value = urlParams.get(key);
            if (value) {
                utmParams[key.replace('utm_', '')] = value;
            }
        });
        
        return utmParams;
    }
    
    init() {
        if (Object.keys(this.utmParams).length > 0) {
            this.storeUTMParameters();
            this.sendToAnalytics();
            this.updateCRM();
        }
    }
    
    storeUTMParameters() {
        // Store in cookies for attribution
        this.storeFirstTouch();
        this.storeLastTouch();
        this.storeTouchpoint();
    }
    
    storeFirstTouch() {
        if (!this.getCookie('first_touch_source')) {
            Object.entries(this.utmParams).forEach(([key, value]) => {
                this.setCookie(`first_touch_${key}`, value, 365);
            });
            this.setCookie('first_touch_timestamp', Date.now(), 365);
        }
    }
    
    storeLastTouch() {
        Object.entries(this.utmParams).forEach(([key, value]) => {
            this.setCookie(`last_touch_${key}`, value, 30);
        });
        this.setCookie('last_touch_timestamp', Date.now(), 30);
    }
    
    storeTouchpoint() {
        let touchpoints = JSON.parse(this.getCookie('touchpoints') || '[]');
        
        touchpoints.push({
            ...this.utmParams,
            timestamp: Date.now(),
            page: window.location.pathname,
            referrer: document.referrer
        });
        
        // Keep last 10 touchpoints
        if (touchpoints.length > 10) {
            touchpoints = touchpoints.slice(-10);
        }
        
        this.setCookie('touchpoints', JSON.stringify(touchpoints), 90);
    }
    
    setCookie(name, value, days) {
        const expires = new Date(Date.now() + days * 24 * 60 * 60 * 1000).toUTCString();
        document.cookie = `${name}=${encodeURIComponent(value)}; expires=${expires}; path=/; secure; samesite=strict`;
    }
    
    getCookie(name) {
        const value = `; ${document.cookie}`;
        const parts = value.split(`; ${name}=`);
        if (parts.length === 2) {
            return decodeURIComponent(parts.pop().split(';').shift());
        }
        return null;
    }
}

// Initialize UTM tracking
document.addEventListener('DOMContentLoaded', () => {
    new UTMTracker();
});
```

### 2. Google Analytics 4 Integration

**GA4 Enhanced Conversion Setup**:
```javascript
// GA4 UTM Integration
class GA4UTMIntegration {
    constructor(measurementId) {
        this.measurementId = measurementId;
        this.init();
    }
    
    init() {
        // Configure GA4 with custom dimensions
        gtag('config', this.measurementId, {
            'custom_map': {
                'custom_parameter_1': 'utm_source',
                'custom_parameter_2': 'utm_medium',
                'custom_parameter_3': 'utm_campaign',
                'custom_parameter_4': 'utm_content',
                'custom_parameter_5': 'utm_term'
            }
        });
        
        // Send UTM parameters as custom events
        this.sendUTMEvent();
    }
    
    sendUTMEvent() {
        const utmTracker = new UTMTracker();
        
        if (Object.keys(utmTracker.utmParams).length > 0) {
            gtag('event', 'utm_parameters_captured', {
                'event_category': 'UTM Tracking',
                'event_label': `${utmTracker.utmParams.source}/${utmTracker.utmParams.medium}`,
                'custom_parameter_1': utmTracker.utmParams.source,
                'custom_parameter_2': utmTracker.utmParams.medium,
                'custom_parameter_3': utmTracker.utmParams.campaign,
                'custom_parameter_4': utmTracker.utmParams.content,
                'custom_parameter_5': utmTracker.utmParams.term
            });
        }
    }
    
    trackConversion(conversionData) {
        const utmTracker = new UTMTracker();
        
        gtag('event', 'conversion', {
            'send_to': 'AW-CONVERSION_ID/CONVERSION_LABEL',
            'value': conversionData.value || 1.0,
            'currency': 'USD',
            'transaction_id': conversionData.transactionId,
            'utm_source': utmTracker.getCookie('last_touch_source'),
            'utm_medium': utmTracker.getCookie('last_touch_medium'),
            'utm_campaign': utmTracker.getCookie('last_touch_campaign')
        });
    }
}
```

### 3. Salesforce CRM Integration

**Lead Attribution Setup**:
```javascript
// Salesforce UTM Integration
class SalesforceUTMIntegration {
    constructor(apiEndpoint, apiKey) {
        this.apiEndpoint = apiEndpoint;
        this.apiKey = apiKey;
    }
    
    async createLeadWithAttribution(leadData, utmData) {
        const enrichedLeadData = {
            ...leadData,
            Lead_Source__c: utmData.source,
            Lead_Medium__c: utmData.medium,
            Campaign_Name__c: utmData.campaign,
            Ad_Content__c: utmData.content,
            Search_Term__c: utmData.term,
            First_Touch_Source__c: this.getCookie('first_touch_source'),
            First_Touch_Medium__c: this.getCookie('first_touch_medium'),
            First_Touch_Campaign__c: this.getCookie('first_touch_campaign'),
            Last_Touch_Source__c: this.getCookie('last_touch_source'),
            Last_Touch_Medium__c: this.getCookie('last_touch_medium'),
            Last_Touch_Campaign__c: this.getCookie('last_touch_campaign'),
            Attribution_Touchpoints__c: this.getCookie('touchpoints'),
            Attribution_Model__c: 'multi_touch'
        };
        
        try {
            const response = await fetch(`${this.apiEndpoint}/leads`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${this.apiKey}`
                },
                body: JSON.stringify(enrichedLeadData)
            });
            
            if (!response.ok) {
                throw new Error(`Salesforce API error: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            console.error('Error creating lead with attribution:', error);
            throw error;
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
}
```

## Form Integration

### 1. Lead Form UTM Capture

**RFI Form Integration**:
```javascript
// RFI Form UTM Integration
class RFIFormUTMIntegration {
    constructor(formSelector) {
        this.form = document.querySelector(formSelector);
        this.utmTracker = new UTMTracker();
        this.init();
    }
    
    init() {
        if (this.form) {
            this.addHiddenUTMFields();
            this.populateUTMFields();
            this.form.addEventListener('submit', this.handleFormSubmit.bind(this));
        }
    }
    
    addHiddenUTMFields() {
        const utmFields = [
            'utm_source', 'utm_medium', 'utm_campaign', 'utm_content', 'utm_term',
            'first_touch_source', 'first_touch_medium', 'first_touch_campaign',
            'last_touch_source', 'last_touch_medium', 'last_touch_campaign'
        ];
        
        utmFields.forEach(fieldName => {
            const hiddenField = document.createElement('input');
            hiddenField.type = 'hidden';
            hiddenField.name = fieldName;
            hiddenField.id = fieldName;
            this.form.appendChild(hiddenField);
        });
    }
    
    populateUTMFields() {
        // Current UTM parameters
        Object.entries(this.utmTracker.utmParams).forEach(([key, value]) => {
            const field = this.form.querySelector(`#utm_${key}`);
            if (field) field.value = value;
        });
        
        // First touch attribution
        ['source', 'medium', 'campaign'].forEach(param => {
            const value = this.utmTracker.getCookie(`first_touch_${param}`);
            const field = this.form.querySelector(`#first_touch_${param}`);
            if (field && value) field.value = value;
        });
        
        // Last touch attribution
        ['source', 'medium', 'campaign'].forEach(param => {
            const value = this.utmTracker.getCookie(`last_touch_${param}`);
            const field = this.form.querySelector(`#last_touch_${param}`);
            if (field && value) field.value = value;
        });
    }
    
    handleFormSubmit(event) {
        // Track form submission with UTM attribution
        gtag('event', 'form_submit', {
            'event_category': 'Lead Generation',
            'event_label': 'RFI Form',
            'utm_source': this.utmTracker.getCookie('last_touch_source'),
            'utm_medium': this.utmTracker.getCookie('last_touch_medium'),
            'utm_campaign': this.utmTracker.getCookie('last_touch_campaign')
        });
    }
}

// Initialize RFI form integration
document.addEventListener('DOMContentLoaded', () => {
    new RFIFormUTMIntegration('#rfi-form');
});
```

### 2. Application Form Integration

**Application Form UTM Capture**:
```javascript
// Application Form UTM Integration
class ApplicationFormUTMIntegration extends RFIFormUTMIntegration {
    constructor(formSelector) {
        super(formSelector);
    }
    
    handleFormSubmit(event) {
        // Track application submission with full attribution
        const touchpoints = JSON.parse(this.utmTracker.getCookie('touchpoints') || '[]');
        
        gtag('event', 'application_submit', {
            'event_category': 'Conversion',
            'event_label': 'Application Form',
            'value': 100, // Application value
            'utm_source': this.utmTracker.getCookie('last_touch_source'),
            'utm_medium': this.utmTracker.getCookie('last_touch_medium'),
            'utm_campaign': this.utmTracker.getCookie('last_touch_campaign'),
            'touchpoint_count': touchpoints.length,
            'journey_duration': this.calculateJourneyDuration()
        });
    }
    
    calculateJourneyDuration() {
        const firstTouch = this.utmTracker.getCookie('first_touch_timestamp');
        if (firstTouch) {
            return Math.round((Date.now() - parseInt(firstTouch)) / (1000 * 60 * 60 * 24)); // Days
        }
        return 0;
    }
}

// Initialize application form integration
document.addEventListener('DOMContentLoaded', () => {
    new ApplicationFormUTMIntegration('#application-form');
});
```

## Testing & Validation

### 1. UTM Parameter Testing

**Testing Checklist**:
```javascript
// UTM Testing Suite
class UTMTestingSuite {
    constructor() {
        this.tests = [];
        this.init();
    }
    
    init() {
        this.addTest('UTM Parameter Extraction', this.testUTMExtraction);
        this.addTest('Cookie Storage', this.testCookieStorage);
        this.addTest('GA4 Integration', this.testGA4Integration);
        this.addTest('Form Integration', this.testFormIntegration);
        this.runTests();
    }
    
    addTest(name, testFunction) {
        this.tests.push({ name, testFunction: testFunction.bind(this) });
    }
    
    async runTests() {
        console.log('Running UTM Testing Suite...');
        
        for (const test of this.tests) {
            try {
                const result = await test.testFunction();
                console.log(`✅ ${test.name}: PASSED`, result);
            } catch (error) {
                console.error(`❌ ${test.name}: FAILED`, error);
            }
        }
    }
    
    testUTMExtraction() {
        // Test URL: ?utm_source=google&utm_medium=cpc&utm_campaign=test
        const testUrl = new URL('https://www.uagc.edu/?utm_source=google&utm_medium=cpc&utm_campaign=test');
        const originalLocation = window.location;
        
        // Mock window.location
        Object.defineProperty(window, 'location', {
            value: testUrl,
            writable: true
        });
        
        const utmTracker = new UTMTracker();
        const expectedParams = {
            source: 'google',
            medium: 'cpc',
            campaign: 'test'
        };
        
        // Restore original location
        window.location = originalLocation;
        
        if (JSON.stringify(utmTracker.utmParams) === JSON.stringify(expectedParams)) {
            return 'UTM parameters extracted correctly';
        } else {
            throw new Error('UTM parameter extraction failed');
        }
    }
    
    testCookieStorage() {
        const utmTracker = new UTMTracker();
        const testValue = 'test_value';
        
        utmTracker.setCookie('test_utm_param', testValue, 1);
        const retrievedValue = utmTracker.getCookie('test_utm_param');
        
        if (retrievedValue === testValue) {
            return 'Cookie storage working correctly';
        } else {
            throw new Error('Cookie storage failed');
        }
    }
    
    testGA4Integration() {
        // Mock gtag function
        window.gtag = window.gtag || function() {
            window.gtag.calls = window.gtag.calls || [];
            window.gtag.calls.push(arguments);
        };
        
        const ga4Integration = new GA4UTMIntegration('GA_MEASUREMENT_ID');
        
        if (window.gtag.calls && window.gtag.calls.length > 0) {
            return 'GA4 integration initialized';
        } else {
            throw new Error('GA4 integration failed');
        }
    }
    
    testFormIntegration() {
        // Create test form
        const testForm = document.createElement('form');
        testForm.id = 'test-form';
        document.body.appendChild(testForm);
        
        const formIntegration = new RFIFormUTMIntegration('#test-form');
        
        // Check if hidden fields were added
        const hiddenFields = testForm.querySelectorAll('input[type="hidden"]');
        
        // Clean up
        document.body.removeChild(testForm);
        
        if (hiddenFields.length > 0) {
            return 'Form integration working correctly';
        } else {
            throw new Error('Form integration failed');
        }
    }
}

// Run tests in development environment
if (window.location.hostname === 'localhost' || window.location.hostname.includes('dev')) {
    new UTMTestingSuite();
}
```

## Deployment Checklist

### Pre-Production Validation
- [ ] UTM parameter taxonomy finalized
- [ ] Campaign naming conventions documented
- [ ] Technical implementation tested
- [ ] GA4 custom dimensions configured
- [ ] Salesforce field mapping verified
- [ ] Form integration tested
- [ ] Cross-domain tracking validated
- [ ] Attribution models configured

### Production Deployment
- [ ] UTM tracking code deployed to all pages
- [ ] Form integrations implemented
- [ ] Analytics tracking verified
- [ ] CRM integration tested
- [ ] Attribution reporting configured
- [ ] Team training completed
- [ ] Documentation updated
- [ ] Monitoring alerts configured

### Post-Deployment Monitoring
- [ ] UTM parameter capture rates monitored
- [ ] Attribution accuracy validated
- [ ] Data quality metrics tracked
- [ ] Campaign performance measured
- [ ] Cross-platform sync verified
- [ ] Error rates monitored
- [ ] User experience impact assessed

---

**Next Steps**: Review [Troubleshooting Guide](./troubleshooting.md) for common issues and solutions, and proceed to [Section 3: Technical Implementation](../03-technical-implementation/_index.md) for technical integration details. 