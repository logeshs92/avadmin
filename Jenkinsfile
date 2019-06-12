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
	      sshPublisher(publishers: [sshPublisherDesc(configName: 'AV-ADMIN', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''date=$(date +\'%Y%m%d%H%M%S\')
status=$(pm2 ls -m | grep status | awk \'{print $3}\')
if [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip ];
then    
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip
fi
if [ "$status" == "online" ];
then
pm2 stop /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js
if [ -d /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN ];
then
#mv /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN /data/vhosts/live.aerovoyce.net_5454/public/Artifact/AV-ADMIN_${date}
zip -r /data/vhosts/live.aerovoyce.net_5454/public/Artifact/AV-ADMIN_${date}.zip /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN
fi
else
if [ -d /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN ];
then
#mv /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN /data/vhosts/live.aerovoyce.net_5454/public/Artifact/AV-ADMIN_${date}
zip -r /data/vhosts/live.aerovoyce.net_5454/public/Artifact/AV-ADMIN_${date}.zip /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN
fi
fi''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: ''), sshTransfer(cleanRemote: false, excludes: '', execCommand: '''post_check() {
cd /data/vhosts/live.aerovoyce.net_5454/public/
if [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip ];
then
unzip AV-ADMIN.zip
post
else
echo "AV-ADMIN.zip does not exist"
fi
}

post () {
if [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/package.json ];
then
cd /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/
npm install
pm2 start /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip
elif [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/package.json ];
then
cd /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/
npm install
pm2 start /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip
fi
}
post_check''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/vhosts/live.aerovoyce.net_5454/public/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'AV-ADMIN.zip')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
      	}
	    }
	
	
	  }
	}
