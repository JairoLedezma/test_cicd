# Sample DMN Project

- This repo containes the source code for a sample DMN Project which implements the iteration over list of objects on which business logic validation is performed.
## Prerequisites
- JBoass EAP v7.3.0
- Red Hat Decision Manager v7.11.0
- Red Hat Decision Manager Kie Server v7.11.0
- Java JDK v11
## Install steps
`git clone`
## Payload Configuration
- Execute the following GET Request from Swagger API under the Kie Server and Kie Containers section when using the Kie development server to retrieve the **container Id**.
![](https://github.com/RutvikPanchal/sampleDMN/blob/master/docs/GET%20Containers.png?raw=true)<br /><br />
- Execute the following GET Request under the DMN models section to retrieve the **model-namespace** and **model-id** by passing in the container Id, set the Response content type to application/json
![](https://github.com/RutvikPanchal/sampleDMN/blob/master/docs/GET%20Info.png?raw=true)<br /><br />
- Execute the following POST Request to check for the validation by passing in the payload configured as shown in **Payload** in the body parameter, set the Parameter content type and Response content type to application/json
![](https://github.com/RutvikPanchal/sampleDMN/blob/master/docs/POST%20Info.png?raw=true)<br /><br />
### Payload:
```
{
  "model-namspace": model-namespace,
  "model-name": model-name,
  "dmn-context":{
    "Input": [
        {"id": id, "value": value},
        {"id": id, "value": value},
        ...
      ]
    }
}
```
