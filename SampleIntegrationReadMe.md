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
<br />

## **Integrations Bundle Summary**
This integration serves as a bi-directional data flow betweeen the University and `AcmeTech Solutions`. \
It contains two integrations: 
- `University Population Biodemo`
- `Roster Refresh`

Summarized below

## **University Population Biodemo**
### Integration Summary
This integration calls the `PeopleSoft` queries: `UNIVERSITY_GREEK_LIFE` & `UNIVERSITY_STUDENTS_BIODEMO`, processes the data to filter out irrelevant data, and merges the remaining into a `CSV` file for drop off on AcmeTech Solution's `SFTP` server.

### Technical Information
* Job starts on a cron, or manually triggerable on `Trigger.Uni_Pop_Biodemo`

* Call `PSQuery`: `UNIVERSITY_GREEK_LIFE`

* Remove all non-active Greek life members

* Calls `PSQuery`: `UNIVERSITY_STUDENTS_BIODEMO`

* Remove all non-Greek life members

* Perform a union join of the data  

* Create a `CSV` file called `University_Students_Biodemo.CSV`

* Archive a copy of the `CSV` file to the volume mount dedicated for this integration (see `uni-greek_life-pvc`)

* Send the `CSV` to AcmeTech Solutions `SFTP` server

### Known Issue(s)
* No known/common issues with the integration at this time.

---
<br />
<br />

## **Roster Refresh**
### Integration Summary
This integration serves the purpose of updating the University's `Greek Life` roster for current and active members. It calls a few of `AcmeTech Solution`'s API endpoints to compile data into a file for import into PeopleSoft.

### Technical Information
* Job starts on a cron, or manually triggerable on `Trigger.Greek_Roster_Refresh`

* Call `AcmeTech Solution` API V2 for a list of Greek organizations

* Call `AcmeTech Solution` API V3  for a list of Greek organizations

* Calls PSQuery: `UNIVERSITY_STUDENTS_BIODEMO`

* Remove all non-Greek life members

* Perform a union join of the data  

* Create a `CSV` file called `University_Students_Biodemo.CSV`

* Archive a copy of the `CSV` file to the volume mount dedicated for this integration (see `uni-greek_life-pvc`)

* Send the `CSV` to AcmeTech Solutions `SFTP` server


## Manual Triggers

| Job                                                 | Trigger
|-----------------------------------------------------| ------------------------
| Manually call PSQuery and drop the Engagement file  | Send any message to `direct:Trigger.queryBiodemoDatafromCS`
| Manually trigger file resend (timestamped to the second) | Send any message to `seda:Trigger.sororityFraternityLifeUpload`


## Common Problems

**Problem:** For unknown reasons we sometimes receive this error while reading OMCS SFTP.

**Immediate Resolution:**  Manually retrigger job named `Do the thing`.

**Possible Long-Term Resolution:** Engage OMCS to diagnose the issue on their end.  Failing that,
add more aggressive retry policies to SFTP calls.

**Problem:** We were unable find sufficient basketweaving materials data in Campus Solutions to complete the nightly batch job.

**Resolution:** Inform PS Admins they need to load more wicker data and get back to us, so we can re-trigger job `loadBasketData`.

**Possible Long-Term Resolution:** Proactively detect the error and send alerts to an email list for relevant staff.

## OPENSHIFT DEPLOYMENT ARTIFACTS

| Artifact     | Name
|--------------| ------------------------
| Config Map   | `ilstu-engage-config`
| Config Map   | `ilstu-logging-config`
| Secret       | `ilstu-ideacenter-sftp`
| Volume Mount | `ilstu-sait-pvc`

*NOTE:* IDEA (CampusLabs's Course Evaluation Tool) ergo the `ilstu-ideacenter-sftp` secret is shared

**Deployment Diagram**
~~~~
WorkStation ->  gitlab      -> Openshift Dev  (intd) [Manual approval]
                            -> Openshift Test (intt) [Manual approval]
              (if tagged)   -> Openshift Prod (pint) [Manual approval]
~~~~
Deployment of this program does require the above configmaps and secrets to be available in the deployment namespace beforehand.

[Deployment Procedure](https://docs.illinoisstate.edu/x/oAE3Aw)


## Primary Users
- Justin Smith
- Adam Listek

## Project Team
- ESB Developer: Kaiser Ahmed
- PS Developer: Jamie Turner 
- Project Manager: Dave Hyland

## OPENSHIFT
