#!groovy

@Library('jenkinslibrary@master') _

//func from shareibrary

def tools = new org.devops.tools()






//pipeline
pipeline{
    agent { 
        node { 
            label "jenkins/jenkins-jenkins-agent"
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