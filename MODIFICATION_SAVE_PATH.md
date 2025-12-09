# Flutter UVCCamera - takePicture savePath 参数修改记录

## 修改目的
为 `takePicture` 方法添加自定义保存路径功能，允许用户指定照片保存的具体路径。

## 修改摘要
在原有 `takePicture()` 方法基础上，增加可选的 `savePath` 参数，支持自定义照片保存路径。

## 涉及文件及修改详情

### 1. `android/src/main/kotlin/com/chenyeju/UVCCameraView.kt`
**修改位置**: 第604行
**修改内容**:
```kotlin
// 修改前
fun takePicture(callback: UVCStringCallback) {
    if (!isCameraOpened()) {
        callback.onError("Camera not open")
        return
    }

    captureImage(object : ICaptureCallBack {
        // 回调实现...
    })
}

// 修改后
fun takePicture(callback: UVCStringCallback, savePath: String? = null) {
    if (!isCameraOpened()) {
        callback.onError("Camera not open")
        return
    }

    captureImage(object : ICaptureCallBack {
        // 回调实现...
    }, savePath)  // <-- 传入 savePath 参数
}
```

### 2. `android/src/main/kotlin/com/chenyeju/FlutterUVCCameraPlugin.kt`
**修改位置**: 第104-117行
**修改内容**:
```kotlin
// 修改前
"takePicture" -> {
    mUVCCameraViewFactory.takePicture(
        object : UVCStringCallback { /* 回调实现 */ }
    )
}

// 修改后
"takePicture" -> {
    val savePath = call.argument<String>("savePath")
    mUVCCameraViewFactory.takePicture(
        object : UVCStringCallback { /* 回调实现 */ },
        savePath  // <-- 新增 savePath 参数
    )
}
```

### 3. `android/src/main/kotlin/com/chenyeju/UVCCameraViewFactory.kt`
**修改位置**: 第32-34行
**修改内容**:
```kotlin
// 修改前
fun takePicture(callback: UVCStringCallback) {
    cameraView.takePicture(callback)
}

// 修改后
fun takePicture(callback: UVCStringCallback, savePath: String? = null) {
    cameraView.takePicture(callback, savePath)
}
```

### 4. `lib/src/uvc_camera_controller.dart`
**修改位置**: 第284-292行
**修改内容**:
```dart
// 修改前
Future<String?> takePicture() async {
  String? path = await _methodChannel?.invokeMethod('takePicture');
  debugPrint("path: $path");
  return path;
}

// 修改后
/// Take a picture
/// If [savePath] is provided, the picture will be saved to that location
Future<String?> takePicture({String? savePath}) async {
  String? path = await _methodChannel?.invokeMethod('takePicture', {
    'savePath': savePath,  // <-- 新增参数传递
  });
  debugPrint("path: $path");
  return path;
}
```

### 5. `example/lib/camera.dart`
**修改位置**: 第418-428行
**修改内容**:
```dart
// 修改前
takePicture(int i) async {
  String? path = await cameraController.takePicture();
  if (path != null) {
    // 处理结果...
  }
}

// 修改后
takePicture(int i) async {
  // 可选：指定自定义保存路径
  // final savePath = "/storage/emulated/0/DCIM/MyApp/photo_${DateTime.now().millisecondsSinceEpoch}.jpg";
  // String? path = await cameraController.takePicture(savePath: savePath);
  String? path = await cameraController.takePicture();
  if (path != null) {
    // 处理结果...
  }
}
```

## 使用方法示例

### 1. 默认路径拍照
```dart
String? path = await cameraController.takePicture();
```

### 2. 指定路径拍照
```dart
final savePath = "/storage/emulated/0/DCIM/MyApp/photo_${DateTime.now().millisecondsSinceEpoch}.jpg";
String? path = await cameraController.takePicture(savePath: savePath);
```

## Android 7 注意事项

1. **权限要求**：需要 `WRITE_EXTERNAL_STORAGE` 权限
2. **推荐路径格式**：`/storage/emulated/0/DCIM/YourAppName/filename.jpg`
3. **路径必须存在**：确保指定的目录已经创建
4. **应用私有目录**：考虑使用 `getExternalFilesDir()` 获取的路径

## 构建说明

修改完成后，使用以下命令清理和构建：
```bash
flutter clean
flutter pub get
flutter run
```

## 版本兼容性

本修改向下兼容，原有不带参数的调用方式仍然可用。