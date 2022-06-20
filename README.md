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
            
To be able to create a binding to the Paho Android Service, the service needs to be declared in the AndroidManifest.xml.

Add the following within the <application> tag:            
            
            <application
                  ........
            
                  <service android:name="org.eclipse.paho.android.service.MqttService" >
                  </service>

            </application>
            
            
**Step 4: Design the layout activity_main.xml**             
            
            <?xml version="1.0" encoding="utf-8"?>
            <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:app="http://schemas.android.com/apk/res-auto"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                tools:context=".MainActivity">

                <Button
                    android:id="@+id/button"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="236dp"
                    android:onClick="published"
                    android:text="PUBLISH"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintHorizontal_bias="0.498"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />

                <TextView
                    android:id="@+id/subText"
                    android:layout_width="141dp"
                    android:layout_height="68dp"
                    android:layout_marginTop="360dp"
                    android:text="TextView"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />

                <Button
                    android:id="@+id/connBtn"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="524dp"
                    android:onClick="conn"
                    android:text="connect"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintHorizontal_bias="0.201"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />

                <Button
                    android:id="@+id/disconnBtn"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="524dp"
                    android:onClick="disconn"
                    android:text="disconnect"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintHorizontal_bias="0.78"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />

            </androidx.constraintlayout.widget.ConstraintLayout>
            
            
**Step 5: Write the code in MainActivity.java**           





            import androidx.appcompat.app.AppCompatActivity;

            import android.os.Bundle;
            import android.view.View;
            import android.widget.TextView;
            import android.widget.Toast;

            import org.eclipse.paho.android.service.MqttAndroidClient;
            import org.eclipse.paho.client.mqttv3.IMqttActionListener;
            import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
            import org.eclipse.paho.client.mqttv3.IMqttToken;
            import org.eclipse.paho.client.mqttv3.MqttCallback;
            import org.eclipse.paho.client.mqttv3.MqttClient;
            import org.eclipse.paho.client.mqttv3.MqttException;
            import org.eclipse.paho.client.mqttv3.MqttMessage;

            public class MainActivity extends AppCompatActivity {

                MqttAndroidClient client;
                TextView subText;

                @Override
                protected void onCreate(Bundle savedInstanceState) {
                    super.onCreate(savedInstanceState);
                    setContentView(R.layout.activity_main);

                    subText = (TextView)findViewById(R.id.subText);

                    String clientId = MqttClient.generateClientId();
                    client = new MqttAndroidClient(this.getApplicationContext(), "tcp://broker.mqttdashboard.com:1883",clientId);
                    //client = new MqttAndroidClient(this.getApplicationContext(), "tcp://192.168.43.41:1883",clientId);

                    try {
                        IMqttToken token = client.connect();
                        token.setActionCallback(new IMqttActionListener() {
                            @Override
                            public void onSuccess(IMqttToken asyncActionToken) {
                                Toast.makeText(MainActivity.this,"connected!!",Toast.LENGTH_LONG).show();
                                setSubscription();

                            }

                            @Override
                            public void onFailure(IMqttToken asyncActionToken, Throwable exception) {
                                Toast.makeText(MainActivity.this,"connection failed!!",Toast.LENGTH_LONG).show();
                            }
                        });
                    } catch (MqttException e) {
                        e.printStackTrace();
                    }

                    client.setCallback(new MqttCallback() {
                        @Override
                        public void connectionLost(Throwable cause) {

                        }

                        @Override
                        public void messageArrived(String topic, MqttMessage message) throws Exception {
                            subText.setText(new String(message.getPayload()));
                        }

                        @Override
                        public void deliveryComplete(IMqttDeliveryToken token) {

                        }
                    });

                }

                public void published(View v){

                    String topic = "topictesting";
                    String message = "publishing message from suraj";
                    try {
                        client.publish(topic, message.getBytes(),0,false);
                        Toast.makeText(this,"Published Message",Toast.LENGTH_SHORT).show();
                    } catch ( MqttException e) {
                        e.printStackTrace();
                    }
                }

                private void setSubscription(){

                    try{

                        client.subscribe("topictesting",0);


                    }catch (MqttException e){
                        e.printStackTrace();
                    }
                }

                public void conn(View v){

                    try {
                        IMqttToken token = client.connect();
                        token.setActionCallback(new IMqttActionListener() {
                            @Override
                            public void onSuccess(IMqttToken asyncActionToken) {
                                Toast.makeText(MainActivity.this,"connected!!",Toast.LENGTH_LONG).show();
                                setSubscription();

                            }

                            @Override
                            public void onFailure(IMqttToken asyncActionToken, Throwable exception) {
                                Toast.makeText(MainActivity.this,"connection failed!!",Toast.LENGTH_LONG).show();
                            }
                        });
                    } catch (MqttException e) {
                        e.printStackTrace();
                    }

                }

                public void disconn(View v){

                    try {
                        IMqttToken token = client.disconnect();
                        token.setActionCallback(new IMqttActionListener() {
                            @Override
                            public void onSuccess(IMqttToken asyncActionToken) {
                                Toast.makeText(MainActivity.this,"Disconnected!!",Toast.LENGTH_LONG).show();


                            }

                            @Override
                            public void onFailure(IMqttToken asyncActionToken, Throwable exception) {
                                Toast.makeText(MainActivity.this,"Could not diconnect!!",Toast.LENGTH_LONG).show();
                            }
                        });
                    } catch (MqttException e) {
                        e.printStackTrace();
                    }
                }

            }
            
            
**Step 6: You can test your mobile app**            
            
- `RUN the app`
- `In browser, open the` [MQTT websocket client](http://www.hivemq.com/demos/websocket-client/)

![1](https://user-images.githubusercontent.com/101108540/174437564-e3b8b6c2-55aa-4722-82c8-d53e479d822f.jpg)


- `Click button connect.`

### wanted to send data from mqtt to android App?.

- `In publish give topic which is "testingtopic" and provide the messessage which you want to display in android app here my message is "hello siwi from suraj".`

![1](https://user-images.githubusercontent.com/101108540/174591006-6a7bd6ce-f804-4285-83e7-d457973e6b6a.jpg)



- Example of Topic in my program:

            topic = topictesting

            QoS = 0

- `Click subscribe.`
- `When click button Publish, you will see the “publishing message from suraj” in the Messages bar. The message is already embedded in the coding.`




- `When click Publish from MQTT websocket client with correct topic (“event”), you will see the “YEAHH!!!” on you mobile app` 


