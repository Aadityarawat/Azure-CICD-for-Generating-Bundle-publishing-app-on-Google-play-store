# Azure-CICD-for-Generating-Bundle-publishing-app-on-Google-play-store
By reading this repo, you will understand how to create a CI/CD pipeline for generating bundles and a release pipeline to publish your app on the Google Play Store.

// Line 32 to 124 is the script for pipeline and if you are copy pasting make sure replace bullets to "-" (hyphen).
CI/CD using Azure DevOps 

What is CI/CD? 

Continuous Integration/Continuous Deployment or Delivery is an automated process that enables development teams to continuously integrate code changes, test those changes, and deploy them to production environments. It helps to streamline software development and delivery, improve code quality, and reduce manual efforts. 

 

Steps 
1. Need to create a new organization or select an existing one, it totally depends on you. 

2. Now you need to import a repo of your project, and it also depends on you, you can import it directly on azure or GitHub. 

3. create a new pipeline  

Select YAML file option based on where your repo exists.  

Select repo whichever you want. 

Select Android, if it's an android project. 

Now in review you need to write some script in it. 

 

PART 1 (Generate Signed Bundle) 

trigger: 

- master 

 

pool: 

vmImage: ubuntu-latest 

 

steps: 

- task: DownloadSecureFile@1 

inputs: 

secureFile: 'keystore1' 

 

(keystore1 is jsk file I have stored in the pipeline -> lib -> secure file) 

 

- task: Gradle@2 

inputs: 

workingDirectory: '$(Build.Repository.LocalPath)' 

gradleWrapperFile: '$(Build.Repository.LocalPath)/gradlew' 

gradleOptions: '-Xmx3072m' 

javaHomeOption: 'JDKVersion' 

jdkVersionOption: '17'   (as I have set java version in gradle 17)   

publishJUnitResults: false 

testResultsFiles: '**/TEST-*.xml' 

tasks: 'clean build test assembleRelease assembleDebug assembleAndroidTest bundleRelease' 

sonarQubeRunAnalysis: false 

 

- task: CmdLine@2 

displayName: 'Signing and aligning AAB file(s) app/build/outputs/bundle/release/app-release.aab' 

inputs: 

script: | 

jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \ 

-keystore $(Agent.TempDirectory)/keystore1 \  

-storepass '12345678' \ 

-keypass '12345678' \ 

$(system.defaultworkingdirectory)/app/build/outputs/bundle/release/app-release.aab \ 

'key0' 

 

- task: CopyFiles@2 

inputs: 

SourceFolder: '$(system.defaultworkingdirectory)/app/build/outputs/bundle/release/' 

Contents: '**' 

TargetFolder: '$(build.artifactstagingdirectory)' 

 

- task: PublishBuildArtifacts@1 

inputs: 

PathtoPublish: '$(Build.ArtifactStagingDirectory)' 

ArtifactName: 'drop' 

publishLocation: 'Container' 

 

 

So, we successfully completed the first half. Run the pipeline and you can see the result inside the publish option that we have successfully generated the signed release bundle. 
 

PART 2 (Release pipeline for google play store) 

Steps 
1. Select pipeline -> release 

2. create new release pipeline. 

3. Add the artifacts that you have create for generating signed bundle 

4. After adding the artifacts click on the thunder symbol for continuous deployment. Enable the option. 

5. Now click on add a stage & select Azure app service deployment. 

6. Then click on 1 job 1 task text, then click to deploy azure app service and click on remove button as we are not deploying app on azure. 

7. Now in Run on agent, click add task -> search google play store release -> add. 

8. After adding that we need to add a new Service connection or existing one, if have. 

https://www.youtube.com/watch?v=vlL6CBGadI4  (This tutorial will help you with how to setup service connections and upcoming errors). 

9. Now you need to go to Google cloud developer account than add new project or existing, than click on service accounts -> add service account -> write details and copy the email address -> next step, select a role “basic -> owner” or “service ac -> service ac user”and we are done with the process. 

10. Now paste that email address in the azure Service Account E-mail  

11. Now in service ac you have to add a key in JSON format. It will download in your local storage. 

12. Open that file and copy in this format  
-----BEGIN PRIVATE KEY-----\nYourPrivateKeyHere\n-----END PRIVATE KEY-----\n 

13. Paste it in azure Private Key and fill other details, then click to save button & we are done with the service connection. 

14. Add: Application id,  
Action : Upload single bundle 
Bundle path : **/*release.aab  (release.aabis the file name in my drop folder) 
Track: Internal 
 
In Advance Option select Release as Draft if the never publish for production. 

15. Click on save button,  

16. Now we need to go to the Google play console to add service account email address. 
Go to User & permissions -> invite new user -> paste email -> account permission -> select admin (all permission) -> click on invite user. 

17. Now go to release pipeline -> create release (if everything works perfectly, we will get an error in the console, we just need to copy the url that is suggested in the error log) 

18. Go to the url and enable Google play Android Developer Api. 

19. Now if we create release, we can be able to see result internal test as draft. 
 
NOTE: We need to publish our app in internal testing or production at least once manually before CICD. 

 

 
 
