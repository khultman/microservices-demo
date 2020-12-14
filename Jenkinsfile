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
      steps {
        script {
          SCENARIO_RUN_ID = sh (
            script: "curl -s -X POST -H 'Content-Type: application/json' -H 'Authorization: Key ${GREMLIN_API_KEY}' --data '{\"hypothesis\": \"${SCENARIO_HYPOTHESIS}\"}' https://api.gremlin.com/v1/scenarios/${SCENARIO_UUID}/runs",
            returnStdout: true
          ).trim()
          echo "see your scenario at https://app.gremlin.com/scenarios/detail/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}"
        }

        script {
          RESPONSE = sh(
            script: "curl -X GET -H 'Authorization: Key ${GREMLIN_API_KEY}' https://api.gremlin.com/v1/scenarios/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}",
            returnStdout: true
          ).trim()
          JSON = readJSON text: RESPONSE
          LIFECYCLE = JSON.graph.nodes.concurrentNode.state.lifecycle
        ​
          while(LIFECYCLE == "NotStarted" || LIFECYCLE == "Active") {
            RESPONSE = sh(
              script: "curl -X GET -H 'Authorization: Key ${GREMLIN_API_KEY}' https://api.gremlin.com/v1/scenarios/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}",
              returnStdout: true
            ).trim()
            JSON = readJSON text: RESPONSE
            LIFECYCLE = JSON.graph.nodes.concurrentNode.state.lifecycle
            sleep(1)
          }
          echo ${LIFECYCLE}
          if(LIFECYCLE == "HaltRequested") {
            error "Scenario Halted"
          }
        }
      }
    }

    /*
    stage('Observce and Halt') {
      input {
        message 'Do you want to halt scenario?'
        parameters {
          choice(choices: ['yes', 'no'], name: 'HALT', description: '')
        }
      }
      steps {
        script {
          if (env.HALT=='yes') {
            sh "curl -s -X POST -H 'Authorization: Key ${GREMLIN_API_KEY}' https://api.gremlin.com/v1/scenarios/halt/${SCENARIO_UUID}/runs/${SCENARIO_RUN_ID}"
            error('Scenario Aborted')
          }
        }
      }
    }
    */
  }

}
