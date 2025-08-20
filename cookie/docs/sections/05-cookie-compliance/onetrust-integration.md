# OneTrust Enterprise Integration Guide

## Executive Summary

OneTrust serves as UAGC's enterprise Consent Management Platform (CMP), providing comprehensive cookie consent management, privacy preference centers, and regulatory compliance automation. This integration ensures GDPR, CCPA, and multi-jurisdictional compliance while maintaining seamless user experience and marketing effectiveness.

## OneTrust Platform Architecture

### Enterprise Platform Components
```yaml
OneTrust Platform Stack:
  Consent Management:
    - Cookie Consent Banner
    - Preference Center
    - Consent Receipt Management
    - Cross-Domain Consent Sync
  
  Privacy Management:
    - Data Subject Rights Automation
    - Privacy Impact Assessments
    - Vendor Risk Management
    - Data Mapping and Inventory
  
  Compliance Automation:
    - Regulatory Framework Updates
    - Policy Template Management
    - Audit Trail and Reporting
    - Breach Response Coordination
```

### UAGC OneTrust Configuration
```typescript
interface UAGCOneTrustConfig {
  organizationId: string;
  environment: 'production' | 'staging' | 'development';
  domains: {
    primary: 'uagc.edu';
    subdomains: ['www.uagc.edu', 'portal.uagc.edu', 'cloud.mail.uagc.edu'];
    crossDomainSync: boolean;
  };
  
  consentModel: {
    framework: 'IAB TCF 2.2' | 'Custom';
    granularity: 'category' | 'vendor' | 'purpose';
    defaultConsent: 'opt-in' | 'opt-out';
    jurisdiction: 'GDPR' | 'CCPA' | 'Multi-jurisdictional';
  };
}
```

## Cookie Consent Banner Implementation

### Banner Configuration and Styling
```javascript
// OneTrust Banner Configuration
window.OptanonWrapper = function() {
  // Custom styling and behavior
  const bannerConfig = {
    layout: 'banner-bottom',
    position: 'fixed',
    styling: {
      primaryColor: '#003366', // UAGC Navy
      secondaryColor: '#FF6B35', // UAGC Orange
      textColor: '#FFFFFF',
      buttonStyle: 'rounded',
      fontFamily: 'Arial, sans-serif'
    },
    
    content: {
      title: 'Your Privacy Choices',
      description: 'We use cookies to enhance your experience and provide personalized education recommendations.',
      acceptAllText: 'Accept All Cookies',
      rejectAllText: 'Reject Non-Essential',
      settingsText: 'Cookie Settings',
      policyLinkText: 'Privacy Policy'
    },
    
    behavior: {
      autoShow: true,
      showCloseButton: true,
      respectDNT: true,
      consentExpiry: 365, // days
      reConsentFrequency: 365 // days
    }
  };
  
  // Apply custom configuration
  OneTrust.applyConfiguration(bannerConfig);
  
  // Custom event handlers
  OneTrust.onConsentChanged(handleConsentChange);
  OneTrust.onBannerDisplayed(trackBannerImpression);
  OneTrust.onPreferenceCenterDisplayed(trackPreferenceCenterView);
};

function handleConsentChange(consentData) {
  // Update UAGC cookie categories based on consent
  updateCookieCategories(consentData);
  
  // Trigger analytics events
  gtag('event', 'consent_update', {
    consent_categories: consentData.categories,
    consent_timestamp: new Date().toISOString()
  });
  
  // Update personalization engine
  PersonalizationEngine.updateConsentPreferences(consentData);
}
```

### Cookie Categorization Framework
```yaml
OneTrust Cookie Categories:
  Strictly Necessary (C0001):
    description: "Essential for website functionality"
    defaultConsent: true
    userControl: false
    cookies:
      - Session management
      - Security tokens
      - Load balancing
      - Basic functionality
  
  Performance & Analytics (C0002):
    description: "Help us understand website usage"
    defaultConsent: false
    userControl: true
    cookies:
      - Google Analytics
      - Adobe Analytics
      - Hotjar
      - Performance monitoring
  
  Functional & Personalization (C0003):
    description: "Enable personalized content and features"
    defaultConsent: false
    userControl: true
    cookies:
      - Personalization engine
      - Content recommendations
      - User preferences
      - Journey tracking
  
  Targeting & Advertising (C0004):
    description: "Used for targeted advertising"
    defaultConsent: false
    userControl: true
    cookies:
      - Google Ads
      - Facebook Pixel
      - LinkedIn Insight
      - Retargeting pixels
```

## Preference Center Implementation

### Custom Preference Center Design
```html
<!-- OneTrust Preference Center Template -->
<div id="ot-pc-wrapper">
  <div id="ot-pc-content">
    <header class="ot-pc-header">
      <h2>Privacy Preference Center</h2>
      <p>Manage your cookie and privacy preferences for UAGC websites.</p>
    </header>
    
    <section class="ot-pc-categories">
      <div class="ot-category-group" data-category="C0001">
        <div class="ot-category-header">
          <h3>Strictly Necessary Cookies</h3>
          <span class="ot-always-active">Always Active</span>
        </div>
        <div class="ot-category-description">
          <p>These cookies are essential for the website to function properly and cannot be disabled.</p>
          <details class="ot-category-details">
            <summary>View Cookies</summary>
            <div class="ot-cookie-list">
              <!-- Cookie details populated by OneTrust -->
            </div>
          </details>
        </div>
      </div>
      
      <div class="ot-category-group" data-category="C0002">
        <div class="ot-category-header">
          <h3>Performance & Analytics</h3>
          <label class="ot-toggle">
            <input type="checkbox" id="ot-category-C0002">
            <span class="ot-slider"></span>
          </label>
        </div>
        <div class="ot-category-description">
          <p>Help us understand how visitors interact with our website by collecting and reporting information anonymously.</p>
          <details class="ot-category-details">
            <summary>View Cookies</summary>
            <div class="ot-cookie-list">
              <!-- Cookie details populated by OneTrust -->
            </div>
          </details>
        </div>
      </div>
    </section>
    
    <footer class="ot-pc-footer">
      <button id="ot-pc-btn-accept-all">Accept All</button>
      <button id="ot-pc-btn-reject-all">Reject All</button>
      <button id="ot-pc-btn-save">Save Preferences</button>
    </footer>
  </div>
</div>
```

### Preference Center Styling
```css
/* OneTrust Preference Center Custom Styles */
#ot-pc-wrapper {
  --ot-primary-color: #003366;
  --ot-secondary-color: #FF6B35;
  --ot-text-color: #333333;
  --ot-background-color: #FFFFFF;
  --ot-border-color: #E0E0E0;
  
  font-family: 'Arial', sans-serif;
  color: var(--ot-text-color);
}

.ot-pc-header {
  background: linear-gradient(135deg, var(--ot-primary-color), #004080);
  color: white;
  padding: 2rem;
  text-align: center;
}

.ot-category-group {
  border: 1px solid var(--ot-border-color);
  border-radius: 8px;
  margin: 1rem 0;
  padding: 1.5rem;
  transition: box-shadow 0.3s ease;
}

.ot-category-group:hover {
  box-shadow: 0 4px 12px rgba(0, 51, 102, 0.1);
}

.ot-toggle input[type="checkbox"] {
  appearance: none;
  width: 50px;
  height: 26px;
  background: #ccc;
  border-radius: 13px;
  position: relative;
  cursor: pointer;
  transition: background 0.3s ease;
}

.ot-toggle input[type="checkbox"]:checked {
  background: var(--ot-primary-color);
}

.ot-toggle input[type="checkbox"]::before {
  content: '';
  position: absolute;
  width: 22px;
  height: 22px;
  background: white;
  border-radius: 50%;
  top: 2px;
  left: 2px;
  transition: transform 0.3s ease;
}

.ot-toggle input[type="checkbox"]:checked::before {
  transform: translateX(24px);
}
```

## Data Subject Rights Automation

### Rights Request Processing
```typescript
class OneTrustDataRightsManager {
  private apiEndpoint = 'https://app.onetrust.com/api/datasubject/v2';
  private apiKey = process.env.ONETRUST_API_KEY;
  
  async submitRightsRequest(requestData: DataRightsRequest): Promise<RequestResponse> {
    const request = {
      type: requestData.type, // 'access', 'delete', 'portability', 'rectification'
      subject: {
        email: requestData.email,
        firstName: requestData.firstName,
        lastName: requestData.lastName,
        country: requestData.country
      },
      customFields: {
        studentId: requestData.studentId,
        enrollmentStatus: requestData.enrollmentStatus,
        dataCategories: requestData.dataCategories
      }
    };
    
    try {
      const response = await fetch(`${this.apiEndpoint}/requests`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(request)
      });
      
      if (!response.ok) {
        throw new Error(`OneTrust API error: ${response.statusText}`);
      }
      
      const result = await response.json();
      
      // Log request for audit trail
      await this.logRightsRequest(request, result);
      
      return result;
    } catch (error) {
      console.error('Data rights request failed:', error);
      throw error;
    }
  }
  
  async getRequestStatus(requestId: string): Promise<RequestStatus> {
    const response = await fetch(`${this.apiEndpoint}/requests/${requestId}`, {
      headers: {
        'Authorization': `Bearer ${this.apiKey}`
      }
    });
    
    return await response.json();
  }
  
  async fulfillDataAccess(requestId: string): Promise<DataExport> {
    // Collect data from all UAGC systems
    const userData = await this.collectUserData(requestId);
    
    // Generate structured data export
    const dataExport = {
      requestId,
      timestamp: new Date().toISOString(),
      dataCategories: {
        personalInfo: userData.personal,
        educationalRecords: userData.educational,
        marketingData: userData.marketing,
        technicalData: userData.technical
      },
      format: 'JSON',
      encryption: true
    };
    
    // Upload to OneTrust secure portal
    await this.uploadDataExport(dataExport);
    
    return dataExport;
  }
}
```

### Automated Data Discovery
```yaml
OneTrust Data Discovery Configuration:
  Data Sources:
    Primary Systems:
      - Salesforce CRM
      - Student Information System
      - Learning Management System
      - Marketing Automation Platform
    
    Secondary Systems:
      - Google Analytics
      - Adobe Analytics
      - Email Marketing Platform
      - Customer Support System
    
    Technical Systems:
      - Web Analytics Cookies
      - Personalization Engine
      - A/B Testing Platform
      - Chat and Support Tools
  
  Discovery Rules:
    Personal Identifiers:
      - Email addresses
      - Student ID numbers
      - Phone numbers
      - IP addresses
    
    Educational Data:
      - Enrollment records
      - Academic transcripts
      - Course progress
      - Degree information
    
    Marketing Data:
      - Campaign interactions
      - Website behavior
      - Email engagement
      - Lead scoring data
```

## Cross-Domain Consent Synchronization

### Multi-Domain Consent Framework
```javascript
class OneTrustCrossDomainSync {
  constructor() {
    this.domains = [
      'uagc.edu',
      'www.uagc.edu',
      'portal.uagc.edu',
      'cloud.mail.uagc.edu'
    ];
    this.syncEndpoint = 'https://optanon.blob.core.windows.net/consent';
  }
  
  async syncConsentAcrossDomains(consentData) {
    const syncPromises = this.domains.map(domain => 
      this.syncConsentToDomain(domain, consentData)
    );
    
    try {
      await Promise.all(syncPromises);
      console.log('Consent synchronized across all domains');
    } catch (error) {
      console.error('Cross-domain consent sync failed:', error);
      // Implement fallback mechanism
      await this.fallbackConsentSync(consentData);
    }
  }
  
  async syncConsentToDomain(domain, consentData) {
    return new Promise((resolve, reject) => {
      const iframe = document.createElement('iframe');
      iframe.style.display = 'none';
      iframe.src = `https://${domain}/consent-sync.html`;
      
      iframe.onload = () => {
        iframe.contentWindow.postMessage({
          type: 'CONSENT_SYNC',
          consent: consentData,
          timestamp: Date.now()
        }, `https://${domain}`);
        
        // Listen for confirmation
        const messageHandler = (event) => {
          if (event.origin === `https://${domain}` && 
              event.data.type === 'CONSENT_SYNC_COMPLETE') {
            window.removeEventListener('message', messageHandler);
            document.body.removeChild(iframe);
            resolve();
          }
        };
        
        window.addEventListener('message', messageHandler);
        
        // Timeout after 5 seconds
        setTimeout(() => {
          window.removeEventListener('message', messageHandler);
          document.body.removeChild(iframe);
          reject(new Error(`Sync timeout for ${domain}`));
        }, 5000);
      };
      
      document.body.appendChild(iframe);
    });
  }
}
```

## Compliance Reporting and Analytics

### OneTrust Dashboard Integration
```typescript
interface OneTrustReporting {
  consentMetrics: {
    totalBannerImpressions: number;
    consentRate: number;
    categoryAcceptanceRates: {
      analytics: number;
      personalization: number;
      advertising: number;
    };
    geographicBreakdown: Record<string, number>;
  };
  
  rightsRequests: {
    totalRequests: number;
    requestTypes: {
      access: number;
      delete: number;
      portability: number;
      rectification: number;
    };
    averageResponseTime: number;
    completionRate: number;
  };
  
  complianceStatus: {
    gdprCompliance: boolean;
    ccpaCompliance: boolean;
    ferpaCompliance: boolean;
    lastAuditDate: string;
    nextAuditDue: string;
  };
}

class OneTrustAnalytics {
  async generateComplianceReport(): Promise<OneTrustReporting> {
    const [consentData, rightsData, complianceData] = await Promise.all([
      this.getConsentMetrics(),
      this.getRightsRequestMetrics(),
      this.getComplianceStatus()
    ]);
    
    return {
      consentMetrics: consentData,
      rightsRequests: rightsData,
      complianceStatus: complianceData
    };
  }
  
  async exportComplianceData(format: 'json' | 'csv' | 'pdf'): Promise<Blob> {
    const reportData = await this.generateComplianceReport();
    
    switch (format) {
      case 'json':
        return new Blob([JSON.stringify(reportData, null, 2)], 
          { type: 'application/json' });
      case 'csv':
        return this.convertToCSV(reportData);
      case 'pdf':
        return await this.generatePDFReport(reportData);
    }
  }
}
```

## Integration with UAGC Systems

### Salesforce CRM Integration
```javascript
// OneTrust + Salesforce Integration
class OneTrustSalesforceSync {
  async syncConsentToSalesforce(contactData, consentPreferences) {
    const salesforceUpdate = {
      Id: contactData.salesforceId,
      OneTrust_Consent_Analytics__c: consentPreferences.analytics,
      OneTrust_Consent_Personalization__c: consentPreferences.personalization,
      OneTrust_Consent_Marketing__c: consentPreferences.marketing,
      OneTrust_Consent_Date__c: new Date().toISOString(),
      OneTrust_Consent_Source__c: 'Website Banner'
    };
    
    try {
      await salesforceAPI.updateContact(salesforceUpdate);
      console.log('Consent preferences synced to Salesforce');
    } catch (error) {
      console.error('Salesforce sync failed:', error);
      // Queue for retry
      await this.queueForRetry(salesforceUpdate);
    }
  }
}
```

### Google Analytics 4 Integration
```javascript
// OneTrust + GA4 Consent Integration
function updateGA4Consent(consentData) {
  gtag('consent', 'update', {
    'analytics_storage': consentData.analytics ? 'granted' : 'denied',
    'ad_storage': consentData.advertising ? 'granted' : 'denied',
    'ad_user_data': consentData.advertising ? 'granted' : 'denied',
    'ad_personalization': consentData.advertising ? 'granted' : 'denied',
    'functionality_storage': consentData.personalization ? 'granted' : 'denied',
    'personalization_storage': consentData.personalization ? 'granted' : 'denied'
  });
  
  // Send consent update event
  gtag('event', 'consent_update', {
    'consent_categories': Object.keys(consentData).filter(key => consentData[key]),
    'consent_method': 'onetrust_banner'
  });
}
```

## Security and Data Protection

### OneTrust Security Configuration
```yaml
Security Framework:
  Data Encryption:
    - TLS 1.3 for data in transit
    - AES-256 for data at rest
    - End-to-end encryption for sensitive data
  
  Access Controls:
    - Role-based access control (RBAC)
    - Multi-factor authentication
    - API key rotation every 90 days
    - Audit logging for all access
  
  Data Retention:
    - Consent records: 7 years post-withdrawal
    - Rights request records: 10 years
    - Audit logs: 7 years
    - Temporary processing data: 30 days
  
  Compliance Monitoring:
    - Real-time compliance dashboard
    - Automated compliance checks
    - Regular security assessments
    - Incident response procedures
```

## Troubleshooting and Support

### Common Integration Issues
```yaml
Common Issues and Solutions:
  Banner Not Displaying:
    Causes:
      - Script loading order
      - Domain configuration
      - Cache issues
    Solutions:
      - Verify script placement in <head>
      - Check domain whitelist in OneTrust
      - Clear CDN cache
  
  Consent Not Persisting:
    Causes:
      - Cookie domain mismatch
      - Third-party cookie blocking
      - Cross-domain sync failure
    Solutions:
      - Verify cookie domain configuration
      - Implement first-party cookie fallback
      - Check cross-domain sync logs
  
  API Authentication Failures:
    Causes:
      - Expired API keys
      - Incorrect endpoint URLs
      - Rate limiting
    Solutions:
      - Rotate API keys
      - Verify endpoint configuration
      - Implement exponential backoff
```

## Cross-References

- **Legal Requirements**: See [Legal Requirements](./legal-requirements.md)
- **Cookie Architecture**: See [Cookie Architecture](../03-technical-implementation/cookie-architecture.md)
- **Compliance Procedures**: See [Compliance Procedures](./compliance-procedures.md)
- **User Rights Management**: See [User Rights Management](./user-rights-management.md)
- **Technical Implementation**: See [Section 3: Technical Implementation](../03-technical-implementation/overview.md)

---

*This document is maintained by the UAGC Digital Privacy team. For OneTrust platform support, contact the OneTrust Customer Success team.* 