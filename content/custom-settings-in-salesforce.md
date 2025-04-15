
---

# Salesforce Custom Settings

Custom Settings in Salesforce are specialized data containers that allow you to store configuration and reference data at an application level. They provide a fast, cached access mechanism from formulas and Apex code, making them ideal for data that seldom changes. This post breaks down what custom settings are, when to use them, their limitations, caching behavior, access methods, and more—all in a minimal, digestible format.

---

## What Are Custom Settings?

Custom Settings enable you to create custom data sets that can be accessed easily across your Salesforce organization. They are similar to custom objects but come with built-in caching, meaning that accessing the data does not count against your standard SOQL query limits.

### Types of Custom Settings

1. **Hierarchy Custom Settings:**  
   - **Overview:** Allow you to personalize settings at the organization, profile, or user level.  
   - **Usage:** Ideal for data that may have different values depending on the context (for example, default settings that vary by user or profile).

2. **List Custom Settings:**  
   - **Overview:** Provide a reusable collection of static data.  
   - **Usage:** Useful for reference data that is common for all users, where a simple list of values needs to be maintained.

---

## When to Use Custom Settings

- **Configuration Data:** Store application configuration or feature toggles.
- **Reference Data:** Keep static data such as thresholds, constants, or environment-specific settings.
- **Formula Fields & Workflows:** Custom settings can be referenced directly in formulas, validation rules, and workflow rules for dynamic behavior.
- **Performance Sensitive Code:** Because custom settings are cached, they minimize the need for frequent SOQL queries, thereby reducing governor limit consumption.

---

## When Not to Use Custom Settings

- **Large Volumes or Relational Data:** For complex data models, relationships, or high data volumes, consider using custom objects or custom metadata.
- **Transactional Data:** Custom settings are not designed for constantly updating records; use them instead for mostly static configuration.
- **Security-Centric Data:** Custom settings do not support standard record-level security; for sensitive data, standard objects or custom metadata might be more appropriate.

---

## Advantages & Limitations

### Advantages

- **Fast Access:** Data is cached and readily available without consuming SOQL limits.
- **Easy Integration:** Accessible from Apex, formulas, and validation rules.
- **User-Level Overrides (Hierarchy):** Supports different values for different users or profiles.
- **Simplified Data Management:** Streamlines configuration management and reduces code complexity.

### Limitations

- **Not Suited for Frequent Updates:** Best for data that rarely changes.
- **Limited Record Structure:** Custom settings have limitations compared to standard objects in terms of relationships and advanced configurations.
- **Security Restrictions:** Lack record-level security; rely on organization-wide settings or use other data types for sensitive info.

---

## Governor Limits and Caching

- **Governor Limits:** Because custom settings are cached, accessing them does not contribute to your SOQL query limits. This can greatly enhance performance in scenarios where multiple Apex transactions repeatedly reference configuration data.
  
- **Caching Mechanism:**  
  - **Automatic Caching:** Salesforce automatically caches custom settings. When a custom setting record is updated, the cache is refreshed.  
  - **Performance Benefits:** This caching mechanism is beneficial for operations that need fast, recurring access to configuration data without incurring database query costs.

---

## Access Methods

### In Apex Code

- **Hierarchy Custom Settings:**  
  Use the `getInstance()` method to retrieve the setting at the appropriate level (user, profile, or organization):
  ```apex
  // Fetches the setting for the current user context
  MyHierarchySetting__c setting = MyHierarchySetting__c.getInstance();
  System.debug('Setting Value: ' + setting.Custom_Field__c);
  ```

- **List Custom Settings:**  
  Use the `getAll()` method to retrieve all records as a map:
  ```apex
  // Retrieves all list custom setting records
  Map<String, MyListSetting__c> settingsMap = MyListSetting__c.getAll();
  for (MyListSetting__c setting : settingsMap.values()) {
      System.debug('Name: ' + setting.Name + ', Value: ' + setting.Custom_Field__c);
  }
  ```

### In Formulas and Other Declarative Tools

- **Direct Reference:** Custom Settings fields can be referenced in formula fields, validation rules, and workflow rules, providing dynamic behavior without additional code.

---

## Inserting and Updating Custom Settings

- **List Custom Settings:**  
  These can be created, updated, and deleted using standard DML operations in Apex.
  ```apex
  // Creating a new list custom setting record
  MyListSetting__c newSetting = new MyListSetting__c();
  newSetting.Name = 'Setting1';
  newSetting.Custom_Field__c = 'Value1';
  insert newSetting;
  
  // Updating an existing custom setting record
  newSetting.Custom_Field__c = 'NewValue';
  update newSetting;
  ```
  
- **Hierarchy Custom Settings:**  
  Typically managed via the Salesforce Setup UI for different user or profile levels. Although updates via Apex are possible, the hierarchical model generally suggests UI-based management for clarity and context.

---

## Required Permissions

- **Creation and Modification:**  
  - Typically, you need the **Customize Application** permission to create or modify custom settings.
  - Users performing DML operations on **List Custom Settings** must have appropriate object-level and field-level permissions.
  
- **Access:**  
  - Accessing custom settings via Apex or declarative tools does not usually require additional permissions beyond standard access rights, thanks to their caching mechanism.

---
Below is the additional section in markdown for testing custom settings in test classes. You can append it to your existing blog post on Salesforce Custom Settings:

---

## Using Custom Settings in Test Classes

When writing test classes for functionality that leverages custom settings, it’s important to create and manipulate test data within the test context. Since tests run in an isolated environment, relying on pre-existing org data is not recommended. Instead, follow these best practices:

### Best Practices

- **Create Test Data:**  
  Always insert custom settings records within your test to ensure the data is isolated and predictable. Avoid relying on existing org data.

- **Avoid Using `SeeAllData=true`:**  
  Using `@isTest(SeeAllData=true)` can introduce dependencies on actual org data. Instead, create your own custom settings records to keep tests robust and self-contained.

- **Test Both List and Hierarchy Settings:**  
  Validate the behavior of both **List** and **Hierarchy** custom settings. For Hierarchy custom settings, although updates via the UI are common, you can still simulate and verify behavior through DML in your test classes.

### Example for List Custom Settings

```apex
@isTest
private class CustomSettingsListTest {
    @isTest static void testListCustomSettings() {
        // Create a List Custom Setting record
        MyListSetting__c setting = new MyListSetting__c(
            Name = 'TestSetting',
            Custom_Field__c = 'Test Value'
        );
        insert setting;
        
        // Retrieve and assert the custom settings data
        Map<String, MyListSetting__c> settingsMap = MyListSetting__c.getAll();
        System.assertEquals(1, settingsMap.size(), 'There should be one custom setting record.');
        System.assertEquals('Test Value', settingsMap.get('TestSetting').Custom_Field__c, 
            'The custom field value should match the inserted value.');
    }
}
```

### Example for Hierarchy Custom Settings

```apex
@isTest
private class CustomSettingsHierarchyTest {
    @isTest static void testHierarchyCustomSettings() {
        // Create a Hierarchy Custom Setting record
        MyHierarchySetting__c setting = new MyHierarchySetting__c(
            Name = 'TestSetting',
            Custom_Field__c = 'Hierarchy Value'
        );
        // Although hierarchy custom settings are typically managed via the setup UI,
        // you can insert them via DML within test classes.
        insert setting;
        
        // Retrieve and assert using getInstance method
        MyHierarchySetting__c retrievedSetting = MyHierarchySetting__c.getInstance();
        System.assertNotEquals(null, retrievedSetting, 'The setting should not be null.');
        System.assertEquals('Hierarchy Value', retrievedSetting.Custom_Field__c, 
            'The custom field should contain the correct hierarchy value.');
    }
}
```

### Tips When Testing

- **Cache Considerations:**  
  Custom settings are cached by Salesforce. In test methods, after inserting records, immediately retrieving them using methods like `getAll()` or `getInstance()` ensures you are working with the up-to-date test data.

- **Isolation of Tests:**  
  Each test method should be self-sufficient and independent. Insert and clean up your custom setting data as part of the test logic to avoid interference between tests.

- **Testing Updates & Deletes:**  
  Validate how changes to custom settings data affect your application behavior. Include tests for updating and, if applicable, deleting custom settings (for List Custom Settings) to ensure that your application responds correctly to dynamic configuration changes.

---
When working with Hierarchy Custom Settings, the key field to be aware of is the **SetupOwnerId**. This field determines whether a record applies at the organization, profile, or user level. You don't *always* have to add a user reference, but you must do so if you intend for the custom setting record to override the default at a specific user (or profile) level.

Here’s a detailed explanation:

- **Organization-Level Record:**  
  If you don’t set the **SetupOwnerId** field when inserting a hierarchical custom setting record, Salesforce treats this record as the organization-wide default. This is often sufficient if you want to define a common default value for all users.

- **User-Level or Profile-Level Record:**  
  To tailor the custom setting for a specific user or profile, you must set the **SetupOwnerId** field to the appropriate ID. For a user-level override, assign it the user’s ID; for a profile-level setting, assign it the profile’s ID. This way, when you invoke `MyHierarchySetting__c.getInstance()`, Salesforce can resolve the record based on the current context (e.g., current user or profile).

### Example: Inserting a Hierarchy Custom Setting with a User Reference

```apex
// For a specific user: Setting the SetupOwnerId to the current user's ID
MyHierarchySetting__c userSetting = new MyHierarchySetting__c(
    SetupOwnerId = UserInfo.getUserId(),
    Custom_Field__c = 'User-Specific Value'
);
insert userSetting;

// Later, when fetching the instance, Salesforce will return this record if the current user matches SetupOwnerId
MyHierarchySetting__c retrievedSetting = MyHierarchySetting__c.getInstance();
System.debug('Retrieved Custom Field Value: ' + retrievedSetting.Custom_Field__c);
```

### Some Considerations

- **No User Reference Needed:**  
  If you want a universal default for your org, you don’t need to add a **SetupOwnerId**; simply create the record without it.

- **User/Profile Specific:**  
  If you need to create settings specific to a user or a profile, you must set the **SetupOwnerId** to the corresponding ID. This ensures that when Salesforce looks up the custom setting, it correctly applies the override based on the user or profile context.

This approach makes hierarchical custom settings very flexible, allowing you to define both global defaults and specific overrides where necessary.
---

## Summary

Salesforce Custom Settings offer a streamlined and efficient way to store configuration data without impacting governor limits. They are perfect for scenarios where static or semi-static data needs to be accessed frequently and quickly. However, keep in mind their limitations when dealing with large datasets or data that requires complex relationships and robust security. By understanding the differences between list and hierarchical custom settings and applying the right access methods, you can optimize your Salesforce applications while keeping the configuration simple and performant.

---
