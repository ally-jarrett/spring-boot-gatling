A sample app to demonstrate revealing a performance problems with load testing.

## Usage

`mvn spring-boot:run` to start the webapp - see the amazing feature at http://localhost:8080/

`mvn gatling:test` while the webapp is running to run the performance test

or `mvn spring-boot:start gatling:test spring-boot:stop`

## Setup OCP 

- `oc apply -f extras/nexus-secret.yaml`
- `oc create configmap --from-file=extras/settings.xml`

Review Jenkinsfile for mounts and usage 
