library identifier: "pipeline-library@v1.5",
retriever: modernSCM(
  [
    $class: "GitSCMSource",
    remote: "https://github.com/redhat-cop/pipeline-library.git"
  ]
)

// The name you want to give your Spring Boot application
// Each resource related to your app will be given this name
appName = "spring-boot-performance"

openshift.withCluster() {
  env.NAMESPACE = openshift.project()
  env.POM_FILE = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/pom.xml" : "pom.xml"
  echo "Starting Pipeline for ${env.APP_NAME}..."
  env.BUILD = "${env.NAMESPACE_BUILD}"
  env.DEV = "${env.NAMESPACE_DEV}"
}

pipeline {
    // Use the 'maven' Jenkins agent image which is provided with OpenShift
    agent {
        kubernetes {
          yaml """\
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: image-registry.openshift-image-registry.svc:5000/openshift/jenkins-agent-maven:latest
                command:
                - cat
                tty: true
                resources:
                  requests:
                    cpu: 500m
                    memory: 512Mi
                  limits:
                    cpu: 750m
                    memory: 1Gb
            """.stripIndent()
        }
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        // Run Maven build, skipping tests
        stage('Maven Build'){
          steps {
           container('maven') {
                sh "mvn -B clean install -DskipTests=true -f ${POM_FILE}"
            }
          }
        }

        // Run Maven unit tests
        stage('Gatling Performance Test'){
          steps {
            container('maven') {
                sh "mvn -B spring-boot:start gatling:test spring-boot:stop"
            }
          }
       }
    }
}
