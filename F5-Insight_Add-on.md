
# Ticket 01 – Update BIG-IP instance (Part 1/3)

### Description

> A new version of BIG-IP is released and your team has been tasked to perform the upgrade using F5 Insight
> 
> The .iso files have been uploaded and verified in F5 Insight. The next step is to distribute the software to the instances.

### Action

Navigate to: **Manage > Automation > Jobs**  

**Click on:** Add Job > "Software Distribution"  

**Fill Form:**  
> Job Name:  17.1.3.4 Distribution  
> 
> Instances:
> > - bigip/NorthRegion-bigip-01.udf.labs  
> > - bigip/NorthRegion-bigip-02.udf.labs  
> 
> Software to Distribute:  BIGIP-17.1.3.4-0.0.12.iso  
> 
> Distribution Type:  Serial (Rolling Execution)  

**Click on:** "Check Instances"  

**Click on:** "Execute Job"  


To confirm the distribution has started:

> - View Distribution Progress using REST API

udf.f5.com GUI: > Deployment > F5 Insight 01 (Primary) > ACCESS > WEB SHELL
```sh
curl -X GET "https://localhost/api/v1/fleet/upgrade/jobs/software-distribution" \
     -H "Authorization: Bearer $(curl -k -s -X POST "https://localhost/api/auth/login" \
                                      -H "Content-Type: application/json" \
                                      -d "{ \"username\": \"admin\", \"password\": \"HelloUDF\" }" \
                                      | jq -r '.access_token')" \
     -k | jq
```
  

---
## Ticket 02 – Explore Top N

### Description

> The Top N dashboard provides a consolidated view of the most active and resource-intensive objects on a BIG-IP.
> 
> This ticket is designed to familiarize you with what information is available and how it can be used to quickly identify busy applications, high resource consumers, and potential availability concerns.
> 
> Your goal is to explore the Top N dashboard and understand the types of operational insights it provides.

### Action

> **Dashboards >> BIG-IP Device >> Top N**
 
From the Device dropdown at the top of the page, select the BIG-IP specified below.

> **CentralRegion-bigip-01**

### Task

> - Which VIP has the highest WAF CPU Utilization on CentralRegion-bigip-01?

---
# Ticket 03 – Update BIG-IP instance (Part 2/3)

### Description

> The software has been successfully distributed to both NorthRegion instances.
> 
> The next step is to begin installation of the new version on both instances.

### Action

Navigate to: **Manage > Automation > Jobs**  

> **Click on:** Add Job > "Software Distribution"  
> 
> **Fill Form:**  
> > Job Name:  17.1.3.4 Distribution  


Navigate to: **Manage > Automation > Jobs**  

**Click on:** Add Job > "Software Installation"  

**Fill Form:**  
> Job Name: 17.1.3.4 Installation  
> 
> Installation Type: HA Pair
> 
> Execution Type: Serial (Rolling Execution)  
> 
> Checkbox: Only select the 2 boxes below (leave the rest un-checked)  
> > - Automatic failback after both instance in HA Pair upgraded  
> > - After reboot of Standby instance, before failover  
> 
> Target Distribution: BIGIP-17.1.3.4-0.0.12.iso  
> 
> Set Target Volume: Next Sequential (New)  
> 
> Instances:  
> > - bigip/NorthRegion-bigip-01.udf.labs  
> > - bigip/NorthRegion-bigip-02.udf.labs  

**Click on:** "Run Checks"  

**Click on:** "Execute Job"  

We have configured F5 Insight to pause when install on the first instance completes.

To confirm the installation has started:

> - View Installation Progress using REST API

Use the REST API to view Installation Jobs  
udf.f5.com GUI: > Deployment > F5 Insight 01 (Primary) > ACCESS > WEB SHELL
```sh
curl -X GET "https://localhost/api/v1/fleet/upgrade/jobs/software-installation" \
     -H "Authorization: Bearer $(curl -k -s -X POST "https://localhost/api/auth/login" \
                                      -H "Content-Type: application/json" \
                                      -d "{ \"username\": \"admin\", \"password\": \"HelloUDF\" }" \
                                      | jq -r '.access_token')" \
     -k | jq
```


---

# Ticket 04 – “What are these unused pools?”

### Description

> During a routine review of the **CentralRegion-bigip-01** configuration, operations suspects there may be unused (orphaned) objects left over from previous testing or decommissioned applications.
> 
> You have been asked to identify any orphaned pools on **CentralRegion-bigip-01** so they can be documented and, if appropriate, cleaned up later.

### Action

> The AI Assistant located at the top of the home screen
> 
> and enter the following prompt:
> 
> `Show all pools on CentralRegion-bigip-01, and indicate which ones are not referenced by any virtual server.`
> 

From the returned information and the TMUI on CentralRegion-bigip-01:
> 
> - Navigate to: **Local Traffic >> Pools >> Pool List**
>     
>     Confirm whether bruce_wayne and oliver_twist appear in the configuration.
>     
> - Verify whether either pool is assigned as a default pool (or referenced in a policy) on any virtual server.    

### Task

> - How many pools are on CentralRegion-bigip-01?
> - How many of those pools are not referenced by any Virtual Server?


---
# Ticket 05 – "Why Is My Application Not Performing the Same Across Regions?”

### Description

> An application owner for the app, _web-app-foo_, has observed inconsistent behavior across devices.
> 
> They would like the configuration reviewed for any anomalies.

### Context

> **Device Names:** EastRegion-bigip-01, WestRegion-bigip-01
> 
> **Virtual Server Name:** web-app-foo
> 
> App Servers and Ports:
> 
> - 10.1.20.100:80
> - 10.1.20.101:80
> - 10.1.20.102:80
> - 10.1.20.201:80
> - 10.1.20.202:80

### Action

> Use the AI Assistant and enter the following prompt:
> 
> > `Can you check the virtual servers named 'web-app-foo' and their associated objects across all BIG-IPs in all Data Centers to verify that the configuration is the same?`
> 
> (Tell the AI assistant yes to all datacenters if it asks you the scope of the question)
> 
> Review the information returned. Do you see any configuration differences between _EastRegion-bigip-01_ and _WestRegion-bigip-01_?
> 
> If differences are identified, validate them by navigating to: **Dashboard >> BIG-IP Device >> Device Virtual Server and Dashboard >> BIG-IP Device >> Device Pools**
> 
> Compare the configuration of _web-app-foo_ on each device.

### Task 

> Identify and explain the configuration differences that could impact application behavior.

---

# Ticket 06 – “Why is one web server so slow?”

### Description

> Operations has reported that the application behind _web_app_42_ feels sluggish at times. Initial checks indicate that one of the pool members is responding significantly more slowly than the others.
> 
> You have been asked to investigate the performance of the pool members behind _web_app_42_ and determine why one server appears slower than the rest.

### Context

> **Device Name:** EastRegion-bigip-01
> 
> **Virtual Server:** /Common/web_app_42
> 
> **Pool:** web-pool

### Action

Traditionally, you would examine connection and request distribution using the BIG-IP GUI or CLI, for example:

> `tmsh show ltm pool /Common/web-pool members`

Use the AI Assistant and enter the following prompt:

> `Show all pool statistics for /Common/web-pool on EastRegion-bigip-01.`

Compare these results to the TMUI interface on EastRegion-bigip-01 under:

> **Local Traffic >> Pools >> web-pool**

Determine whether the AI results match the TMUI data. If necessary, refine the prompt.

### Task

 > - Identify which specific pool member appears slower based on available metrics


---

# Ticket 07 – “Security test shows WAF blocks Struts exploit attempt”

### Description

> The security team wants confirmation that the WAF is capable of detecting Apache Struts exploit attempts.
> 
> You have been asked to run a safe test request and demonstrate that the WAF properly detects and blocks the attack.

### Context

> **Device Name:** EastRegion1-bigip-01
> 
> **VIP:** /Common/web_app_42
> 
> **Policy:** /Common/my_afm_policy (Java/Struts signatures enabled)
> 
> **Application URL:** /upload.action

### Action

> Navigate to:
> 
> > **udf.f5.com GUI >> Deployment >> F5 Insight >> ACCESS >> WEB SHELL**
> 
> Send the following Struts-style test request:
```
curl -k -v \
  "https://ast66.demo.f5/upload.action" \
  -H 'Content-Type: ${(#_="multipart/form-data").(#context["com.opensymphony.xwork2.dispatcher.HttpServletResponse"].addHeader("X-Struts-POC","1"))}' \
  --data-binary 'test'
```
> Review the information returned from that request:
> 
> - IP address of FQDN “ast66.demo.f5” (Can be found near the top of output)
> - Response code of 200
> - Response html that says “Request Rejected”
> - Support ID in response (Can be found in the html of the response)
> 
> Use the AI Assistant and enter the following prompt:
> 
> > `Check for WAF events from "EastRegion-bigip-01" in the Antarctica datacenter over the last 14 days.`
> 
> Review the returned information.

### Task

> - Provide the “Incident Type” and number of “Requests Blocked”

Review the following for additional information and trends:

> - Summary of WAF-related events for that source IP
> - Key findings summary provided by the AI agent

---

# Ticket 08 – Update BIG-IP instance (Part 3/3)

### Description

> The installation on the first HA pair is completed. Your team has validated that the system is healthy and config are loaded correctly.
> 
> The next step is to begin installation on the second instance. This time, use the REST API to resume install.

### Action

> Resume Installation using REST API

udf.f5.com GUI: > Deployment > F5 Insight 01 (Primary) > ACCESS > WEB SHELL
```sh
ACCESS_TOKEN=$(curl -k -s -X POST "https://localhost/api/auth/login" \
     -H "Content-Type: application/json" \
     -d "{ \"username\": \"admin\", \"password\": \"HelloUDF\" }" | jq -r '.access_token')
    

PAUSED_JOB_ID=$(curl -k -s -X GET "https://localhost/api/v1/fleet/upgrade/jobs/software-installation?status=PAUSED" \
     -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    | jq -r '.data | sort_by(.updatedAt) | reverse | .[0].id')


curl -X POST "https://localhost/api/v1/fleet/upgrade/jobs/software-installation/${PAUSED_JOB_ID}/resume" \
     -H "Authorization: Bearer ${ACCESS_TOKEN}" \
     -k | jq
```

To confirm the installation has resumed:
```sh
curl -X GET "https://localhost/api/v1/fleet/upgrade/jobs/software-installation" \
     -H "Authorization: Bearer $(curl -k -s -X POST "https://localhost/api/auth/login" \
                                      -H "Content-Type: application/json" \
                                      -d "{ \"username\": \"admin\", \"password\": \"HelloUDF\" }" \
                                      | jq -r '.access_token')" \
     -k | jq
```
