#!groovy

@Library('jenkinslibrary@master') _

//func from shareibrary

String BUILD_NUMBER = "${BUILD_NUMBER}"


def tools = new org.devops.detail_print()






//pipeline
pipeline{
    agent { 
        node { 
            label "python"
             //label "jenkins-jenkins-agent"
            //label "maven"
            //label "jenkins-python"
            }
        }
    
    
    stages{

        stage("CheckOut"){
            steps{
                script{
                   
                    
                    println("${BUILD_NUMBER}")
                
                    tools.Printcollor("获取代码","green")
                    //result =  sh returnStdout: true, script: "whoami"
                    result = sh(script: "echo ${PYTHON_SETUPTOOLS_VERSION}", returnStdout: true).trim()

                    tools.Printcollor("${result}","blue")


                }
            }
        }
 
    
    
}

}