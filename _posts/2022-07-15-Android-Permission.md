---
title: Android RequestMultiplePermissions()을 사용하여 권한 요청
author: Narvis2
date: 2022-07-15 15:22:00 +0900
categories: [Android, Permission]
tags: [android, permission]
---

안녕하세요. narvis2 입니다.
이번 포스팅에서는 Android에서 위험 권한을 요청하는 방법에 대해서 설명해볼까 합니다.  
기존에는 startActivityForResult()를 사용하여 권한 요청을 하였으나 해당 함수가 Deprecated됨에 따라 startActivityForResult()를 대체하는 방법을 Code를 통해 간단히 알아보겠습니다.

## RequestMultiplePermissions() 사용
``` kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding 

    // 필요한 권한을 배열에 담습니다.
    private val permissions = arrayOf(
        Manifest.permission.RECORD_AUDIO, 
        Manifest.permission.CAMERA, 
        Manifest.permission.WRITE_EXTERNAL_STORAGE
    )

    private val permissionResultLauncher = registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) {
        if(it.all { permissions -> permissions.value == ture}) {
            // TODO: 권한이 승인되었을 경우
        } else {
            // 권한이 거부 되었을 경우
        }
    }

    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.lifecycleOwner = this

        // 권한 체크 후 권한이 허가되지 않았으면 권한 요청
        if (!checkPermission(permissions)) {
            permissionResultLauncher.launch(permissions)
        }

        binding.startSubActivity.setOnClickListener {
            onStartSubActivity()
        }
    }

    // 권한을 체크하는 함수
    private fun checkPermissions(permissions: Array<String>): Boolean = permissions.all { it: String
        ContextCompat.checkSelfPermission(this, it) == PackageManager.PERMISSION_GRANTED
    }

    private fun onStartSubActivity() {
        // 권한이 거부된 경우 Dialog 생성
        if (!checkPermissions(permissions)) {
            val alert = AlertDialog.Builder(this)
            alert.apply {
                setTitle("권한 설정")
                setMessage("권한 거절로 인해 일부 기능이 제한될 수 있습니다.")
                setPositiveButton("권한 설정하러 가기") { _, _ ->
                    try {
                        // 권한이 없을 때 "애플리케이션 정보" 화면으로 이동
                        val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
                            .setData(Uri.fromParts("package", packageName, null))
                        startActivity(intent)
                    } catch (e: ActivityNotFoundException) {
                        e.printStackTrace()
                        // 나의 Application을 못찾았을 때 -> "애플리케이션" 목록 페이지로 이동
                        val intent = Intent(Settings.ACTION_MANAGE_APPLICATIONS_SETTINGS)
                        startActivity(intent)
                    }
                }
                setNegativeButton("취소") { dialog, _ ->
                    // 권한이 거부 되었을 경우 
                    dialog.dismiss()
                }
                create()
                show()
            }

            return
        } 

        // 권한을 승인한 경우 SubActivity로 이동
        startActivity(Intent(this, SubActivity::class.java))
    }
}
```