// https://github.com/jfrog/project-examples/blob/master/jenkins-examples/pipeline-examples/scripted-examples/conan-example/Jenkinsfile

def serverId = 'Artifactory'
def conanStagingRepo = 'lde-conan-staging'
def ref = 'conan-artifactory-issue/1.0.0@user/testing'

node('Linux') {
    // Clone the code from github:
    git url :'https://github.com/demirole/conan-artifactory-issue.git'

    // Obtain an Artifactory server instance, defined in Jenkins --> Manage Jenkins --> Configure System:
    def server = Artifactory.server serverId

    // Create a local build-info instance:
    String conanHome = "${env.WORKSPACE}/conan_home"
    def buildInfo = Artifactory.newBuildInfo(userHome: conanHome)
    buildInfo.name = "Conan-pipeline"

    // Create a conan client instance:
    def conanClient = Artifactory.newConanClient()

    // Add a new repository named 'conan-local' to the conan client.
    // The 'remote.add' method returns a 'serverName' string, which is used later in the script:
    String serverName = conanClient.remote.add server: server, repo: conanStagingRepo
    conanClient.run(command: "profile new --detect")

    // Run a conan build. The 'buildInfo' instance is passed as an argument to the 'run' method:
    String command = "create . ${ref} -pr:b default -pr default"
    conanClient.run(command: command, buildInfo: buildInfo)

    // Create an upload command. The 'serverName' string is used as a conan 'remote', so that
    // the artifacts are uploaded into it:
    String command = "upload * --all -r ${serverName} --confirm"

    // Run the upload command, with the same build-info instance as an argument:
    conanClient.run(command: command, buildInfo: buildInfo)

     // Publish the build-info to Artifactory:
    server.publishBuildInfo buildInfo
}
