def AUTH_HEADER

pipeline {
   agent any

   stages {
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
 }
}
