# EffectiveForIOS_Command

* Previous preparation error: The developer disk image could not be mounted on this device
  1. Xcode 디바이스 연결 오류 해결 (폴더 접근 권한 허용)
     - sudo chmod a+w /private/var/tmp/ 

* Other Linker Flags Option
  1. 링커 Objetivc-C 및 Old-Linker 사용 플래그
     -ObjC
     -Wl
     -ld_classic
