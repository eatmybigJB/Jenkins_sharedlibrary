#!groovy

@Library('jenkinslibrary@master') _

//func from shareibrary

String BRANCH_NAME = "${env.BRANCH_NAME}"


def tools = new org.devops.detail_print()






//pipeline
pipeline{
    agent { 
        node { 
            label "jenkins-jenkins-agent"
            }
        }
    
    
    stages{

        stage("CheckOut"){
            steps{
                script{
                   
                    
                    println("${BRANCH_NAME}")
                
                    tools.Printcollor("获取代码","green")


                }
            }
        }
 
    
    
}

}