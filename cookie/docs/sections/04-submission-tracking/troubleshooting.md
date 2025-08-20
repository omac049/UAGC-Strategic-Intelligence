# Section 4: RFI & Application Submission Tracking - Troubleshooting Guide

## Common Issues and Solutions

This troubleshooting guide addresses the most frequently encountered issues in the UAGC submission tracking system, providing step-by-step solutions and preventive measures.

## Form Submission Issues

### Issue 1: Form Submissions Not Being Tracked

**Symptoms:**
- Form submissions complete successfully but don't appear in analytics
- No leads created in Salesforce CRM
- Missing submission data in reporting dashboards

**Diagnosis Steps:**
```javascript
// Check if tracking is initialized
console.log('Tracking Status:', window.uagcSubmissionTracker?.isInitialized);

// Verify configuration
console.log('Config:', window.uagcConfig);

// Check event queue
console.log('Event Queue:', window.uagcSubmissionTracker?.eventQueue);
```

**Common Causes & Solutions:**

1. **Tracking Script Not Loaded**
   ```html
   <!-- Ensure tracking script is loaded before form initialization -->
   <script src="/js/uagc-submission-tracker.js"></script>
   <script>
     if (typeof SubmissionTracker === 'undefined') {
       console.error('SubmissionTracker not loaded');
     }
   </script>
   ```

2. **Configuration Missing**
   ```javascript
   // Verify all required configuration is present
   const requiredConfig = [
     'integrations.salesforce.endpoint',
     'integrations.ga4.measurementId',
     'tracking.enabled'
   ];
   
   requiredConfig.forEach(path => {
     const value = path.split('.').reduce((obj, key) => obj?.[key], window.uagcConfig);
     if (!value) {
       console.error(`Missing config: ${path}`);
     }
   });
   ```

### Issue 2: Form Validation Errors

**Symptoms:**
- Forms won't submit despite appearing complete
- Validation messages not displaying correctly
- Fields marked as invalid incorrectly

**Solutions:**

1. **Field Validation Rules**
   ```javascript
   const validationRules = {
     email: {
       required: true,
       pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
       message: 'Please enter a valid email address'
     },
     phone: {
       required: true,
       pattern: /^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$/,
       message: 'Please enter a valid phone number'
     }
   };
   ```

## Integration Issues

### Issue 3: Salesforce Lead Creation Failures

**Symptoms:**
- Form submissions tracked but no leads in Salesforce
- Authentication errors in console
- API timeout errors

**Common Causes & Solutions:**

1. **Authentication Issues**
   ```javascript
   // Check authentication credentials
   const authConfig = window.uagcConfig.integrations.salesforce;
   
   if (!authConfig.clientId || !authConfig.clientSecret) {
     console.error('Missing Salesforce credentials');
   }
   ```

2. **API Rate Limiting**
   ```javascript
   // Implement retry logic with exponential backoff
   class SalesforceLeadService {
     async createLeadWithRetry(submissionData, maxRetries = 3) {
       for (let attempt = 1; attempt <= maxRetries; attempt++) {
         try {
           return await this.createLead(submissionData);
         } catch (error) {
           if (error.message.includes('RATE_LIMIT') && attempt < maxRetries) {
             const delay = Math.pow(2, attempt) * 1000;
             await new Promise(resolve => setTimeout(resolve, delay));
             continue;
           }
           throw error;
         }
       }
     }
   }
   ```

### Issue 4: Google Analytics 4 Tracking Problems

**Symptoms:**
- Conversion events not appearing in GA4
- Enhanced conversions not working
- Custom parameters not being captured

**Solutions:**

1. **Measurement ID Verification**
   ```javascript
   // Verify measurement ID format
   const measurementId = window.uagcConfig.integrations.ga4.measurementId;
   
   if (!/^G-[A-Z0-9]{10}$/.test(measurementId)) {
     console.error('Invalid GA4 Measurement ID format:', measurementId);
   }
   ```

2. **Debug Mode**
   ```javascript
   // Enable GA4 debug mode
   gtag('config', 'GA_MEASUREMENT_ID', {
     debug_mode: true
   });
   ```

## Performance Issues

### Issue 5: Slow Form Submission Processing

**Symptoms:**
- Long delays between form submission and confirmation
- Timeout errors during submission
- Poor user experience with loading states

**Solutions:**

1. **Asynchronous Processing**
   ```javascript
   // Process integrations in parallel
   async processSubmission(submissionData) {
     const startTime = Date.now();
     
     try {
       const integrationPromises = [
         this.salesforceService.createLead(submissionData),
         this.ga4Service.trackSubmission(submissionData),
         this.marketingAutomationService.triggerWorkflow(submissionData)
       ];
       
       const results = await Promise.allSettled(integrationPromises);
       
       results.forEach((result, index) => {
         if (result.status === 'rejected') {
           console.error(`Integration ${index} failed:`, result.reason);
         }
       });
       
       return submissionData.submissionId;
     } finally {
       const endTime = Date.now();
       this.performanceMonitor.trackSubmissionPerformance(
         submissionData.submissionId,
         startTime,
         endTime,
         this.getIntegrationLatencies()
       );
     }
   }
   ```

2. **Queue Management**
   ```javascript
   // Implement submission queue with batch processing
   class SubmissionQueue {
     constructor(batchSize = 10, flushInterval = 5000) {
       this.queue = [];
       this.batchSize = batchSize;
       this.flushInterval = flushInterval;
       this.startProcessor();
     }
     
     add(submission) {
       this.queue.push(submission);
       
       if (this.queue.length >= this.batchSize) {
         this.flush();
       }
     }
     
     async flush() {
       if (this.queue.length === 0) return;
       
       const batch = this.queue.splice(0, this.batchSize);
       
       try {
         await this.processBatch(batch);
       } catch (error) {
         console.error('Batch processing error:', error);
         this.queue.unshift(...batch);
       }
     }
     
     startProcessor() {
       setInterval(() => {
         this.flush();
       }, this.flushInterval);
     }
   }
   ```

## Data Quality Issues

### Issue 6: Incomplete or Corrupted Submission Data

**Symptoms:**
- Missing form fields in CRM
- Corrupted special characters
- Attribution data not captured

**Solutions:**

1. **Data Sanitization**
   ```javascript
   // Sanitize form data before processing
   function sanitizeFormData(formData) {
     const sanitized = {};
     
     Object.entries(formData).forEach(([key, value]) => {
       if (typeof value === 'string') {
         sanitized[key] = value
           .replace(/[<>]/g, '')
           .replace(/[^\w\s@.-]/g, '')
           .trim();
       } else {
         sanitized[key] = value;
       }
     });
     
     return sanitized;
   }
   ```

2. **Data Validation Pipeline**
   ```javascript
   // Implement comprehensive data validation
   class DataValidator {
     static validateSubmission(submissionData) {
       const errors = [];
       
       if (!this.validateRequiredFields(submissionData)) {
         errors.push('Missing required fields');
       }
       
       if (!this.validateEmail(submissionData.formData?.email)) {
         errors.push('Invalid email format');
       }
       
       if (!this.validatePhone(submissionData.formData?.phone)) {
         errors.push('Invalid phone format');
       }
       
       return {
         isValid: errors.length === 0,
         errors
       };
     }
   }
   ```

## Monitoring and Alerting

### Setting Up Comprehensive Monitoring

```javascript
// Implement comprehensive monitoring
class SubmissionMonitor {
  constructor() {
    this.metrics = {
      totalSubmissions: 0,
      successfulSubmissions: 0,
      failedSubmissions: 0,
      averageProcessingTime: 0,
      integrationFailures: {}
    };
    
    this.alerts = {
      highErrorRate: 0.05,
      slowProcessing: 10000,
      integrationFailure: 3
    };
  }
  
  trackSubmission(submissionData, processingTime, integrationResults) {
    this.metrics.totalSubmissions++;
    
    if (integrationResults.every(result => result.success)) {
      this.metrics.successfulSubmissions++;
    } else {
      this.metrics.failedSubmissions++;
      this.checkErrorRateAlert();
    }
    
    this.updateAverageProcessingTime(processingTime);
    
    if (processingTime > this.alerts.slowProcessing) {
      this.triggerAlert('SLOW_PROCESSING', { processingTime, submissionData });
    }
    
    this.trackIntegrationFailures(integrationResults);
  }
  
  checkErrorRateAlert() {
    const errorRate = this.metrics.failedSubmissions / this.metrics.totalSubmissions;
    
    if (errorRate > this.alerts.highErrorRate) {
      this.triggerAlert('HIGH_ERROR_RATE', { 
        errorRate: errorRate * 100,
        threshold: this.alerts.highErrorRate * 100
      });
    }
  }
  
  triggerAlert(type, data) {
    console.error(`ALERT: ${type}`, data);
    
    fetch('/api/alerts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        type,
        data,
        timestamp: new Date().toISOString()
      })
    });
  }
}
```

## Quick Reference

### Common Console Commands
```javascript
// Check tracking status
window.uagcSubmissionTracker.getStatus();

// View recent submissions
window.uagcSubmissionTracker.getRecentSubmissions();

// Test Salesforce connection
window.uagcSubmissionTracker.testSalesforceConnection();

// Test GA4 connection
window.uagcSubmissionTracker.testGA4Connection();

// Clear event queue
window.uagcSubmissionTracker.clearEventQueue();

// Enable debug mode
window.uagcConfig.tracking.debug = true;
```

### Error Code Reference
- **ERR_TRACKING_001**: Tracking not initialized
- **ERR_FORM_002**: Form validation failed
- **ERR_SF_003**: Salesforce authentication failed
- **ERR_SF_004**: Salesforce API rate limit exceeded
- **ERR_GA4_005**: GA4 measurement ID invalid
- **ERR_DATA_006**: Required fields missing
- **ERR_PERF_007**: Processing timeout exceeded

This troubleshooting guide provides comprehensive solutions for maintaining reliable submission tracking operations. 