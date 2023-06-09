// https://github.com/jfrog/project-examples/blob/master/jenkins-examples/pipeline-examples/scripted-examples/conan-example/Jenkinsfile

def serverId = 'Artifactory-Prod'
def conanStagingRepo = 'lde-conan-staging'
def ref = 'conan-artifactory-issue/1.0.0@user/testing'
String command

node('Windows') {

    cleanWs()
    env.CONAN_USE_ALWAYS_SHORT_PATHS = 'True'
    env.CONAN_ENABLE_REVISIONS = 'True'

    // Clone the code from github:
    checkout(scm)

    // Obtain an Artifactory server instance, defined in Jenkins --> Manage Jenkins --> Configure System:
    def server = Artifactory.server serverId

    // Create a local build-info instance:
    String conanHome = "${env.WORKSPACE}/conan_home"
    def buildInfo = Artifactory.newBuildInfo()

    // Create a conan client instance:
    def conanClient = Artifactory.newConanClient(userHome: conanHome)

    // Add a new repository named 'conan-local' to the conan client.
    // The 'remote.add' method returns a 'serverName' string, which is used later in the script:
    String serverName = conanClient.remote.add server: server, repo: conanStagingRepo
    conanClient.run(command: "profile new --detect default")

    // Run a conan build. The 'buildInfo' instance is passed as an argument to the 'run' method:
    command = "create . ${ref} -pr:b default -pr default"
    conanClient.run(command: command, buildInfo: buildInfo)
    buildInfo.env.collect()

    // Create an upload command. The 'serverName' string is used as a conan 'remote', so that
    // the artifacts are uploaded into it:
    command = "upload * -r ${serverName} --confirm --all --parallel --check --force"

    // Run the upload command, with the same build-info instance as an argument:
    conanClient.run(command: command, buildInfo: buildInfo)

     // Publish the build-info to Artifactory:
    server.publishBuildInfo buildInfo
}
