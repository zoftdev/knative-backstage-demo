# Knative Backstage Demo

## Installation

Install Knative Eventing and Serving:

```bash
./000-infra/01-kn-serving.sh
./000-infra/02-kn-eventing.sh
```


TODO:
- Install Knative Eventing and Serving
- Create broker, services and other resources
- Install Backstage backend
- Configure Backstage plugin


## Building demo images

There are 2 images:

- [event-generator](apps/event-generator): A simple app that generates events and sends them to a sink (for instance, to a broker).
- [generic-service](apps/generic-service): A simple app that receives events and logs them. It can also send a reply event (optional).

### event-generator

```bash
  # go to the app directory
  cd apps/event-generator
  
  # install dependencies
  npm install
  
  # run locally
  K_SINK="https://example.com" node index.js
  
  # Output
  > Sending event: {"id":"d33cc086-bb7a-4e8d-a90e-d11789a35e6d","time":"2024-02-02T10:49:08.053Z","type":"com.example.event","source":"event-generator","specversion":"1.0","data":{"message":"Hello 1"}} to https://example.com
  
  # build docker image
  docker build . -t aliok/event-generator --platform=linux/amd64
  
  # run docker image locally
  docker run --name=test-event-generator --detach=false --rm --env K_SINK="https://example.com" aliok/event-generator:latest

  # publish
  docker push aliok/event-generator
```

### generic-service

```bash
  # go to the app directory
  cd apps/generic-service
  
  # install dependencies
  npm install
  
  # run locally
  REPLY_TYPE="test-reply" node index.js
  
  # Send a test event
  curl -i 'http://localhost:8080/'                      \
  -H 'ce-time: 2023-09-26T12:35:14.372688+00:00'      \
  -H 'ce-type: com.mycompany.paymentreceived'           \
  -H 'ce-source: test'   \
  -H 'ce-id: a9254f41-4d32-45d2-8293-e90d96876de1'    \
  -H 'ce-specversion: 1.0'                            \
  -H 'accept: */*'                                    \
  -H 'accept-encoding: gzip, deflate'                 \
  -H 'content-type: '                                 \
  -d $'{"foo": "bar"}'
  
  # Output (reply)
  {"id":"a9254f41-4d32-45d2-8293-e90d96876de1","time":"2023-09-26T12:35:14.372Z","type":"com.mycompany.paymentreceived","source":"test","specversion":"1.0","data_base64":"eyJmb28iOiAiYmFyIn0=","data":{"type":"Buffer","data":[123,34,102,111,111,34,58,32,34,98,97,114,34,125]}} 

  # See the logs
  =======================
    Request headers:
    {
      host: 'localhost:8080',
      'user-agent': 'curl/8.4.0',
      'ce-time': '2023-09-26T12:35:14.372688+00:00',
      'ce-type': 'com.mycompany.paymentreceived',
      'ce-source': 'test',
      'ce-id': 'a9254f41-4d32-45d2-8293-e90d96876de1',
      'ce-specversion': '1.0',
      accept: '*/*',
      'accept-encoding': 'gzip, deflate',
      'content-length': '14'
    }
    
    Request body - to string:
    {"foo": "bar"}
    =======================
    
    Received event:
    {"id":"a9254f41-4d32-45d2-8293-e90d96876de1","time":"2023-09-26T12:35:14.372Z","type":"com.mycompany.paymentreceived","source":"test","specversion":"1.0","data_base64":"eyJmb28iOiAiYmFyIn0="}
    =======================
  
  # build docker image
  docker build . -t aliok/generic-service --platform=linux/amd64
  
  # run docker image locally
  docker run --name=test-generic-service --detach=false --rm -p 8080:8080 -e REPLY_TYPE="test-reply" aliok/generic-service:latest

  # publish
  docker push aliok/generic-service
```
