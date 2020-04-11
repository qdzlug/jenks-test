def AUTH_HEADER

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

                def TRegion = "EU"

                def rBody = """
                {"region": "EU"}
                """

                httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: false, name: 'Authorization', value: '$AUTH_HEADER']], httpMode: 'POST', requestBody: '  \'{"region": "EU", "appinst": { "key": { "app_key": {"name": "testapp", "version": "1.0" }}}}', url: 'https://console.mobiledgex.net/api/v1/auth/ctrl/ShowAppInst '

                println("Status: "+response.status)
                println("Content: "+response.content)
                }
        }
    }
 }
}
