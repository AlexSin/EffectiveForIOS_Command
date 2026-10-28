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


* Beyond Compare 초기화
  1. cd Library/Application\ Sypport/Beyond\ Compare/ 5
  2. rm -f registry.dat 초기화
  3. Beyond Compare 다시 사용 ㅋㅋ

 
* extension UIImage 정리
import Foundation
import UIKit
import ImageIO

/// 이미지를 Base64로 변환할 때 사용할 포맷과 품질을 정의합니다.
public enum ImageBase64Format {
    case png // 투명도가 유지되는 PNG
    case jpeg(quality: CGFloat) // 용량 조절이 가능한 JPEG (0.0 ~ 1.0)
}

extension UIImage {

    /// base64 encoding
    public var base64: String {
        return self.jpegData(compressionQuality: 0.8)?.base64EncodedString() ?? ""
    }

    /// init
    /// - Parameters:
    ///   - base64: Base64 문자열
    ///   - withPrefix: Prefix 추가 유/무
    convenience init?(base64: String, withPrefix: Bool) {
        var finalData: Data?

        if withPrefix {
            guard let url = URL(string: base64) else { return nil }
            finalData = try? Data(contentsOf: url)
        } else {
            finalData = Data(base64Encoded: base64)
        }

        guard let data = finalData else { return nil }
        self.init(data: data)
    }
    
    /// UIImage를 Base64 문자열로 변환합니다.
    /// - Parameter format: 변환할 이미지 포맷 (기본값은 JPEG 80% 품질)
    /// - Returns: Base64로 변환된 문자열. 변환 실패 시 nil 반환.
    public func toBase64(format: ImageBase64Format = .jpeg(quality: 0.8)) -> String? {
        let imageData: Data?
        
        switch format {
        case .png:
            // PNG는 투명도(Alpha) 채널을 보존해야 할 때 사용합니다.
            imageData = self.pngData()
        case .jpeg(let quality):
            // JPEG는 압축 품질(0.0 ~ 1.0)을 지정하여 용량을 줄일 때 사용합니다.
            // 1.0은 원본 유지(용량 큼), 보통 0.8 정도가 웹 전송에 적합합니다.
            imageData = self.jpegData(compressionQuality: quality)
        }
        
        // 데이터가 정상적으로 추출되었다면 Base64 문자열로 인코딩하여 반환합니다.
        return imageData?.base64EncodedString()
    }
    
    /// Base64 문자열을 이용해 UIImage를 생성합니다.
    /// "data:image/png;base64," 등의 접두사(Prefix)가 있어도 자동으로 제거하고 처리합니다.
    /// - Parameter base64String: 이미지 데이터가 담긴 Base64 문자열
    public convenience init?(base64String: String) {
        var cleanBase64 = base64String
        
        // 접두사(Prefix) 자동 감지 및 제거
        // 웹이나 하이브리드 앱에서 넘어오는 "data:image/...;base64," 포맷을 자동으로 잘라냅니다.
        if cleanBase64.hasPrefix("data:image") {
            if let commaIndex = cleanBase64.firstIndex(of: ",") {
                let afterComma = cleanBase64.index(after: commaIndex)
                cleanBase64 = String(cleanBase64[afterComma...])
            }
        } 
        
        // Base64 -> Data 변환
        // .ignoreUnknownCharacters 옵션을 주면 문자열 사이에 포함된 줄바꿈(\n)이나 공백을 무시하여 파싱 에러를 방지합니다.
        guard let data = Data(base64Encoded: cleanBase64, options: .ignoreUnknownCharacters) else {
            printSC("Base64 문자열을 Data로 변환 실패")
            return nil
        }
        
        // Data -> UIImage 초기화
        self.init(data: data)
    }
    
    static func from(color: UIColor) -> UIImage {
        let rect = CGRect(x: 0, y: 0, width: 1, height: 1)
        UIGraphicsBeginImageContext(rect.size)
        let context = UIGraphicsGetCurrentContext()
        context!.setFillColor(color.cgColor)
        context!.fill(rect)
        let img = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return img!
    }
    
    static func UIColorFromRGB(rgbValue: UInt) -> UIColor {
          return UIColor(
              red: CGFloat((rgbValue & 0xFF0000) >> 16) / 255.0,
              green: CGFloat((rgbValue & 0x00FF00) >> 8) / 255.0,
              blue: CGFloat(rgbValue & 0x0000FF) / 255.0,
              alpha: CGFloat(1.0)
          )
    }
    
    // gif 이미지 데이터 쪼개서 불러 오기
    public class func gifImageName(_ name: String) -> UIImage? {
            guard let bundleURL = Bundle.main.url(forResource: name, withExtension: "gif"),
                  let imageData = try? Data(contentsOf: bundleURL) else {
                return nil
            }
            return gifImageData(imageData)
        }

    // gif 데이터 애니메션 이미지로 변환
    private class func gifImageData(_ data: Data) -> UIImage? {
        guard let source = CGImageSourceCreateWithData(data as CFData, nil) else {
            return nil
        }
        
        let count = CGImageSourceGetCount(source)
        var images = [UIImage]()
        var duration = 0.0
        
        for i in 0..<count {
            if let cgImage = CGImageSourceCreateImageAtIndex(source, i, nil) {
                images.append(UIImage(cgImage: cgImage))
                // 프레임당 0.07초 딜레이 (필요시 조절 가능: 숫자가 낮을 수록 빠름)
                duration += 0.07
            }
        }
        
        return UIImage.animatedImage(with: images, duration: duration)
    }
    
    //    let imageName = "apngb-animated.apng"
    public class func apngImageName(_ name: String) -> UIImage? {
        guard let bundleURL = Bundle.main.url(forResource: name, withExtension: "apng"),
              let imageData = try? Data(contentsOf: bundleURL) else {
            return nil
        }
        return apngImageData(imageData as CFData)
    }
    
    // apng ani 데이터 이미지로 변환
    private class func apngImageData(_ data: CFData) -> UIImage? {
        // 이미지소스 반환
        guard let source = CGImageSourceCreateWithData(data, nil) else {
            return nil
        }
                
        let count = CGImageSourceGetCount(source)   // 이미지소스 갯수
        var images:[UIImage] = []                   // 이미지 배열
        var duration: TimeInterval = 0.0            // 딜레이 시간
        
        for i in 0..<count {
            if let cgImage = CGImageSourceCreateImageAtIndex(source, i, nil) {
                // 인덱스 별 이미지 저장
                images.append(UIImage(cgImage: cgImage))
                // 프레임당 0.03초 딜레이
                // duration += 0.03
                // 각 프레임의 딜레이 시간 계산
                if let properties = CGImageSourceCopyPropertiesAtIndex(source, i, nil) as? [String: Any],
                   let pngProperties = properties[kCGImagePropertyPNGDictionary as String] as? [String: Any] {
                    
                    let delayTime = pngProperties[kCGImagePropertyAPNGUnclampedDelayTime as String] as? Double ??
                    pngProperties[kCGImagePropertyAPNGDelayTime as String] as? Double ?? 0.1
                    
                    printSC("duration: \(duration), delayTime: \(delayTime)")

                    duration += delayTime
                }
            }
        }
        
        printSC("duration: \(duration)")

        return UIImage.animatedImage(with: images, duration: duration)
    }
}
