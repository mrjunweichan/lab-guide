## Upgrade BIG-IP Instance

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
