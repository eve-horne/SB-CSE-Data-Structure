# About SB-CSE Data Structure
This new and improved data structure takes the required data from CASE API and condenses it into a cohesive, lighter, and manageable format. 
Because this format is based on JSON, a Document Store Database would be the ideal implementation method. (e.x. MongoDB or Couchbase)
## JSON:
``` JSON {  
   "claim":{  
      "title":"string",
      "claimNumber": "string",
      "grade":"string",
      "subject":"string",
      "description":"string",
      "shortCode":"string",
      "domain": "string",
      "targets":[  
         {  
            "title":"string",
            "shortCode":"string",
	    "description": "string",
            "standards":[  
               "string"
            ],
            "stdDesc":[  
               "string"
            ],
            "DOK":[  
               "string"
            ],
            "DOKDesc":[  
               "string"
            ],
	    "shortDOK": [
	    	"string"
	    ],
            "type":"string",
            "clarification":"string",
            "heading":"string",
            "evidence":[  
               "string"
            ],
            "vocab":"string",
            "tools":"string",
            "stimInfo":"string",
            "devNotes":"string",
            "complexity":"string",
            "dualText":"string",
            "accessibility":"string",
            "stem":[  
               "string"
            ],
            "taskDescription":[  
               "string"
            ],
            "taskModel":[  
               "string"
            ],
            "examples":[  
               "string"
            ],
            "rubrics":[  
               "string"
            ],
            "stimulus":[  
               "string"
            ],
            "shortStem":[  
               "string"
            ],
            
         }
      ]
   }
}
```



## Claim:

|  **Key**  | **Description** | **CASE API Scope** | **Data Location** | 
|  :------ | :------ | :------ | :------ | 
|  title (String)  | Claim Title | getAllCFDocuments() | ``CFDocument[index].title`` | 
| claimNumber (String)| Claim # | getAllCFDocuments() | ``CFDocument[index].title`` |
|  grades [String] | Grade # 01-12 also includes HS for 09-12 | getAllCFDocuments() | ``CFDocument[index].notes`` |
|  subject (String) | "Mathematics" or  "English Language Arts" | getAllCFDocuments() | ``CFDocument[index].subjectURI[0].title`` |
|  description (String) | Claim Fulltext | getCFPackage(SB ELA/Math Content Spec) | ``CFItem[indexOfClaim].fullStatement`` |
|  shortCode (String) | Claim shortname | getCFPackage(SB ELA/Math Content Spec) | ``CFItem[indexOfClaim].abbreviatedStatement`` | 
|  domain (String) | Optional Domain Fulltext | getCFPackage(SB ELA/Math Content Spec) | ``CFItem[indexOfDomain].fullStatement`` | 
|  targets [String] | Array of target objects | getCFPackage(Target Document) | see below | 

## Target:

|  **Key** | **Description** | **CASE API Scope** | **Data Location** |
|  :------ | :------ | :------ | :------ |
|  title (String) | Target Name | getCFPackage(Claim-Target Document) | ``CFDocument.title`` |
|  shortCode (String) | Target shortname | getCFPackage(Claim-Target Document) | ``CFDocument.notes`` |
|  description (String) | Target Fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfTarget].fullStatement`` |  
|  standards [String] | Array of standard identifiers | getCFPackage(Claim-Target Document) | ``CFItems[idxOfMeasuredSkill].humanCodingScheme`` |
|  stdDesc [String] | Fulltext Description of above standard | getCFPackage(CCSS Import Doc) | ``CFItems[idxOfStdIdentifier].fullStatement`` |
|  DOK [String] | Depth of knowledge fulltext WIP | getCFPackage(SB ELA/Math Content Spec) | ``CFAssociations[idxOfTarget].destinationNodeURI.uri`` |
|  type (String) | CAT, "PT", or "Both" - Math only has CAT - | getCFPackage(Claim-Target Document) | ``CFDocument.creator`` |
|  clarificiation (String) | Clarification fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfClarification].fullStatement`` |
|  heading (String) | Section Heading fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfHeading].fullStatement`` |
|  wvidence [String] | Array of Required Evidence fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfEvidenceReq].fullStatement`` |
|  vocab (String) | key/construct vocabulary | getCFPackage(Claim-Target Document) | ``CFItems[idxOfGeneralReqs].fullStatement`` |
|  tools (String) | allowed tools | getCFPackage(Claim-Target Document) | ``CFItems[idxOfGeneralReqs].fullStatement`` |
|  stimInfo (String) | Stimuli info | getCFPackage(Claim-Target Document) | ``CFItems[idxOfGeneralReqs].fullStatement`` |
|  devNotes (String) | Developer notes | getCFPackage(Claim-Target Document) | ``CFItems[idxOfGeneralReqs].fullStatement`` |  
|  complexity (String) | Stimuli complexity | getCFPackage(Claim-Target Document) | ``CFItems[idxOfGeneralReqs].fullStatement`` |
|  dualText (String) | Dual Text complexity | getCFPackage(Claim-Target Document) | ``CFItems[idxOfGeneralReqs].fullStatement`` |
|  accessibility (String) | Accessibility fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfAccessibility].fullStatement`` |
|  taskModel [String] | Array of task model fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfTaskModel].fullStatement`` |  
|  taskDescription [String] | Parallel array to taskmodel of description fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfTaskDesc].fullStatement`` | 
|  stimulus (String) | Stimulus Fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfStimulus].fullStatement`` | 
|  stem [String] | Array of Stems fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfStems].fullStatement`` |  
|  example [String] | Array of TaskModel Examples | getCFPackage(Claim-Target Document) | ``CFItems[idxOfExample].fullStatement`` |  
|  rubric [String] | Array of TaskModel Scoring rules Fulltext | getCFPackage(Claim-Target Document) | ``CFItems[idxOfRubric].fullStatement`` |  

## From CASE to SB-CSE Data Structure (Step-by-step):
This section describes exactly what is required to scrape data from CASE to our new structure. This guide should be referenced when designing a script to scrape all data from CASE for our new DB.


### Step 1: Getting All Content Spec Documents

* Make a GET Request to CASE's getAllCFDocuments() API Call Ex:

``
https://case.smarterbalanced.org/ims/case/v1p0/CFDocuments?limit=99999999999&offset=0&sort&orderBy&filter&fields
``

You'll notice that the ``Limit`` parameter is 999999999, this is just to ensure the API call returns ALL of the Content Spec Documents.

*Note: CASE will always return 2-3 less entries than specified in the ``Limit`` parameter.*

* The Response Body will be an array of all the CFDocument objects stored in the CASE DB

The majority of these will be SmarterBalanced Spec Documents with titles containing Subject, Grade, Claim, and Target.

*However*, a few of these Documents will **not** be Grade/Claim/Target, but still important.
Mainly these 4 Documents:

* **Smarter Balanced ELA Content Specification** 	(GUID: ``cbbb8d01-63fc-4adf-9803-a3781839b6b6``)
* **Smarter Balanced Math Content Specification** (GUID: ``255adad7-2854-47d7-9aa4-c5e1750eb8ca``)
* **CCSS Imported from Digital Library** (GUID: ``1a66f1ae-9f7d-4c86-9e24-d0f022279578``)
* **Norm Webb's Depth of Knowledge [DOK] Levels of Cognitive Difficulty** (GUID: ``b7bc9d8c-cb5f-4dc8-96da-13ce635053d1``)

Each of these 4 documents will come in handy later when we need information like Standards and Depth of Knowledge.


### Step 2: Parsing Subject, Grade, Claim, and Target Values from these Documents
We can now iterate through the array of CFDocuments to scrape their Subject, Grade and Claim information.

* Subject can be found in ``CFDocument[index].subjectURI[0].title`` *(Always use 0th index for subjectURI)*

* Grade, Claim, and Target can be found in ``CFDocument[index].title``

* The string found in ``CFDocument[index].title`` will be in one of the following formats:

#### ELA:
```
1. English Language Arts Specification: Grade *X* Claim *Y* Target *Z*

or

2. English Language Arts Performance Task Specification: Grade *X* "Performance Task Name"
```
 


#### Math:
```
1. Grade *X* Mathematics Item Specification C*Y* T*Z*

or

2. *HS* Mathematics Item Specification C*Y* T*Z*

or

3. Math Grades *X-X*, Claim *Y*

or

4. Math *High School*, Claim *Y* 
```

* **X** = Grade (Either INT from 3-8 **or** String HS/High School to represent 9-12)

* **Y** = Claim (INT 0-9)

* **Z** = Target (If ELA: INT 0-99, If Math: Char A-Z)



### Step 3: Getting Claim Description, Domain and Shortname
To get the fulltext description for a claim, its Domains, and a shortname that will be useful later, we need to move our scope from getAllCFDocuments() to getCFPackage().

* Make a GET Request to CASE's getPackage() API Call with the GUID of your subject's corresponding content spec document as a parameter (remember the 4 important documents from above?)

* **Smarter Balanced ELA Content Specification** 	(GUID: ``cbbb8d01-63fc-4adf-9803-a3781839b6b6``)
* **Smarter Balanced Math Content Specification** (GUID: ``255adad7-2854-47d7-9aa4-c5e1750eb8ca``)

**Example:**
**ELA Content Spec as Parameter:**

```
https://case.smarterbalanced.org/ims/case/v1p0/CFPackages/cbbb8d01-63fc-4adf-9803-a3781839b6b6
```

The Response Body will have a singular CFDocument object containing mostly unimportant information like the title of this document and its subject.
However, it also returns an array of CFItems that contain important information about claims like their descriptions, shortnames, and domains.

The main difficulty here is distinguishing "Claim" CFItems from other CFItems like "Measured Skills" and "Components"

* Filter out any CFItems that are not of type "Claim". 
* Item Types are found at ``CFItems[index].CFItemType``
* Claim Descriptions are found at ``CFItems[indexOfClaim].fullStatement``
* Claim Shortnames are found at ``CFItems[indexOfClaim].humanCodingScheme``
* The Domain **MIGHT** be found at ``CFItems[indexOfClaim + 1].fullStatement`` **IF**

``CFItem[indexOfClaim + 1].CFItemType === "Domain"``

Domains are optionally included with claims.

#### Code Example:
``` javascript

for (var i = 0; i < CFItems.length; i++) {

	if (CFItems[i].CFItemType === "Claim") {
	
		console.log(CFItems[i].fullStatement);
		console.log(CFItems[i].humanCodingScheme);
		
		if(CFItems[i+1].CFItemType === "Domain") {
		
			console.log(CFItems[i+1].fullStatement);
			
		}
	}	
		
}
```


### Step 4: Getting Target Document Information
So far we've covered the first section of the new SB-CSE Data Format, mainly Claims. The next big step is getting our Targets information.
From here, we'll be using getAllCFDocuments() to get our target GUIDs as parameters for getCFPackage().

#### In getAllCFDocuments():

* Find the index of the Target Document you want information from. This requires filtering based on values in the ``CFDocument[index].title`` string. 
* Find the GUID of the Target Document you found from the above step. This can be found at ``CFDocuments[index].identifier``
* Use that GUID in the getCFPackage() section below.

#### In getCFPackage()
* Make a GET Request using the above Target Document GUID as a parameter.

**Example using GUID of ELA Grade 3, Claim 1, Target 9:**
```
https://case.smarterbalanced.org/ims/case/v1p0/CFPackages/444c5d50-ceb4-11e7-9cd4-75d19dc0a557
```
The Response Body will have a Singular CFDocument containing basic info about the document (title, grade, subject, etc)
It will also have an array of CFItems containing almost everything we need to fill the rest of our Data structure.

From here, referencing the above table should be helpful in filling out the rest of the Data Structure.

### Brief Summary of each key and value:

#### Easy:
* Target Title can be found at ``CFDocument.title``
* Target Shortname can be found at CFDocument.notes
* KeyDetails can be found at ``CFItem[index].fullStatement`` where the ``CFItem[index].CFItemType === "Target"``
* Standards Entries can be found at each ``CFItem[index].humanCodingScheme`` where the ``CFItem[index].CFItemType === "Measured Skill"``
* Descriptions for above standards can be found at ``CFItem[index].fullStatement`` where the index is the same as above.
* Type can be found at ``CFDocument.creator``. This string will contain "CAT" or "PT"
* Clarification can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Clarification"``
* Heading can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Section Heading"``
* Evidence entries can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Evidence Required"``
* Accessibility can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Accessibility"``
* TaskModel Entries can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Task Model"``
* TaskDescription entries can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Task Description"``
* Stimulus can be found at ``CFItems[index].fullStatement`` where the ``CFItem[index].CFItemType === "Stimulus"

#### Not-So-Easy:
**The following items all share the same CFItemType "General Requirements" but have very different values and sub-types:**

* Vocab can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].abbreviatedStatement === "Key\Construct Relevant Vocabulary"``
* Tools can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].abbreviatedStatement === "Allowable Tools"``
* StimInfo can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].abbreviatedStatement === "Stimuli/Passages"``
* DevNotes can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].abbreviatedStatement === "Development Notes"``
* Complexity can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].abbreviatedStatement === "Stimuli/Text Complexity"``
* DualText can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].abbreviatedStatement === "Dual-Text Stimuli"``


**The following items are related to TaskModel entries:**

* Stem entries can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].CFItemType === "Stem"``

If the ``CFItemType`` at the index immediately following the above Stem is "Stem", this will be a Dual-Text Stem. It will also have "Appropriate Stems for Dual Text Stimuli" as its ``abbreviatedStatement``

* Example Entries can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].CFItemType === "Example"``
* Rubric Entries can be found at ``CFItems[index].fullStatement`` where ``CFItems[index].CFItemType === "Rubric"``

The order that these items are entered into the array matter because they are parallel to the TaskModel Array.
Each Task Model has a regular Stem, a Dual-Text Stem, an Example, and a Rubric.
So the Stem array should have 2x as many entries as TaskModel, Example, or Rubric, does.


### Step 5: Getting Depth of Knowledge
This can get pretty difficult as **sometimes** DOK information is there for a target, and **sometimes** its not.

Essentially, you want to find the ``CFItem[index].humanCodingScheme`` where ``CFItemType === "Target"``
then call getCFPackage on the corresponding subject's Content Specification:
* **Smarter Balanced ELA Content Specification** 	(GUID: ``cbbb8d01-63fc-4adf-9803-a3781839b6b6``)
* **Smarter Balanced Math Content Specification** (GUID: ``255adad7-2854-47d7-9aa4-c5e1750eb8ca``)

Then find all indexes that have  ``CFAssociations[index].originNodeURI.title === "your-shortname-here"``
Then find all CFAssociations[j].destinationNodeURI.uri that follows this format:

```
urn:fdc:edplancms.com:2016:cfr:DOK:X-DOK%YYY
```
* **X** is the DOK Type (Types are listed in the DOK Document Package)
* **YYY** is the DOK Level (Typically in 20*Y* format where the last digit is the actual level)

**Parsing the value example:**


In getCFPackage(ELA Grade 3 Claim 1 Target 9):

``CFItems[index].humanCodingScheme = "E.G3.C1RI.T9"``

In getCFPackage(Smarter Balanced ELA Content Specification):

``` javascript
for(var i = 0; i < CFAssociations.length(); i++) {

	if (CFAssociations[i].originNodeURI.title === "E.G3.C1RI.T9") {
	
		if(CFAssociations[i].destinationNodeURI.uri.includes("DOK%") {
			console.log(CFAssociations[i].destinationNodeURI.uri);
		}
	}
}
```
**Output:**

```
urn:fdc:edplancms.com:2016:cfr:DOK:R-DOK%202
urn:fdc:edplancms.com:2016:cfr:DOK:R-DOK%203
```

This means that our DOKs are **R-DOK2** and **R-DOK3**

* The DOK fulltexts can be found in the DOK Document Package at ``CFItem[index].fullStatement`` where ``CFItem[index].CFItemType === "R-DOK2"`` or ``"R-DOK3"``
