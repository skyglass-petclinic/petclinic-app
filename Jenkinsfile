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

    stage('Build & Push'){
        echo '=== Packaging Petclinic Application ==='
        docker.withRegistry(registry, 'dockerHubCredentials') {
            imageTest.inside(" -v $PWD/target:/app/target -v $HOME/.m2:/root/.m2 -u root") {
                sh "mvn -B -DskipTests clean package"
                sh "mvn jib:build"
            }
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