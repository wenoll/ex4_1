# 实验四：基于TensorFlow Lite实现的Android花卉识别应用

## 向应用中添加TensorFlow Lite

最终TensorFlow Lite模型被成功导入，并生成摘要信息

![Screenshot 2025-05-14 130534](https://github.com/user-attachments/assets/e0169068-bd1b-46c7-96d6-af53ea402c47)


## 添加TODO代码重新运行APP

![Screenshot 2025-05-14 130808](https://github.com/user-attachments/assets/cdef692f-5839-4b9f-9742-e78fc2aacf9b)


## 真机运行完成的花卉识别应用

![f9cb79b3292ae27d5ad7934c3b43e32b](https://github.com/user-attachments/assets/3bf2e5b8-436a-4849-955c-384a9c50fbb2)


## 思考

改变应用程序界面和功能的代码，查看效果。

![Screenshot 2025-05-14 132819](https://github.com/user-attachments/assets/7a822b76-1b6b-4b1d-aac9-8d2db85aeba5)
![Screenshot 2025-05-14 132831](https://github.com/user-attachments/assets/9d2eca1b-80d5-446e-9c6e-40bebaa4346d)


```kotlin
// 在activity_main.xml中添加拍照按钮
<Button
    android:id="@+id/captureButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="拍照"
    android:layout_gravity="bottom|center_horizontal"
    android:layout_marginBottom="24dp"
    android:backgroundTint="#4CAF50"
    android:textColor="#FFFFFF"
    android:paddingStart="24dp"
    android:paddingEnd="24dp"
    android:paddingTop="12dp"
    android:paddingBottom="12dp"
    android:elevation="8dp"/>

// 在MainActivity中添加处理逻辑
private val captureButton by lazy {
    findViewById<Button>(R.id.captureButton)
}

// 在onCreate中设置点击事件
captureButton.setOnClickListener {
    takePhoto()
}

// 添加拍照方法
private fun takePhoto() {
    val imageCapture = ImageCapture.Builder()
        .setTargetRotation(viewFinder.display.rotation)
        .build()
    
    try {
        // 创建临时文件
        val photoFile = File(
            getExternalFilesDir(Environment.DIRECTORY_PICTURES),
            "IMG_${System.currentTimeMillis()}.jpg"
        )
        
        // 创建输出选项
        val outputOptions = ImageCapture.OutputFileOptions.Builder(photoFile).build()
        
        // 拍照
        imageCapture.takePicture(
            outputOptions,
            ContextCompat.getMainExecutor(this),
            object : ImageCapture.OnImageSavedCallback {
                override fun onImageSaved(outputFileResults: ImageCapture.OutputFileResults) {
                    showStatus("照片已保存: ${photoFile.absolutePath}")
                    
                    // 可选：在保存的照片上运行识别
                    runRecognitionOnSavedImage(photoFile)
                }
                
                override fun onError(exception: ImageCaptureException) {
                    Log.e(TAG, "拍照错误: ${exception.message}", exception)
                    showStatus("拍照失败")
                }
            }
        )
    } catch (e: Exception) {
        Log.e(TAG, "拍照异常: ${e.message}", e)
        showStatus("拍照异常")
    }
}

// 对保存的图片运行识别（可选）
private fun runRecognitionOnSavedImage(file: File) {
    // 加载图片
    val bitmap = BitmapFactory.decodeFile(file.absolutePath)
    
    // 调整图片大小
    val resizedBitmap = Bitmap.createScaledBitmap(bitmap, 224, 224, true)
    
    // 运行识别
    val tfImage = TensorImage.fromBitmap(resizedBitmap)
    val outputs = flowerModel.process(tfImage)
        .probabilityAsCategoryList.apply {
            sortByDescending { it.score }
        }.take(MAX_RESULT_DISPLAY)
    
    // 更新UI
    val items = mutableListOf<Recognition>()
    for (output in outputs) {
        items.add(Recognition(output.label, output.score))
    }
    recogViewModel.updateData(items)
}
```

