
def doBuild() {
    dir("src/github.com/keybase/dokany") {
        retry(3) {
            checkout([
                scm: [
                    $class: 'GitSCM',
                    branches: [[name: Revision]],
                    userRemoteConfigs: [[url: "https://github.com/keybase/dokany.git"]],
                ]
            ])
        }
    }
    dir("src/github.com/keybase/dokany/sys") {
        bat "if EXIST sys.vcxproj.user del sys.vcxproj.user"
        bat "copy sys.vcxproj.user.keybase sys.vcxproj.user"
    }
    
    withEnv(["SIGNTOOL=signtool",
             "ADDITIONALCERT=/a",
             "CERTISSUER=\"DigiCert\"",
             "KEYBASE_WINBUILD=${env.BUILD_NUMBER}"]) {

        stage('Build Drivers') {
            dir("src/github.com/keybase/dokany/dokan_wix") {
                bat '''
                    powershell -Command "(gc version.xml) -replace 'BuildVersion=\".+\"', 'BuildVersion=\"%BUILD_NUMBER%\"' | sc version.xml"
                    powershell -Command "(gc version.xml)"
                    echo ADDITIONALCERT %ADDITIONALCERT%
                    echo CERTISSUER %CERTISSUER%
                    echo KEYBASE_WINBUILD %KEYBASE_WINBUILD%
                    pwd
                    dir /s /b version.xml
                    "%ProgramFiles(x86)%\\Microsoft Visual Studio 14.0\\vc\\bin\\vcvars32.bat" && build.bat
                '''
                archiveArtifacts 'src/github.com/keybase/dokany/dokan_wix/DokanSetup.exe,src/github.com/keybase/dokany/dokan_wix/DokanSetup_redist.exe,src/github.com/keybase/dokany/Win32/**/*,src/github.com/keybase/dokany/x64/**/*,src/github.com/keybase/dokany/dokan_wix/**/*'                
            }
        } 
    }
}

node ('windows-release') {
    ws("${WORKSPACE}_${EXECUTOR_NUMBER}") {
        withEnv(["GOPATH=${pwd()}"]) {
            doBuild()
        }
    }
}