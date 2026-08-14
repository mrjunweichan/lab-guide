## Task 01 – Explore Top N

### Description

> The Top N dashboard provides a consolidated view of the most active and resource-intensive objects on a BIG-IP.
> 
> This ticket is designed to familiarize you with what information is available and how it can be used to quickly identify busy applications, high resource consumers, and potential availability concerns.
> 
> Your goal is to explore the Top N dashboard and understand the types of operational insights it provides.

### Tasks

> Navigate to:
> 
> > **Dashboards >> BIG-IP Device >> Top N**


From the Device dropdown at the top of the page, select the BIG-IP specified below.

> **CentralRegion-bigip-01**


### Deliverables

> Briefly answer the following:
> 
> > - Which VIP has the highest WAF CPU Utilization on CentralRegion-bigip-01?


---

# Task 02 – Identifying Orphaned Objects

## Title: “What are these unused pools and nodes?”

### Description

> During a routine review of the _CentralRegion-bigip-01_ configuration, operations suspects there may be unused (orphaned) objects left over from previous testing or decommissioned applications.
> 
> You have been asked to identify any orphaned pools on _CentralRegion-bigip-01_ so they can be documented and, if appropriate, cleaned up later.

### Context

> **Device Name:** CentralRegion-bigip-01
> 
> These objects are believed not to be referenced by any active virtual servers.

### Tasks

> Use the AI Assistant located at the top of the home screen and enter the following prompt:
> 
> `Show all pools and nodes on CentralRegion-bigip-01, and indicate which ones are not referenced by any virtual server.`
> 
> From the returned information and the TMUI on CentralRegion-bigip-01:
> 
> > - Navigate to: **Local Traffic >> Pools >> Pool List**
> >     
> >     Confirm whether bruce_wayne and oliver_twist appear in the configuration.
> >     
> > - Verify whether either pool is assigned as a default pool (or referenced in a policy) on any virtual server.
> >     
> 
> Summarize which of the above pools and nodes are truly orphaned (that is, not referenced by any virtual server or pool). Do not delete anything as part of this exercise. The goal is only to locate and document orphaned objects.

### Deliverables

> Briefly answer the following:
> 
> > - How many pools are on CentralRegion-bigip-01?
> > - How many of those pools are not referenced by any Virtual Server?


---
# Task 03 – Verify Application Configuration Consistency Across BIG-IPs

## Title: “Why Is My Application Not Performing the Same Across Regions?”

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

### Tasks

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
> Compare the configuration of _web-app-foo_ on each device:
> 
> > Identify and explain the configuration differences that could impact application behavior.

---

# Task 04 – Upgrade BIG-IP Instance

#### **1. Distribute new version to HA Pair**

**Navigate to:** Manage > Automation > Jobs  


**Click on:** Add Job > "Software Distribution"  


**Fill Form:**  
Job Name:  17.1.3.4 Distribution  

Instances:
  - bigip/NorthRegion-bigip-01.udf.labs  
  - bigip/NorthRegion-bigip-02.udf.labs  

Software to Distribute:  BIGIP-17.1.3.4-0.0.12.iso  

Distribution Type:  Serial (Rolling Execution)  

  
**Click on:** "Check Instances"  


**Click on:** "Execute Job"  


---

**To View Distribution Progress**

Click on execution number next to "Job name" 
Click on execution ID of running job
View the progress of distribution job


Alternatively, use the REST API to view Distribution Jobs
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

#### **2. Install new version to HA Pair**

**Navigate to:** Manage > Automation > Jobs  


**Click on:** Add Job > "Software Installation"  


**Fill Form:**  
Job Name: 17.1.3.4 Installation  

Installation Type: HA Pair

Execution Type: Serial (Rolling Execution)  

Checkbox: Only select the 2 boxes below (leave the rest un-checked)  
  - Automatic failback after both instance in HA Pair upgraded  
  - After reboot of Standby instance, before failover  

Target Distribution: BIGIP-17.1.3.4-0.0.12.iso  

Set Target Volume: Next Sequential (New)  

Instances:  
  - bigip/NorthRegion-bigip-01.udf.labs  
  - bigip/NorthRegion-bigip-02.udf.labs  

**Click on:** "Run Checks"  

**Click on:** "Execute Job"  

**Note:** 
> F5 Insight will install on first instance first and will pause when done.

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
  
Resume Installation using REST API
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
