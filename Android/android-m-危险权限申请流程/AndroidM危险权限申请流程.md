## Android M 危险权限申请流程

危险权限基本都涉及到用户的隐私，诸如拍照、读取短信、写存储、录音等，Android 系统将这些危险权限分为 9 组，**获取分组中某个权限的同时也就获取了同组中的其他权限**，危险权限不仅需要在 AndroidManifest.xml 中注册，还需要**动态地申请权限**。

| Permission Group | Permissions
| ---------------- | -------------
| CALENDAR         | `READ_CALENDAR`<br/>`WRITE_CALENDAR`
| CAMERA           | `CAMERA`
| CONTACTS         | `READ_CONTACTS`<br/>`WRITE_CONTACTS`<br/>`GET_ACCOUNTS`
| LOCATION         | `ACCESS_FINE_LOCATION`<br/>`ACCESS_COARSE_LOCATIO`
| MICROPHONE       | `RECORD_AUDIO`
| PHONE            | `READ_PHONE_STATE`<br/>`CALL_PHONE`<br/>`READ_CALL_LOG`<br/>`WRITE_CALL_LOG`<br/>`ADD_VOICEMAIL`<br/>`USE_SIP`<br/>`PROCESS_OUTGOING_CALLS`
| SENSORS          | `BODY_SENSORS`
| SMS              | `SEND_SMS`<br/>`RECEIVE_SMS`<br/>`READ_SMS`<br/>`RECEIVE_WAP_PUSH`<br/>`RECEIVE_MMS`
| STORAGE          | `READ_EXTERNAL_STORAGE`<br/>`WRITE_EXTERNAL_STORAGE`

需要动态申请权限的三个条件，缺一不可
1. 危险权限
2. Android 版本 >= 6.0
3. targetSdkVersion >= 23

## SDK 提供的 API

为方便开发者实现权限管理，Google 提供了 4 个 API：

| API                                    | 作用
|----------------------------------------|---
| checkSelfPermission()                  | 判断权限是否具有某项权限
| requestPermissions()                   | 申请权限
| onRequestPermissionsResult()           | 申请权限回调方法
| shouldShowRequestPermissionRationale() | 是否要提示用户申请该权限的缘由

### 判断权限是否已申请
```java
// ActivityCompat.java
public static int checkSelfPermission(
    @NonNull Context context, 
    @NonNull String permission)
```

### 请求权限
```java
// ActivityCompat.java
public static void requestPermissions(
    final @NonNull Activity activity, 
    final @NonNull String[] permissions, // 需要申请的权限数组
    final @IntRange(from = 0) int requestCode) // from 0x0000 to 0xffff
```

### 是否需要提示用户申请权限的理由
```java
// ActivityCompat.java
public static boolean shouldShowRequestPermissionRationale(
    @NonNull Activity activity, 
    @NonNull String permission)
```

| 序号 | 是否授予了权限 | ssrpr()返回 | 是否勾选不再询问 | 说明
|-------|----|-------|---|---
| 1     | 否 | **false** | - | 首次请求权限，不提示用户理由
| 2     | 否 | true  | 否 | 需要提示用户理由
| 3     | 否 | true  | 否 | 需要提示用户理由
| …     | …	 | …     | … | 需要提示用户理由
| i     | 否 | true  | 是 | 需要提示用户理由
| i + 1 | -  | **false** | - | 用户勾选了不再提示，不再需要提示理由

可以看出只有两种情况`shouldShowRequestPermissionRationale`返回了 false

### 权限申请结果回调
```java
// FragmentActivity.java
@Override
public void onRequestPermissionsResult(
    int requestCode, 
    @NonNull String[] permissions, // 权限数组 
    @NonNull int[] grantResults) // 结果数组
```

## 权限申请流程
![权限申请流程](http://upload-images.jianshu.io/upload_images/2048485-f7ec08e39f7fc79e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 权限动态申请库的封装，使用 KOTLIN 语言

### 定义接口代理
```kotlin
interface RequestPermissionsDelegate {

    fun onRequestPermissionsResult(
        permissions: Array<out String>, 
        grantResults: IntArray)
}
```

### BaseActivity
```kotlin
abstract class MPsActivity : AppCompatActivity() {

    private val mRequestPermissionsDelegateMap : MutableMap<Int, RequestPermissionsDelegate> = HashMap()

    fun addRequestPermissionsDelegate(requestCode: Int, requestPermissionsDelegate: RequestPermissionsDelegate) {
        mRequestPermissionsDelegateMap.put(requestCode, requestPermissionsDelegate)
    }

    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        val requestPermissionsDelegate : RequestPermissionsDelegate ?= mRequestPermissionsDelegateMap[requestCode]
        requestPermissionsDelegate?.onRequestPermissionsResult(permissions, grantResults)
        mRequestPermissionsDelegateMap.remove(requestCode)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when (requestCode) {
            REQUEST_CODE_SETTING -> {
                onSettingReturn()
            }
        }
    }

    /**
     * 从设置页返回
     */
    abstract fun onSettingReturn()
}
```
使用一个 MAP 管理所有的权限回调结果

### 定义九组权限的 RequestCode
```kotlin
val PERMISSION_REQUEST_CODE_CALENDAR = 0x0001
val PERMISSION_REQUEST_CODE_CAMERA = 0x0002
val PERMISSION_REQUEST_CODE_CONTACTS = 0x0003
val PERMISSION_REQUEST_CODE_LOCATION = 0x0004
val PERMISSION_REQUEST_CODE_MICROPHONE = 0x0005
val PERMISSION_REQUEST_CODE_PHONE = 0x0006
val PERMISSION_REQUEST_CODE_SENSORS = 0x0007
val PERMISSION_REQUEST_CODE_SMS = 0x0008
val PERMISSION_REQUEST_CODE_STORAGE = 0x0009
```

### Abstract Helper
```kotlin
abstract class Helper {

    interface Listener {
        fun onPermissionDenied()
        fun onPermissionGranted()
    }

    private fun checkSelfPermission(context: Context): Boolean {
        val result = ActivityCompat.checkSelfPermission(context, permission())
        return PackageManager.PERMISSION_GRANTED == result
    }

    private fun requestPermission(activity: MPsActivity, packageName: String, listener: Listener) {
        activity.addRequestPermissionsDelegate(requestCode(), object : RequestPermissionsDelegate {
            override fun onRequestPermissionsResult(permissions: Array<out String>, grantResults: IntArray) {
                if (grantResults.isEmpty() || PackageManager.PERMISSION_GRANTED != grantResults[0]) {
                    if (!shouldShowRequestPermissionRationale(activity)) {
                        // 说明用户勾选了不再提示
                        showMustGrantDialog(activity, packageName, listener)
                    } else {
                        // 首次拒绝或未勾选不再提示的拒绝
                        listener.onPermissionDenied()
                    }
                } else {
                    listener.onPermissionGranted()
                }
            }
        })

        ActivityCompat.requestPermissions(activity, arrayOf(permission()), requestCode())
    }

    private fun shouldShowRequestPermissionRationale(activity: Activity): Boolean {
        return ActivityCompat.shouldShowRequestPermissionRationale(activity, permission())
    }

    /**
     * 跳转系统设置
     */
    private fun goSettings(activity: MPsActivity, packageName: String) {
        val localIntent = Intent()
        // localIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        localIntent.action = "android.settings.APPLICATION_DETAILS_SETTINGS"
        localIntent.data = Uri.fromParts("package", packageName, null)
        activity.startActivityForResult(localIntent, REQUEST_CODE_SETTING)
    }

    private fun showMustGrantDialog(activity: MPsActivity, packageName: String, listener: Listener) {

        val builder: AlertDialog.Builder = AlertDialog.Builder(activity)
        builder.setTitle("APP 怒了")
        builder.setMessage(permissionRequiredHint())
        builder.setNegativeButton("残忍拒绝") { dialog, which ->
            listener.onPermissionDenied()
            dialog.dismiss()
        }
        builder.setPositiveButton("欣然同意") { dialog, which ->
            goSettings(activity, packageName)
            dialog.dismiss()
        }
        builder.create().show()
    }

    fun requestPermissionIfNeed(activity: MPsActivity, packageName: String, listener: Listener) {
        if (!checkSelfPermission(activity)) {
            requestPermission(activity, packageName, listener)
        } else {
            listener.onPermissionGranted()
        }
    }

    /**
     * @see Manifest.permission.READ_CALENDAR
     * @see Manifest.permission.WRITE_CALENDAR
     *
     * @see Manifest.permission.CAMERA
     *
     * @see Manifest.permission.READ_CONTACTS
     * @see Manifest.permission.WRITE_CONTACTS
     * @see Manifest.permission.GET_ACCOUNTS
     *
     * @see Manifest.permission.ACCESS_FINE_LOCATION
     * @see Manifest.permission.ACCESS_COARSE_LOCATION
     *
     * @see Manifest.permission.RECORD_AUDIO
     *
     * @see Manifest.permission.READ_PHONE_STATE
     * @see Manifest.permission.CALL_PHONE
     * @see Manifest.permission.READ_CALL_LOG
     * @see Manifest.permission.WRITE_CALL_LOG
     * @see Manifest.permission.ADD_VOICEMAIL
     * @see Manifest.permission.USE_SIP
     * @see Manifest.permission.PROCESS_OUTGOING_CALLS
     *
     * @see Manifest.permission.BODY_SENSORS
     *
     * @see Manifest.permission.SEND_SMS
     * @see Manifest.permission.RECEIVE_SMS
     * @see Manifest.permission.READ_SMS
     * @see Manifest.permission.RECEIVE_WAP_PUSH
     * @see Manifest.permission.RECEIVE_MMS
     *
     * @see Manifest.permission.WRITE_EXTERNAL_STORAGE
     * @see Manifest.permission.READ_EXTERNAL_STORAGE
     */
    protected abstract fun permission(): String

    /**
     * @see PERMISSION_REQUEST_CODE_CALENDAR
     * @see PERMISSION_REQUEST_CODE_CAMERA
     * @see PERMISSION_REQUEST_CODE_CONTACTS
     * @see PERMISSION_REQUEST_CODE_LOCATION
     * @see PERMISSION_REQUEST_CODE_MICROPHONE
     * @see PERMISSION_REQUEST_CODE_PHONE
     * @see PERMISSION_REQUEST_CODE_SENSORS
     * @see PERMISSION_REQUEST_CODE_SMS
     * @see PERMISSION_REQUEST_CODE_STORAGE
     */
    protected abstract fun requestCode(): Int

    @StringRes
    protected abstract fun permissionRequiredHint(): Int
}
```

![helper](http://upload-images.jianshu.io/upload_images/2048485-a08b0c4d020b6e0b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 如何使用
```kotlin
class RequestPermissionActivity : MPsActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        checkPermissions()
    }

    override fun onSettingReturn() {
        checkPermissions()
    }

    private fun checkPermissions() {
        StoragePermissionHelper.requestPermissionIfNeed(this, packageName, object : Helper.Listener {
            override fun onPermissionDenied() {
                toast("拒绝存储")
            }

            override fun onPermissionGranted() {
                toast("同意存储")
            }
        })
    }
}
```


## 源码
[MPermissions](https://github.com/hayukleung/MPermissions)