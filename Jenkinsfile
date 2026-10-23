pipeline {
    agent any

    environment {
        TESTLAB_URL     = 'https://durango-dms.andolasoft.co.in'
        TESTLAB_PROJECT = 'bd1069318a3d03fb0836ff5986406470'
    }

    stages {
        stage('Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                        python3 -m venv .venv
                        .venv/bin/pip install -r requirements.txt
                        .venv/bin/pytest --junitxml=results.xml
                    '''
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'results.xml'
            withCredentials([string(credentialsId: 'testlab-token', variable: 'TESTLAB_TOKEN')]) {
                sh '''
                    curl -sS --fail-with-body -X POST "$TESTLAB_URL/test-lab/api/automation/results" \
                      -H "Authorization: Bearer $TESTLAB_TOKEN" \
                      -F project_id="$TESTLAB_PROJECT" \
                      -F name="Jenkins #$BUILD_NUMBER" \
                      -F source_ref="$BUILD_URL" \
                      -F junit=@results.xml
                '''
            }
        }
    }
}
