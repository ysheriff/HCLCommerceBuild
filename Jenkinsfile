pipeline {
  agent any
  stages {
    stage('ts-app Build') {
      stages {
       stage('Start TS-Util container') {
          steps {
            sh 'echo branch path is is $BRANCH_PATH'
            sh 'docker rm  ts -f'
            sh 'docker run -d -e LICENSE=accept --name ts --volume /opt/wcbd:/wcbd --volume $BRANCH_PATH:/code "${BUILD_UTIL_IMAGE}"'
          }
        }
        stage('Ts-app WCBD build') {
          steps {
            sh 'echo tag is $TAG'
            sh 'docker exec ${IMAGE_TYPE_UTIL} bash -c "cd /opt/WebSphere/CommerceServer90/wcbd && cp -Rn /wcbd/workspace/Stores/* /code/Stores && ./wcbd-ant -buildfile wcbd-build.xml -Dbuild.type=local -Dapp.type=ts -Dbuild.label=str-ts-$TAG -Dmodule.dir=/code -Dwork.dir=/wcbd/ -Dext.compile.class.path=/wcbd/lib/com.ibm.websphere.appserver.api.json_1.0.17.jar:/wcbd/lib/com.ibm.ws.prereq.jackson.jar:/wcbd/lib/org.eclipse.emf.common.jar:/wcbd/lib/org.eclipse.emf.ecore.jar && exit"'
          }
        }
        stage('Search-app WCBD build') {
          steps {
            sh 'echo tag is $TAG'
            sh 'docker exec ${IMAGE_TYPE_UTIL} bash -c "cd /opt/WebSphere/CommerceServer90/wcbd &&     ./wcbd-ant -buildfile wcbd-build.xml -Dbuild.type=local -Dapp.type=search -Dbuild.label=str-search-$TAG -Dmodule.dir=/code -Dwork.dir=/wcbd/ -Dext.compile.class.path=-Dext.compile.class.path=/wcbd/lib/com.ibm.websphere.appserver.api.json_1.0.17.jar:/wcbd/lib/com.ibm.ws.prereq.jackson.jar:/wcbd/lib/org.eclipse.emf.common.jar:/wcbd/lib/org.eclipse.emf.ecore.jar && exit"'
          }
        }
        stage('Copy TS-app files and expand') {
          steps {
                 sh '''
                 cd /opt/dockerBuild/ts/CusDeploy && rm -rf *
                 cp /opt/wcbd/dist/server/wcbd-deploy-server-local-ts-str-ts-$TAG.zip /opt/dockerBuild/ts/CusDeploy
                 cd /opt/dockerBuild/ts/CusDeploy && unzip wcbd-deploy-server-local-ts-str-ts-$TAG.zip
                 chown -R strbuild:strbuild /opt/dockerBuild/ts/CusDeploy
                '''
          }
        }
          stage('Copy Search-app files and expand') {
          steps {
                 sh '''
                  cd /opt/dockerBuild/search/CusDeploy && rm -rf *
                 cp /opt/wcbd/dist/server/wcbd-deploy-server-local-search-str-search-$TAG.zip /opt/dockerBuild/search/CusDeploy
                 cd /opt/dockerBuild/search/CusDeploy && unzip wcbd-deploy-server-local-search-str-search-$TAG.zip
                '''
          }
        }
        stage('Ts-app Docker Build') {
          steps {
             sh 'cd /opt/dockerBuild/ts && docker build --tag ts-app:$TAG .'
          }
        }
        stage('Search-app Docker Build') {
          steps {
               sh 'cd /opt/dockerBuild/search && docker build --tag search-app:$TAG .'
          }
        }
        stage('Push ts-app to IBMCloud Registry') {
          steps {
             sh '''
              docker tag ts-app:$TAG $DOCKER_REG$BUILD_IMAGE_TS:$TAG
              ibmcloud login --apikey=${API_KEY} -r=us-south
              ibmcloud cr login
              docker push $DOCKER_REG$BUILD_IMAGE_TS:$TAG
              echo "Docker build complete"
            '''
          }
        }
         stage('Push search-app to IBMCloud Registry') {
          steps {
              sh '''
              docker tag search-app:$TAG $DOCKER_REG$BUILD_IMAGE_SEARCH:$TAG
              ibmcloud login --apikey=${API_KEY} -r=us-south
              ibmcloud cr login
              docker push $DOCKER_REG$BUILD_IMAGE_SEARCH:$TAG
              echo "Docker build complete"
            '''
          }
        }
      }
    }
  }
  environment {
    BUILD_IMAGE_TS = 'ts-app'
    BUILD_UTIL_IMAGE = 'us.icr.io/str-np/ts-utils:9.0.1.10'
    BUILD_IMAGE_SEARCH = 'search-app'
    BUILD_TYPE = 'ts'
    BUILD_TYPE_UTIL = 'ts'
    DOCKER_REG = 'us.icr.io/str-np/'
    IMAGE_TYPE = 'ts-app'
    IMAGE_TYPE_UTIL = 'ts'
    WC9_VERSION = '9.0.1.10'
    BUILD_APP_NAME = 'Tsapp'
    TAG = "${sh(script:'date +"%m%d%y.%H%M"', returnStdout: true).trim()}"
    BRANCH_PATH = "${env.WORKSPACE}"
    API_KEY = "YOUR_IBM_CLOUD_API_KEY"
  }
  post {
    success {
      sh '''
         echo "success"
      '''
    }
    failure {
      sh '''
        echo "failure"
      '''
    }
  }
}
