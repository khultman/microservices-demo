pipeline {

  agent any

  environment {
    GREMLIN_API_KEY = credentials('gremlin-api-key')
    SCENARIO_RUN_ID = ''
  }

  parameters {
    /*
     * Unreliable Networks:                                    522ceaff-995f-43f4-acea-ff995f03f44e
     * Unavailable Dependency - Currency/Ad/Email:             58ea876e-f74f-4af9-aa87-6ef74f6af927
     * Unavailable Dependency - Recommendation/Payment/Cart:   fd5c8763-1d28-47b8-9c87-631d2877b8f6
    */
    string(name: 'SCENARIO_UUID', defaultValue: '522ceaff-995f-43f4-acea-ff995f03f44e', description: 'UUID Of Scenario')
    string(name: 'SCENARIO_HYPOTHESIS', defaultValue: 'Automated scenario run from Jenkins')
  }

  stages {

    stage('Checkout Source') {
      steps {
        git url:'https://github.com/khultman/microservices-demo.git', branch:'master'
      }
    }

    stage('Deploy Gremlin Shop') {
      steps {
        script {
          sh "/usr/local/bin/kubectl apply -f $WORKSPACE/release/kubernetes-manifests.yaml"
        }
      }
    }

    stage('Execute Gremlin Scenario') {
      parallel {
        stage('Loadtest') {
          steps {
            // Initiate the Loadtest
            script{
              sh '/usr/bin/python3 -m bzt.cli ${WORKSPACE}/loadtest.yml -report'
            }
          }
        }

        stage('Gremlin') {
          steps {
            // Initiate the Gremlin Scenario
            script {
              SCENARIO_RUN_ID = sh (
                script: "curl -s -X POST -H 'Content-Type: application/json' -H 'Authorization: Key ${GREMLIN_API_KEY}' --data '{\"hypothesis\": \"${SCENARIO_HYPOTHESIS}\"}' https://api.gremlin.com/v1/scenarios/${SCENARIO_UUID}/runs",
                returnStdout: true
              ).trim()
              SCENARIO_RUN_ID = (SCENARIO_RUN_ID =~ /(\d+)/)[0][1]
              echo "see your scenario at https://app.gremlin.com/scenarios/detail/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}"
            }

            // Follow the Gremlin Scenario to ensure it wasn't halted or failed
            script {
              RESPONSE = sh(
                script: "curl -X GET -H 'Authorization: Key ${GREMLIN_API_KEY}' https://api.gremlin.com/v1/scenarios/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}",
                returnStdout: true
              ).trim()
              JSON = readJSON text: RESPONSE
              LIFECYCLE = JSON.stage_info.stage
              while(LIFECYCLE == "NotStarted" || LIFECYCLE == "Active") {
                RESPONSE = sh(
                  script: "curl -X GET -H 'Authorization: Key ${GREMLIN_API_KEY}' https://api.gremlin.com/v1/scenarios/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}",
                  returnStdout: true
                ).trim()
                JSON = readJSON text: RESPONSE
                LIFECYCLE = JSON.stage_info.stage
                sleep(10)
              }
              echo LIFECYCLE
              if(LIFECYCLE == "HaltRequested" || LIFECYCLE == "Halted") {
                error "Scenario Halted"
              }
              if(LIFECYCLE == "Failed") {
                error "Scenario Failed: " + JSON.stage_info.stageMetadata.failedReason
              }
            }
          }
        }
      }
    }
  }

}
