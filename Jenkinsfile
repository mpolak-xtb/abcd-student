pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/mpolak-xtb/abcd-student', branch: 'main'
                }
            }
        }
        stage("Juice Shop") {
         steps {
                        sh '''
                            docker run --name juice-shop -d --rm \
                            -p 3000:3000 bkimminich/juice-shop
                        '''
            }
        }
        stage('osv-scanner') {
            steps {
            sh 'mkdir -p ${WORKSPACE}/results'
            sh 'osv-scanner scan --lockfile package-lock.json --format json --output ${WORKSPACE}/results/sca-osv-scanner.json'
//                 sh '''
//                      docker run --name osv \
//                                     -v C:/Users/polak/IdeaProjects/abcd-student/.zap:/src/:rw \
//                                     -w /src \
//                                     -t ghcr.io/google/osv-scanner:latest bash -c \
//                                     "ls; osv-scanner scan --lockfile package-lock.json --output scan-results.txt" \
//                                     || true
//                      '''
            }
            post {
                    always {
                        sh '''


                        '''
                    }
                }
        }
    }
    post {
    always {
            sh '''
                docker stop juice-shop
            '''
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo...'
            defectDojoPublisher(artifact: 'results/sca-osv-scanner.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'maciej.polak@xtb.com')
        }

    }
}
