node('maven'){
    // define commands
    def ocCmd = "oc --token=`cat /var/run/secrets/kubernetes.io/serviceaccount/token` --server=https://openshift.default.svc.cluster.local --certificate-authority=/run/secrets/kubernetes.io/serviceaccount/ca.crt"
    def mvnCmd = "mvn -s configuration/cicd-settings.xml"
    stage('build')
        git branch: 'master', url: 'https://github.com/jr00n/docker-workshop-java-todolist.git'
        sh "${mvnCmd} clean install -DskipTests=true"
    stage('test')
        sh "${mvnCmd} test"
    stage 'Push to Nexus'
        sh "${mvnCmd} deploy -DskipTests=true"
    stage('deploy')
        sh "rm -rf oc-build && mkdir -p oc-build/deployments"
        sh "cp todolist-web-servlet-jsp/target/todolist.war oc-build/deployments/ROOT.war"
        // clean up. keep the image stream
        //sh "${ocCmd} delete bc,dc,svc,route -l app=todo -n team-a"
        openshiftDeleteResourceByLabels(types:'bc,dc,svc,route,build',keys:'app',values:'todo')
        // build WildFly image with WAR
        //sh "${ocCmd} new-build --name=todo --image-stream=jboss-eap70-openshift --binary=true --labels=app=todo -n team-a || true"
        sh "${ocCmd} new-build --name=todo --image-stream=wildfly:latest --binary=true --labels=app=todo -n team-a || true"
        sh "${ocCmd} start-build todo --from-dir=oc-build --wait=true -n team-a"
        // deploy image
        sh "${ocCmd} new-app todo:latest -n team-a"
        // expose, make a route for todo app
        sh "${ocCmd} expose svc/todo -n team-a"
}
