
# **Project Comprehensive Details**

---

## **1. Removing Dependency on EFS and Implementing Multi-Tenant Architecture**

**Company:** Uniphore Technologies  
**Role:** Software Engineer

---

### **Objective**  
Transformed the **U-Analyze** speech analytics platform by migrating from **AWS Elastic File System (EFS)** to **AWS S3** and implementing a robust **multi-tenant architecture** to enhance scalability, security, and cost efficiency.

---

### **System Architecture Flow Chart**

#### **Before Migration: EFS-Based Architecture**

**Flow Chart Description**:

1. **User Request Handling**:
   - Users access the application, triggering requests.
2. **Monolithic Application**:
   - The application processes requests using a monolithic codebase.
3. **EFS Storage Access**:
   - All user-generated data, irrespective of the tenant, is stored in a shared **Elastic File System (EFS)**.
   - This leads to **data sharing** between tenants with **limited scalability** and potential **security risks**.

**Challenges**:

- **Scalability Issues**: Limited performance improvements due to shared storage.
- **Security Concerns**: Risks of data exposure due to a shared file system.
- **High Costs**: Over-provisioning and underutilization of storage resulted in cost inefficiencies.

---

#### **After Migration: S3 Multi-Tenant Architecture**

**Flow Chart Description**:

1. **User Request Handling**:
   - Users make requests, which are now routed through a **gateway**.
2. **Microservices-Based Application**:
   - Requests are processed by multiple **microservices** (e.g., Authentication Service, File Management Service).
   - Each microservice is built to handle specific tasks and scales independently.
3. **S3 Bucket per Tenant**:
   - Tenant-specific requests are directed to **individual S3 buckets**.
   - Each tenant has a dedicated bucket, ensuring **data isolation**.
   - **AWS IAM Roles** manage access, and **AWS KMS** provides encryption for each bucket.

**Scalability and Security Improvements**:

- Transition to **AWS S3** eliminated bottlenecks related to shared storage.
- **Tenant Isolation** through dedicated buckets and IAM roles, ensuring a robust security model.
- Event-driven workflows were integrated with **AWS Lambda** for **real-time data processing**.

---

### **Technical Implementation**

#### **1. AWS S3 Integration**

- **Bucket-per-Tenant Strategy**:
  - **Naming Convention**: `s3://u-analyze-<tenant_id>/`
  - **Advantages**: Ensured data isolation, simplified access control, and streamlined management.

**Code Example: AWS S3 File Upload**  
```java
public void uploadFile(String tenantId, File file) {
    String bucketName = "u-analyze-" + tenantId;
    String objectKey = "files/" + file.getName();
    PutObjectRequest request = new PutObjectRequest(bucketName, objectKey, file)
                                  .withCannedAcl(CannedAccessControlList.Private);
    s3Client.putObject(request);
}
```

#### **2. Security Enhancements**

- **IAM Policy for Tenant Isolation**:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "s3:*",
        "Resource": [
          "arn:aws:s3:::u-analyze-tenant123",
          "arn:aws:s3:::u-analyze-tenant123/*"
        ]
      }
    ]
  }
  ```
- **Encryption**:
  - Implemented **SSE-KMS** with tenant-specific encryption keys to meet compliance requirements.

#### **3. Migration Strategy**

- **Data Migration Without Downtime**:
  - Utilized **AWS DataSync** to transfer data efficiently.
  - Real-time synchronization ensured data consistency.
- **Data Integrity Verification**:
  - Performed **checksum validations** post-migration.
  - Enabled **S3 Versioning** for added data protection.

---

### **Challenges and Solutions**

| **Challenge**                   | **Solution**                                                 |
|---------------------------------|--------------------------------------------------------------|
| **Seamless Data Migration**     | Used **AWS DataSync** for real-time data transfer.           |
| **Ensuring Data Consistency**   | Implemented checksum validations and enabled S3 versioning.  |
| **Access Control Complexity**   | Automated IAM policy generation for each tenant.             |
| **Scalability Concerns**        | Leveraged S3's scalability and Kubernetes auto-scaling.      |

---

### **Impact**

| **Metric**                | **Before Migration** | **After Migration**       |
|---------------------------|----------------------|---------------------------|
| File Access Latency       | ~200ms               | ~130ms (**35% reduction**)|
| Storage Costs             | High (fixed)         | Reduced by **40%**        |
| Concurrent Users Supported| Limited              | **5x Increase**           |

### **Mentoring and Leadership**

- **Led a Team** of **6 engineers** to implement AWS best practices and manage the migration process.
- **Conducted Training** on AWS services and architecture principles.

---

## **2. Automated MongoDB Data Purging Script**

**Company:** Uniphore Technologies  
**Role:** Software Engineer

---

### **Objective**

Automated the deletion of outdated documents from MongoDB collections, adhering to data retention policies, and optimizing database performance.

---

### **Architecture Flow Chart**

1. **Data Purging Workflow**:
   - **Cron Scheduler** triggers the Node.js script at scheduled times.
   - The **Node.js Purging Script** connects to **MongoDB Replica Set**.
   - Based on **retention policies** defined in configuration files, outdated documents are identified and deleted in batches.
   - Logs and **Nagios Alerts** are generated to monitor the process.

2. **Monitoring and Automation**:
   - Logs are integrated into **ELK Stack** for visualization and tracking.
   - **Nagios** is configured to send alerts in case of errors or failures.

---

### **Technical Implementation**

**Code Example**:
```javascript
const fs = require('fs');
const MongoClient = require('mongodb').MongoClient;

const config = JSON.parse(fs.readFileSync('config.json', 'utf8'));
const uri = 'mongodb://username:password@host:port/database';

(async () => {
  const client = await MongoClient.connect(uri, { useUnifiedTopology: true });
  const db = client.db('database');

  for (const col of config.collections) {
    const retentionDate = new Date(Date.now() - col.retentionDays * 24 * 60 * 60 * 1000);
    const result = await db.collection(col.name).deleteMany({ createdAt: { $lt: retentionDate } });
    console.log(`Purged ${result.deletedCount} documents from ${col.name}.`);
  }

  client.close();
})();
```

---

### **Challenges and Solutions**

| **Challenge**                             | **Solution**                                                   |
|-------------------------------------------|---------------------------------------------------------------|
| **Minimizing Impact on Live Traffic**     | Scheduled tasks during low-traffic periods and used batch operations. |
| **Dynamic Retention Requirements**        | Used configuration files for easy updates to retention policies. |

### **Impact**

| **Metric**                | **Before**         | **After**                |
|---------------------------|--------------------|--------------------------|
| Query Latency             | ~300ms             | ~180ms (25% reduction)   |
| Storage Usage             | High (90%)         | Reduced by **30%**       |

### **Mentoring and Collaboration**

- **Created Documentation** for future maintenance.
- **Conducted Workshops** on MongoDB performance optimization.

---

## **3. Automated Vulnerability and Risk Assessment Integration**

**Company:** Uniphore Technologies  
**Role:** Software Engineer

---

### **Objective**

Integrated comprehensive security vulnerability scanning tools into the CI/CD pipeline to proactively identify and remediate critical security issues, enhancing the security posture of the **U-Analyze** platform.

---

### **CI/CD Pipeline Flow Chart**

1. **Source Code Management**:
   - Developers push code changes to **GitHub** or **GitLab**.
2. **Trigger Build Pipeline**:
   - CI/CD tools trigger builds, which include **unit tests**.
3. **Security Scanning Integration**:
   - **Static Application Security Testing (SAST)** is performed using **Veracode** immediately after the build.
   - **Software Composition Analysis (SCA)** with **Snyk** scans the dependencies for vulnerabilities.
   - **Dynamic Application Security Testing (DAST)** is performed on the **staging environment** before final deployment.
4. **Notifications**:
   - Developers receive alerts for any **high-severity vulnerabilities** found.

---

### **Technical Implementation**

#### **Types of Scanning Implemented**

1. **Static Application Security Testing (SAST)**:
   - **Tool Used**: **Veracode Static Scan**.
   - **Purpose**: Identifies vulnerabilities in source code before execution.
   - **Implementation**: Integrated in GitHub/GitLab workflows, running scans on every new commit or pull request.

2. **Software Composition Analysis (SCA)**:
   - **Tool Used**: **Snyk**.
   - **Purpose**: Analyzes third-party libraries for known vulnerabilities.
   - **Implementation**: Automatically scans dependency files (`pom.xml`, `package.json`) in GitHub/GitLab for vulnerabilities.

3. **Dynamic Application Security Testing (DAST)**:
   - **Tool Used**: **Veracode Dynamic Analysis**.
   - **Purpose**: Tests the running application for security issues that can only be detected during runtime.
   - **Implementation**: DAST scans are performed in the **staging environment** to find and fix vulnerabilities before deploying to production.

4. **Interactive Application Security Testing (IAST)**:
   - **Tool Used**: **Veracode Interactive Analysis**.
   - **Purpose**: Monitors vulnerabilities during runtime in real-world conditions.
   - **Implementation**: Deployed agents in staging to monitor the application while running end-to-end tests.

---

### **Major Vulnerabilities Identified and Solutions**

1. **Log4j Vulnerability (CVE-2021-44228)**:
   - **Issue**: Remote code execution vulnerability in Apache Log4j.
   - **Detection**: Identified through **SCA** scans that flagged outdated Log4j dependencies.
   - **Solution**: Upgraded to secure versions and implemented log sanitization.

2. **SQL Injection Flaws**:
   - **Issue**: Unsanitized input parameters in database queries.
   - **Detection**: Detected by **SAST** scans and confirmed by **DAST**.
   - **Solution**: Refactored code to use **prepared statements** and added **input validation**.

3. **Cross-Site Scripting (XSS)**:
   - **Issue**: Reflected and stored XSS vulnerabilities in the web interface.
   - **Detection**: Identified via **DAST** scans.
   - **Solution**: Implemented **output encoding** and added **Content Security Policy (CSP)** headers.

4. **Outdated Libraries and Dependencies**:
   - **Issue**: Use of outdated third-party libraries with known vulnerabilities.
   - **Detection**: Detected via

 **Snyk** scans.
   - **Solution**: Automated dependency updates using tools like **Dependabot** and manual verification for critical updates.

---

### **Challenges and Solutions**

| **Challenge**                          | **Solution**                                                         |
|----------------------------------------|----------------------------------------------------------------------|
| **High Number of False Positives**     | Tuned scanning tools and created suppression rules with justifications. |
| **Team Resistance to Change**          | Conducted training sessions on secure coding practices and how to interpret scan results. |

---

### **Impact**

| **Metric**                 | **Before Integration** | **After Integration**      |
|----------------------------|------------------------|----------------------------|
| Risk Score                 | **99/100**             | **20/100** (Significant Reduction) |
| Issue Resolution Time      | High                   | Decreased by **60%**       |

---

### **Mentoring and Leadership**

- **Training Sessions**: Conducted workshops on **secure coding practices**, **interpreting scan results**, and **security in the SDLC**.
- **Policy Development**: Developed **Secure Development Lifecycle (SDLC)** policies to align with industry best practices.

---

## **4. Automated Reports for Transaction Monitoring**

**Company:** Comviva  
**Role:** Software Engineer

---

### **Objective**

Developed an automated reporting system for the **Mobiquity Money** product to monitor transaction details, ensure data consistency, and support strategic decision-making.

---

### **Architecture Flow Chart**

1. **ETL Workflow**:
   - **Data Extraction**: Extracted data from **Oracle** and **PostgreSQL** databases.
   - **Data Transformation**: Cleaned, normalized, and merged data using **Pentaho Data Integration**.
   - **Data Loading**: Loaded into a centralized reporting database for analysis.
2. **Reporting and Automation**:
   - Reports were generated using **Pentaho Report Designer**.
   - **Automation Scheduler**: Scheduled reports were sent to stakeholders using built-in scheduling tools.

---

### **Technical Implementation**

#### **ETL Processes**

- Extracted data from multiple databases, ensuring consistency across **Oracle** and **PostgreSQL**.
- Transformation included **data cleaning** and **normalization** to produce reliable insights.
- Automated report scheduling using **Pentaho** ensured timely delivery.

---

### **Challenges and Solutions**

| **Challenge**                  | **Solution**                                                  |
|--------------------------------|---------------------------------------------------------------|
| **Data Consistency Issues**    | Implemented validation checks and reconciled data regularly.  |
| **Performance Bottlenecks**    | Optimized SQL queries and applied indexing on key fields.     |

### **Impact**

| **Metric**                | **Before**           | **After**                 |
|---------------------------|----------------------|---------------------------|
| Report Accuracy           | Manual Verification  | **100% Automated Validation** |
| Reporting Time            | Long Manual Process  | **Reduced by 80%**        |

---

## **5. Spring Boot Microservices for Biller Category**

**Company:** Comviva  
**Role:** Software Engineer

---

### **Objective**

Migrated the billing module of the **Mobiquity Money** product from a monolithic architecture to microservices using **Spring Boot** to improve scalability and maintainability.

---

### **Architecture Flow Chart**

1. **User Request**:
   - User initiates a request, routed through an API gateway.
2. **Microservices Workflow**:
   - **Biller Management Service** handles billers.
   - **Payment Processing Service** manages payments.
   - **Notification Service** sends notifications.
3. **Database Access**:
   - Each microservice uses separate databases.
4. **Orchestration**:
   - Services are containerized using **Docker** and managed by **Kubernetes**.

---

### **Technical Implementation**

#### **Microservices Development**

- **Technologies**: Leveraged **Spring Boot**, **Spring Cloud** for rapid development.
- **Service Discovery** and **Load Balancing** were handled by **Eureka** and **Ribbon**.

#### **Deployment and Orchestration**

- **Continuous Deployment** using Jenkins and Docker.
- **Monitoring** with **Prometheus** and **Grafana**.

---

### **Challenges and Solutions**

| **Challenge**                       | **Solution**                                                   |
|-------------------------------------|---------------------------------------------------------------|
| **Service Communication**           | Utilized **Feign Clients** and **Hystrix** for fault tolerance.|
| **Data Consistency**                | Implemented **Saga Pattern** for distributed transaction management.|

### **Impact**

| **Metric**                | **Before Migration**  | **After Migration**       |
|---------------------------|-----------------------|---------------------------|
| System Throughput         | Limited               | Increased by **30%**      |
| Deployment Time           | Lengthy               | **Reduced by 40%**        |

### **Mentoring and Leadership**

- **Code Review**: Led code review sessions to ensure adherence to standards.
- **Collaboration**: Worked closely with DevOps and QA for smoother deployments.

---

# **Conclusion**

These comprehensive project overviews showcase strong expertise in:

- **Backend Development**: Proficient in Java, Node.js, and microservices.
- **Cloud Infrastructure**: Expertise in AWS services (S3).
- **Database Optimization**: MongoDB data purging and database scaling.
- **Security Enhancement**: Robust implementation of application security in CI/CD.
- **Leadership and Mentoring**: Led teams, conducted training, and established best practices.

---
