# Как начать работу с GalenIT SDK

Библиотека GalenIT SDK для платформы Android 8.1 и выше высылается по запросу в виде 
набора подключаемых файлов и состоит из:

- libstage1.so - ядро SDK
- com.galenit.galenit-base - интерфейсы 
- com.galenit.adaptor-android-native - адаптор для solib
- com.galenit.galenit-api - модуль для взаимодействия с API сервера, реализованным в соответствии 
  со спецификацией OpenAPI
- com.galenit.galenit-tests - модуль для работы с функциональными тестами
- com.galenit.component-binah - представление для работы с измерением показателей здоровья
  посредством камеры
- com.galenit.component-common - вспомогательные представления

Чтобы создать приложение с измерениями показателей здоровья человека:

## Шаг 1. Получите необходимые файлы и ключ для работы с GalenIT SDK
Отправьте письмо на почту [info@it-galen.com](mailto:info@it-galen.com) с запросом о 
предоставлении GalenIT SDK для вашей организации. Дальнейшие инструкции придут на почту.

Полученный ключ можно использовать для работы в нескольких приложениях.

## Шаг 2. Установите библиотеку GalenIT SDK
1. Создайте новый проект или откройте существующий, например, в Android Studio.
2. Откройте файл build.gradle проекта. В секции repositories добавьте репозиторий Maven Central:
```sh
repositories {
    google()
    mavenCentral()
}
```
3. Откройте файл build.gradle приложения (модуля) и укажите минимальную версию Android.SDK
```gradlew
android {
  defaultConfig {
    // ...
    minSdk 27
  }
}
```
3. В файле build.gradle приложения (модуля) укажите адрес репозитория и добавьте требуемые зависимости:
```gradlew
dependencies {
    //Подключаем предоставленные библиотеки
    implementation fileTree(include: ['*.aar'], dir: 'libs')
    ...
    implementation "org.jetbrains.kotlinx:atomicfu:0.16.3"
    implementation "org.jetbrains.kotlinx:kotlinx-datetime:0.3.1"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.5.2-native-mt"
    implementation "org.jetbrains.kotlinx:kotlinx-serialization-core:1.3.0"
    implementation "org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.0"
    implementation "com.google.mlkit:face-detection:16.1.2"
    implementation "com.babylon.certificatetransparency:certificatetransparency-android:0.2.0"
}
```
4. Разместите библиотеку libstage1.so в приложении (модуле) в директории app/src/main/jniLibs/arm64-v8a/
5. Синхронизируйте проект, чтобы применить изменения. Например, в Android Studio можно нажать Sync Now или выбрать в меню File → Synchronize. Дождитесь окончания синхронизации.

Если синхронизация завершилась успешно, при компиляции библиотека будет добавлена в проект автоматически. При ошибке компиляции, убедитесь что вы правильно указали репозиторий и зависимость и синхронизируйте проект снова.

## Шаг 3. Настройте библиотеку для измерения по лицу
1. Добавьте разрешения приложению в AndroidManifest.xml:
```xml
<manifest>
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.BLUETOOTH" />
  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.CAMERA" />
  <application>...</application>
</manifest>

```
2. Добавьте представление на Activity:
```xml
<com.galenit.component.binah.BinahView
        android:id="@+id/cameraView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintDimensionRatio="W, 4:3"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@+id/buttonStart"
        app:mode="face" />
```
2. Импортируйте библиотеку в Activity:
```kotlin
import com.galenit.device.api.lib.GalenIT
```

3. После проверки требуемых разрешений инициализируйте работу библиотеки с лицензионным ключом
```kotlin
CoroutineScope(Dispatchers.Main).launch {
  try {
    val apiKey = BuildConfig.GALENIT_TOKEN
    if (!GalenIT.started)
      GalenIT.start(apiKey)
      ...
  }
  catch (e: Exception) {
      ...
  }
}
```
3. Передайте события onFace и onFaceHide и добавьте методы запуска и остановки измерений:
```kotlin
binding.cameraView.onFace = { bitmap, _ ->
  // ...
}
binding.cameraView.onFaceHide = { ->
  // ...
  CoroutineScope(Dispatchers.Main).launch {
    binding.cameraView.stopMeasurement()
    binding.cameraView.startWithoutMeasurements()
  }
}

binding.cameraView.startMeasurements() // запустить представление с измерением
// binding.cameraView.startWithoutMeasurements() // запустить представление без измерения

```
4. Добавьте подписчики на получение параметров измерения
```kotlin
binding.cameraView.startWithoutMeasurements()
CoroutineScope(Dispatchers.Main).launch {
  try {
    val parameter = binding.cameraView.readLongParameter(Parameter.LongBodyOxygen::class)
    parameter.setPartListener(object : WavePartListener {
      override fun listen(param: Parameter<Any>) {
        // param.value
      }
    })
  }
  catch (e: Exception) {
    // ...
  }
}
CoroutineScope(Dispatchers.Main).launch {
  try {
    val parameter = binding.cameraView.readParameter(Parameter.Pressure::class)
    // parameter.value.systolic, parameter.value.diastolic
  }
  catch (e: java.lang.Exception) {
    // ...
  }
}
```
для измерения по лицу доступны следующие параметры:
- Parameter.LongBodyOxygen::class
- Parameter.LongPulseRate::class
- Parameter.LongBreathingRate::class
- Parameter.SDNNHeartRateVariability::class
- Parameter.StressLevel::class
- Parameter.Pressure::class

## Шаг 4. Соберите и запустите приложение
При работе в Android Studio приложение можно сразу запустить, сборка будет выполнена автоматически. Следуйте инструкциям ниже, чтобы запустить приложение на:
- [Android-устройстве](http://developer.android.com/training/basics/firstapp/running-app.html#RealDevice), подключенном через USB к компьютеру разработчика (в режиме отладки).

## Пример приложения
