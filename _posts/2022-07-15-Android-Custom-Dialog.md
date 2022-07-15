---
title: Android Custom Dialog 
author: Narvis2
date: 2022-07-15 16:38:00 +0900
categories: [Android, Custom]
tags: [android, custom, dialog]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅은 조금 가벼운 주제로 쉬어가고자 합니다.  
Custom Dialog를 만들어 필요할때 마다 간단히 호출하여 사용하는 방법을 익혀보고자 합니다.  

## Custom Dialog Class 생성
---
``` kotlin
class BasicDialog(
    private context: Context,
    root: ViewGroup? = null
) {
    private val builder = AlertDialog.Builder(context, R.style.AlertDialogTheme)

    private val binding: DialogBasicPopupBinding by lazy {
        val b = DialogBasicPopupBinding.inflate(LayoutInflater.from(context), root, false)
        // DataBinding 을 사용합니다.
        b.body = null
        b.okCallback = null
        b.cancelCallback = null

        return@lazy b
    }

    // androidx 의 AlertDialog를 import 해주셔야 합니다.
    private var dialog: AlertDialog? = null

    // Dialog 의 Title을 지정해주는 Method 입니다. 
    // R.string.confirm 의 형식을 인자에 넣어주는 Method 입니다.
    fun setTitle(@StringRes titleId: Int): BasicDialog {
        return setTitle(context.getText(titleId))
    }
    // 위의 setTitle 의 return 에서 사용하는 Method 입니다.
    // resources.getString() 로 string resource를 가져와 인자에 넣어주는 방식의 Method 입니다.
    fun setTitle(title: CharSequence): BasicDialog {
        // Data Binding 을 사용하여 xml에 넣어줍니다.
        binding.subject = title
        return this
    }

    // Dialog 의 Message를 지정해주는 Method 입니다.
    // R.string.confirm 의 형식을 인자에 넣어주는 Method 입니다.
    fun setMessage(@StringRes messageId: Int): BasicDialog {
        return setMessage(context.getText(messageId))
    }
    // 위의 setMessage 함수의 return에서 사용하는 Method 입니다.
    // resource.getString() 로 string resource를 가져와 인자에 넣어주는 방식의 Method 입니다. 
    fun setMessage(message: CharSequence): BasicDialog {
        // Data Binding 을 사용하여 xml에 넣어줍니다.
        binding.body = message
        return this
    }

    // Dialog 의 Message를 지정해주는 Method 입니다.
    // R.string.confirm 의 형식을 인자에 넣어주는 Method 입니다.
    fun setMessageStyle1(@StringRes messageId: Int): BasicDialog {
        return setMessageStyle1(context.getText(messageId))
    }
    // 위의 setMessageStyle1 함수의 return에서 사용하는 Method 입니다.
    // resource.getString() 로 string resource를 가져와 인자에 넣어주는 방식의 Method 입니다.
    fun setMessageStyle1(message: CharSequence): BasicDialog {
        // SpannableStringBuilder 를 사용하면 Code 상에서 TextView에 넣을 문자 일부의 색, 크기, 스타일을 변경할 수 있습니다.
        return setMessage(SpannableStringBuilder(message).apply {
            // 지정된 개체로 지정된 텍스트 범위를 표시합니다. 
            // Flag 는 범위 범위의 시작 또는 끝에 텍스트가 삽입될 때 범위가 작동하는 방식을 결정합니다.
            // start 변수에서부터 end 변수까지를 what의 설정으로 변경합니다.
            setSpan(
                // 텍스트 크기를 물리적 픽셀 크기로 설정하거나 'dip'이 true인 경우 장치 독립적 픽셀 크기로 설정합니다.
                what = AbsoluteSizeSpan(
                    TypedValue.applyDimension(
                        TypedValue.COMPLEX_UNIT_SP,
                        17f, // Text 크기를 17f로 고정
                        binding.root.context.resources.displayMetrics
                    ).toInt(), 
                    dip = false
                ), 
                start = 0, 
                end = length, 
                flags = Spanned.SPAN_EXCLUSIVE_EXCLUSIVE // 길이가 0일 수 없으며 포함하는 모든 텍스트가 제거되면 버퍼에서 자동으로 제거됩니다.
            )
        })
    }

    // Dialog의 "확인" 버튼 Click Listener 라고 생각하시면 됩니다.
    // R.string.confirm 의 형식을 인자에 넣어주고 Click Listener Callback을 받는 Method 입니다.
    fun setPositiveButton(@StringRes textId: Int, listener: View.OnClickListener?): BasicDialog {
        return setPositiveButton(context.getText(textId), listener)
    }
    // resource.getString() 로 string resource를 가져와 인자에 넣어주고 Click Listener Callback을 받는 Method 입니다.
    fun setPositiveButton(text: CharSequence, listener: View.OnClickListener?): BasicDialog {
        // 버튼 이름을 Data Binding 을 통해 넣어줍니다.
        binding.ok = text
        if (listener != null) {
            // Click Listener를 Data Binding을 통해 넣어줍니다.
            binding.okCallback = View.OnClickListener { v ->
                listener.onClick(v)
                dismiss()
            }
        } else {
            binding.okCallback = listener
        }

        return this
    }

    // Dialog "부정" 버튼 Click Listener 라고 생각하시면 됩니다.
    // R.string.confirm 의 형식을 인자에 넣어주고 Click Listener Callback을 받는 Method 입니다.
    fun setNegativeButton(@StringRes textId: Int, listener: View.OnClickListener): BasicDialog {
        return setNegativeButton(context.getText(textId), listener)
    }
    // resource.getString() 로 string resource를 가져와 인자에 넣어주고 Click Listener Callback을 받는 Method 입니다.
    fun setNegativeButton(text: CharSequence, listener: View.OnClickListener?): BasicDialog {
        // 부정 버튼 이름을 Data Binding 을 통해 넣어줍니다.
        binding.cancel = text
        if (listener != null) {
            // Click Listener를 Data Binding을 통해 넣어줍니다.
            binding.cancelCallback = View.OnClickListener { v ->
                listener.onClick(v)
                dismiss()
            }
        } else {
            binding.cancelCallback = listener
        }

        return this
    }
    
    // Dialog 취소가 되는지 여부를 설정합니다.
    fun setCancelable(cancelable: Boolean): BasicDialog {
        builder.setCancelable(cancelable)
        return this
    }
    // Dialog 의 취소 Listener 를 설정합니다.
    fun setOnCancelListener(onCancelListener: DialogInterface.OnCancelListener): BasicDialog {
        builder.setOnCancelListener(onCancelListener)
        return this
    }
    
    // Dialog 가 dismiss 될때 호출되는 Listener 를 설정합니다.
    // Dialog 가 dismiss 될때 해야할 적업이 있을 때 사용하시면 됩니다.
    fun setOnDismissListener(onDismissListener: DialogInterface.OnDismissListener): BasicDialog {
        builder.setOnDismissListener(onDismissListener)
        return this
    }

    // Dialog 에 Custom View를 세팅해주고 dialog를 create() 해주는 Method 입니다.
    private fun create(): AlertDialog {
        builder.setView(binding.root)
        dialog = builder.create()

        return dialog!!
    }

    // Dialog 를 보여주는 함수입니다. Dialog 는  마지막에 show()를 호출하지 않으면 동작하지 않습니다.
    fun show() {
        create().show()
    }
    
    // Dialog 를 dismiss 하는 함수입니다.
    fun dismiss() {
        dialog?.dismiss()
    }
}
```

## xml 쪽 알아보기
---
DialogBasicPopupBinding 내부를 살펴보겠습니다.
``` xml
<?xml version="1.0" encoding="utf-8"?>
<!-- Data Binding 을 사용하기 위해 alt + enter 를 통해 layout 테그를 만들어 줍니다. -->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <!-- View의 Visible 처리를 해주기 위해 import 합니다. -->
        <import type="android.view.View" />

        <!-- 제목, 즉 Title을 담당하는 변수입니다. -->
        <variable
            name="subject"
            type="CharSequence" />

        <!-- Message를 담당하는 변수입니다. -->
        <variable
            name="body"
            type="CharSequence" />

        <!-- 취소 버튼의 이름을 담당하는 변수입니다. -->
        <variable
            name="cancel"
            type="CharSequence" />

        <!-- 확인 버튼의 이름을 담당하는 변수입니다. -->
        <variable
            name="ok"
            type="CharSequence" />

        <!-- 취소 버튼의 Click Listenr 를 담당해줄 변수입니다. -->
        <variable
            name="cancelCallback"
            type="android.view.View.OnClickListener" />

        <!-- 확인 버튼의 Click Listenr 를 담당해줄 변수입니다. -->
        <variable
            name="okCallback"
            type="android.view.View.OnClickListener" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:paddingHorizontal="20dp"
        tools:background="@color/colorSignUpHint">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="20dp"
            android:background="@drawable/white_round_frame">

            <LinearLayout
                android:id="@+id/basicDialogTitleLl"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:minHeight="120dp"
                android:orientation="vertical"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent">

                <!-- 삼항식을 사용하여 title이 null 이 아니면 보여주고 null 이면 GONE 처리합니다. -->
                <TextView
                    android:id="@+id/subjectTv"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:gravity="center"
                    android:letterSpacing="-0.02"
                    android:lineSpacingExtra="6sp"
                    android:paddingBottom="7.5dp"
                    android:lineSpacingMultiplier="0.65"
                    android:maxLines="3"
                    android:paddingTop="2.2dp"
                    android:text="@{subject}"
                    android:textColor="#000000"
                    android:textSize="17sp"
                    android:textStyle="bold"
                    android:visibility="@{subject != null ? View.VISIBLE : View.GONE}"
                    tools:text="인증에 실패하였습니다.\n다시 인증해주세요." />

                <!-- 삼항식을 사용하여 Message가 null 이 아니면 보여주고 null 이면 GONE 처리합니다. -->
                <TextView
                    android:id="@+id/bodyTv"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginBottom="24.5dp"
                    android:gravity="center_horizontal"
                    android:letterSpacing="-0.03"
                    android:lineSpacingExtra="6sp"
                    android:lineSpacingMultiplier="0.65"
                    android:maxLines="5"
                    android:text="@{body}"
                    android:textColor="#000000"
                    android:textSize="14sp"
                    android:visibility="@{body != null ? View.VISIBLE : View.GONE}"
                    tools:text="인증번호를 받지 못한 경우 재요청하여 인증번호를 수신 후 진행하여 주십시오." />
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:orientation="horizontal"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toBottomOf="@id/basicDialogTitleLl">

                <TextView
                    android:layout_width="0dp"
                    android:layout_height="48dp"
                    android:layout_weight="1"
                    android:background="@drawable/common_button_cancel"
                    android:gravity="center"
                    android:letterSpacing="-0.02"
                    android:lineSpacingExtra="-4sp"
                    android:text="@{cancel}"
                    android:textColor="#ffffff"
                    android:textSize="16sp"
                    android:visibility="@{cancelCallback != null ? View.VISIBLE : View.GONE}"
                    android:onClick="@{cancelCallback}"
                    tools:text="@string/str_cancel" />
                
                <!-- Space를 통해 취소 버튼과 확인 버튼 사이에 공백을 만듭니다. -->
                <!-- 취소 버튼의 ClickListener 가 Null 이 아니고 확인 버튼의 ClickListener 가 Null 이 아닐때만 보여주고 둘 중 하나라도 NUll 일 경우 Gone 처리 됩니다. -->
                <!-- &amp;&amp; = && 와 같습니다. -->
                <Space
                    android:id="@+id/space"
                    android:layout_width="8dp"
                    android:layout_height="48dp"
                    android:visibility="@{cancelCallback != null &amp;&amp; okCallback != null ? View.VISIBLE : View.GONE}" />

                <TextView
                    android:layout_width="0dp"
                    android:layout_height="48dp"
                    android:layout_weight="1"
                    android:background="@drawable/common_button_ok"
                    android:gravity="center"
                    android:letterSpacing="-0.02"
                    android:lineSpacingExtra="-4sp"
                    android:text="@{ok}"
                    android:textColor="#ffffff"
                    android:textSize="16sp"
                    android:visibility="@{okCallback != null ? View.VISIBLE : View.GONE}"
                    android:onClick="@{okCallback}"
                    tools:text="@string/confirm" />
            </LinearLayout>
        </androidx.constraintlayout.widget.ConstraintLayout>
    </LinearLayout>
</layout>
```

## 실제로 사용해 보기
---
``` kotlin
class HomeFragment : Fragment() {
    private lateinit var binding: FragmentHomeBinding
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = DataBindingUtil.inflate(inflater, layoutResId, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.lifecycleOwner = viewLifecycleOwner

        binding.showDialogButton.setOnClickListener {
            BasicDialog(requireContext(), binding.root as? ViewGroup)
                .setMessageStylel(
                    getSpannedFromHtml(
                        resources.getString(R.string.str_hello)
                            .format("narvis2")
                    )
                )
                .setPositiveButton(resources.getString(R.string.str_confirm)) {
                    // TODO: 확인 버튼 클릭 시 동작 작성
                }
                .setNegativeButton(resources.getString(R.string.str_cancel)) {
                    // TODO: 취소 버튼 클릭 시 동작 작성
                    // setNegativeButton 함수 내부에 dialog.dismiss()가 포함되어 있으므로 빈칸으로 놔둬도 클릭 시 자동 dismiss처리 됩니다.
                }
                .show() // show() 함수 내부에 create().show() 로 되어있습니다.
        }
    }

    // 제공된 HTML 문자열에서 표시 가능한 스타일 텍스트를 반환하는 Method 입니다.
    private fun getSpannedFromHtml(text: String?): Spanned? {
        return if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N) {
            Html.fromHtml(text)
        } else {
            Html.fromHtml(text, Html.FROM_HTML_MODE_LEGACY)
        }
    }
}
```

## strings 파일
---
``` xml
<!-- 이렇게하면 <b> </b> 안의 글자만 Bold 처리 됩니다. -->
<string name="str_hello"><![CDATA[<b>%1$s</b> 님 안녕하십니까?]]></string>
<string name="str_confirm">확인</string>
<string name="str_cancel">취소</string>
```

## drawable
---
### 1. common_button_cancel.xml 입니다.  
- 취소 버튼 색상이 회색으로 바뀌며 모서리가 14dp 만큼 둥글게 됩니다.  

``` xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <corners android:radius="14dp" />
    <solid android:color="#a8a9ae" />
</shape>
```
### 2. common_button_ok.xml 입니다.
- 확인 버튼 색상이 남색으로 바뀌며 모서리가 14dp 만큼 둥글게 됩니다.  

``` xml
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <corners android:radius="14dp" />
    <solid android:color="#152348" />
</shape>
```

## Style
---
<h3>AlertDialog.Builder(context, R.style.AlertDialogTheme) 에 적용되는 Style 입니다.</h3>
- 뒷 배경을 투명으로 만듭니다.
- dialog 가 화면의 위, 아래 기준으로 중간에 오도록 합니다.
- 상태바 색상을 투명으로 만듭니다.  

``` xml
<style name="AlertDialogTheme" parent="transparent">
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:layout_gravity">center_vertical</item>
    <item name="android:statusBarColor">@android:color/transparent</item>
</style>
```