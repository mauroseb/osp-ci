pipeline {
   agent any
   
   parameters {
        string(name: 'BUILD_HOSTNAME', description: 'The name of the hypervisor')
        string(name: 'ILO_IP', description: 'The IP address for the server ilo')
        booleanParam(name: 'SATELLITE', description: 'Use internal Satellite instead of CDN', defaultValue: false)
    }

   stages {
      stage('Clean up') {
           when {
                expression {
                     params.skip_cleanup == false
                }
            }
         steps {
            echo 'Clean up'
         }
      }
      stage ('Prepare Hypervisor') {
         when {
                expression {
                     params.skip_hypervisor == false
                }
         }
      
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
          when {
                expression {
                     params.skip_undercloud == false
                }
         }
         steps {
            echo 'Setup undercloud base system'
            echo 'Prepare undercloud.conf'
            echo 'Deploy undercloud'
            echo 'Take snapshot of undercloud'
         }
      }
      stage ('Overcloud') {
         when {
                expression {
                     params.skip_overcloud == false
                }
         }
         steps {
            echo 'Setup overcloud config'
            echo 'Clone templates'
            echo 'Deploy overcloud'
         }
      }
      stage ('Test') {
         when {
                expression {
                     params.skip_test == false
                }
         }
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
         when {
                expression {
                     params.skip_test_tenant == false
                }
         }
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
