---
layout: post
title: Supercharge Process Builder with APEX
---

Process Builder is one of Salesforce's power tools of autommation. It is declarative development power and limitation. I found while working with builder that while it makes things simple, it is missing some basic things. Some things are not supported. One such example is how to set a record type ID for new created record in Process Builder?

One way to do this is hardcode the 18 digit ID, really not a solution considering sandboxes changes and generally bad practice.

There is a solution, PB can make a call to APEX class with [Invokable Methods](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_InvocableMethod.htm) to acomplish some task. Hey, I thought this will be perfect to enhance Process Builder with APEX method to get dynamic Record Types, at least based on Develeopr names. Theer are some limitation obviously to make this method

+ The invocable method must be static and public or global, and its class must be an outer class.
+ Only one method in a class can have the InvocableMethod annotation.
+ Triggers can’t reference invocable methods.
+ Other annotations can’t be used with the InvocableMethod annotation.

I created a `Invokable` method to get record type by name with expecteation I can call that in my process and get the value to use.
Class Method code below

```
@InvocableMethod(label='Get Record Type' description='Returns the Lead record type IDs for given Develper name.') 
public static Id getRecTypeLead(String devName){
	Map<String, Id> recordTypeIdMap = new Map<String, Id>();
	String sobjectType = 'Lead';
	List<RecordType> rtLst = [Select Id, Name, DeveloperName FROM RecordType 
                               WHERE  
                               SObjectType = :sobjectType];
        for(RecordType rt : rtLst){
        	recordTypeIdMap.put(rt.DeveloperName, rt.Id);
        }     
	return recordTypeIdMap.get(devName);
}

```

Ok Tested my method works as expected, now try to use it in my Process. I can see the method in a Process step and can set parameter value for Developer Name string. But here is a catch-22, Process Builder cannot use any of the method return values. If I was using this method in a Flow I can get a record ID. But in PB it is not possible. Need to consider another solution.

Given this limitation I decide to make a method to call to actually set/change a record type for a given record by PB. And to be more generic define parameters object to set as input for my Process so I can set any record this way.

```
global class PBRecordTypeSelector {

    // Update given record with record requested Object Type, Record Name and data record ID
    // Result of this method - given data Record is updated with Record Type
    @InvocableMethod(label='Set Record Type' description='Update record type ID for given Object by Developer Name.')     
    global static void setRecordType(List<RecordTypeRequest> recType){
        Map<String,Id> recordTypeIdMap = new Map<String,Id>();
        String obj = recType[0].getObjectType();
        String devName = recType[0].getDeveloperName();
        String recId = recType[0].getRecordID();
		List<RecordType> rtLst = [Select Id, Name, DeveloperName 
                                  FROM RecordType 
                               		WHERE  SObjectType = :obj];
        for(RecordType rt : rtLst){
        	recordTypeIdMap.put(rt.DeveloperName, rt.Id);
        }     
		if (recordTypeIdMap.containsKey(devName)){
        	// Find SObject with ID and set its record type      	
            String query = 'SELECT Id, Name, RecordTypeId FROM ' + obj + ' WHERE Id = \'' + recId+'\'';       
            List<SObject> lst = Database.query(query);
			SObject record = (SObject)lst[0];
			record.put('RecordTypeId', recordTypeIdMap.get(devName));
            update record;
        }
    }
    
    // Request object with required parameters to set record types
    global class RecordTypeRequest {
        @InvocableVariable(required=true)
        global ID recordId;

        @InvocableVariable(required=true)
        global String objectType;
        
        @InvocableVariable(required=true)
		global String developerName;
        
        global String getObjectType(){return objectType;}
        global String getDeveloperName(){return developerName;}
        global String getRecordID(){return recordId;}
    }
}
```

Now if we make a Process step action to call APEX method `setRecordType` we can set values for Object type of Lead, Opprtunity or custom object name, Record Developer Name and ID for record to update.
Expected result that our recrd will have the correct record type from Process Builder action step.

 




 
