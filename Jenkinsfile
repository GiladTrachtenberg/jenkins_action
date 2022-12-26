pipeline{

    agent any

    tools {
        maven 'maven-3.6.2'
        jdk 'jdk8'
    }

    stages{
        stage('VERSION-CALCULATION stage'){
            when { expression { env.GIT_BRANCH ==~ /release(.+)/ } }   
            steps {
                script {
                    sh "git fetch --tags || true"
                    
                    TAG=env.GIT_BRANCH.split('\\/') 
                    VERSION= TAG[1]
                    HIGHEST = sh(script: "git tag -l --sort=version:refname \"${VERSION}.*\" | tail -1", returnStdout: true).trim()
                    if (HIGHEST.isEmpty()) {
                        REG =".0"
                        NEW_TAG=VERSION+REG
                    } else {
                        NEW_TAG=HIGHEST.split('\\.')
                        NEW_TAG[2]=NEW_TAG[2].toInteger()+1
                        NEW_TAG=NEW_TAG.join('.')
                    }
                    sh "mvn versions:set -DnewVersion=${NEW_TAG}"
                    sh "mvn versions:commit"
                }
            }
        } 
       stage('BUILDING stage'){
            steps {
                configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML clean compile"
                }
            }
        }
        stage('TESTING stage'){
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
                script{
                    env.GIT_COMMIT_MSG = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                    }
            }
        }
        stage('E2E testing'){
            when { expression {env.GIT_BRANCH == 'master' || (env.GIT_BRANCH ==~ /feature(.+)/ && env.GIT_COMMIT_MSG == 'e2e') }}
            steps{
                script {
                         configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS')]){
                            sh "mvn -s $MAVEN_SETTINGS dependency:get -DrepoUrl=http://artifactory:8081/artifactory/libs-snapshot-local -Dartifact=com.lidar:simulator:99-SNAPSHOT:jar -Dtransitive=false -Ddest=simulator.jar"
                            sh "mvn -s $MAVEN_SETTINGS dependency:get -DrepoUrl=http://artifactory:8081/artifactory/libs-snapshot-local -Dartifact=com.lidar:analytics:99-SNAPSHOT:jar -Dtransitive=false -Ddest=analytics.jar"
                        }
                        sh "cp target/*.jar app.jar"
                        sh "java -cp app.jar:analytics.jar:simulator.jar com.lidar.simulation.Simulator"   
                }
                

            }
        }
        stage('PUBLISH stage'){
            when { expression {env.GIT_BRANCH == 'master' || env.GIT_BRANCH ==~ /release(.+)/}}
            steps {
                configFileProvider([configFile(fileId: 'settings', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML clean deploy -DskipTests"
                }
            }
        }
        stage('RELEASE-TAG stage'){
            when { expression {env.GIT_BRANCH ==~ /release(.+)/ } }
            steps{
                sh "git clean -f"
                sh "git tag ${NEW_TAG}"
                withCredentials([gitUsernamePassword(credentialsId: 'GITLAB',
                 gitToolName: 'git-tool')]) {
                   sh "git push --tags"
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