# AuraComponentBase

AuraComponentBase is a super/base component for Salesforce Lightning component framework.
The intention is provide common functionalities to be used sub components. I am also trying to demonstrate an example of Lightning component coding standard, e.g. Component controller is used to manage UI level behaviours and /actions, while the helper class handles business logic and server side communication.   

Current functions:
* Display messages using lightning:notificationsLibrary.
* Show / hide spinner.
* Handle server action errors.


Usage:

Sample.cmp
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId" 
                description="Sample Sub Component"
                extends="c:AuraComponentBase"
                controller="SampleApexController"
                access="global" >
<aura:attribute name="recordId" type="String"/>
<aura:attribute name="items" type="List"/>
<aura:handler name="init" value="{!this}" action="{!c.doInit}"/>

</aura:component>


SampleController.js
({
	doInit : function(component, event, helper) {
		 //show spinner
     helper.togglerSpinner(component);
     var recordId = component.get('v.recordId');
     helper.loadItems(component, event, helper, recordId, function(result){
        component.set('v.items', result);
        //hide spinner
        helper.togglerSpinner(component);
     });   
	}
})

SampleHelper.js
({
	loadItems : function(component, event, helper, recordId, callback) {
        var action = component.get('c.loadItems');
        action.setParams({"recordId": recordId});
        
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (component.isValid() && state === "SUCCESS") {
                var result = response.getReturnValue();
                if (result && callback) {
                    callback(result);
                }
            } else {
                helper.handleActionError(component, response, function() {
                 	  console.log('Error on calling getLocation: ' + response);                                                               });
            }
        });
        $A.enqueueAction(action);
	}
})

SampleApexController
public with sharing class SampleApexController {
    @AuraEnabled
    public static List<Item__c> loadItems(Id recordId) {
        return [SELECT Id, Name FROM Item__c WHERE parentId__c = :recordId];
    }
}
