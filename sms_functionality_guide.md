# SMS Functionality in CanIGo Leave Management System

## Overview

The CanIGo system uses **Twilio SMS API** to send automated notifications to parents when students submit leave applications. This ensures parents are informed about their child's leave requests in real-time.

## How SMS Works

### 1. **Technology Stack**
- **Twilio SDK**: PHP library for sending SMS messages
- **Composer**: Dependency management for Twilio SDK
- **REST API**: Communication with Twilio's servers

### 2. **SMS Trigger Points**

#### **Student Leave Application**
- **Location**: `student/leave_form.php`
- **Trigger**: When student submits a leave request
- **Recipient**: Parent's mobile number
- **Content**: Leave details in Marathi language

### 3. **SMS Implementation Details**

#### **Twilio Configuration**
```php
// Twilio Account Credentials (from leave_form.php)
$sid = "AC2e9425daadf393c37c936f1f6c42a94c";  // Account SID
$token = "2fa61f577081c8f2efaf1b3546e4b664";  // Auth Token
$from_number = '+19377876028';  // Twilio phone number
```

#### **SMS Sending Process**
```php
// 1. Include Twilio SDK
require __DIR__ . '/../twilio-php-app/vendor/autoload.php';

// 2. Create Twilio Client
$client = new Twilio\Rest\Client($sid, $token);

// 3. Send SMS
$client->messages->create(
    "+91" . $parent_mobile,  // Recipient number
    [
        'from' => $from_number,
        'body' => $message_content
    ]
);
```

### 4. **Message Content**

#### **Marathi SMS Content** (Primary Language)
```
तुमच्या पाल्याने [Student Name] ने खालील तारखांसाठी रजा अर्ज सादर केला आहे: [Start Date] ते [End Date].
रजा घेण्याचे कारण: [Reason]
काही शंका असल्यास संपर्क साधावा.
धन्यवाद,
AITRC VITA
```

#### **English SMS Content** (Alternative)
```
This is a notification to inform you that [Student Name] has submitted a leave request for the following dates: [Start Date] to [End Date].
Please note that the student is requesting leave with the following reason: [Reason]
Feel free to reach out if you have any questions or concerns.
Thank you,
AITRC VITA
```

### 5. **Data Flow for SMS**

#### **Step 1: Student Login**
- Student logs in with username/password
- Session variables are set in `student/home.php`
- Parent contact number stored in `$_SESSION['parent_mo']`

#### **Step 2: Leave Form Submission**
- Student fills leave form with dates and reason
- Form data processed in `student/leave_form.php`
- Leave record inserted into database

#### **Step 3: SMS Trigger**
- After successful database insertion
- Twilio SDK loads and initializes
- SMS sent to parent's mobile number
- Success confirmation shown to student

### 6. **Database Integration**

#### **Parent Contact Storage**
- Stored in `students` table as `ParentContactNo`
- Retrieved during student login session
- Used for SMS notifications

#### **Leave Record Creation**
```sql
INSERT INTO leaves (
    StudentID, StartDate, EndDate, Reason,
    TeacherApprovalStatus, HODApprovalStatus, ByGuard
) VALUES (
    ?, ?, ?, ?,
    'Pending', 'Pending', 'Pending'
);
```

### 7. **SMS Limitations**

#### **Current Implementation Issues**
- **No SMS for approvals**: Only sent when leave is applied, not when approved/rejected
- **Single language**: Only Marathi SMS (English version exists but not used)
- **No delivery confirmation**: No tracking of SMS delivery status
- **Hardcoded credentials**: Twilio credentials are in source code

#### **Missing SMS Notifications**
- **HOD Approval**: No SMS when HOD approves/rejects leave
- **Teacher Approval**: No SMS when teacher approves/rejects leave
- **Guard Status**: No SMS when student checks in/out
- **Complaint Resolution**: No SMS when complaints are resolved

### 8. **Twilio Setup Requirements**

#### **Account Requirements**
- Active Twilio account
- Purchased phone number for SMS
- Sufficient SMS credits/balance
- Verified phone numbers (for testing)

#### **Technical Requirements**
- PHP 7.0+ with Composer
- Twilio PHP SDK installed
- Internet connection for API calls
- Valid Indian mobile numbers (+91 prefix)

### 9. **SMS Cost Structure**

#### **Twilio Pricing** (Approximate)
- **SMS to India**: ~$0.02 - $0.05 per message
- **Setup fee**: One-time account verification
- **Monthly fees**: Phone number rental (~$1/month)

### 10. **Security Considerations**

#### **Current Security Issues**
- **Exposed credentials**: SID and token in source code
- **No encryption**: SMS content sent in plain text
- **No rate limiting**: Potential for SMS spam
- **No authentication**: Anyone can trigger SMS if they access the code

#### **Recommended Improvements**
- Move credentials to environment variables
- Implement SMS rate limiting
- Add SMS delivery tracking
- Use HTTPS for all communications

### 11. **Testing SMS Functionality**

#### **Test Environment Setup**
1. Get Twilio test credentials
2. Use test phone numbers
3. Monitor Twilio dashboard for delivery
4. Check SMS logs and delivery status

#### **Production Deployment**
1. Purchase dedicated phone number
2. Add SMS credits to account
3. Update credentials in production
4. Test with real Indian numbers

### 12. **Error Handling**

#### **Common SMS Errors**
- **Invalid phone number**: Wrong format or non-existent number
- **Insufficient balance**: No SMS credits in Twilio account
- **Rate limiting**: Too many SMS sent in short time
- **Network issues**: Connectivity problems with Twilio API

#### **Error Responses**
```php
try {
    $message = $client->messages->create($to, $options);
    echo "SMS sent successfully. SID: " . $message->sid;
} catch (Exception $e) {
    echo "SMS failed: " . $e->getMessage();
}
```

### 13. **Future Enhancements**

#### **Recommended SMS Features**
- **Multi-language support**: English, Hindi, Marathi
- **Approval notifications**: SMS when leave is approved/rejected
- **Status updates**: SMS for each approval stage
- **Emergency alerts**: SMS for urgent situations
- **Bulk notifications**: SMS to multiple recipients
- **Delivery receipts**: Confirmation of SMS delivery

#### **Advanced Features**
- **SMS templates**: Pre-defined message formats
- **Scheduled SMS**: Time-based notifications
- **SMS analytics**: Delivery statistics and reports
- **Two-way SMS**: Allow parents to reply
- **SMS preferences**: Allow users to opt-in/opt-out

## Summary

The CanIGo system currently implements **basic SMS functionality** using Twilio API to notify parents when students submit leave applications. The system sends SMS in Marathi language containing leave details. However, it lacks comprehensive SMS notifications for the complete leave approval workflow and has some security and scalability limitations that should be addressed for production deployment.

**Current SMS Coverage**: Only leave application notifications
**Missing SMS Coverage**: Approval notifications, status updates, complaint resolutions
**Technology**: Twilio PHP SDK with REST API
**Cost**: ~$0.02-0.05 per SMS to India
**Languages**: Primarily Marathi (English available but unused)