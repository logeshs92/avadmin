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
	node --max_old_space_size=3000 node_modules/@angular/cli/bin/ng build --configuration=staging	
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
cp -r /data/deployment/AV-ADMIN/Node_Server/configExternal/baseconfig.json AV-ADMIN/Node_Server/configExternal/
zip -r AV-ADMIN.zip AV-ADMIN'''
      }
    }

    stage('deploy') {
      steps {
sshPublisher(publishers: [sshPublisherDesc(configName: 'AV-ADMIN', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''date=$(date +\'%Y%m%d%H%M%S\')
status=$(pm2 ls -m | grep -A10 "+--- stage" | grep status | awk '{print $3}' | head -1)
if [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip ];
then    
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip
fi
if [ "$status" == "online" ] && [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js ];
then
pm2 stop /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js
if [ -d /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN ];
then
cd /data/vhosts/live.aerovoyce.net_5454/public/
zip -r /data/vhosts/live.aerovoyce.net_5454/public/Artifact/AV-ADMIN_${date}.zip AV-ADMIN
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN
fi
else
if [ -d /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN ];
then
cd /data/vhosts/live.aerovoyce.net_5454/public/
zip -r /data/vhosts/live.aerovoyce.net_5454/public/Artifact/AV-ADMIN_${date}.zip AV-ADMIN
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
pm2 start /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js --name stage
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip
elif [ -f /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/package.json ];
then
cd /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/
npm install
pm2 start /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN/Node_Server/node.js --name stage
rm -r /data/vhosts/live.aerovoyce.net_5454/public/AV-ADMIN.zip
fi
}
post_check''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/vhosts/live.aerovoyce.net_5454/public/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'AV-ADMIN.zip')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
	  }
	}
stage('test') {
      steps {
	      sh label: '', script: '''#!/bin/bash
sleep 30
curl --connect-timeout 10 --insecure -sf "https://live.aerovoyce.net:5443" >/dev/null
curl_test=$(echo $?)
node_test=$(pm2 ls -m | grep -A10 "+--- stage" | grep status | awk \'{print $3}\' | head -1)
if [[ "$curl_test" -eq "0" ]] && [[ "$node_test" -eq "online" ]];
then
echo "Build stable"
else
echo "Build unstable"  
exit 1
fi'''
      }
}
	
     }
  }
