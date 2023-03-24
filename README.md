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

