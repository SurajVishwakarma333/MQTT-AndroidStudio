# MQTT Android Studio Example

**Step 1: Add dependencies into setting.gradle.**

Add Paho Android Service as a dependency to your app add the following parts to your gradle file

      
         dependencyResolutionManagement {
            repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
            repositories {
            .......
            maven {
                  url "https://repo.eclipse.org/content/repositories/paho-releases/"
                  }
            }
          }
          
          
          
**Step 2: Add dependencies into build.gradle app level**          

            dependencies {
                implementation('org.eclipse.paho:org.eclipse.paho.android.service:1.0.2') {
                    exclude module: 'support-v4'
                }
            }    
      
**Step 3: Add permission on AndroidManifest.xml** 

The Paho Android Service needs the following permissions to work:

            <uses-permission android:name="android.permission.WAKE_LOCK" />
            <uses-permission android:name="android.permission.INTERNET" />
            <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
            <uses-permission android:name="android.permission.READ_PHONE_STATE" />
            
To be able to create a binding to the Paho Android Service, the service needs to be declared in the AndroidManifest.xml. Add the following within the <application> tag:            
            
            <application
                  ........
            
                  <service android:name="org.eclipse.paho.android.service.MqttService" >
                  </service>

            </application>
            
            
            
            
            
