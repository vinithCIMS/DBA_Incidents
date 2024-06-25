### RCA & Corrective and Preventive Action (CAPA) Report

---

### Summary

**Incident:** Email Reference: Re: Your request has been logged with request id ##667270##

**Issue:** The CIMS application was not generating UPS tracking numbers for the attached LPNs.

**Reported By:** Krishna Tummala [ktummala@hybridapparel.com](mailto:ktummala@hybridapparel.com)

**Date of Incident:** June 24, 2024, at 4:49 PM PST

---

### Input for CAPA

1. **Application Error Log:**
    
    ```
    LOGGER Error: 0 : 6/14/2024 2:45:38 PM : System.Data.SqlClient.SqlException (0x80131904): Execution Timeout Expired. The timeout period elapsed prior to completion of the operation or the server is not responding.
    A transaction that was started in a MARS batch is still active at the end of the batch. The transaction is rolled back.
    Operation cancelled by user.
    
    ```
    
2. **SQL Server Error Log:**
    
    ```
    2024-06-14 14:45:38.51 spid65 Autogrow of file 'HA_cIMSDev_Blank_DATA' in database 'CIMSProd' was cancelled by user or timed out after 20592 milliseconds. Use ALTER DATABASE to set a smaller FILEGROWTH value for this file or to explicitly set a new file size.
    
    ```
    
3. **Client Attached LPN:**
The client attached a list of LPNs that were stuck and not generating UPS tracking numbers.
4. **Microsoft Support Case Q&A:**
Referenced support case for guidance on managing execution timeouts and autogrow settings.

---

### Immediate Actions

From Ramanamurthy V:

- Identified that the reported records were stuck on the same day (06/14/2024).
- Noted that there are still 48 records stuck on the same day, but those Shipvias have been changed to LTL (Less than Truckload).

---

### Root Cause Analysis (RCA)

**A. Description of the Problem:**

On June 14, 2024, at 2:45:38 PM, users reported an application error indicating a timeout issue. The error log showed an execution timeout expired error from the application, which coincided with a SQL Server error log entry indicating an autogrow operation timeout.

**B. Error Log Details:**

**Application Error Log:**

```
LOGGER Error: 0 : 6/14/2024 2:45:38 PM : System.Data.SqlClient.SqlException (0x80131904): Execution Timeout Expired. The timeout period elapsed prior to completion of the operation or the server is not responding.
A transaction that was started in a MARS batch is still active at the end of the batch. The transaction is rolled back.
Operation cancelled by user.

```

**SQL Server Error Log:**

```
2024-06-14 14:45:38.51 spid65 Autogrow of file 'HA_cIMSDev_Blank_DATA' in database 'CIMSProd' was cancelled by user or timed out after 20592 milliseconds. Use ALTER DATABASE to set a smaller FILEGROWTH value for this file or to explicitly set a new file size.

```

**C. Root Cause:**

The autogrow operation for the database file `HA_cIMSDev_Blank_DATA` in the `CIMSProd` database was set to grow by 5120 MB (5 GB). This large autogrow increment caused a significant delay in the operation, leading to a timeout. The delay caused the application to experience a timeout error, resulting in the user-facing issue.

The error indicates that the transaction/session cannot wait until the file growth completes and then the session is cancelled. Due to that, the file growth is also cancelled. The reason is that the file growth cannot complete in time. The growth operation would kick in and the transaction would be halted until growth is complete. The time taken to grow is dependent on two factors:

1. Size of growth
2. Disk speed.

---

### Corrective and Preventive Actions (CAPA)

**1. Immediate Corrective Actions:**


From Ramanamurthy V:

- Identified that the reported records were stuck on the same day (06/14/2024).
- Noted that there are still 48 records stuck on the same day, but those Shipvias have been changed to LTL (Less than Truckload).

---

**2. Long-Term Preventive Actions:**

- **Changing the Auto-grow Settings Based on Recommendations:**
We will review and implement Microsoft's recommendations for autogrow settings, ensuring they are valid and suitable for our environment.
    
    **Reference:** [Microsoft Case Request logged for Recommendations](https://learn.microsoft.com/en-us/answers/questions/1722897/execution-timeout-expired-error-with-ups-api-reque?page=1&orderby=Helpful&comment=answer-1556448&sharingId=332E480B15C1EE13)
    
- **Proactive Monitoring and Space Management:**
Implement a SQL Server Agent job to regularly monitor the database file sizes and automatically increase the file size during off-peak hours if necessary.

 **3. Documentation and Training:**   
     **Reference:**
     
https://techyaz.com/sql-server/performance-tuning/understanding-database-autogrowth-sql-server/

https://www.red-gate.com/simple-talk/databases/sql-server/database-administration-sql-server/sql-server-database-growth-and-autogrowth-settings/
            
### Conclusion

The timeout issue was caused by a large autogrow increment, leading to delays in the autogrow operation and subsequent application timeout. By adjusting the autogrow settings, implementing proactive monitoring and space management, and conducting a thorough review of the storage performance, we have taken significant steps to prevent similar issues in the future. The DBA team is now better equipped with the knowledge and tools to manage database growth and performance proactively.

**Report Prepared By:**

[Vinith Ankam]

Senior SQL Server Database Administrator
