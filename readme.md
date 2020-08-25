
# Mapping json structure to data model

This project shows how to map the result of REST WIH into a data model.

![P1 Business Process](https://github.com/salifou/rhpam-error-handling-subprocess/blob/json-mapping/src/main/resources/com/demo/error_handling_sp/p1-svg.svg)

The business process used (P1) has 2 tasks:

1. `REST - Get Group Info` REST task which performs the following
   1. Execute [GET] http://localhost/data.json query
   2. Map the result into GroupInfo data object

2. `Log Group Info` Log the Group info object in the console.


The following sections describe how to:

1. Build, deploy and start the process
2. Serve the JSON locally using caddy docker container

## Building, Deploying and Starting the business process

### Building

```sh
git clone https://github.com/salifou/rhpam-error-handling-subprocess.git
git checkout json-mapping
mvn clean instal
```

### Deploying the KIE Container

#### Request

[PUT] localhost:8080/kie-server/services/rest/server/containers/c0

```xml
<kie-container>
  <release-id>
    <artifact-id>error-handling-sp</artifact-id>
    <group-id>com.demo</group-id>
    <version>1.0.1-SNAPSHOT</version>
  </release-id>
</kie-container>
```

#### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<response type="SUCCESS" msg="Container c0 successfully deployed with module com.demo:error-handling-sp:1.0.1-SNAPSHOT.">
    <kie-container container-id="c0" status="STARTED">
        <messages>
            <content>Container c0 successfully created with module com.demo:error-handling-sp:1.0.1-SNAPSHOT.</content>
            <severity>INFO</severity>
            <timestamp>2020-08-25T10:53:10.935-04:00</timestamp>
        </messages>
        <release-id>
            <artifact-id>error-handling-sp</artifact-id>
            <group-id>com.demo</group-id>
            <version>1.0.1-SNAPSHOT</version>
        </release-id>
        <resolved-release-id>
            <artifact-id>error-handling-sp</artifact-id>
            <group-id>com.demo</group-id>
            <version>1.0.1-SNAPSHOT</version>
        </resolved-release-id>
        <scanner status="DISPOSED"/>
    </kie-container>
</response>
```

### Starting the business Process

#### Request

[POST] http://localhost:8080/kie-server/services/rest/server/containers/c0/processes/p1/instances

```json
{}
```

#### Response

Status 201

```
7
```

### Verifying the process variable

[GET] http://localhost:8080/kie-server/services/rest/server/containers/c0/processes/instances/7/variables/instances

### Response

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<variable-instance-list>
    <variable-instance>
        <name>initiator</name>
        <old-value></old-value>
        <value>rhpamAdmin</value>
        <process-instance-id>7</process-instance-id>
        <modification-date>2020-08-25T11:00:15.618-04:00</modification-date>
    </variable-instance>
    <variable-instance>
        <name>groupInfo</name>
        <old-value></old-value>
        <value>GroupInfo[groupName=Abe Group, groupStatus=Active-Inforce, groupNumber=1, groupCoverages=[GroupCoverage[prodStatus=Active-Inforce, groupProduct=Accident], GroupCoverage[prodStatus=Active-Inforce, groupProduct=Critical Illness]]]</value>
        <process-instance-id>7</process-instance-id>
        <modification-date>2020-08-25T11:00:15.783-04:00</modification-date>
    </variable-instance>
</variable-instance-list>
```

## Serving the JSON using Caddy docker container

```sh
### 1. Create data dir
mkdir data

### 2. Create the json file
cat > ./data/data.json << EOF
{
   "groupName":"Abe Group",
   "groupStatus":"Active-Inforce",
   "groupNumber":"1",
   "groupCoverages":[
      {
         "prodStatus":"Active-Inforce",
         "groupProduct":"Accident"
      },
      {
         "prodStatus":"Active-Inforce",
         "groupProduct":"Critical Illness"
      }
   ]
}
EOF

### 3. Start Caddy web server
docker run -d -p 8082:80 \
  -v ./data/data.json:/usr/share/caddy/data.json:z \
  caddy

### 4. Test the endpoint
$ curl localhost:8082/data.json
{
   "groupName":"Abe Group",
   "groupStatus":"Active-Inforce",
   "groupNumber":"1",
   "groupCoverages":[
      {
         "prodStatus":"Active-Inforce",
         "groupProduct":"Accident"
      },
      {
         "prodStatus":"Active-Inforce",
         "groupProduct":"Critical Illness"
      }
   ]
}
```
