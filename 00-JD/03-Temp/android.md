# Android

## 1、插件单独编译配置项

* application的manifest

  ```xml
  <!-- TODO: Please copy declarations of all activities to the Manifest.xml in the main project. -->
  <activity
      android:name="com.jd.lib.JDCashReward.DemoMainActivity"
      android:launchMode="singleTask" >
      <intent-filter>
          <action android:name="android.intent.action.MAIN" />
          <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
  </activity>
  ```

  ```xml
  <!-- TODO: Please copy declarations of all services to the Manifest.xml in the main project. -->
  <service
      android:name="com.jd.lib.JDCashReward.DemoService"
      android:enabled="true"
      android:exported="false" >
  </service>
  ```

* application的build.gradle

  ```xml
  isUseAura=false
  ```



