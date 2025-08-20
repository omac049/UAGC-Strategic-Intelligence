# Success Metrics Troubleshooting Guide

## Overview

This troubleshooting guide provides solutions for common issues encountered with the UAGC success metrics tracking and measurement system. Use this guide to quickly diagnose and resolve problems with analytics tracking, conversion measurement, and reporting.

## Quick Diagnostic Commands

### Analytics Data Verification
```javascript
// Check GA4 data layer
console.log('dataLayer:', window.dataLayer);

// Verify GTM container
console.log('GTM loaded:', typeof window.google_tag_manager !== 'undefined');

// Check conversion tracking
gtag('event', 'test_conversion', {
  debug_mode: true,
  event_category: 'Testing',
  event_label: 'Troubleshooting'
});
```

### Salesforce Integration Check
```apex
// Query recent leads for verification
SELECT Id, Email, UTM_Source__c, Lead_Score__c, CreatedDate 
FROM Lead 
WHERE CreatedDate = TODAY 
ORDER BY CreatedDate DESC 
LIMIT 10
```

## Common Issues and Solutions

### 1. GA4 Conversion Tracking Issues

#### Problem: Conversions Not Recording
**Symptoms:**
- Zero conversions in GA4 reports
- Missing conversion events in real-time reports
- Form submissions not triggering events

**Diagnostic Steps:**
```javascript
// Enable debug mode
gtag('config', 'GA_MEASUREMENT_ID', {
  debug_mode: true
});

// Check if events are firing
gtag('event', 'debug_test', {
  event_category: 'Debug',
  event_label: 'Manual Test',
  debug_mode: true
});
```

**Solutions:**
1. **Verify GTM Configuration**
   ```javascript
   // Check GTM container status
   if (typeof window.google_tag_manager === 'undefined') {
     console.error('GTM not loaded');
   } else {
     console.log('GTM loaded successfully');
   }
   ```

2. **Fix Event Firing**
   ```javascript
   // Ensure proper event structure
   function trackConversion(conversionType, value) {
     gtag('event', 'conversion', {
       send_to: `GA_MEASUREMENT_ID/${conversionType}`,
       value: value,
       currency: 'USD',
       event_category: 'Lead Generation',
       event_label: conversionType
     });
   }
   ```

3. **Validate Measurement ID**
   ```javascript
   // Check measurement ID format
   const measurementId = 'G-XXXXXXXXXX';
   if (!measurementId.match(/^G-[A-Z0-9]{10}$/)) {
     console.error('Invalid measurement ID format');
   }
   ```

#### Problem: Duplicate Conversions
**Symptoms:**
- Inflated conversion numbers
- Multiple events for single actions
- Inconsistent conversion data

**Solutions:**
1. **Implement Conversion Deduplication**
   ```javascript
   const conversionTracker = {
     trackedConversions: new Set(),
     
     trackConversion(type, identifier) {
       const conversionKey = `${type}_${identifier}`;
       
       if (this.trackedConversions.has(conversionKey)) {
         console.log('Conversion already tracked, skipping');
         return false;
       }
       
       this.trackedConversions.add(conversionKey);
       
       gtag('event', 'conversion', {
         send_to: `GA_MEASUREMENT_ID/${type}`,
         transaction_id: identifier
       });
       
       return true;
     }
   };
   ```

### 2. Salesforce Integration Problems

#### Problem: Leads Not Creating
**Symptoms:**
- Form submissions not creating Salesforce leads
- API errors in browser console
- Missing lead data in Salesforce

**Diagnostic Steps:**
```javascript
// Test Salesforce API connection
async function testSalesforceConnection() {
  try {
    const response = await fetch('/api/salesforce/test', {
      method: 'GET',
      headers: {
        'Authorization': 'Bearer ' + salesforceToken
      }
    });
    
    console.log('Salesforce connection:', response.status);
    return response.ok;
  } catch (error) {
    console.error('Salesforce connection failed:', error);
    return false;
  }
}
```

**Solutions:**
1. **Verify API Credentials**
   ```javascript
   // Check token expiration
   function checkSalesforceToken() {
     const token = localStorage.getItem('sf_token');
     const expiry = localStorage.getItem('sf_token_expiry');
     
     if (!token || Date.now() > parseInt(expiry)) {
       console.error('Salesforce token expired');
       refreshSalesforceToken();
     }
   }
   ```

2. **Fix Lead Creation Payload**
   ```javascript
   function createSalesforceLead(formData) {
     const leadPayload = {
       FirstName: formData.firstName,
       LastName: formData.lastName,
       Email: formData.email,
       Phone: formData.phone,
       Company: formData.company || 'Unknown',
       UTM_Source__c: getUTMParameter('utm_source'),
       UTM_Medium__c: getUTMParameter('utm_medium'),
       UTM_Campaign__c: getUTMParameter('utm_campaign'),
       Program_Interest__c: formData.program,
       Lead_Source: 'Website'
     };
     
     // Validate required fields
     if (!leadPayload.FirstName || !leadPayload.LastName || !leadPayload.Email) {
       throw new Error('Missing required lead fields');
     }
     
     return submitToSalesforce(leadPayload);
   }
   ```

#### Problem: Lead Scoring Issues
**Symptoms:**
- Incorrect lead scores
- Missing lead grades
- Scoring triggers not firing

**Solutions:**
1. **Debug Scoring Logic**
   ```apex
   // Add debug statements to scoring trigger
   trigger LeadScoringTrigger on Lead (before insert, before update) {
       for (Lead lead : Trigger.new) {
           System.debug('Processing lead: ' + lead.Email);
           
           Integer score = 0;
           
           // Debug each scoring component
           if (lead.Education_Level__c == 'Bachelor\'s Degree') {
               score += 15;
               System.debug('Education score added: 15');
           }
           
           System.debug('Final score: ' + score);
           lead.Lead_Score__c = score;
       }
   }
   ```

### 3. Attribution Tracking Problems

#### Problem: UTM Parameters Not Captured
**Symptoms:**
- Missing attribution data
- All traffic showing as direct
- UTM parameters not in Salesforce

**Diagnostic Steps:**
```javascript
// Check UTM parameter capture
function debugUTMCapture() {
  const urlParams = new URLSearchParams(window.location.search);
  const utmParams = {};
  
  ['utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content'].forEach(param => {
    utmParams[param] = urlParams.get(param);
  });
  
  console.log('UTM Parameters:', utmParams);
  console.log('Stored UTMs:', JSON.parse(localStorage.getItem('utm_data') || '{}'));
}
```

**Solutions:**
1. **Fix UTM Parameter Storage**
   ```javascript
   function captureUTMParameters() {
     const urlParams = new URLSearchParams(window.location.search);
     const utmData = {};
     
     ['utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content'].forEach(param => {
       const value = urlParams.get(param);
       if (value) {
         utmData[param] = value;
       }
     });
     
     // Store with expiration
     const expirationTime = Date.now() + (30 * 24 * 60 * 60 * 1000); // 30 days
     localStorage.setItem('utm_data', JSON.stringify(utmData));
     localStorage.setItem('utm_expiry', expirationTime.toString());
     
     console.log('UTM data captured:', utmData);
   }
   ```

### 4. Cost Per Acquisition (CPA) Issues

#### Problem: Cost Data Not Syncing
**Symptoms:**
- Missing cost information
- Incorrect CPA calculations
- API connection failures

**Diagnostic Steps:**
```javascript
// Test ad platform API connections
async function testAdPlatformAPIs() {
  const platforms = ['google_ads', 'facebook_ads', 'linkedin_ads'];
  
  for (const platform of platforms) {
    try {
      const response = await fetch(`/api/${platform}/test`);
      console.log(`${platform} API:`, response.status === 200 ? 'OK' : 'Failed');
    } catch (error) {
      console.error(`${platform} API error:`, error);
    }
  }
}
```

**Solutions:**
1. **Fix API Authentication**
   ```javascript
   class AdPlatformManager {
     constructor() {
       this.tokens = {
         google_ads: process.env.GOOGLE_ADS_TOKEN,
         facebook_ads: process.env.FACEBOOK_ADS_TOKEN,
         linkedin_ads: process.env.LINKEDIN_ADS_TOKEN
       };
     }
     
     async refreshToken(platform) {
       try {
         const response = await fetch(`/api/${platform}/refresh-token`, {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' }
         });
         
         if (response.ok) {
           const data = await response.json();
           this.tokens[platform] = data.access_token;
           console.log(`${platform} token refreshed`);
         }
       } catch (error) {
         console.error(`Failed to refresh ${platform} token:`, error);
       }
     }
   }
   ```

### 5. Dashboard and Reporting Issues

#### Problem: Data Not Appearing in Dashboards
**Symptoms:**
- Empty dashboard widgets
- Stale data in reports
- Connection errors

**Diagnostic Steps:**
```python
# Python script to test data connections
import requests
import json

def test_data_sources():
    sources = {
        'ga4': 'https://analyticsreporting.googleapis.com/v4/reports:batchGet',
        'salesforce': 'https://your-instance.salesforce.com/services/data/v52.0/',
        'marketing_apis': 'https://api.marketing-platform.com/v1/test'
    }
    
    for source, url in sources.items():
        try:
            response = requests.get(url, timeout=10)
            print(f"{source}: {'OK' if response.status_code == 200 else 'Failed'}")
        except Exception as e:
            print(f"{source}: Error - {e}")
```

**Solutions:**
1. **Fix Data Source Connections**
   ```python
   # Automated data refresh script
   import schedule
   import time
   
   def refresh_dashboard_data():
       try:
           # Refresh GA4 data
           ga4_data = fetch_ga4_metrics()
           
           # Refresh Salesforce data
           sf_data = fetch_salesforce_metrics()
           
           # Update dashboard
           update_dashboard(ga4_data, sf_data)
           
           print("Dashboard data refreshed successfully")
       except Exception as e:
           print(f"Dashboard refresh failed: {e}")
           send_alert_email(str(e))
   
   # Schedule refresh every hour
   schedule.every().hour.do(refresh_dashboard_data)
   ```

## Performance Optimization

### 1. Slow Loading Analytics
**Solutions:**
```javascript
// Lazy load analytics for better performance
function loadAnalyticsAsync() {
  const script = document.createElement('script');
  script.async = true;
  script.src = 'https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID';
  document.head.appendChild(script);
  
  script.onload = function() {
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'GA_MEASUREMENT_ID');
  };
}

// Load after page load
window.addEventListener('load', loadAnalyticsAsync);
```

### 2. Large Data Volume Issues
**Solutions:**
```python
# Implement data pagination for large datasets
def fetch_large_dataset(start_date, end_date, page_size=1000):
    all_data = []
    offset = 0
    
    while True:
        batch = fetch_data_batch(start_date, end_date, offset, page_size)
        
        if not batch:
            break
            
        all_data.extend(batch)
        offset += page_size
        
        # Rate limiting
        time.sleep(0.1)
    
    return all_data
```

## Monitoring and Alerts

### Set Up Automated Monitoring
```python
# Monitoring script for critical metrics
import smtplib
from email.mime.text import MIMEText

def check_system_health():
    issues = []
    
    # Check conversion tracking
    if not verify_conversion_tracking():
        issues.append("Conversion tracking not working")
    
    # Check Salesforce integration
    if not verify_salesforce_connection():
        issues.append("Salesforce integration failed")
    
    # Check data freshness
    if not verify_data_freshness():
        issues.append("Data not updating")
    
    if issues:
        send_alert_email(issues)
    
    return len(issues) == 0

def send_alert_email(issues):
    msg = MIMEText(f"System Issues Detected:\n" + "\n".join(issues))
    msg['Subject'] = 'UAGC Analytics Alert'
    msg['From'] = 'alerts@uagc.edu'
    msg['To'] = 'analytics-team@uagc.edu'
    
    # Send email
    server = smtplib.SMTP('smtp.uagc.edu')
    server.send_message(msg)
    server.quit()
```

## Emergency Procedures

### 1. Complete System Failure
1. **Immediate Actions:**
   - Switch to backup tracking (basic GA4)
   - Notify stakeholders
   - Document the issue

2. **Recovery Steps:**
   ```bash
   # Restart analytics services
   sudo systemctl restart analytics-service
   
   # Check service status
   sudo systemctl status analytics-service
   
   # Review logs
   sudo journalctl -u analytics-service -f
   ```

### 2. Data Loss Prevention
```javascript
// Implement local backup for critical events
const eventBackup = {
  queue: [],
  
  addEvent(eventData) {
    this.queue.push({
      ...eventData,
      timestamp: Date.now()
    });
    
    // Try to send immediately
    this.flush();
  },
  
  flush() {
    if (this.queue.length === 0) return;
    
    // Attempt to send queued events
    fetch('/api/events/batch', {
      method: 'POST',
      body: JSON.stringify(this.queue)
    }).then(() => {
      this.queue = [];
    }).catch(() => {
      // Keep events in queue for retry
      console.log('Events queued for retry');
    });
  }
};
```

## Support Contacts

### Internal Teams
- **Analytics Team:** analytics@uagc.edu
- **IT Support:** it-support@uagc.edu
- **Marketing Team:** marketing@uagc.edu

### External Vendors
- **Google Analytics Support:** [GA4 Help Center](https://support.google.com/analytics)
- **Salesforce Support:** [Salesforce Help](https://help.salesforce.com)
- **OneTrust Support:** [OneTrust Documentation](https://my.onetrust.com)

## Related Documentation

- [Success Metrics Overview](./overview.md)
- [Implementation Guide](./implementation-guide.md)
- [Technical Implementation](../03-technical-implementation/overview.md)
- [Cookie Compliance](../05-cookie-compliance/overview.md) 