# Cloud Armor Builder \- User Guide

## Table of Contents

1. [Introduction](#introduction)  
2. [What Cloud Armor Builder Does (and Doesn't Do)](#what-cloud-armor-builder-does-and-doesnt-do)  
3. [Getting Started](#getting-started)  
4. [Step-by-Step Guide](#step-by-step-guide)  
5. [Understanding Security Rules](#understanding-security-rules)  
6. [Automatic Configurations](#automatic-configurations)  
7. [Deploying Your Policy](#deploying-your-policy)  
8. [Best Practices](#best-practices)  
9. [Common Use Cases](#common-use-cases)  
10. [Troubleshooting](#troubleshooting)  
11. [FAQ](#faq)

---

# Introduction

The Cloud Armor Builder is a visual configuration tool that simplifies the creation of Google Cloud Armor security policies. It guides you through building comprehensive web application firewall (WAF) rules and generates production-ready Terraform code that follows Google Cloud best practices.

### Key Benefits

- **No manual priority management** \- The tool automatically assigns and manages rule priorities  
- **Best practices built-in** \- Configurations follow Google Cloud's recommended security patterns  
- **Visual rule building** \- See your policy configuration in real-time as you build it  
- **Clean Terraform output** \- Generate maintainable infrastructure-as-code

---

# What Cloud Armor Builder Does (and Doesn't Do)

### What the Tool DOES

**Automated Configurations:**

- **Generates Terraform code** for Cloud Armor security policies  
- **Automatically assigns rule priorities** based on security best practices (no manual priority management needed)  
- **Sets optimal logging levels** \- Automatically enables VERBOSE logging when OWASP rules are added for better troubleshooting  
- **Enables advanced parsing** \- Automatically configures STANDARD\_WITH\_GRAPHQL parsing for comprehensive request analysis  
- **Creates meaningful rule descriptions** \- Automatically generates descriptions for all rule types except custom rules and Address Groups  
- **Configures Address Groups** with a default capacity of 2,000 IPs/CIDRs (expandable to 150,000 in GCP console if needed)  
- **Validates configurations** in real-time to prevent errors

### What the Tool DOES NOT Do

**Infrastructure Requirements (Must be handled separately):**

- **Does NOT create load balancers** \- You must have an existing Global or Regional External Application Load Balancer  
- **Does NOT apply policies to targets** \- You must manually attach the policy to your backend services or load balancer  
- **Does NOT create GCP projects or enable APIs** \- Your project must exist with required APIs enabled  
- **Does NOT manage existing policies** \- This is for creating new policies only  
- **Does NOT handle policy updates** \- Use Terraform's native capabilities for updating deployed policies

### Prerequisites Before Using the Tool

1. **Google Cloud Project** with billing enabled  
2. **Load Balancer** already created (Global or Regional External ALB)  
3. **Required APIs enabled:** `compute.googleapis.com`  
4. **IAM Permissions:** `compute.securityAdmin` and `compute.loadBalancerAdmin`  
5. **Terraform installed** (version 1.0 or higher) for deployment (Comes preinstalled in Cloud Shell)

---

# Getting Started

### Choosing Your Mode

The tool offers two modes to accommodate different experience levels:

#### 🎯 Guided Tour Mode (Recommended for beginners)

- Step-by-step walkthrough of each security category  
- Explanations for each rule type  
- Best practice recommendations  
- Ability to switch to Expert Mode at any time

#### ⚡ Expert Mode

- Direct access to all rule categories  
- Faster configuration for experienced users  
- No guided prompts  
- Full control over rule selection

### Understanding Tiers and Load Balancers

Your choice of **Cloud Armor Tier** and **Load Balancer Type** determines which features are available:

| Configuration | Available Features |
| :---- | :---- |
| **Standard Tier** \+ Any Load Balancer | Basic protection: IP/Geo blocking, OWASP, Rate Limiting, Bot Management |
| **Enterprise Tier** \+ **Regional** Load Balancer | Same as Standard (Enterprise features require Global ALB) |
| **Enterprise Tier** \+ **Global** Load Balancer | All features including Adaptive Protection, Threat Intelligence, Address Groups |

---

# Step-by-Step Guide

## Step 1: Define Your Policy

#### Policy Configuration Fields

**Google Cloud Project ID (**Required**)**

- Enter your GCP project ID where the policy will be deployed  
- Example: `my-project-123456`  
- This is your actual GCP project identifier, not a display name

**Policy Name** (Required)

- Use descriptive names like `production-web-app-policy` or `api-gateway-security`  
- Lowercase letters, numbers, and hyphens only  
- Must be unique within your project

**Load Balancer Type**

- **Global External ALB**: Choose for worldwide applications and full Enterprise feature access  
- **Regional External ALB**: Choose for region-specific applications (limits Enterprise features)

**Cloud Armor Tier**

- **Standard**: Core WAF protection suitable for most applications  
- **Enterprise**: Advanced ML-based protection and threat intelligence (requires Global ALB for full features)

**Default Rule Action**

- **Allow (Fail-Open)**: Default allows all traffic, rules block specific threats  
    
  - Best for: Public websites, e-commerce sites, content platforms  
  - Priority order: Block rules execute first (10,000-799,999), then allow rules (900,000-999,999)


- **Deny (Fail-Closed)**: Default blocks all traffic, rules allow specific sources  
    
  - Best for: Internal applications, admin panels, B2B platforms  
  - Priority order: Allow rules execute first (10,000-499,999), then block rules (900,000-999,999)

### Optional Advanced Settings

The tool includes an **"Optional Advanced Settings"** section that can be expanded to configure additional security parameters:

#### User IP Request Headers

- **Purpose**: Identify the true client IP when traffic passes through proxies, CDNs, or load balancers  
- **Options**:  
  * X-Forwarded-For \- Standard proxy header  
  * X-Real-IP \- Common with Nginx  
  * X-Client-IP \- Alternative client IP header  
  * CF-Connecting-IP \- Cloudflare's header  
  * True-Client-IP \- Akamai and other CDNs  
  * Custom headers  
- **Default**: If not configured, Cloud Armor uses the connecting server's IP (may be proxy/CDN IP)  
- **Important**: Configure this if your application sits behind any proxy or CDN to ensure accurate IP-based rules

#### Request Body Inspection Size

- **Purpose**: Define how much of POST/PATCH request bodies to inspect for WAF rules  
- **Options**:  
  * 8 KB (minimal inspection, best performance)  
  * 16 KB  
  * 32 KB   
  * 48 KB  
  * 64 KB (maximum inspection, higher latency)  
- **Trade-off**: Larger sizes provide better protection against attacks in request bodies but may produce additional false positives  
- **Applies to**: OWASP rules and custom rules that inspect request bodies  
- **Default**: Cloud Armor's default is 8kb, if not specified

### 

## Step 2: Configure Security Rules

The interface provides three panels:

- **Left Panel**: Rule category navigation (shows availability based on your tier/ALB selection)  
- **Middle Panel**: Active rule builder with configuration forms  
- **Bottom Panel**: Live preview of your policy configuration

#### Available Rule Categories

##### 🛡️ Standard Tier Rules (Always Available)

**1\. IP Blocking/Allowing**

- Block or allow specific IP addresses or CIDR ranges  
- Supports both IPv4 and IPv6  
- Example: Block `192.168.1.0/24` or allow office IP `203.0.113.45`

**2\. Geographic Controls**

- Control access based on country of origin  
- Uses ISO country codes  
- Can block or allow multiple countries simultaneously

**3\. OWASP Top 10 Protection**

- Industry-standard protection against common web vulnerabilities  
- Includes: SQL injection, XSS, RCE, LFI, RFI, and more  
- **Automatic Configuration**: Verbose logging enabled for all OWASP rules  
- Sensitivity levels 1-4 (higher \= stricter but more false positives possible)  
- All rules are set to Sensitivity Level 1 by default unless they are manually overridden

**4\. Rate Limiting**

- Prevent abuse and DDoS attacks  
- Configure requests per minute/hour thresholds  
- Actions: Throttle, temporary ban, or deny  
- Can be applied globally or per-IP

**5\. Bot Management**

- Control access for 92 predefined bots across 7 categories  
- Separate actions for legitimate bots vs impersonators  
- Categories include: Search engines, social media, monitoring, AI crawlers

**6\. Custom Rules**

- Leverage drop down menus and a UI to build Cloud Armor Common Expression Language (CEL) rules  
- Apply Transformations easily like lower(), decodeUrl(), etc.  
- Support for creating multiple sub-expressions, up to a limit of 5 per rule.  
- Create groups of rules to define complex and prescriptive protection

##### 🚀 Enterprise Tier Rules (Global ALB \+ Enterprise Required)

**7\. Adaptive Protection**

- ML-powered anomaly detection  
- Automatically identifies and mitigates attacks  
- Self-learning based on your traffic patterns

**8\. Threat Intelligence**

- Real-time threat feeds from Google  
- Includes: Tor exit nodes, known attackers, malicious IPs  
- Multiple feed options for different threat types

**9\. Address Groups**

- Create reusable lists of IPs/CIDRs  
- Default capacity: 2,000 entries (expandable to 150,000 in GCP console)  
- Simplifies management of large IP lists

## Step 3: Review and Export

Once you've configured your rules:

1. Review the complete configuration in the preview panel  
2. Check that priorities are correctly assigned (automatic)  
3. Click "Generate Terraform" to create your infrastructure code  
4. Download or copy the generated `.tf` file

---

# Understanding Security Rules

### How Rules Are Processed

Rules are evaluated in **priority order** (lowest number first):

- Priority 10,000 executes before priority 20,000  
- First matching rule wins  
- If no rules match, the default action applies

### Automatic Priority Assignment

The tool uses intelligent priority ranges to ensure proper rule execution:

#### For "Allow" Default (Fail-Open)

```
  10,000-99,999     → IP Deny Rules (Block attackers first)
  100,000-199,999   → Geographic Deny Rules
  200,000-299,999   → Address Groups Deny Rules
  300,000-399,999   → Threat Intelligence
  400,000-499,999   → Adaptive Protection
  500,000-599,999   → OWASP Rules
  600,000-699,999   → Rate Limiting
  700,000-799,999   → Bot Management
  800,000-899,999   → Custom Rules
  900,000-919,999   → IP Allow Rules
  920,000-939,999   → Geographic Allow Rule
  940,000-999,999   → Address Groups Allow Rules
```

#### For "Deny" Default (Fail-Closed)

```
  10,000-99,999     → Address Groups Allow Rules (Allow trusted first)
  100,000-199,999   → IP Allow Rules
  200,000-299,999   → Geographic Allow Rules
  300,000-399,999   → Bot Management
  400,000-499,999   → Custom Rules (Allow)
  450,000-459,999   → Custom Rules (Deny/Throttle/Rate-Based Ban)
  500,000-599,999   → Rate Limiting
  600,000-699,999   → OWASP Rules
  700,000-799,999   → Threat Intelligence
  800,000-899,999   → Adaptive Protection
  900,000-919,999   → IP Deny Rules
  920,000-939,999   → Geographic Deny Rules
  940,000-999,999   → Address Groups Deny Rules
```

---

# Automatic Configurations

The tool automatically configures several advanced settings based on best practices:

### Verbose Logging

- **Automatically enabled** when any OWASP rule is added  
- **Normal logging** for all other rule types  
- Helps troubleshoot false positives and understand traffic patterns

### User-IP Settings

- **Automatically** uses `origin.user_ip` attribute in CEL rules to detect real users IPs instead of connecting IPs  
- If no user-ip is defined in Step 1 (Policy Setup), Cloud Armor falls back to using the connecting IP automatically.

### Request Parsing

- **STANDARD\_WITH\_GRAPHQL** parsing automatically enabled  
- Ensures GraphQL queries are properly inspected  
- No manual configuration needed

### Rule Descriptions

- **Automatically generated** for all predefined rule types  
- Includes context about what the rule does  
- Custom rules and Address Groups require manual descriptions

### Address Group Capacity

- **Default: 2,000** IPs/CIDRs per group (tool limit for performance)  
- **Maximum in GCP: 150,000** entries (can be increased after deployment)  
- Modify in GCP Console if larger capacity needed

### Priority Spacing

- **1,000 priority gap** between rules  
- Allows insertion of new rules without renumbering  
- Automatic gap detection fills unused priorities  
- OWASP rules added in Preview mode are prioritized higher than Enforce mode rules to allow for proper inspection & evaluation

---

# Deploying Your Policy

### Step 1: Prepare Your Environment

```shell
# Authenticate with Google Cloud
gcloud auth login

# Set your project
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable compute.googleapis.com
```

### Step 2: Deploy with Terraform

```shell
# Create a directory for your Terraform files
mkdir cloud-armor-policies
cd cloud-armor-policies

# Copy your generated .tf file here
# Then initialize Terraform
terraform init

# Review the plan
terraform plan

# Apply the configuration
terraform apply
```

### Step 3: Attach Policy to Load Balancer

After the policy is created, attach it to your backend service:

```shell
# For Global Load Balancer
gcloud compute backend-services update YOUR_BACKEND_SERVICE \
    --security-policy YOUR_POLICY_NAME \
    --global

# For Regional Load Balancer
gcloud compute backend-services update YOUR_BACKEND_SERVICE \
    --security-policy YOUR_POLICY_NAME \
    --region YOUR_REGION
```

### Step 4: Verify Deployment

```shell
# List all security policies
gcloud compute security-policies list

# Describe your specific policy
gcloud compute security-policies describe YOUR_POLICY_NAME

# Check if policy is attached to backend
gcloud compute backend-services describe YOUR_BACKEND_SERVICE --global
```

---

# Best Practices

### Policy Design

#### Start Conservative

- Begin with logging/preview mode before enforcement  
- Monitor for false positives for 24-48 hours  
- Gradually increase protection levels

#### Layer Your Defenses

For example:

1. **First Layer**: Geographic and IP blocking for known bad actors  
2. **Second Layer**: Rate limiting to prevent abuse  
3. **Third Layer**: OWASP rules for application protection  
4. **Fourth Layer**: Bot management for crawler control

#### Use Preview Mode

- Test new rules by enabling `Preview` mode  
- Review logs before enabling enforcement  
- Especially important for OWASP sensitivity changes

### Rule Configuration

#### IP and Geographic Blocking

- Use CIDR ranges instead of individual IPs when possible  
- Regularly review and update blocked regions  
- Consider business requirements before blocking countries  
- Maintain an allow-list for office/VPN IPs

#### Rate Limiting

- **Conservative starting points:**  
  - API endpoints: 100 requests per minute per IP  
  - Login pages: 10 requests per minute per IP  
  - General pages: 1000 requests per minute per IP  
- Adjust based on legitimate traffic patterns  
- Consider burst traffic during promotions/events

#### OWASP Protection

- **Sensitivity Level Guideline Example:**  
  - Level 1: Start here (least false positives)  
  - Level 2: After 1 week without issues  
  - Level 3: For high-security applications  
  - Level 4: Only with extensive testing  
- Monitor verbose logs for false positives  
- Create exceptions for legitimate traffic patterns

#### Bot Management

- Let **legitimate bots** in, at least initially  
- **Block** or **Throttle impersonators** by default  
- Review bot traffic monthly and adjust

### Monitoring

#### Log Analysis

- Use Cloud Logging to analyze blocked requests  
- Look for patterns in false positives  
- Identify new attack vectors  
- Track effectiveness metrics  
- Leverage out of the box Cloud Armor Dashboards in Cloud Monitoring

---

# Common Use Cases and Example Guidelines

### E-commerce Website

**Strategy**: Default Allow (fail-open)

```
Priority Order:
1. Block known malicious IPs (10,000-99,999)
2. Block high-risk countries if not selling there (100,000-199,999)
3. OWASP Level 2 protection (500,000-599,999)
4. Rate limit: 500 req/min per IP (600,000-699,999)
5. Block bot impersonators (700,000-799,999)
```

### Corporate Application

**Strategy**: Default Deny (fail-closed)

```
Priority Order:
1. Allow office IP ranges (10,000-99,999)
2. Allow VPN endpoints (100,000-199,999)
3. Allow specific countries where employees are (200,000-299,999)
4. Rate limit: 100 req/min per IP (500,000-599,999)
5. OWASP Level 3 protection (600,000-699,999)
```

### Public API

**Strategy**: Default Allow with strict rate limiting

```
Priority Order:
1. Block known abusive IPs (10,000-99,999)
2. Threat intelligence feeds (300,000-399,999)
3. Tiered rate limiting:
   - Free tier: 60 req/min (600,000)
   - Paid tier: 600 req/min (600,100)
4. OWASP Level 2 for injection protection (500,000-599,999)
5. Block all bot traffic (700,000-799,999)
```

### Development/Staging Environment

**Strategy**: Default Deny with specific allowlists

```
Priority Order:
1. Allow developer IPs only (10,000-99,999)
2. Allow CI/CD pipeline IPs (100,000-199,999)
3. Allow monitoring service IPs (200,000-299,999)
4. Block everything else (default deny)
```

---

# Troubleshooting

### Common Issues and Solutions

#### "Policy not blocking traffic"

- **Check**: Is the policy attached to your backend service?  
- **Verify**: `gcloud compute backend-services describe YOUR_BACKEND --global`  
- **Solution**: Attach policy using gcloud command or Terraform

#### "Legitimate users being blocked"

- **Check**: Review logs for which rule is triggering  
- **Common causes**:  
  - OWASP sensitivity too high  
  - Rate limits too aggressive  
  - Geographic blocking too broad  
- **Solution**: Adjust rule configuration or add exceptions

#### "OWASP false positives"

- **Immediate fix**: Lower sensitivity level  
- **Long-term**: Create custom rule exceptions for specific patterns via Fine Tuning  
- **Use verbose logs** (automatically enabled) to identify patterns

#### "Rate limiting not working"

- **Check**: Enforcement threshold vs ban threshold  
- **Verify**: Time windows are appropriate  
- **Common issue**: Threshold too high for the time window

#### "Can't see all Enterprise features"

- **Requirements**: Must have BOTH:  
  1. Enterprise tier selected  
  2. Global External ALB selected  
- **Solution**: Change load balancer type to Global

#### "Address Group capacity exceeded"

- **Tool limit**: 2,000 entries  
- **GCP limit**: 150,000 entries  
- **Solution**: The Cloud Armor Builder limits users to creating Address Groups with a maximum capacity of 2000\. If more than 2000 IPs/CIDRs are required for your Address Group, you may create the Address Group outside of this tool via the GCP console or `gcloud`, where you can set the capacity to a maximum of 150,000.

---

# FAQ

### General Questions

**Q: Can I modify the generated Terraform code?** A: Yes\! The code is clean and well-commented. You can customize it further, add variables, or integrate it with your existing Terraform modules.

**Q: How often should I update my security policies?** A: Review quarterly at minimum. Update threat intelligence monthly, and immediately respond to new attack patterns.

**Q: Can I import existing Cloud Armor policies?** A: No, this tool is for creating new policies. Use Terraform import for existing resources.

**Q: Can I modify the priority ranges?** A: The priorities ranges per rule type are hardcoded into the Cloud Armor Builder. However, you can modify the priority of any individual rule you create on the Preview Panel in Step 2, or on the Review page in Step 3\.

### Technical Questions

**Q: Why is verbose logging only for OWASP rules?** A: OWASP rules are most likely to generate false positives. Verbose logging helps identify patterns for tuning without overwhelming logs for other rule types.

**Q: Can I use preview mode for testing?** A: Of course. It is encouraged and recommended to do so before applying blocking rules.

**Q: Why is the Address Group limit 2,000 in the tool?** A: This acts as a natural constraint to prevent users from maxing out their total Address Group capacity across their GCP Projects or Organization.  A capacity of 2,000 is a good starting point.  If larger capacity than 2,000 is required, use the GCP console or `gcloud` to create your address group.

**Q: How do priorities work with manual override?** A: The tool automatically assigns priorities, but you can manually override them in the generated Terraform if needed by clicking on the priority number. 

### Best Practices Questions

**Q: Should I use fail-open or fail-closed?** A:

- **Fail-open** for public services where availability is critical  
- **Fail-closed** for internal services where security is paramount

**Q: What's the recommended OWASP sensitivity level?** A: Start with Level 1, monitor for a week, then gradually increase. Most production sites run successfully at Level 2\.

**Q: How many rules can I have in one policy?** A: Google Cloud Armor supports up to 200 rules per policy. The tool will work within these limits.

**Q: Should I block all bots?** A: No. We recommending not taking any action initially on legitimate bots unless they are sending requests too frequently. We recommend blocking impersonators and manage scrapers based on your needs.

---

# Getting Help

### Support Resources

**Need Help?**

- Click the "Need Support?" button in the top-right corner of the tool  
- Access this user guide anytime  
- Submit feature requests  
- Report bugs

**Additional Resources:**

- [Google Cloud Armor Documentation](https://cloud.google.com/armor/docs)  
- [Terraform Google Provider Documentation](https://registry.terraform.io/providers/hashicorp/google/latest/docs)  
- [Cloud Armor Pricing](https://cloud.google.com/armor/pricing)

### Feedback

We continuously improve the tool based on user feedback. Please share:

- Feature requests  
- Bug reports  
- Use case examples  
- Configuration patterns that work well

---

*Last Updated: August 2025*  
