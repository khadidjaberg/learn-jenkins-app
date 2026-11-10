pipeline {
    agent any

    stages {
        stage('build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
              
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm config set registry https://registry.npmmirror.com
                    npm config set replace-registry-host always
                    npm config set fetch-timeout 300000
                    npm config set fetch-retries 5
                    npm config set fetch-retry-mintimeout 20000
                    npm config set fetch-retry-maxtimeout 120000
                    npm ci --prefer-offline --no-audit --no-fund
                    npm run build
                    ls -la
                        '''

            }
        }
        stage('test'){
              agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    test -f build/index.html 
                    npm test
                  '''
        }
        }
    }

    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}