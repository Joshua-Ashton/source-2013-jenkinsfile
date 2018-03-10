def GitURL = "https://github.com/ValveSoftware/source-sdk-2013"
def GitCredentialsId = ''
def SrcPath = "sp/src"
def GameBinFolder = "sp/game/episodic/bin"
def VPCScript = "creategameprojects"
def SolutionName = "games"

stage ('Build Source SDK 2013') {
	parallel "Linux": {
		node ('linux') {
			stage('Git') {
				git url: GitURL, credentialsId: GitCredentialsId
			}

			stage('Generate VPC on Linux') {
				dir (SrcPath) {
					sh "./${VPCScript}"
				}
			}

			stage('Compile on Linux') {
				sh "if [ -d ${GameBinFolder} ]; then echo \"Dir already exists...\"; else mkdir -p ${GameBinFolder}; fi"

				dir ('sp/src') {
					sh "/valve/steam-runtime/shell.sh --arch=i386 make -f ${SolutionName}.mak"
				}
			}

			stage('Get Artifacts on Linux') {
				archiveArtifacts(artifacts: "${GameBinFolder}/*.so", onlyIfSuccessful: true)
			}
		}
	},
	"Windows": {
		node ('windows') {
			stage('Git') {
				git url: GitURL, credentialsId: GitCredentialsId
			}

			stage('Generate VPC on Windows') {
					dir (SrcPath) {
						bat "${VPCScript}.bat"
						bat "copy ${SolutionName}.sln+sln_fix.txt ${SolutionName}.sln"
				}
			}

			stage('Compile on Windows') {
				bat "if not exist ${GameBinFolder} (mkdir ${GameBinFolder})"
				
				dir ('sp/src') {
					bat """call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Enterprise\\Common7\\Tools\\VsDevCmd.bat"
					msbuild ${SolutionName}.sln /t:Build /p:Configuration=Release /m:4"""
				}
			}

			stage('Get Artifacts on Windows')
			{
				archiveArtifacts(artifacts: "${GameBinFolder}/*.dll", onlyIfSuccessful: true)
			}
		}
	}
}