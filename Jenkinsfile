/**
* Fichier permettant de mettre à jour le script sonar pour iOS. 
* Ce script doit être lancé depuis une autre job avec ka Groovy Sandbox de désactivé pour avoir accès à la liste des nodes :
*
*   node("linux") {
*    stage 'GIT Checkout'
*    git branch: 'develop', url: 'https://github.com/STEAMULO/sonar-objective-c.git'
*    load 'Jenkinsfile'
*   }
*
*/

import hudson.model.Node
import hudson.model.Slave
import jenkins.model.Jenkins
    
try {
    def nodeNameOSX = []
    def nodeNameOffLine = []
    stage ("Liste Slaves") {
        int nbNodes = Jenkins.instance.nodes.size()
        //On ne peut pas itérer directement sur un node, car il n'est pas sérializable
        for (i = 0; i <nbNodes; i++) {
            def nodeName = Jenkins.instance.nodes[i].nodeName
            def isOnline = Jenkins.instance.nodes[i].toComputer().online
            if (nodeName.toLowerCase().contains("osx")) {
                if (isOnline) {
                    nodeNameOSX.add(nodeName)
                }
                else {
                    nodeNameOffLine.add(nodeName)
                }
            }
        }
    }
    
    stage ("Installation runner sonar") {
        installRunnerSonar(nodeNameOSX)
    }
        
    if (nodeNameOffLine.size() > 0) {
        stage ("Send Mail OffLine") {
            echo "Des nodes ne l'ont pas installé car offline"
            emailext body: """Bonjour,

Le run-sonnar Objs n\'a pas \u00E9t\u00E9 install\u00E9 sur ces Slaves car ils sont hors ligne.
Merci de les mettre onLine et de relancer le job une fois ceux-ci disponibles.
        
$nodeNameOffLine

Bonne journ\u00E9e
        """, subject: 'Installation du runner sonar Objc impossible sur Slave OffLine', to: 'tlanquetin@steamulo.com'
        }
    }
} catch (any) {
        currentBuild.result = 'FAILURE'
        throw any //rethrow exception to prevent the build from proceeding
} finally {
    step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'tlanquetin@steamulo.com', sendToIndividuals: true])
}


def installRunnerSonar(List nodeNameList) {
    for (nodeName in nodeNameList) {
        node ("$nodeName") {
            echo ("Installation sur $nodeName")
            git branch: 'develop', url: 'https://github.com/STEAMULO/sonar-objective-c.git'
            def scriptPath = "src/main/shell/run-sonar.sh"
            def osxPath = "/usr/local/bin/"
            sh "cp -f ${scriptPath} ${osxPath}."
            step([$class: 'WsCleanup'])
        }
    }
}