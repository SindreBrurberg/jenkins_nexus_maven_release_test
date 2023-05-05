def vers
def outFile
def release = false
pipeline {
    agent any
    tools {
        go 'Go 1.20'
        maven 'Mvn'
    }
    environment {
        NEXUS_CREDS = credentials('Cantara-NEXUS')
    }
    stages {
		stage('release') {
            def performRelease = input  message             : "Perform Maven Release?",
                                        ok                  : "Schedule Maven Release Build",
                                        submitter           : env.ALLOWED_SUBMITTER_RELEASE,
                                        submitterParameter  : 'APPROVING_SUBMITTER',
                                        parameters:
                                        [
                                            booleanParam
                                            (
                                                defaultValue: true,
                                                description: '',
                                                name: 'Dry run only?'
                                            ),
                                            string
                                            (
                                                defaultValue: '',
                                                description: '',
                                                name: 'Release Version'
                                            ),
                                            string
                                            (
                                                defaultValue: '',
                                                description: '',
                                                name: 'Development version'
                                            )
                                        ]

            if( performRelease )
            {
                echo "Release values ${performRelease['Dry run only?']}, ${performRelease['Development version']}, ${performRelease['Release Version']}"
                dir( env.PROJECT_FOLDER )
                {
                    echo "Release values ${performRelease['Dry run only?']}, ${performRelease['Development version']}, ${performRelease['Release Version']}"
                }
            }
        }
        stage("pre") {
            steps {
                script {
                    if (env.TAG_NAME) {
                        vers = "${env.TAG_NAME}"
                        release = true
                    } else {
                        vers = "${env.GIT_COMMIT}"
                    }
                    artifactId = "nerthus2"
                    outFile = "${artifactId}-${vers}"
                    echo "New file: ${outFile}"
                }
            }
        }
        stage("test") {
            steps {
                script {
                    testApp()
                }
            }
        }
        stage("build") {
            steps {
                script {
                    echo "V: ${vers}"
                    echo "File: ${outFile}"
                    buildApp(outFile, vers)
                }
            }
        }
        stage("deploy") {
            steps {
                script {
                    echo 'deplying the application...'
                    echo "deploying version ${vers}"
                    if (release) {
                        //sh "find . -name '${outFile}-*' -type f -exec curl -v -u "+'$NEXUS_CREDS'+" --upload-file {} https://mvnrepo.cantara.no/content/repositories/releases/no/cantara/gotools/${artifactId}/${vers}/{}  \\;"
                    } else {
                        //sh "find . -name '${outFile}-*' -type f -exec curl -v -u "+'$NEXUS_CREDS'+" --upload-file {} https://mvnrepo.cantara.no/content/repositories/snapshots/no/cantara/gotools/${artifactId}/${vers}/{}  \\;"
                    }
                    sh "rm ${outFile}-*"
                }
            }
        }
    }
}

def testApp() {
    echo 'testing the application...'
    sh './testRecursive.sh'
}

def buildApp(outFile, vers) {
    echo 'building the application...'
    //buildFlags = "-X 'github.com/cantara/gober/webserver/health.Version=${vers}' -X 'github.com/cantara/gober/webserver/health.BuildTime=\$(date)' -X 'github.com/cantara/gober/webserver.Name=${artifactId}' "
}
