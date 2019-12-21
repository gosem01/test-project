# test-project

# Setup and General Obaservations
The documentation provided was not enough to do a proper install of everything required from a clean machine. There was considerable time and effort in confirming proper setup steps for the tools, SDK, emulation, and test execution. Below are the installations steps that I underwent in order to set up a working environment:

1. Firstly, I encountered a blocking issue with the iOS test execution environment and that was that I was unable to sign even temporarily as an apple developer so I could not execute the framework on iOS simulator nor my iPhone device. I reached out to apple for the developer account but they have not gotten back to me. I reached out to the team and Drew provided me a tip as a work around but that did not work either. Hence I proceeded with Android. The same principles apply to iOS it is just a matter of having a working apple developer account and properly setting up a WebDriverAgent runner.
    
2. Secondly, when attempting to execute the test existing test suite I ran into missing AWS credentials which blocked proper initialization of the framework driver. I reached out and was promptly provided the credentials.

# Android Environment Setup
### Install Java 8
* brew tap adoptopenjdk/openjdk
* brew cask install adoptopenjdk8

### Install maven
* brew install maven

### Install Android SDK
* brew install android-sdk


### Set PATH variables in bash profile
Set PATH variables in my .bash_profile file:  
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)  
export ANDROID_HOME=/usr/local/share/android-sdk   
export ANDROID_SDK_ROOT=/usr/local/share/android-sdk   
export PATH=$ANDROID_HOME/tools/emulator:$PATH    
export PATH=$ANDROID_HOME/tools:$PATH   
export PATH=$ANDROID_HOME/platform-tools:$PATH  
export PATH=$ANDROID_HOME/build-tools/23.0.1:$PATH   



Add any necessary android packages:
Execute commands in terminal (more about packages: https://gist.github.com/mrk-han/66ac1a724456cadf1c93f4218c6060ae )

Create the following file file   
touch ~/.android/repositories.cfg

Paste below contents insite the cfg file that was just created and save  
sdkmanager --update   
sdkmanager "adb"   
sdkmanager "emulator"   
sdkmanager "platform-tools"   
sdkmanager "system-images;android-24;default;x86_64"   


### Create an android emulator:
avdmanager create avd -n emulator_name -k "system-images;android-24;default;x86_64" -g "default"

### Run emulator
$ANDROID_HOME/tools/emulator -avd emulator_name

### Download Appium Desktop & Install 
Open Appium Desktop and click <Edit configuration> button and set ANDROID_HOME and JAVA_HOME  
ANDROID_HOME = /usr/local/share/android-sdk  
For JAVA_HOME execute command in terminal and copy path  
echo $(/usr/libexec/java_home -v 1.8)

Before running tests make sure that you set AWS credentials  
Open terminal (use the same terminal that you want user for running test because env vars with credentials must be executed in the same session as with the test)  

export AWS_ACCESS_KEY_ID="Provided-AWS-KEY"  
export AWS_SECRET_ACCESS_KEY="Provided-AWS-Secret-key"  

### Command to Run tests:
mvn compile test -Ddebug=true -DsuiteFile=testng-login.xml -DdriverConfig=local_android_phone.json

### Locator Strategy
The locator strategies I was trying to use were from the perspective of performance and effeciency. The most performant/efficient is selecting an element by it's accessbility Id (iOS) or contentDescription (Android) because you knock out to birds with one stone there in grabbing elements for accessibility tests as well as normal tests. The second best is by element id which is synonymous with accessiblity id for iOS but for android it is different than it's native accessibility selector. Typically those are added by app developers. Lastly there is XPath, which is the lowest performing and most brittle of all however is widely used. I try to only use XPath if i dont see other more stable ways of element locating.

# NOTES on base test:
Had to comment out this line of code from the Menu Chunk since it was blocking the debug test when logging the user in:

protected void waitUntilVisible() {  
	//syncHelper.waitForElementToAppear(getLocator(this, "getShowTutorialButton"));   
}


In AccountView it was necessary to update the android XPATH so that the account page assertion was looking for the correct locators and could pass the assertion:   
@MobileElementLocator(android = "//android.view.View[@content-desc=\"test\"]", iOS = "//XCUIElementTypeOther[3]/XCUIElementTypeStaticText[1]")   
protected Text getNameText() {   
	return new Text(getLocator(this, "getNameText"));   
}   

@MobileElementLocator(android = "//android.view.View[@content-desc=\"Account\"]", iOS = "Account")
protected Text getAccountHeaderText() {
	return new Text(getLocator(this, "getAccountHeaderText"));
}

# Screenshots of grabbing selectors with Appium:
![Alt text](/example1.png?raw=true "Optional Title")
![Alt text](/example2.png?raw=true "Optional Title")
![Alt text](/example3.png?raw=true "Optional Title")
