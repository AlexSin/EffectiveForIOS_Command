# EffectiveForIOS_Command

* Previous preparation error: The developer disk image could not be mounted on this device
  1. Xcode 디바이스 연결 오류 해결 (폴더 접근 권한 허용)
     - sudo chmod a+w /private/var/tmp/ 

* Other Linker Flags Option
  1. 링커 Objetivc-C 및 Old-Linker 사용 플래그
     -ObjC
     -Wl
     -ld_classic

* 빌드 설정 분리
  1. PROJECT 선택 -> Configurations -> + 로 Configuration File 생성
  2. Target -> Build Settings -> Product Bundle Identifier -> 새로 생성한 Configuration 맞게 번들ID 설정
  3-1. Target -> General -> Identify에서 Display Name / Bundle Idenetifier 추가
  3-2. Target -> Build Settings -> User-Defined 에 원하느 값을 추가 (ex) API_BASE_URL, APP_DISPLAY_NAME 등)
  4. Info.plist -> User-Defined 선언된 값을 정의 ( SERVER_TYPE $(SERVER_TYPE), API_BASE_URL $(API_BASE_URL), APP_DISPLAY_NAME $(APP_DISPLAY_NAME)
  5. 상단 Scheme 선택 -> New Scheme 생성 -> 설정 하고 싶은 Configurations을 new Scheme에서 설정
 

* 선언한 데이터 설정 및 확인
ex) // 환경에 따라 dev 도메인 혹은 prod 도메인 자동
if let domainUrl = Bundle.main.object(forInfoDictionaryKey: "API_BASE_URL") as? String {
    print("현재 연결된 서버 도메인: \(domainUrl)")
    // 네트워킹 코드에 이 domainUrl을 사용하세요.
}
