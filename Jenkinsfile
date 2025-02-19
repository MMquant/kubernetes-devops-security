@Library('slack') _

/////// ******************************* Code for fectching Failed Stage Name ******************************* ///////
import io.jenkins.blueocean.rest.impl.pipeline.PipelineNodeGraphVisitor
import io.jenkins.blueocean.rest.impl.pipeline.FlowNodeWrapper
import org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper
import org.jenkinsci.plugins.workflow.actions.ErrorAction

// Get information about all stages, including the failure cases
// Returns a list of maps: [[id, failedStageName, result, errors]]
@NonCPS
List < Map > getStageResults(RunWrapper build) {

  // Get all pipeline nodes that represent stages
  def visitor = new PipelineNodeGraphVisitor(build.rawBuild)
  def stages = visitor.pipelineNodes.findAll {
    it.type == FlowNodeWrapper.NodeType.STAGE
  }

  return stages.collect {
    stage ->

      // Get all the errors from the stage
      def errorActions = stage.getPipelineActions(ErrorAction)
    def errors = errorActions?.collect {
      it.error
    }.unique()

    return [
      id: stage.id,
      failedStageName: stage.displayName,
      result: "${stage.status.result}",
      errors: errors
    ]
  }
}

// Get information of all failed stages
@NonCPS
List < Map > getFailedStages(RunWrapper build) {
  return getStageResults(build).findAll {
    it.result == 'FAILURE'
  }
}

/////// ******************************* Code for fectching Failed Stage Name ******************************* ///////


pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "mmquant/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://172.31.40.19"
    applicationURI = "/increment/99"
  }

  stages {
    stage('Testing Slack') {
      steps {
        sh 'exit 0'
      }
    }
    stage('Testing Slack - Error Stage') {
      steps {
        sh 'exit 1'
      }
    }
  }

  post {
    //    always { 
    //      junit 'target/surefire-reports/*.xml'
    //      jacoco execPattern: 'target/jacoco.exec'
    //      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    //      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
    //      publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report'])

    // Use sendNotifications.groovy from shared library and provide current build result as parameter 
    //      sendNotification currentBuild.result
    //    }

    success {
      script {
        /* Use slackNotifier.groovy from shared library and provide current build result as parameter */
        env.failedStage = "none"
        env.emoji = ":white_check_mark: :tada: :thumbsup_all:"
        sendNotification currentBuild.result
      }
    }

    failure {
      script {
        //Fetch information about  failed stage
        def failedStages = getFailedStages(currentBuild)
        env.failedStage = failedStages.failedStageName
        env.emoji = ":x: :red_circle: :sos:"
        sendNotification currentBuild.result
      }
    }

  }
}
