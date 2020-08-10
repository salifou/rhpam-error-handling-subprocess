# Rest WIH  Exception Handling via a Sub process

See: https://github.com/salifou/rhpam-error-handling-subprocess

## kie-deployment-descriptor.xml

```xml
<work-item-handler>
    <resolver>mvel</resolver>
    <identifier>new org.jbpm.process.workitem.rest.RESTWorkItemHandler(classLoader, "error-handler", "COMPLETE")</identifier>
    <parameters/>
    <name>Rest</name>
</work-item-handler>
```

**classLoader is needed for marshalling to work as expected**. See [2].

## Rest WorkItem Handler parameter

- `HandleResponseErrors` - optional parameter that instructs handler to throw errors in case
 of non successful response codes (other than 2XX). Example: HandleResponseErrors="true"
 
 - `ResultClass` FQCN of the result. Needed for marshalling. See [2]

[1] https://github.com/kiegroup/jbpm/blob/7.33.x/jbpm-workitems/jbpm-workitems-rest/src/main/java/org/jbpm/process/workitem/rest/RESTWorkItemHandler.java#L766

[2] https://access.redhat.com/solutions/3763421

## Error handler sub process

- The sub process can return vairable to the main task. (See outcome process variable is example above)


