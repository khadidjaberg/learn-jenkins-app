pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = '6f6ac398-b9c5-4df1-8818-a81165b8dec4'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
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
        stage('tests'){
            parallel{
                stage('Unit tests'){
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

    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
                }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh'''
                            npm config set registry https://registry.npmmirror.com
                            npm config set replace-registry-host always
                            npm config set fetch-timeout 300000
                            npm config set fetch-retries 5
                            npm config set fetch-retry-mintimeout 20000
                            npm config set fetch-retry-maxtimeout 120000
                            npm install serve --no-audit --no-fund
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

    post{
        always{
           publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: 'playwright-report',
            reportFiles: 'index.html',
            reportName: 'Playwright HTML Report',
            reportTitles: '',
            useWrapperFileDirectly: true
        ])
        }
    }
                }
                 stage('Deploy') {
                     agent{
                           docker{
                               image 'node:18-alpine'
                               reuseNode true
                             }
                         }
                    steps {

                      sh'''
                            npm config set registry https://registry.npmmirror.com
                            npm config set replace-registry-host always
                            npm config set fetch-timeout 300000
                            npm config set fetch-retries 5
                            npm config set fetch-retry-mintimeout 20000
                            npm config set fetch-retry-maxtimeout 120000

                            # sharp telecharge son binaire depuis GitHub (bloque chez toi) -> on utilise le miroir
                            export npm_config_sharp_binary_host="https://npmmirror.com/mirrors/sharp"
                            export npm_config_sharp_libvips_binary_host="https://npmmirror.com/mirrors/sharp-libvips"

                            npm install netlify-cli --no-audit --no-fund
                            node_modules/.bin/netlify --version
                            echo "Deploying to production, Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                        '''

                      }
        }
            }
        }
    }

}