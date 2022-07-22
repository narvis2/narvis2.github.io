---
title: Android Glide를 이용한 Image Load
author: Narvis2
date: 2022-07-22 10:19:00 +0900
categories: [Android, Glide]
tags: [android, glide]
---

안녕하세요. Narvis2입니다.  
이번 포스팅에서는 Glide에 대하여 알아보도록 하겠습니다.  
Glide는 Android에서 이미지를 빠르고 효율적으로 불러올 수 있게 도와주는 라이브러리입니다.  
이미지, Gif, 비디오 스틸의 로딩과 디코딩, 캐싱 등의 다양한 API를 사용할 수 있습니다.  
Glide는 **_어떠한 종류의 이미지이더라도 빠르고 부드럽게 스크롤 하는 것을 목적으로 합니다._**
  
[Glide Github](https://github.com/bumptech/glide)
## Glide Cache
- Glide는 기존적으로 메모리 & 디스크에 이미지를 Caching 하여 다음번에 호출하였을 때 빠른 Image Loading을 지원합니다.
- 메모리 캐시
> - Glide는 기본적으로 메모리 캐싱을 하기 때문에 메모리 캐싱을 위해 추가적으로 할 일은 없습니다.
> - **_주의_**❗️ 단, Url 이미지 로딩 시 한번 Load한 이미지는 Cache에 저장되어 서버에서 해당 이미지를 변경해도 App의 이미지는 갱생되지 않습니다.
>> 이런 경우 .skipMemoryCache(true) 로 메모리 캐시를 사용하지 않을 수 있습니다.
- 디스크 캐시
> - Glide는 기본적으로 디스크 캐싱을 수행합니다. 기본적인 개념은 메모리 캐시와 같습니다.
> - .diskCacheStrategy()를 사용하여 캐싱에 관한 설정이 가능합니다.
>> - DiskCacheStrategy.ALL : 모든 이지를 캐싱합니다.
>> - DiskCacheStrategy.AUTOMATIC : default 값이며, RESOURCE 를 기반으로 전략적인 캐싱을 수행합니다.
>> - DiskCacheStrategy.DATA : 원본 이미지를 캐싱합니다.
>> - DiskCacheStrategy.RESOURCE : 해상도를 줄인 이미지만 캐싱합니다.
>> - DiskCacheStrategy.NONE : 디스크 캐싱을 수행하지 않습니다.

## Glide 이미지 불러오기 절차 (여러 캐시를 확인)
1. Active Resources : 현재 이 이미지가 다른 View에 나타나는가? 를 확인합니다.
2. Memory Cache : 이 이미지가 최근에 메모리에 load 되었거나 여전히 메모리에 남아있는가? 를 확인합니다.
3. Resource : 이 이미지가 예전에 decode 되었거나, 변형 되었거나, 디스크 캐시에 기록되어 있는가? 를 확인합니다.
4. Data : 이 이미지가 이전에 디스크 캐시에 기록 되었던 데이터 였는가? 를 확인합니다.

## Glide Image Loading
- Glide는 빌더 패턴으로 구성되어 있고, 3개의 필수 파타미러를 요구합니다.
- Glide는 기본적으로 비동기로 이미지를 load 합니다.
> **_참고_** : 필수 파라미터
> - with() : context를 넣어줍니다.
> - load() : 대상 이미지를 넣어줍니다.(URL, Resource Id, File Url ...)
> - into() : 이미지를 보여줄 ImageView를 넣어줍니다.
- ✔️ **이미지 변형**
> **_참고_** : 이미지 변형 함수
> - centerCrop() : 이미지가 ImaveView보다 클 때, ImageView의 크기에 맞춰 이미지 크기를 조정 후 자릅니다. ImageView가 완전히 채워지지만, 전체 이미지가 표시되지 않을 수 있습니다.
> - fitCenter() : 이미지가 ImageView의 사이즈와 다를 때, ImageView 크기와 같거나 작도록 이미지의 크기를 조정합니다. 이미지가 완전히 표시되지만, 전체 ImageView 크기를 채우지 않을 수 있습니다.
> - circleCrop() : 이미지가 원으로 변경됩니다.(CenterCrop()과 동일한 방식입니다.)
- ✔️ **Option** : 사용자 정의할 수 있는 유형 독립적 옵션을 제공합니다.
- 🚩 RequestOption() 인스턴스를 생성하고 apply()를 활용하여 원하는 Option을 넣습니다.
> - placeHolder(@DrawableRes int resourceId) : 요청받은 이미지가 나타나기 전까지 지정한 이미지를 보여줍니다. 네트워크로 이미지를 요청하거나 큰 이미지를 요청하여 시간이 오래 걸릴 때 PlaceHolder 로 지정한 이미지를 보여줍니다.
> - error(@DrawableRes int resourceId) : 이미지 로드에 실패한 경우 Error 에 지정된 이미지를 보여줍니다.
> - fallback(@Nullable Drawable drawable) : 요청된 URL or 모델이 null 인 경우 fallback으로 지정한 이미지를 보여줍니다.
``` kotlin
Glide.into(context)
    .load("Image Url 넣기")
    .apply(
        RequestOption().placeHoler(이미지 로딩 중 표시될 이미지 넣기).error(이미지 로드 실패 시 표시될 이미지 넣기)
    )
    .into(imageView)
```
- ✔️ **Target**
- Glide에서 Target 을 사용하면 bitmap, drawable, placeHolder 를 받아 처리할 수 있습니다.
- 다음은 예제 코드 입니다. 해당 예제 코드는 암시적 인텐트 사용하여 핸드폰의 갤러리에 접근해 받아온 Uri을 비트맵으로 바꾸는 로직입니다. 👇
``` kotlin
private val startIntentGallery: ActivityResultLauncher<Intent> = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
) { result: ActivityResult ->
    result.data?.extras?.clear()
    if (result.data?.data != null && result.resultCode == AppCompatActivity.RESULT_OK) {
        val context = binding.imageView.context

        // ContentProvider를 통해 Image Uri를 bitmap으로 변경합니다.
        val bitmap = MediaStore.Images.Media.getBitmap(
            context.contentResolver,
            result.data?.data!!
        )

        Glide.with(context)
            .asBitmap()
            .load(bitmap)
            override(1024, 1024)
            .into(object : CustomTarget<Bitmap>() {
                override fun onResourceReady(
                    resource: Bitmap,
                    transition: Transition<in Bitmap>?
                ) {
                    // TODO : ImageView에 이미지 넣기
                }

                override fun onLoadCleared(placeholder: Drawable?) {}
            })
    }
}
```

## Glide Blur 처리
- Blur를 통해 이미지를 흐리게 만듭니다.
- gradle 파일에 다음 dependency 를 추가합니다.👇
``` gradle
implementation "jp.wasabeef:glide-transformations::4.3.0" 
```
> **_참고_** : 다음은 Glide Blur 처리 예제 코드입니다. 👇
> BlurTransformation 의 첫 번째 인자는 radius 입니다. 해당 radius 값에 따라 흐림의 정도가 설정됩니다.  
> **_주의_**❗️ : BlurTransformation을 사용할때 centerCrop()을 먼저 사용하고 BlurTransformation을 사용해야합니다. 그렇지 않으면 오류가 발생합니다.
``` kotlin
fun blurImageLoader(
    context: Context,
    imageView: ImageView,
    url: String
) {
    Glide.with(context)
        .setDefaultRequestOptions(
            RequestOptions()
                .diskCacheStrategy(DiskCacheStrategy.RESOURCE) // 해상도를 줄인 이미지만 캐싱
                .centerCrop()
                .transform(BlurTransformation(10, 3)) // Blur 적용
        )
        .load(url)
        .into(imageView)
}
```

## Glide error, placeHolder 처리 예제
- 다음은 Glide를 이용하여 Image를 Load할때 error, placeHolder를 처리하는 예제입니다. 👇
``` kotlin
fun loadImage(
    view: ImageView,
    imageUrl: String,
    @DrawableRes
    placeholder: Int = 0,
    @DrawableRes
    error: Int = 0,
) {
    // imageUrl 이 비어있을 때 로직 추가
    if (imageUrl.isEmpty()) {
        if (placeholder != 0) {
            view.setImageResource(placeholder)
        }

        return
    }

    val options = RequestOptions()

    @SuppressLint("CheckResult")
    if (error != 0) {
        // 로드가 실패할 경우 표시할 리소스를 설정합니다.
        options.error(error)
    }

    Glide.with(view.context)
        .setDefaultRequestOptions(options)
        .load(imageUrl).apply {
            @SuppressLint("CheckResult")
            if (placeholder != 0) {
                // 리소스가 로드되는 동안 표시할 드로어블 리소스의 Android 리소스 ID를 설정합니다.
                apply(options.placeholder(placeholder))
            }
        }
        .into(object : CustomTarget<Drawable>() {
            override fun onResourceReady(resource: Drawable, transition: Transition<in Drawable>?) {
                view.setImageDrawable(resource)
            }

            override fun onLoadCleared(placeholder: Drawable?) {
                view.setImageDrawable(placeholder)
            }
        })
}
```

## 마치며
이번 포스팅에서는 Glide의 기본 개념에 대하여 알아보았습니다.  
Glide는 기본적으로 이미지를 비동기로 load하며, 기본적으로 캐싱을 지원하여 보다 빠르게 이미지를 불러올 수 있다는 점을 기억해주시면 되겠습니다.  