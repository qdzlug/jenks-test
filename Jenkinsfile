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

    stage('Clone repository') {
        steps {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
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

     stage('Check for Running Instance') {
        steps {
            script{

              REGION="EU"

              def rBody = """
              {"region": "$REGION"}
              """

              def response = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: "$AUTH_HEADER"]], httpMode: 'POST', requestBody: rBody, url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/ShowAppInst'

                println("Status: "+response.status)
                println("Content: "+response.content)
                }
        }
    }
 }
}
