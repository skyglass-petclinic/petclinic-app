def imageName = 'skyglass/petclinic'
def registry = 'https://registry.hub.docker.com'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    stage('Unit Tests'){
        def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
        echo '=== Testing Petclinic Application ==='
        sh "docker run --rm -v $PWD/app:/app ${imageName}-test mvn test"
        junit '$PWD/app/target/surefire-reports/*.xml'
    }

    stage('Package'){
        echo '=== Testing Petclinic Application ==='
        sh "docker run --rm -v $PWD/target:/app/target ${imageName}-test mvn -B -DskipTests clean package"
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
        sh("docker rmi -f skyglass/petclinic-online:latest || :")
        if (env.BRANCH_NAME == 'master') {
            sh("docker rmi -f skyglass/petclinic-online:master || :")
        }
    }      

    stage('Deploy'){
        if(env.BRANCH_NAME == 'master'){
            build job: "watchlist-deployment/master"
        }
    }    
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}