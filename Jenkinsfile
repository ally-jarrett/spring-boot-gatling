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
  env.MAVEN_SERVER_USERNAME = "${env.MAVEN_SERVER_USERNAME}"
  env.MAVEN_SERVER_PASSWORD = "${env.MAVEN_SERVER_PASSWORD}"
  env.MAVEN_SETTINGS = "/etc/data/settings.xml"
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
                    cpu: 200m
                    memory: 256Mi
                  limits:
                    cpu: 500m
                    memory: 512Mi
                volumeMounts:
                - name: maven-settings
                  mountPath: /etc/data
                env:
                  - name: MAVEN_MIRROR_URL
                    value: https://nexus-nexus.apps.ocp1.purplesky.cloud/repository/maven-public/
                  - name: MAVEN_SERVER_USERNAME
                    valueFrom:
                      secretKeyRef:
                        name: nexus-secret
                        key: username
                  - name: MAVEN_SERVER_PASSWORD
                    valueFrom:
                      secretKeyRef:
                        name: nexus-secret
                        key: password
              volumes:
                - name: maven-settings
                  configMap:
                    name: maven-settings
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
                sh "mvn -B clean install -DskipTests=true -f ${POM_FILE} -s ${MAVEN_SETTINGS}"
            }
          }
        }

        // Run Maven unit tests
        stage('Maven Deploy'){
          steps {
            container('maven') {
                sh "mvn -B spring-boot:start gatling:test spring-boot:stop -s ${MAVEN_SETTINGS}"
            }
          }
        }
    }
}
