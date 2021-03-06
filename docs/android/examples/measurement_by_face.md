# Пример измерения по лицу

## Шаг 1. Настройте проект
Проведите необходимые настройки согласно [инструкции](../getstarted.md)

## Шаг 2. Настройте библиотеку
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
Доступные параметры компонента:
- **mode** - _face_ - измерение по лицу, _finger_ - измерение по пальцу;
- **sessionTime** - время сессии измерения;
- **showFacePoint** - показывать точки контура лица;
- **showLinesBetweenFacePoints** - показывать соединительные линии между точками контура лица;
- **pointsPulsationFrequency** - частота изменения цвета точек контура лица;
- **facePointsColorEnd** - цвет точек контура лица конечный (при пульсации);
- **linesPulsationFrequency** - частота изменения цвета соединительных линий между точками контура лица;
- **faceLinesColorEnd** - цвет соединительных линий между точками контура лица конечный (при пульсации);
- **faceLinesColor** - цвет линий контура лица в начале измерения;
- **facePointsColor** - цвет точек контура лица в начале измерения.

> Обратите внимание на параметр **app:mode** - он отвечает за то, как будет отображен компонент

3. Импортируйте необходимые пакеты
```kotlin
import com.galenit.device.api.lib.GalenIT
import com.galenit.component.binah.BinahView
import com.galenit.device.api.base.WavePartListener
import com.galenit.device.api.parameter.Parameter
```

4. Запросите разрешения в методе _onCreate_ MainActivity
```kotlin
ActivityCompat.requestPermissions(
    this,
    arrayOf(
        Manifest.permission.ACCESS_FINE_LOCATION,
        Manifest.permission.ACCESS_COARSE_LOCATION,
        Manifest.permission.BLUETOOTH,
        Manifest.permission.BLUETOOTH_ADMIN,
        Manifest.permission.INTERNET,
        Manifest.permission.ACCESS_NETWORK_STATE,
        Manifest.permission.CAMERA
    ),
    1
)
```   
5. После проверки требуемых разрешений инициализируйте работу библиотеки с лицензионным ключом
```kotlin
CoroutineScope(Dispatchers.Main).launch {
  try {
    val apiKey = BuildConfig.GALENIT_TOKEN
    if (!GalenIT.started)
      GalenIT.start(apiKey)
      // ...
  }
  catch (e: Exception) {
      // ...
  }
}
```
6. Передайте события onFace и onFaceHide и добавьте методы запуска и остановки измерений:
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
7. Добавьте подписчики на получение параметров измерения
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
Для измерения доступны следующие параметры:
- Parameter.LongBodyOxygen::class
- Parameter.LongPulseRate::class
- Parameter.LongBreathingRate::class
- Parameter.SDNNHeartRateVariability::class
- Parameter.StressLevel::class
- Parameter.Pressure::class