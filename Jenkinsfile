set SIGNTOOL=signtool
set ADDITIONALCERT=/a
set CERTISSUER="DigiCert"
call "%ProgramFiles(x86)%\Microsoft Visual Studio 14.0\vc\bin\vcvars32.bat"
set KEYBASE_WINBUILD=%BUILD_NUMBER%
pushd src\github.com\keybase\dokany\sys
rename sys.vcxproj.user.keybase sys.vcxproj.user
popd
cd src\github.com\keybase\dokany\dokan_wix
powershell -Command "(gc version.xml) -replace 'BuildVersion=\".+\"', 'BuildVersion=\"%BUILD_NUMBER%\"' | sc version.xml"
powershell -Command "(gc version.xml)"
build.bat

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
        bat "rename sys.vcxproj.user.keybase sys.vcxproj.user"
    }
    
    withEnv(["SIGNTOOL=signtool",
             "ADDITIONALCERT=/a",
             "CERTISSUER=\"DigiCert\"",
             "KEYBASE_WINBUILD=${env.BUILD_NUMBER}"]) {

        stage('Build Drivers') {
            dir("src/github.com/keybase/dokany/dokan_wix") {
                bat '"%ProgramFiles(x86)%\\Microsoft Visual Studio 14.0\\vc\\bin\\vcvars32.bat" && build.bat'
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