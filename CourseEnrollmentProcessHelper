// Method called from Course Enrollment Process to create or update 'Coach - Level' Accomplishments
    // NOTE: There is only ever 1 'Coach - Level' Accomplishment record per Contact
    @InvocableMethod(label='Update Coach Accomplishments' description='Creates or Updates Coach Accomplishment from Course Enrollment' category= 'Course Enrollments')
    public static void updateCoachAccomplishments(List<docebo_v3__CourseEnrollment__c> courseEnrollments) {
        // START -- System.debug to log input size by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18
        System.debug('courseEnrollments size: ' + courseEnrollments.size());
        // END -- System.debug to log input size by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18

        Map<Id, docebo_v3__CourseEnrollment__c> contactEnrollmentMap = new Map<Id, docebo_v3__CourseEnrollment__c>();
        Map<Id, Engagement_Type__c> contactAccomplishmentMap = new Map<Id, Engagement_Type__c>();
        Set<Id> contactIds = new Set<Id>();
        Id coachAccomplishmentRT = Schema.SObjectType.Engagement_Type__c.getRecordTypeInfosByName().get('Coach - Level').getRecordTypeId();

        for(docebo_v3__CourseEnrollment__c ce : courseEnrollments){
            if(!String.isBlank(ce.Contact__c) && ce.docebo_v3__Status__c == 'Completed'){
                if (contactEnrollmentMap.containsKey(ce.Contact__c)){
                    // check level earned
                    // if less than the new record.. update map
                    if (contactEnrollmentMap.get(ce.Contact__c).Coach_Level_Earned_Numeric__c < ce.Coach_Level_Earned_Numeric__c){
                        contactEnrollmentMap.put(ce.Contact__c, ce);
                    }
                }else{
                    contactIds.add(ce.Contact__c);
                    contactEnrollmentMap.put(ce.Contact__c, ce);
                }
            }
        }

        // START -- System.debug to log enrollments associated to each contact by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-19
        System.debug('contactEnrollmentMap: ' + contactEnrollmentMap);
        // END -- System.debug to log enrollments associated to each contact by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-19

        // Added ORDER BY Contact__c in SOQL query by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-07-29
        for(Engagement_Type__c accomplishment : [SELECT Id, RecordTypeId, Contact__c, Coach_Level__c, Coach_Program_Start_Date__c, Coaches_Act_Reassessments_Date__c, L1_Training_Date__c, L1_Post_Tng_Document_Submission_Date__c, L1_Annual_Assessment_Completion_Date__c, 
                                                L2_Pre_Assessment_Date__c, L2_Training_Date__c, L2_Post_Tng_Document_Submission_Date__c, L2_Annual_Assessment_Completion_Date__c,Chapter_Trainer_Training__c,  
                                                L3_Pre_Assessment_Date__c, L3_Training_Date__c, L3_Post_Tng_Document_Submission_Date__c, Three_Year_Cycle_Start_Date__c, Three_Year_Cycle_End_Date__c, CEU_Status__c 
                                                FROM Engagement_Type__c WHERE Contact__c IN :contactIds AND RecordTypeId = :coachAccomplishmentRT ORDER BY Contact__c]){
            contactAccomplishmentMap.put(accomplishment.Contact__c, accomplishment);
        }

        Map<Id, Contact> contactMap = new Map<Id, Contact>([SELECT Id, Chapter_Affiliation__c, Active__c, Coach_Level__c FROM Contact WHERE Id IN :contactIds]);

        // loop through enrollment records
        for(docebo_v3__CourseEnrollment__c enrollment : contactEnrollmentMap.values()){
            // datetime to date
            DateTime dT = enrollment.docebo_v3__EnrollmentCompletionDate__c;
            Date completionDate = date.newinstance(dT.year(), dT.month(), dT.day());
           
      
           // IF ASSISTANT COACH TRAINING COMPLETED
           if(enrollment.Docebo_Course_Name__c == 'First Tee Assistant Coach Training'){
            // Does Accomplishment exist already?
            if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                // No Accomplishment, create one
                Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Assistant Coach');
                contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                // Update Contact
                contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Assistant Coach';
                contactMap.get(enrollment.Contact__c).Coach_Program_Start_Date__c = completionDate;
                contactMap.get(enrollment.Contact__c).Active__c = true;
            }else{
                // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Assistant Coach')){
                    // Update Accomplishment
                    Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                    eg.Coach_Program_Start_Date__c = completionDate;
                    eg.Coach_Level__c = 'Assistant Coach';
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Assistant Coach';
                    contactMap.get(enrollment.Contact__c).Coach_Program_Start_Date__c = completionDate;
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
            // Mark Contact as Active IF completion date is in the last 2 days
            if(completionDate > Date.today().addDays(-2)){
                contactMap.get(enrollment.Contact__c).Active__c = true;
            }
        }
              
            //IF JUNIOR COACH TRAINING COMPLETED
           else if(enrollment.Docebo_Course_Name__c == 'Junior Coach Training'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'None');
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    eg.Coach_Program_Start_Date__c = null;
                    contactAccomplishmentMap.get(enrollment.Contact__c).Junior_Coach_Completion_Date__c = completionDate;
    
                }
             // Mark Contact as Active IF completion date is in the last 2 days
             if(completionDate > Date.today().addDays(-2)){
                contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            } 
            // IF LEVEL 1 Post-Training COURSE COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 1 Knowledge Check'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Level 1 Completed');
                    eg.L1_Post_Tng_Document_Submission_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 1 Completed';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Level 1 Completed')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L1_Post_Tng_Document_Submission_Date__c = completionDate;
                        eg.Coach_Level__c = 'Level 1 Completed';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 1 Completed';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L1_Post_Tng_Document_Submission_Date__c = completionDate;
                    }
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
            // IF LEVEL 2 PRE-REQUISITE COURSE COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 2 Prerequisite' || enrollment.Docebo_Course_Name__c == 'Level II Prerequisite'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Assistant Coach');
                    eg.L2_Pre_Assessment_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Assistant Coach';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Assistant Coach')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L2_Pre_Assessment_Date__c = completionDate;
                        eg.Coach_Level__c = 'Assistant Coach';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Assistant Coach';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L2_Pre_Assessment_Date__c = completionDate;
                    }
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
        
            // IF LEVEL 2 POST TRAINING SURVEY COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 2 Post Training Survey' || enrollment.Docebo_Course_Name__c == 'Level II Post Training Survey'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Level 2');
                    eg.L2_Training_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 2';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Level 2')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L2_Training_Date__c = completionDate;
                        eg.Coach_Level__c = 'Level 2';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 2';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L2_Training_Date__c = completionDate;
                    }
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
            // IF LEVEL 2 POST TRAINING DOCUMENTS COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 2 Post Training Documents' || enrollment.Docebo_Course_Name__c == 'Level II Post Training Documents'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Level 2 Completed');
                    eg.L2_Post_Tng_Document_Submission_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 2 Completed';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Level 2 Completed')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L2_Post_Tng_Document_Submission_Date__c = completionDate;
                        eg.Coach_Level__c = 'Level 2 Completed';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 2 Completed';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L2_Post_Tng_Document_Submission_Date__c = completionDate;
                    }
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
              // IF LEVEL 2 ANNUAL KNOWLEDGE CHECK COMPLETED
              else if(enrollment.Docebo_Course_Name__c == 'Level 2 Annual Knowledge Check'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, null);
                    eg.L2_Annual_Assessment_Completion_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg); 
                      //contactMap.get(enrollment.Contact__c).Active__c = true
                 }else{
                    contactAccomplishmentMap.get(enrollment.Contact__c).L2_Annual_Assessment_Completion_Date__c = completionDate;
                }
             // Mark Contact as Active IF completion date is in the last 2 days
             if(completionDate > Date.today().addDays(-2)){
                contactMap.get(enrollment.Contact__c).Active__c = true;
            }
        }
            // IF LEVEL 3 PRE-REQUISITE COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 3 Prerequisite' || enrollment.Docebo_Course_Name__c == 'Level III Prerequisite'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Level 2 Completed');
                    eg.L3_Pre_Assessment_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 2 Completed';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Level 2 Completed')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L3_Pre_Assessment_Date__c = completionDate;
                        eg.Coach_Level__c = 'Level 2 Completed';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 2 Completed';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L3_Pre_Assessment_Date__c = completionDate;
                    }
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
            // IF LEVEL 3 POST TRAINING SURVEY COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 3 Post Training Survey' || enrollment.Docebo_Course_Name__c == 'Level III Post Training Survey'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Level 3');
                    eg.L3_Training_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 3';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Level 3')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L3_Training_Date__c = completionDate;
                        eg.Coach_Level__c = 'Level 3';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 3';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L3_Training_Date__c = completionDate;
                    }
                }
                 // Mark Contact as Active IF completion date is in the last 2 days
                 if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
              // IF LEVEL 4 POST TRAINING SURVEY COMPLETED
              else if(enrollment.Docebo_Course_Name__c == 'Level 4 Post Training Survey'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Chapter Trainer');
                    eg.Chapter_Trainer_Training__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Chapter Trainer'; 
                 }else{
                    contactAccomplishmentMap.get(enrollment.Contact__c).Chapter_Trainer_Training__c = completionDate;
                }
             // Mark Contact as Active IF completion date is in the last 2 days
             if(completionDate > Date.today().addDays(-2)){
                contactMap.get(enrollment.Contact__c).Active__c = true;
                }
              }
            // IF LEVEL 3 POST TRAINING DOCUMENTS COMPLETED
            else if(enrollment.Docebo_Course_Name__c == 'Level 3 Post Training Documents' || enrollment.Docebo_Course_Name__c == 'Level III Post Training Documents'){
                // Does Accomplishment exist already?
                // No Accomplishment, create one
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'The First Tee Coach');
                    eg.L3_Post_Tng_Document_Submission_Date__c = completionDate;
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                    // Update Contact
                    contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 3 Completed';
                    //contactMap.get(enrollment.Contact__c).Active__c = true;
                }else{
                    // If incoming Coach Level is higher than current Coach Level, update Accomplishment and Contact
                    if(isHigherCoachLevel(contactMap.get(enrollment.Contact__c).Coach_Level__c, 'Level 3 Completed')){
                        // Update Accomplishment
                        Engagement_Type__c eg = contactAccomplishmentMap.get(enrollment.Contact__c);
                        eg.L3_Post_Tng_Document_Submission_Date__c = completionDate;
                        eg.Coach_Level__c = 'Level 3 Completed';
                        contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                        // Update Contact
                        contactMap.get(enrollment.Contact__c).Coach_Level__c = 'Level 3 Completed';
                        //contactMap.get(enrollment.Contact__c).Active__c = true;
                    }else{
                        contactAccomplishmentMap.get(enrollment.Contact__c).L3_Post_Tng_Document_Submission_Date__c = completionDate;
                    }
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }
            }
            // IF LEVEL 3 ANNUAL KNOWLEDGE CHECK COMPLETED
            else if (enrollment.Docebo_Course_Name__c == 'Level 3 Annual Knowledge Check') {
              // Does Accomplishment exist already?
              if (!contactAccomplishmentMap.containsKey(enrollment.Contact__c)) {
                Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, null);
                eg.L3_Annual_Knowledge_Check_Date__c = completionDate;
                contactAccomplishmentMap.put(enrollment.Contact__c, eg);
              } else {
                contactAccomplishmentMap.get(enrollment.Contact__c).L3_Annual_Knowledge_Check_Date__c = completionDate;
              }
              // Mark Contact as Active IF completion date is in the last 2 days
              if (completionDate > Date.today().addDays(-2)) {
                contactMap.get(enrollment.Contact__c).Active__c = true;
              }
            }
            // CEU Three Year -- EVOL-905
            else if(enrollment.Docebo_Course_Name__c == 'CEU Three Year Cycle'){
                if(!contactAccomplishmentMap.containsKey(enrollment.Contact__c)){
                    Engagement_Type__c eg = newAccomplishment(completionDate, enrollment.Contact__c, contactMap.get(enrollment.Contact__c).Chapter_Affiliation__c, coachAccomplishmentRT, 'Assistant Coach');
                    eg.Three_Year_Cycle_Start_Date__c = completionDate;
                    eg.Three_Year_Cycle_End_Date__c = completionDate.addYears(3);
                    eg.CEU_Status__c = 'Current';
                    contactAccomplishmentMap.put(enrollment.Contact__c, eg);
                }else{
                    // accomplishment
                    contactAccomplishmentMap.get(enrollment.Contact__c).Three_Year_Cycle_Start_Date__c = completionDate;
                    contactAccomplishmentMap.get(enrollment.Contact__c).Three_Year_Cycle_End_Date__c = completionDate.addYears(3);
                    contactAccomplishmentMap.get(enrollment.Contact__c).CEU_Status__c = 'Current';
                }
                // Mark Contact as Active IF completion date is in the last 2 days
                if(completionDate > Date.today().addDays(-2)){
                    contactMap.get(enrollment.Contact__c).Active__c = true;
                }    
            }
        }


        // START -- Upsert/Update with retry logic by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-07-29
        List<Engagement_Type__c> accomplishmentList = contactAccomplishmentMap.values();
        List<Contact> contactList = contactMap.values();

        // lists to store failed upsert records
        List<Engagement_Type__c> failedAccomplishments = new List<Engagement_Type__c>();
        List<Contact> failedContacts = new List<Contact>();

        // START -- Store successful requests for logging by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18
        List<Engagement_Type__c> successfulAccomplishments = new List<Engagement_Type__c>();
        List<Contact> successfulContacts = new List<Contact>();
        // END -- Store successful requests for logging by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18

        Integer maxRetries = 3;
        Integer aRetries = 0;
        Integer cRetries = 0;

        // update contacts -- Coach Active and Coach Level using Database Class Method; AllOrNone flag false.
        Database.SaveResult[] cResults = Database.update(contactList, false);

        // cache failed contacts update
        for(Integer i = 0; i < cResults.size(); i++){
            if(!cResults[i].isSuccess()){
                failedContacts.add(contactList[i]);
            } else {
                successfulContacts.add(contactList[i]);
            }
        }

        // execute recursive retry logic for failed contacts
        while(failedContacts.size() > 0 && cRetries < maxRetries){
            List<Contact> retryBatch = failedContacts.clone();
            failedContacts.clear();

            Database.SaveResult[] retriedList = Database.update(retryBatch, false);

            for(Integer i = 0; i < retriedList.size(); i++){
                if(!retriedList[i].isSuccess()){
                    failedContacts.add(retryBatch[i]);
                } else {
                    successfulContacts.add(retryBatch[i]);
                }
            }

            cRetries++;
        }

        // START -- log successful and failed requests by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18
        System.debug('successful Contact updates: ' + successfulContacts.size() + ' records: ' + successfulContacts);
        System.debug('failed Contact updates: ' + failedContacts.size() + ' records: ' + failedContacts);
        // END -- log successful and failed requests by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18

        // If there are still failed upserts after exhausting retries, throw an exception.
        if(failedContacts.size() > 0) throw new AssertException('Error while retrying failed upserts -- contacts');
        // END -- Upsert/Update with retry logic by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-07-29

        // update accomplishments using Database Class Method; AllOrNone flag false.
        Database.UpsertResult[] aResults = Database.upsert(accomplishmentList, false);

        // cache failed accomplishments upserts
        for(Integer i = 0; i < aResults.size(); i++){
            if(!aResults[i].isSuccess()){
                failedAccomplishments.add(accomplishmentList[i]);
            } else {
                successfulAccomplishments.add(accomplishmentList[i]);
            }
        }

        // execute recursive retry logic for failed accomplishments
        while(failedAccomplishments.size() > 0 && aRetries < maxRetries){
            List<Engagement_Type__c> retryBatch = failedAccomplishments.clone();
            failedAccomplishments.clear();

            Database.UpsertResult[] retriedList = Database.upsert(retryBatch, false);

            for(Integer i = 0; i < retriedList.size(); i++){
                if(!retriedList[i].isSuccess()){
                    failedAccomplishments.add(retryBatch[i]);
                } else {
                    successfulAccomplishments.add(retryBatch[i]);
                }
            }

            aRetries++;
        }

        // START -- log successful and failed requests by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18
        System.debug('successful Engagement_Type__c upserts: ' + successfulAccomplishments.size() + ' records: ' + successfulAccomplishments);
        System.debug('failed Engagement_Type__c upserts: ' + failedAccomplishments.size() + ' records: ' + failedAccomplishments);
        // END -- log successful and failed requests by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-08-18

        // If there are still failed upserts after exhausting retries, throw an exception.
        if(failedAccomplishments.size() > 0) throw new AssertException('Error while retrying failed upserts -- accomplishments');

        // START -- Commented out by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-07-29
        // // update accomplishments
        // upsert contactAccomplishmentMap.values();

        // // update contacts -- Coach Active and Coach Level
        // update contactMap.values();
        // END -- Commented out by Patrick Fortaleza (Agilitek Solutions Ltd.) for Jira FTID-28 on 2022-07-29
    }

    /*
     * returns true if the incoming level is higher than the current level, false otherwise
     * 
     * Coach Level rankings from highest to lowest:
     * 
     *      national trainer
     *      chapter trainer
     *      level 3 completed
     *      level 3
     *      level 2 completed
     *      level 2
     *      level 1 completed
     *      level 1
     *      PLAYer
     *      assistant coach
     */

    @TestVisible
    private static Boolean isHigherCoachLevel(String current, String incoming){
        if(incoming == 'Assistant Coach'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed' && current != 'Level 3' && current != 'Level 2 Completed' && current != 'Level 2' && current != 'Level 1 Completed' && current != 'Level 1' && current != 'PLAYer');
        }else if(incoming == 'PLAYer'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed' && current != 'Level 3' && current != 'Level 2 Completed' && current != 'Level 2' && current != 'Level 1 Completed' && current != 'Level 1');
        }else if(incoming == 'Level 1'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed' && current != 'Level 3' && current != 'Level 2 Completed' && current != 'Level 2' && current != 'Level 1 Completed');
        }else if(incoming == 'Level 1 Completed'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed' && current != 'Level 3' && current != 'Level 2 Completed' && current != 'Level 2');
        }else if(incoming == 'Level 2'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed' && current != 'Level 3' && current != 'Level 2 Completed');
        }else if(incoming == 'Level 2 Completed'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed' && current != 'Level 3');
        }else if(incoming == 'Level 3'){
            return (current != 'Chapter Trainer' && current != 'National Trainer' && current != 'Level 3 Completed');
        }else if(incoming == 'Level 3 Completed'){
            return (current != 'Chapter Trainer' && current != 'National Trainer');
        }else if(incoming == 'Chapter Trainer'){
            return (current != 'National Trainer');
        }else{
            return false;
        }
    }

    private static Engagement_Type__c newAccomplishment(Date completionDate, Id con, Id chapter, Id rtId, String level){
        Engagement_Type__c eg = new Engagement_Type__c();
        eg.Coach_Program_Start_Date__c = completionDate;
        eg.Contact__c = con;
        eg.Chapter_Affiliation__c = chapter;
        eg.RecordTypeId = rtId;
        eg.Coach_Level__c = level;
        return eg;
    }

}
