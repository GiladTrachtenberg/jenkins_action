pipeline{

    agent any

    tools {
        maven 'maven-3.6.2'
        jdk 'jdk8'
    }

    stages{

       stage('BUILDING stage'){
            steps {
                configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML clean compile"
                }
            }
        }

        stage('UNIT-TESTING stage'){
            steps {
                configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML clean test"
                }
            }
        }

        stage('PACKAGING stage'){
            steps {
                configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML clean package"
                }
            }
        }

        stage('E2E stage'){
            when { expression {env.GIT_BRANCH == 'master' }}
            steps{
                script {
                         configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS')]){
                            sh "mvn -s $MAVEN_SETTINGS dependency:get -DrepoUrl=http://artifactory:8081/artifactory/libs-snapshot-local -Dartifact=com.lidar:analytics:99-SNAPSHOT:jar -Dtransitive=false -Ddest=analytics.jar"
                            sh "mvn -s $MAVEN_SETTINGS dependency:get -DrepoUrl=http://artifactory:8081/artifactory/libs-snapshot-local -Dartifact=com.lidar:telemetry:99-SNAPSHOT:jar -Dtransitive=false -Ddest=telemetry.jar"
                        }
                        sh "cp target/*.jar app.jar"
                        sh "java -cp app.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"   
                }
                
            }
        }

        stage('PUBLISH stage'){
            steps {
                configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML clean deploy -DskipTests"
                }
            }
        }
    }
    
    post{
        always {
            deleteDir()
        }
    }
}