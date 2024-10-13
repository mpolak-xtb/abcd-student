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
        stage('DAST') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                    -p 3000:3000 bkimminich/juice-shop
                    sleep 15
                '''
                sh '''
                    docker run name zap \
                        -v /Users/maciejpolak/Projects/Bezpiecznykod/demos/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy: stable \
                        bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml || true
                '''
            }
        }
    }
}
