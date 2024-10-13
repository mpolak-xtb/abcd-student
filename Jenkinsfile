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
        stage('DAST') {
            steps {
                sh '''
                     docker run --name zap \
                                    --add-host=host.docker.internal:host-gateway \
                                    -v /mnt/c/users/polak/IdeaProjects/abcd-student/.zap:/zap/wrk/:rw \
                                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                                    "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                                    || true
                     '''
            }
            post {
                    always {
                        sh '''
                            docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                            docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        '''
                    }
                }
        }
    }
    post {
        always {
            sh '''
                docker stop zap juice-shop
                docker rm zap
            '''
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo...'
            defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'maciej.polak@xtb.com')
        }

    }
}
