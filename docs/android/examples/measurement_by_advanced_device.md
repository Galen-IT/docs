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
2. Запросите разрешения в методе _onCreate_ MainActivity
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
3. После проверки требуемых разрешений инициализируйте работу библиотеки с лицензионным ключом
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
3. Получите разовое измерение с помощью прибора
```kotlin
CoroutineScope(Dispatchers.Main).launch {
    val parameterMeasurement = Parameter.LongPulseRate::class
    
    val manager = GalenIT.newBleManager(it) // создать менеджер доступных BLE-устройств
    val device = manager.scanForParameter(parameterMeasurement) // сканировать эфир по устройствам измеряющих параметр пульса
        .first() // возьмем первый найденный
    
    try {
        if (device.module.requireBonding) { // Если девайс хочет сопряжения
            device.bond() // Сопрягаемся заранее
        }

        val connectedDevice = device.connect() // соединяемся с устройством
        val parameter = connectedDevice.readLongParameter(parameterMeasurement) // начинаем измерение продолжительного параметра
        parameter.setPartListener(object : WavePartListener {  // создаем слушатель
            override fun listen(param: Parameter<Any>) {
                // param - каждый элемент измеряемого параметра
            }
        })
        
        CoroutineScope(Dispatchers.Main).launch {
            delay(30000)
            parameter.stop() // Останавливаем измерение через 30 секунд
        } 

        val param = parameter.waitForParameter() // ждём готового продолжительного параметра
        // param - полученный продолжительный параметр
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```
Для измерения доступны следующие параметры:
- Parameter.PulseWaveform::class
- Parameter.LongPulseRate::class
- Parameter.LongBodyOxygen::class
- Parameter.Ecg::class
- Parameter.Pressure::class
