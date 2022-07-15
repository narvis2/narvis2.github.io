---
title: Android 4대 컴포넌트 중 ContentProvider 에 대하여 
author: Narvis2
date: 2022-07-15 10:32:00 +0900
categories: [Android, 4 Component]
tags: [contentProvider, android, contentResolver]
---

안녕하세요. Narvis2입니다.  
지난 포스팅에서는 [BroadcastReceiver](https://narvis2.github.io/posts/Android-BroadcastReceiever/)에 대하여 알아보았습니다.  
이번 포스팅에서는 지난 시간에 이어 Android 4대 Component 중 하나인 **_ContentProvider_**에 관하여 알아보도록 하겠습니다.  
ContentProvider는 Application 사이에서 Data를 공유하는 통로 역할을 합니다. 즉, Android System의 각종 설정값이나 DB에 접근하게 해줍니다.  
여기에 이어 ContentResolver라는 녀석이 나오는데 이 녀석은 ContentProvider는 미리 만들어져 있는 ContentProvider로 부터 데이터를 가져오는 도구 라고 생각하시면 됩니다.  
간단하게 요약하면 ContentProvider는 Android System 의 각종 설정값이나 DB에 접근하게 해주고 ContentResolver는 그 결과 값을 가져오는 역할 입니다.

## ContentProvider
- Application의 Data를 다른 Application과 공유하는 것을 도와줍니다.
- 나의 Application 이 ContentProvider를 구현하여 제공하면, 다른 Application들은 ContentResolver를 통하여 나의 Application 에 구현된 ContentProvider에 접근할 수 있습니다.
- 다른 Application들은 ContentProvider가 제공하는 API들을 이용하여 데이터를 읽거나 저장, 삭제 할 수 있습니다.  
물론 내부적으로 SQLite와 같은 Database를 사용하여 Data를 관리해야 합니다.
- ContentProvider는 다른 Application과 Data를 공유하기 위한 인터페이스라고 이해하시면 됩니다.

## ContentResolver
- ContentProvider 가 제공하는 Data를 가져오기 위해 ContentResolver를 사용합니다.
- **_"미리 만들어져 있는 ContentProvider로 부터 Data를 가져오는 도구"_**라고 이해하시면 됩니다.
- Application이 ContentProvider에 접근할 때는 ContentResolver를 통해 접근하게 됩니다.
- 기본적으로 CRUD 함수를 제공합니다.(다른 Application의 Database를 조작할 수 있습니다.)
- 주로 Android에 있는 연락처, 갤러리, 음악 파일과 같은 기본 데이터를 가져오는 용도로 사용됩니다.
> **참고** : Android는 Media 정보를 저장하는 저장소 용도로 **_MediaStore_**을 사용합니다.

## ContentProvider, ContentResolver 를 통해 Image를 가져오기
ContentResolver를 통하여 갤러리에 접근하여 원하는 Image를 가져오는 예제를 작성해 보겠습니다.  

1. MediaStore에서 이미지를 가져오기
``` kotlin
    private fun onAttachFileClick() {
        // Android 의 4대 Component 의 통신은 Intent로 합니다.
        val intent = Intent(Intent.ACTION_PICK).apply { this: Intent
            type = MediaStore.Images.Media.CONTENT_TYPE
            type = "image/*"
        }

        startIntentImage.launch(intent)
    }
```
2. ContentResolver를 통하여 Image 가져오기
``` kotlin
private val startIntentImage: ActivityResultLauncher<Intent> = 
    registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result: ActivityResult ->
        result.data?.extras?.clear()

        val data = result.data?.data
        if (data != null && result.resultCode == AppCompatActivity.RESULT_OK) {
            val context = binding.imageView.context

            val bitmap = MediaStore.Images.Media.getBitmap(
                context.contentResolver,
                data
            )

            // Bitmap 의 비율을 지정합니다.
            val ratio = if (max(bitmap.width.toDouble(), bitmap.height.toDouble()) > 1270) {
                when {
                    bitmap.width > bitmap.height -> {
                        1270.0 / bitmap.width.toDouble()
                    }

                    else -> {
                        1270.0 / bitmap.height.toDouble()
                    }
                }
            } else {
                1.0
            }

            val width = (bitmap.width.toDouble() * ratio).toInt()
            val height = (bitmap.height.toDouble() * ratio).toInt()

            Glide.with(context)
                .asBitmap()
                .load(bitmap)
                .override(width, height)
                .centerCrop()
                .into(object : CustomTarget<Bitmap>() {
                    override fun onResourceReady(
                        resource: Bitmap,
                        transition: com.bumptech.glide.request.transition.Transition<in Bitmap>?,
                    ) {
                        // 해당 함수를 통해 Custom 된 Bitmap을 ImageView에 넣습니다.
                        BindingAdapters.loadBitmapCorner(
                            view = binding.imageView,
                            bitmap = resource,
                            corner = 5,
                            placeHolder = R.drawable.round_frame_gray_5
                        )

                        // bitmap 과 Glide 의 load에 넣었던 bitmap이 다른 경우 bitmap을 지웁니다.
                        if (bitmap != resource) {
                            bitmap.recycle()                            
                        }
                    }

                    override fun onLoadCleared(placeholder: Drawable?) {}
                })
        }
    }
```
3. loadBitmapCorner() 함수 
DataBinding을 위한 BindingAdapter를 사용한 함수입니다.
``` kotlin
@BindingAdapter(value = ["loadBitmapCorner", "corner", "placeholder", "error", "mask"])
fun loadBitmapCorner(
        view: ImageView,
        bitmap: Bitmap,
        corner: Int = 0,
        @DrawableRes
        placeholder: Int = 0,
        @DrawableRes
        error: Int = 0
) {
    val arrayList = arrayListOf<Transformation<Bitmap>>()
    arrayList.add(CenterCrop())
    // image의 Corners 를 지정해줍니다.
    if (corner != 0) {
        arrayList.add(RoundedCorners(corner.pxToDp(view.context)))
    }
        
    val options = RequestOptions()
        .transform(*(arrayList.toTypedArray()))

    // 이미지 load에 실패한 경우 error에 저장된 이미지를 보여줍니다.
    @SuppressLint("CheckResult")
    if (error != 0) {
        options.error(error)
    }

    Glide.with(view.context)
        .setDefaultRequestOptions(options)
        .load(bitmap).apply {
            // placeHolder -> 요청받은 이미지가 나타나기 전까지 지정한 이미지를 보여줍니다.
            // 네트워크로 이미지를 요청하거나 큰 이미지를 요청하여 시간이 오래 걸릴 때 PlaceHolder로 지정한 이미지를 보여줍니다.
            @SuppressLint("CheckResult")
            if (placeholder != 0) {
                apply(options.placeholder(placeholder))
            }
        }
        .into(view)
}
```

## 마치며
이번 포스팅에서는 ContentProvider, ContentResolver에 대하여 알아보았습니다.  
이번 포스팅을 요약하자면 ContentProvider 가 제공하는 Data를 가져오기 위해 ContentResolver를 사용하며 Application이 ContentProvider에 접근할 때는 ContentResolver를 통해 접근하게 됩니다.  
다음 포스팅에서는 Android 4대 Component 간에 통신을 위해 사용하는 Intent에 대하여 알아보도록 하겠습니다.