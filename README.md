# Lambda_Image_Resizing

Lambda 함수를 이용한 이미지 썸네일 제작 서버리스 아키텍처를 실습해보려한다! 

## 1. Blue Print of Architecture 
![image](https://user-images.githubusercontent.com/88131652/227571382-92e184d2-6164-49ee-8ad1-71d87dd816fd.png)

## 2. Hands On 
### 2-1. S3 버킷생성 
![image](https://user-images.githubusercontent.com/88131652/227571902-9e6167fe-e2ba-4f3f-9c89-631e4f3d6332.png)

원본 이미지가 저장 될 버킷과 썸네일 이미지가 저장될 버킷을 생성한다. 

### 2-2. Lambda 함수 생성 
![image](https://user-images.githubusercontent.com/88131652/227572375-7ae1497e-21ad-4abe-9bd2-f1ef2c623bee.png)

람다함수를 생성한 후 IAM Role을 사용자와 연결해 준다. (IAM Role.Json 참조) 

### 2-3. 실패시 로그 저장 할 SQS 대기열을 생성 
![image](https://user-images.githubusercontent.com/88131652/227573673-67ec91ee-9f02-4537-9644-5388cb441e66.png)

실패시 로그를 메시지로 받을 SQS 대기열을 생성해준다 

### 2-4. Lambda 함수에 트리거 추가 
![image](https://user-images.githubusercontent.com/88131652/227573783-91922854-6010-4bbc-ad87-10ef690d7139.png)

- 원본 이미지 버킷에 객체가 업로드되면 Lambda 함수가 실행될 수 있도록 트리거를 설정
- 이벤트 유형은 모든 객체 생성 이벤트로 해준다 

### 2-5. SQS 대기열 대상 추가 
![image](https://user-images.githubusercontent.com/88131652/227574322-a60a1946-b68e-4a88-abfd-24fbe9947365.png)

실패 시 로그를 저장할 SQS 대기열을 Lambda 함수 대상에 추가해준다. 

### 2-6. Python PIL 모듈 레이어 추가 
![image](https://user-images.githubusercontent.com/88131652/227575711-8b0b07b0-cab2-42ba-beb7-1812e9c71b58.png)

**이 부분이 가장 힘들었다,,,, Lambda 함수 Python은 최소한 모듈만 가지고 있어서 따로 레이어 모듈을 추가해줘야 한다** 

로컬 Python --> Site Package에서 PIL 라이브러리를 복사하여 레이어에 추가해줄 경우,,, **PIL_Imaging can not import** 오류를 볼 것이다,,,, Lambda가 Pillow 패키지의 **Compiled Binary** 패키지를 인식하지 못해서 생기는 오류라고 한다. 

**https://api.klayers.cloud/api/v2/p3.9/layers/latest/ap-northeast-2/html** 

위 html에서 Pillow 패키지의 ARN을 복사해서 2-6 사진처럼 ARN 지정을 통해 레이어에 추가해준다

### 2-7. Lambda 코드 작성 
**Lambda_Image_Resize 코드 참조** 

### 2-8. API GateWay 생성 
![image](https://user-images.githubusercontent.com/88131652/227578424-153cc996-5ce8-4feb-9564-278f010d349b.png)

Rest로 API GateWay 생성 후 리소스{bucket}/{filename} 두 가지를 생성 후 Filename 아래 PUT 메서드 생성 후 2-8 사진과 같이 세팅해준다 

### 2-9. URL 경로 파라미터 설정 
![image](https://user-images.githubusercontent.com/88131652/227578803-e4b4c793-e84a-4aa4-8452-da85596e7ecf.png)

URL 경로 파라미터를 사진과 같이 추가해준다 

### 2-10. 이진 미디어 형식 추가 
![image](https://user-images.githubusercontent.com/88131652/227578911-b702f18c-7670-4279-805a-a427aec00250.png)

파일 포멧을 세부적으로 필터링 할 이진 미디어 형식을 추가해준다. 이 실습에서는 모든 포멧을 업로드 하는 */* 형식으로 설정해준다

### 2-11. API 배포 
![image](https://user-images.githubusercontent.com/88131652/227579150-23ad65cd-71e3-4e29-80a8-d4e8d462b4b6.png)

스테이지 이름을 설정해주어서 API를 배포한다 

### 2-12. PostMan을 통해 API에 PUT Request 테스트 
![image](https://user-images.githubusercontent.com/88131652/227579283-1f916545-4432-4ac3-82c2-2afdcfe2a863.png)

프론트 엔드가 없으므로 POSTMAN을 이용한다. 이때 URL 형식은 아래와 같다 
**API GateWay URL/스테이지명/버킷명/올릴파일이름.확장자** --> **Body의 Binary를 선택하고 파일을 업로드 해준다 

### 2-13. 원본 버킷 확인 
![image](https://user-images.githubusercontent.com/88131652/227579617-4cdf7332-6098-4d13-94fd-08127ff7ab01.png)

### 2-14. 썸네일 버킷 확인 
![image](https://user-images.githubusercontent.com/88131652/227579684-0586b419-8643-4c2a-83ab-8236383b30aa.png)

### 2-15. DynamoDB 테이블 확인 
![image](https://user-images.githubusercontent.com/88131652/227579805-bddf35fe-63ec-44da-b8c2-8570841cdb2b.png)

##3. 느낀점 

- 정말 많은 오류를 마주쳤다,, 특히 **Pillow** 오류는 진짜 몇일 동안 구글링 해서 찾아냈다! 
- 403 Deny : IAM 권한과 역할 설정을 다시 체크할 것! or Lambda 함수 환경 변수에 유효한 IAM Access Key와 Secret Key인지 확인 할 것 
- 404 NOT FOUND : 코드의 오타를 확인할 것 ,, ,, 
