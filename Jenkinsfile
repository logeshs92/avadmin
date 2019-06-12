pipeline {
  agent any
  stages {
    stage('checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '4d360902-7a89-4e66-86a8-0135affcbeb3', url: 'http://projects.snovabits.net/git/snovabits/av-admin.git']]])
      }
    }

    stage('npm install') {
      steps {
        sh label: '', script: '''npm install
	npm rebuild node-sass --force
	node --max_old_space_size=2048 node_modules/@angular/cli/bin/ng build --configuration=staging	
	tee AV-ADMIN/.htaccess << EOF
	<IfModule mod_rewrite.c>
	RewriteEngine On
	RewriteBase /
	RewriteRule ^index.html$ - [L]
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule . ./AV-ADMIN/index.html [L]
	</IfModule>
	EOF
	cp -r Node_Server AV-ADMIN/
	cp -r package.json AV-ADMIN/
	zip -r AV-ADMIN.zip AV-ADMIN'''
      }
    }

    stage('deploy') {
      steps {
sshPublisher(publishers: [sshPublisherDesc(configName: 'AV-ADMIN', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/vhosts/live.aerovoyce.net_5454/public/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'AV-ADMIN.zip')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
	  }
	}
	
     }
  }
