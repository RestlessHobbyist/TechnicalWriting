# **Greek Chapters Roster - AcmeTech Solutions**
Mock Internal Integration Documentation (ReadMe.md)
<br />
<br />

## **Relevant Contacts:**
### Project POCs:
* Technical Analyst - Jane Alt
* Business Analyst - Tom Write
* Database Administrator - John Smith
* Integrations Developer - Cristian Sandu

### Functional POCs:
* RSO Advisor - Terrance James
* Greek Life Coordinator - River Song

### AcmeTech POCs:
* Lead Client Relations - Bugsly Rabbot
* Technical SME - Don Quail
<br />

## **Integrations Bundle Summary**
This integration serves as a bi-directional data flow betweeen the University and `AcmeTech Solutions`. \
It contains two integrations: 
- `University Population Biodemo`
- `Roster Refresh`

Summarized below

---
<br />

## **University Population Biodemo**
### **Integration Summary**
This integration calls the `PeopleSoft` queries: `UNIVERSITY_GREEK_LIFE` & `UNIVERSITY_STUDENTS_BIODEMO`, processes the data to filter out irrelevant data, and merges the remaining into a `CSV` file for drop off on AcmeTech Solution's `SFTP` server.

### Technical Information
* Job starts on a cron, or manually triggerable on `Trigger.Uni_Pop_Biodemo`

* Call `PSQuery`: `UNIVERSITY_GREEK_LIFE`

* Remove all non-active Greek life members

* Call `PSQuery`: `UNIVERSITY_STUDENTS_BIODEMO`

* Remove all non-Greek life members

* Perform a union join of the data  

* Create a `CSV` file called `University_Students_Biodemo.CSV`

* Archive a copy of the `CSV` file to the volume mount dedicated for this integration (see `uni-greek_life-pvc`)

* Send the `CSV` to AcmeTech Solutions `SFTP` server

### Known Issue(s)
* No known/common issues with the integration at this time.
---
<br />

## **Roster Refresh**
### **Integration Summary**
This integration serves the purpose of updating the University's `Greek Life` roster for current and active members. It calls a few of `AcmeTech Solution`'s API endpoints to compile data into a file for import into PeopleSoft.

### Technical Information
* Job starts on a cron, or manually triggerable on `Trigger.Greek_Roster_Refresh`

* Call `AcmeTech Solution` API V2 for a list of Greek organizations

* Call `AcmeTech Solution` API V3 for each org to get positions in that org

* Merge all rows on student ID, only keeping a list of `positions` for that ID

* Note that some `positions` are office held `positions`, others are their `member` status, others yet are `live-in` status

* Align `positions` to mapping requirements and create `CSV`

* Drop off to University `SFTP` server for Greek roster update

### Known Issue(s)
* Vendor has announced that they will be retiring V2 of the API "sometime" in the future. This means we will need to point to a V3 equivalent of that call once they have created it, making changes as necessary to the Endpoint URI and message body parsing.
---
<br />

## Manual Triggers

| Job                                                 | Trigger
|-----------------------------------------------------| ------------------------
| University Population Biodemo | Send any message to `seda:Trigger.Uni_Pop_Biodemo`
| Roster Refresh | Send any message to `seda:Trigger.Greek_Roster_Refresh`


## Common Problems

**Problem:** The PSQuery endpoint for the `University Population Biodemo` run, sometimes gets an influx of requests and cannot handle our call immediately.

**Immediate Resolution:**  If issue is observed during business hours we can manually trigger the job and hope the endpoint has the resources to handle our request

**Possible Long-Term Resolution:** We implement a retry policy for up to an hour. If within an hour of retries we cannot get a response we fail and log an issue reaching the PSQuery endpoint.

## OPENSHIFT DEPLOYMENT ARTIFACTS

| Artifact     | Name
|--------------| ------------------------
| Config Map   | `uni-greekchapters-config`
| Config Map   | `uni-logging-config`
| Secret       | `uni-acmetech-sftp`
| Volume Mount | `uni-unigreekroster-pvc`

<br />


**Deployment Diagram**
~~~~
WorkStation ->  gitlab      -> Openshift Dev  (dev) [Manual approval]
                            -> Openshift Test (test) [Manual approval]
              (if tagged)   -> Openshift Prod (prod) [Manual approval]
~~~~
Deployment of this program does require the above configmaps and secrets to be available in the deployment namespace beforehand.

[Deployment Procedure](http://example.com/)
