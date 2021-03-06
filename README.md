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

![to connect](https://user-images.githubusercontent.com/101108540/174592633-cf5e3c2d-f396-4218-bdf8-ddcd7e5a9c86.jpg)

- `It will show connected.`

![connected](https://user-images.githubusercontent.com/101108540/174593956-c63b1a5f-ffc0-43f5-87c7-c01f9c5d5d1a.jpg)


# Q. Wanted to receive data from MQTT to android App ?.

- Example of Topic in my program:

            topic = topictesting

            QoS = 0

- `When click Publish from MQTT websocket client with correct topic (???topictesting???), you will see the ???hello siwi from suraj !!!!??? on you mobile app` 

- `Now click on publish to display the message in android app.`

![17](https://user-images.githubusercontent.com/101108540/174600612-66f0e62c-5080-4819-b213-312fd18cbd4a.jpg)


# Q. Wanted to send data from android App to MQTT ?.

- `Add Subscriptions by clicking on "Add New Topic Subscription"`

![subs](https://user-images.githubusercontent.com/101108540/174595374-6721027e-f18a-43b5-bc04-55425e2e70ef.jpg)


- `Provide topic name i.e "topictesting" and also set QoS=0. click on Subscribe.`

![sub](https://user-images.githubusercontent.com/101108540/174598301-19971578-5bcf-43ba-b48d-ae108ffc71b7.jpg)


- `When click button Publish, you will see the ???publishing message from suraj??? in the Messages bar. The message is already embedded in the coding.`

![3](https://user-images.githubusercontent.com/101108540/174601860-29ff869c-94eb-4013-b8f7-e0626809dc15.jpg)




### Done!!!????
