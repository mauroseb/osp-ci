#!/usr/bin/env groovy

@Library('common') _

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
            sh cleanup.sh $BUILD_HOSTNAME
         }
      }
      stage ('Setup Shared Resources') {
         steps {
            git url: "https://github.com/mauroseb/bootstrap-rhosp.git"
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
          ansiblePlaybook('bootstrap-rhosp/site.yml') {
          inventoryContent($BUILD_HOSTNAME, true)
             tags('prereqs')
             extraVars {
                extraVar("cdnuser", "mycdnuser", true)
                extraVar("cdnpass", "mycdnpass", true)
                extraVar("poolid", "mysubspool", true)
                extraVar("rhosp_version", $RHOSP_VERSION, true)
             }
          }
        }
      }
      stage ('Create Resources') {
         steps {
            echo 'Create virtual infrastructure like networks and VMs.'
            ansiblePlaybook('bootstrap-rhosp/site.yml') {
               inventoryContent($BUILD_HOSTNAME, true)
               tags('libvirt,vbmc,')
               extraVars {
                  extraVar("cdnuser", "mycdnuser", true)
                  extraVar("cdnpass", "mycdnpass", true)
                  extraVar("poolid", "mysubspool", true)
                  extraVar("rhosp_version", $RHOSP_VERSION, true)
               }
            }
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
            ansiblePlaybook('bootstrap-rhosp/site.yml') {
               inventoryContent($BUILD_HOSTNAME, true)
               tags('undercloud')
               extraVars {
                  extraVar("cdnuser", "mycdnuser", true)
                  extraVar("cdnpass", "mycdnpass", true)
                  extraVar("poolid", "mysubspool", true)
                  extraVar("rhosp_version", $RHOSP_VERSION, true)
               }
            }
            echo 'Prepare undercloud.conf'
            echo 'Deploy undercloud'
            echo 'Take snapshot of undercloud'
         }
      }
      stage ('Import and Introspect') {
         when {
                expression {
                     params.skip_introspection == false
                }
         }
         steps {
            echo 'Setup overcloud config'
            ansiblePlaybook('bootstrap-rhosp/site.yml') {
               inventoryContent($BUILD_HOSTNAME, true)
               tags('import,introspect')
               extraVars {
                  extraVar("cdnuser", "mycdnuser", true)
                  extraVar("cdnpass", "mycdnpass", true)
                  extraVar("poolid", "mysubspool", true)
                  extraVar("rhosp_version", $RHOSP_VERSION, true)
               }
            }
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
            ansiblePlaybook('bootstrap-rhosp/site.yml') {
               inventoryContent($BUILD_HOSTNAME, true)
               tags('undercloud')
               extraVars {
                  extraVar("cdnuser", "mycdnuser", true)
                  extraVar("cdnpass", "mycdnpass", true)
                  extraVar("poolid", "mysubspool", true)
                  extraVar("rhosp_version", $RHOSP_VERSION, true)
               }
            }
            echo 'Clone templates'
            git url: "https://github.com/mauroseb/tht-rhosp${RHOSP_VERSION}"
            echo 'Deploy overcloud'
            echo "RUN: sh -x tht-rhosp${RHOSP_VERSION}/deploy.sh"
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
    stage ('Deploy Test Tenant') {
         when {
                expression {
                     params.skip_test_tenant == false
                }
         }
         steps {
            echo 'Deploy a test tenant with some workload'
            git url: "https://github.com/mauroseb/ansible-osp-smoketest"
            ansiblePlaybook('ansible_osp_smoketest/site.yml') {
               inventoryContent($BUILD_HOSTNAME, true)
               extraVars {
                  extraVar("OS_CLOUD", "overcloud", false)
            }
         }
      }
   }
   post {
    always {
      telegramSend '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!'
    }
  }
}
