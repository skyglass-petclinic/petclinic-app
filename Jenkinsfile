def imageName = 'skyglass/petclinic'
def registry = 'https://registry.hub.docker.com'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    stage('Unit Tests'){
        def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
        echo '=== Testing Petclinic Application ==='
                '-v /media/data/tmp/go:/tmp/go'
        imageTest.inside(" -v $PWD/target:/app/target -v $HOME/.m2:/root/.m2 -u root") {
            sh "mvn clean test"
        }
        junit "target/surefire-reports/*.xml"

    }

    stage('Package'){
        def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
        echo '=== Packaging Petclinic Application ==='
        imageTest.inside(" -v $PWD/target:/app/target -v $HOME/.m2:/root/.m2 -u root") {
            sh " mvn -B -DskipTests install"
        }
    }

    stage('Build'){
        docker.build(imageName)
    } 

    stage('Push'){
        docker.withRegistry(registry, 'dockerHubCredentials') {
            docker.image(imageName).push(commitID())

            if (env.BRANCH_NAME == 'master') {
                docker.image(imageName).push('master')
            }
        }
    }

    stage('Remove local images'){
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