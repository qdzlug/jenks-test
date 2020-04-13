def AUTH_HEADER
def ACCPORTS
def ACCTYPE
def ALLIP
def APPNAME
def APPVER
def CLDLET
def CLSTR
def FLAVOR
def IMGPATH
def IMGTYPE
def INTPORTS
def IPACC
def OPKEY
def ORG
def REGION
def REV
def TYPE
def CLUSTERSTATE
def APPSTATE
def APPINSTSTATE
def DOCKERAPP


pipeline {
   agent any

   stages {

    stage('Clone Repository') {
        steps {
        /*
         * Let's make sure we have the repository cloned to our workspace
         */
        checkout scm
        }
    }

    stage('Set Variables') {
        steps {
           /*
            * In the real world, we'd want to read these
            * from a config file and not have them hardcoded
            * in the Jenkins file.
            */

            script {
              ACCPORTS = "TCP:8000"
              ACCTYPE = 0
              ALLIP = "dynamic"
              APPNAME = "testapp"
              APPVER = "1.1"
              CLDLET = "munich-main"
              CLSTR = "testcluster"
              FLAVOR = "m4.small"
              IMGPATH = "docker.mobiledgex.net/demoorg/images/test:1.1"
              IMGTYPE = 1
              INTPORTS = "True"
              IPACC = 1
              MEXPASS = "helloworld"
              MEXUSER = "demo"
              OPKEY = "TDG"
              ORG = "demoorg"
              REGION = "EU"
              REV = 0
              TYPE = "docker"
            }
        }
    }

    stage('Build Docker Image') {
        steps{
          /*
           * Build our image - if we fail here there
           * is no real point to continuing on.
           */
          DOCKERAPP = docker.build("qdzlug/jenks-test")
        }
    }

    state('Test Docker Image') {
      steps{
        /*
         * Test our application; this is obviously
         * not a very comprehensive test.
         */
         DOCKERAPP.inside {
            "curl -f http://127.0.0.1:8000 || exit 1"
         }
      }
    }


    stage('Login and get JWT') {
         steps {
            script{

            /*
             * We need to take our credentials (which are stored in Jenkins) and bind
             * them to vars so we can get our JWT. That is then used as the Auth Header
             * for all subsequent calls to the API
             */

            withCredentials([usernamePassword(credentialsId: 'MeX-Demo', passwordVariable: 'MEXPASS', usernameVariable: 'MEXUSER')]) {

                def rBody = """
                {"username": "$MEXUSER", "password": "$MEXPASS"}
                """

                def response = httpRequest contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/login'

                println("Status: "+response.status)
                println("Content: "+response.content)

                def props = readJSON text: response.content
                props.each { key, value ->
                    echo "Walked through key $key and value $value"
                    AUTH_HEADER = "Bearer $value"
                }
            }
            println("Header is "+ AUTH_HEADER)
          }
       }
    }

     stage('Check for Running Cluster') {
        steps {
            script{

            /*
             * We check to see if we already have a cluster running; the next
             * pass at this would be to pull the response code and either
             * decide to leave it and reuse it or delete it to go with a
             * fully clean slate on deploy.
             */

              def rBody = """
              {
              "region": "$REGION",
              "clusterinst": {
              "key": {
                "cluster_key": {
                  "name": "$CLSTR"
                },
                "cloudlet_key": {
                  "operator_key": {
                    "name": "$OPKEY"
                  },
                  "name": "$CLDLET"
                },
                "developer": "$ORG"
                 }
                }
              }
              """

              def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/ShowClusterInst'

              println("Status: "+response.status)
              println("Content: "+response.content)
              CLUSTERSTATE=response.content
          }
      }
  }

    stage('Check for Running AppInst') {
       steps {
           script{

           /*
            * If there is a running app instance we want to remove it (and the app)
            * so we can deploy our new version.
            */

             def rBody = """
             {
               "Region": "$REGION",
               "appinst": {
                 "key": {
                   "app_key": {
                     "developer_key": {
                       "name": "$ORG"
                     },
                     "name": "$APPNAME",
                     "version": "$APPVER"
                   },
                   "cluster_inst_key": {
                     "cluster_key": {
                       "name": "$CLSTR"
                     },
                     "cloudlet_key": {
                       "operator_key": {
                         "name": "$OPKEY"
                       },
                       "name": "$CLDLET"
                     },
                     "developer": "$ORG"
                   }
                 }
               }
             }
             """

            def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/ShowAppInst', validResponseCodes: '100:499'

            println("Status: "+response.status)
            println("Content: "+response.content)
            APPINSTSTATE=response.content

         }
      }
   }

   stage('Check for Defined Application') {
      steps {
          script{

           /*
            * If there is a defined app version we want to delete it
            * so we can deploy our new version.
            */

            def rBody = """
            {
              "Region": "$REGION",
              "App": {
                "key": {
                    "developer_key": {
                      "name": "$ORG"
                    },
                    "name": "$APPNAME",
                    "version": "$APPVER"
                  }
                }
            }
            """

            def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/ShowApp', validResponseCodes: '100:499'

            println("Status: "+response.status)
            println("Content: "+response.content)
            APPSTATE=response.content
          }
      }
  }

  stage('Delete Existing Application Instance') {
     steps {
         script{

          /*
           * Ideally this will check the response from the earlier step where
           * we detected a running app instance. However, for our purposes here
           * we just allow response codes in the 4xx series to account for us
           * trying to delete a non-existent object.
           */

           def rBody = """
           {
             "Region": "$REGION",
             "AppInst": {
               "key": {
                 "app_key": {
                   "organization": "$ORG",
                   "name": "$APPNAME",
                   "version": "$APPVER",
                   "developer_key": {
                     "name": "$ORG"
                   }
                 },
                 "cluster_inst_key": {
                   "cluster_key": {
                     "name": "$CLSTR"
                   },
                   "cloudlet_key": {
                     "operator_key": {
                       "name": "$OPKEY"
                     },
                     "name": "$CLDLET"
                   },
                   "developer": "$ORG"
                 }
               }
             }
           }
           """

           def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/DeleteAppInst', validResponseCodes: '100:499'

           println("Status: "+response.status)
           println("Content: "+response.content)
         }
     }
 }

 stage('Delete Existing Application') {
    steps {
        script{

          /*
           * Ideally this will check the response from the earlier step where
           * we detected a defined application However, for our purposes here
           * we just allow response codes in the 4xx series to account for us
           * trying to delete a non-existent object.
           */

          def rBody = """
          {
            "Region": "$REGION",
            "App": {
              "key": {
                "developer_key": {
                  "name": "$ORG"
                },
                "name": "$APPNAME",
                "organization": "$ORG",
                "version": "$APPVER"
              }
            }
          }
          """

          def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/DeleteApp', validResponseCodes: '100:499'

          println("Status: "+response.status)
          println("Content: "+response.content)
       }
    }
}

stage('Delete Existing Cluster') {
   steps {
       script{

          /*
           * Ideally this will check the response from the earlier step where
           * we detected a running cluster However, for our purposes here
           * we just allow response codes in the 4xx series to account for us
           * trying to delete a non-existent object.
           */

         def rBody = """
         {
           "Region": "$REGION",
           "ClusterInst": {
             "key": {
               "cloudlet_key": {
                 "operator_key": {
                   "name": "$OPKEY"
                 },
                 "name": "$CLDLET"
               },
               "developer": "$ORG",
               "developer_key": {
                 "name": "$ORG"
               },
               "cluster_key": {
                 "name": "$CLSTR"
               },
               "organization": "$ORG"
             }
           }
         }
         """

         def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/DeleteClusterInst', validResponseCodes: '100:499'

          println("Status: "+response.status)
          println("Content: "+response.content)
      }
   }
}


    stage('Build Application') {
       steps {
           script{

            /*
             * The first step in deployment is in creating an APP.
             */

             def rBody = """
             {
             "Region": "$REGION",
             "App": {
               "key": {
                 "developer_key": {
                   "name": "$ORG"
                 },
                 "name": "$APPNAME",
                 "organization": "$ORG",
                 "version": "$APPVER"
               },
               "deployment": "$TYPE",
               "access_type": $ACCTYPE,
               "image_path": "$IMGPATH",
               "image_type": $IMGTYPE,
               "default_flavor": {
                 "name": "$FLAVOR"
               },
               "revision": $REV,
               "access_ports": "$ACCPORTS"
             }
             }
             """

             def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/CreateApp'

             println("Status: "+response.status)
             println("Content: "+response.content)
         }
      }
   }

   stage('Build Cluster Instance') {
      steps {
          script{

          /*
           * Next we build a cluster instance that we can deploy to;
           * note that a smarter workflow will account for us wanting
           * to reuse our cluster.
           */

            def rBody = """
            {
              "Region": "$REGION",
              "ClusterInst": {
                "allocated_ip": "$ALLIP",
                "ip_access": $IPACC,
                "key": {
                  "cloudlet_key": {
                    "operator_key": {
                      "name": "$OPKEY"
                    },
                    "name": "$CLDLET"
                  },
                  "developer": "$ORG",
                  "developer_key": {
                    "name": "$ORG"
                  },
                  "cluster_key": {
                    "name": "$CLSTR"
                  },
                  "organization": "$ORG"
                },
                "node_flavor": "$FLAVOR",
                "deployment": "$TYPE",
                "flavor": {
                  "name": "$FLAVOR"
                }
              }
            }

            """

            def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/CreateClusterInst'

            println("Status: "+response.status)
            println("Content: "+response.content)
         }
      }
   }

  stage('Build Application Instance') {
     steps {
         script{

         /*
          * The final step is to deploy the application instance.
          */

           def rBody = """
           {
           "Region": "$REGION",
           "AppInst": {
             "key": {
               "app_key": {
                 "organization": "$ORG",
                 "name": "$APPNAME",
                 "version": "$APPVER",
                 "developer_key": {
                   "name": "$ORG"
                 }
               },
               "cluster_inst_key": {
                 "cluster_key": {
                   "name": "$CLSTR"
                 },
                 "cloudlet_key": {
                   "operator_key": {
                     "name": "$OPKEY"
                   },
                   "name": "$CLDLET"
                 },
                 "developer": "$ORG"
               }
             },
             "flavor": {
               "name": "$FLAVOR"
             }
           }
           }

           """

           def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/CreateAppInst'

           println("Status: "+response.status)
           println("Content: "+response.content)
        }
     }
  }

 }
}
