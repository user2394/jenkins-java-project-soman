pipeline {
  agent none

  environment {
   	MAJOR_VERSION = 1

   }

  stages {
	stage('Say Hello') {
	  agent any
          steps {
            sayHello 'AWESOME student!'
           }
    }
 
  stages {
    stage('Unit Tests'){
      agent {
          label 'apache'
       }
      steps {
	sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
   }
    
  


  stage('build') {
      agent {
          label 'apache'
       }
      steps {
	sh 'ant -f build.xml -v'
       }
     post {
       always {
          archiveArtifacts  artifacts: 'dist/*.jar', fingerprint:true
        }
    }
     }
   
   stage('deploy') {
      agent {
          label 'apache'
       }
     steps {
       sh "if ![-d '/var/www/html/rectangles/all/${env.BRANCH_NAME}']; then mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
       sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
       }
     }
 
   stage("Running on CentOS") {
      agent {
          label 'apache'
       }
     steps{
       sh "wget http://iamsoman1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
       sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
	}
     }


  stage("Test on Debian"){
	agent {
	  docker 'openjdk:8u121-jre'
         }
        steps{
           sh "wget http://iamsoman1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"  
           sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
        }
    }

 stage("Promote to green") {
    agent{
       label 'apache'
       }
     when {
      branch 'master'
      }
     steps{
        sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar  /var/www/html/rectangles/green/"
	}
      }
    
   stage("Promote Development Branch to Master") {
     agent {
       label 'apache'

    }
     when {
       branch 'development'
      }
     steps {
        echo 'checking into development'
        sh 'git checkout development'
        sh 'git pull origin development'
        echo 'checkingout master'
        sh 'git checkout master'
        sh 'git pull origin master'
        echo 'merging development into master branch'
        sh 'git merge development'
        echo 'Pushing to origin master'
        sh 'git push origin master'
        echo 'Tagging the release'
        sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
}
 

    post {
      success {
         emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master!",
            body:"""<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master!":</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "iamsoman94@gmail.com"
           )
}
}
}
} 
     post {
        failure {
         emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] FAILED!",
            body:"""<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' FAILED!":</p>
            <p>Check console output att &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "iamsoman94@gmail.com"
           )
}
}
}

