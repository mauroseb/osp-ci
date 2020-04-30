pipeline {
   agent any

   stages {
      stage('Clean up') {
         steps {
            echo 'Clean up'
         }
      }
      stage ('Prepare Hypervisor') {
         steps {
            echo 'Run necessary tasks in the hypervisor to deploy virtual infrastructure.'
         }
      }
      stage ('Create Resources') {
         steps {
            echo 'Create virtual infrastructure like networks and VMs.'
         }
      }
      stage ('Undercloud') {
         steps {
            echo 'Setup undercloud base system'
            echo 'Prepare undercloud.conf'
            echo 'Deploy undercloud'
            echo 'Take snapshot of undercloud'
         }
      }
      stage ('Overcloud') {
         steps {
            echo 'Setup overcloud config'
            echo 'Clone templates'
            echo 'Deploy overcloud'
         }
      }
      stage ('Test') {
         steps {
            echo 'Test overcloud deployment'
            echo 'Validation framework'
            echo 'Tempest smoke test'
            echo 'Rally'
            echo 'Browbeat'
            echo "Post Results"
         }
      }
    stage ('Deploy Tenant') {
         steps {
            echo 'Deploy a test tenant'
         }
      }
   }
   post {
    always {
      telegramSend '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!'
    }
  }
}
