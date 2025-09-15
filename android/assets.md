# Assets 폴더
Assets는 주로 `app/src/main`위치에 존재한다     
애플리케이션 배포 시 패키지에 포함시킬 리소스를 저장하는 폴더이다       
`\res` 폴더에 저장된 리소스와의 차이점을 폴더의 이름과 같이 font 혹은 이미지처럼 저작권이 있는 파일을 관리하는 폴더이다     

그리고 res 폴더는 R 클래스로 접근 가능하지만 assets는 Context 내에 있는 AssetManager를 통해서만 접근이 가능하다