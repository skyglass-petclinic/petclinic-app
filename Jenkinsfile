def imageName = 'skyglass/petclinic'
def registry = 'https://registry.hub.docker.com'

node('workers'){
    def imageTest

    stage('Checkout'){
        checkout scm
    }

    stage('Unit Tests'){
        imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
        echo '=== Testing Petclinic Application ==='
                '-v /media/data/tmp/go:/tmp/go'
        imageTest.inside(" -v $PWD/target:/app/target -v $HOME/.m2:/root/.m2 -u root") {
            sh "mvn clean test"
        }
        junit "target/surefire-reports/*.xml"

    }

    stage('Build'){
        echo '=== Packaging Petclinic Application ==='
        imageTest.inside(" -v $PWD/target:/app/target -v $HOME/.m2:/root/.m2 -u root") {
            sh "mvn -B -DskipTests package"
        }
    }

    stage('Push'){
        echo '=== Pushing Docker Image ==='
        docker.withRegistry(registry, 'dockerHubCredentials') {
            imageTest.inside(" -v $PWD/target:/app/target -v $HOME/.m2:/root/.m2 -u root") {
                sh "mvn jib:build"
            }
        }
    }

    stage('Cleanup'){
        echo '=== Delete the local docker images ==='
        sh("docker rmi -f ${imageName}:latest || :")
        if (env.BRANCH_NAME == 'master') {
            sh("docker rmi -f ${imageName}:master || :")
        }
    }      

    stage('Deploy'){
        if(env.BRANCH_NAME == 'master'){
            build job: "petclinic-deployment/master"
        }
    }    
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}