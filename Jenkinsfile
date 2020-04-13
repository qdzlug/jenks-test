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


pipeline {
   agent any

   stages {

    stage('Clone Repository') {
        steps {
        /* Let's make sure we have the repository cloned to our workspace */

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


    stage('Login and get JWT') {
         steps {
            script{

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
                }
        }
    }
    stage('Build Application') {
       steps {
           script{


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
               "internal_ports": $INTPORTS,
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

 }
}
