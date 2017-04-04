## 코드 디지털 사인의 이유

파일 전달과정에서 조작을 막기 위함이다.

## xLua의 디지털사인기능 사용

* Tools/KeyPairsGen.exe를 통해 공개키 및 개인키를 생성한다. key_ras파일에 개인키, key_ras.pub에 공개키가 저장된다. 이 두 파일은 잘 관리해야 한다. 개인키는 게임의 보안성에 연관되므로 암호화처리를 잘 해야한다.
* Tools/FilesSignature.exe으로 소스코드에 디지털사인을 한다.
  * key_ras파일을 실행디렉토리에 둔다.
  * 인자는 소스디렉토리와 타겟디렉토리이다. 이 툴은 자동으로 소스디렉토리의 아래의 모든 .lua파일에 디지털사인을 하여 타겟디렉토리에 트리구조와 함께 저장한다.
* SignatureLoader를 통해 CustomLoader로 포장 후 사용한다.
  * SignatureLoader의 constructor는 두개의 인자를 받는다. 하나는 공개키, 다른 하나는 원래의 Loader이다.
