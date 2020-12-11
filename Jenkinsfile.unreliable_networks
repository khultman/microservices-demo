pipeline {

  agent any

  environment {
    GREMLIN_API_KEY = credentials('gremlin-api-key')
    SCENARIO_EXECUTION_ID = ''
  }

  parameters {
    string(name: 'UNRELIABLE_NETWORK_SCENARIO', defaultValue: '522ceaff-995f-43f4-acea-ff995f03f44e', description: 'UUID Of Scenario: Unreliable Network')
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

    stage('Execute Scenario') {
      steps {
        script {
          SCENARIO_EXECUTION_ID = sh (
            script: "curl -s -X POST -H 'Content-Type: application/json' -H 'Authorization: Key ${GREMLIN_API_KEY}' --data '{\"hypothesis\": \"Scenario Run from Jenkins\"}' https://api.gremlin.com/v1/scenarios/${UNRELIABLE_NETWORK_SCENARIO}/runs",
            returnStdout: true
          ).trim()
          echo "see your scenario at https://app.gremlin.com/scenarios/detail/${UNRELIABLE_NETWORK_SCENARIO}/${SCENARIO_EXECUTION_ID}"
        }
      }
    }

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
            sh "curl -s -X POST -H 'Authorization: Key ${GREMLIN_API_KEY}' https://api.gremlin.com/v1/scenarios/halt/${UNRELIABLE_NETWORK_SCENARIO}/runs/${SCENARIO_EXECUTION_ID}"
          }
        }
      }
    }

  }

}
