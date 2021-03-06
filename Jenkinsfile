pipeline {
 agent none
environment {
MAJOR_VERSION = 1
    }

 stages {
   stage ('Unit test')
   {
       agent {
label 'apache'
           }
steps {
 sh 'ant -f test.xml -v'
 junit 'reports/result.xml'
    }
       }
   stage ('build'){

       agent {
label 'apache'
           }
steps {
     sh 'ant -f build.xml -v'
   }
post {
   success {
    archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
}
}
stage ('deploy') {
 agent {
label 'apache'
           }
steps {
sh "if ![ -d 'var/www/html/rectangles/all/${env.BRANCH_NAME}']; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi "
sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}"
    }
    }
stage ('Running on Centos'){
    agent {
label 'Centos'
        }
steps {
sh "wget http://a4645082b.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
    }
 }
stage ('Running on Debian')
{
agent {
docker 'openjdk:8u121-jre'
    }
    steps {
sh "wget http://a4645082b.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
    }
    }
stage ('Stage Promotion'){
agent {
label 'apache'
}

when {
branch 'master'
    }

steps {
sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar "
    }

    }
stage ('Promote Development branch to master')
{
agent {
label 'apache'
    }
    when {
branch 'development'
        }

    steps {
echo " stashing local changes"
sh 'git stash'
echo "Checking out Development branch"
sh 'git checkout development'
sh 'git pull origin'
echo "Checking out master branch"
sh 'git checkout master'
echo "Merging development into master branch"
sh 'git merge development'
echo 'Pushing to master'
sh 'git push origin master'
echo "Tagging release"
sh "git tag ${env.MAJOR_VERSION}-${env.BUILD_NUMBER}"
sh "git push origin ${env.MAJOR_VERSION}-${env.BUILD_NUMBER}"
        }
}
}
}
