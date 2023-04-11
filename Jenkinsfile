pipeline {
  agent any


  stages {
    stage('Setup') {
      steps {
        script {
          echo 'Pulling...' + env.BRANCH_NAME
          if (env.GIT_BRANCH == 'origin/develop' || env.GIT_BRANCH ==~ /(.+)feature-(.+)/) {
            target = 'dev'
          } else if (env.GIT_BRANCH ==~ /(.+)release-(.+)/) {
            target = 'pre'
          } else if (env.GIT_BRANCH == 'origin/master'  || env.GIT_BRANCH == 'origin/main') {
            target = 'pro'
          } else {
            error "Unknown branch type: ${env.GIT_BRANCH}"
          }

        
        }
      }
    }

    stage('APP Build') {
      steps {
        sh 'echo APP build'
      }
    }
  
   

    stage('DEV Deploy') {
      steps {
        sh 'echo Deploy to DEV . . .'
      }
    }

    stage('PRE Deploy') {
      when {
        expression { return target == 'pre' || target == 'pro' }
      }
      steps {
        sh 'echo Deploy to PRE . . .'
      }
    }

    stage('UAT Test') {
      when {
        expression { target == 'pro' }
      }
      steps {
        sh 'echo UAT Test . . .'
      }
    }

    stage('Approval') {
      when {
        expression { target == 'pro' }
      }
      steps {
        timeout(time:30, unit:'MINUTES') {
          input message: "Deploy to Production?", id: "approval"
        }
      }
    }

    stage('PRO Deploy') {
      when {
        expression { return target == 'pro' }
      }
      steps {
        sh 'echo Deploy to PRO . . .'
      }
    }
  }

  post {
    success {
      script {
        currentBuild.result = "SUCCESS"

        /* Custom data map for InfluxDB */
        def custom = [:]
        custom['branch']      = env.GIT_BRANCH
        custom['environment'] = target
        custom['part']        = 'jenkins'
        custom['version']     = version

        step([$class: 'InfluxDbPublisher', customData: custom, target: 'devops-kpi'])
      }
    }

    failure {
      script {
        currentBuild.result = "FAILURE"

        /* Custom data map for InfluxDB */
        def custom = [:]
        custom['branch']      = env.GIT_BRANCH
        custom['environment'] = target
        custom['part']        = 'jenkins'
        custom['version']     = version

        step([$class: 'InfluxDbPublisher', customData: custom, target: 'devops-kpi'])
      }
    }

    unstable {
      script {
        currentBuild.result = "FAILURE"

        /* Custom data map for InfluxDB */
        def custom = [:]
        custom['branch']      = env.GIT_BRANCH
        custom['environment'] = target
        custom['part']        = 'jenkins'
        custom['version']     = version

        step([$class: 'InfluxDbPublisher', customData: custom, target: 'devops-kpi'])
      }
    }

    aborted {
      script {
        currentBuild.result = "FAILURE"

        /* Custom data map for InfluxDB */
        def custom = [:]
        custom['branch']      = env.GIT_BRANCH
        custom['environment'] = target
        custom['part']        = 'jenkins'
        custom['version']     = version

        step([$class: 'InfluxDbPublisher', customData: custom, target: 'devops-kpi'])
      }
    }
  }
}
