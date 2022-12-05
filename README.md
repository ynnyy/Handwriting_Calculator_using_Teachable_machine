# Handwriting Calculator_using machine learning✍️


![header](https://capsule-render.vercel.app/api?type=waving&color=ADD8E6&height=300&section=header&text=Handwriting%20calculator&desc=using%20machine%20learning&fontSize=50&demo=wave&fontColor=696969)

# :pushpin:Project Description

Teachable machine을 이용해 사용자가 필기체로 입력한 수식을 인식하여 계산해주는 계산기 프로그램이다.
--------------------------------------------------------
# :pushpin:Project video

https://user-images.githubusercontent.com/105347300/205647125-74062a5f-0be9-419a-8472-40762602ea31.mp4

youtube : https://www.youtube.com/watch?v=JaJqFwNpuyE﻿

---------------------------------------------------------
# :pushpin:Project function

- GUI : 사용자와 상호작용하는 계산기 화면 기능  
- 문자인식 : 18개의 문자를 인식하는 기능  
 0,1,2,3,4,5,6,7,8,9,(,A,N,S,+,-,x,/  
- 계산기 기능 : 계산(=) , ANS , CE , AC기능  
![image](https://user-images.githubusercontent.com/105347300/205648138-54bd5977-ac56-44d4-9db3-47a5f13a4ada.png)

# :pushpin:Project Block diagram

![image](https://user-images.githubusercontent.com/105347300/205648749-8a029d2f-93d6-4fdd-b4c7-5b9a26d010ba.png)
----------------------------------------------------------

# :pushpin:Project algorithm
## 문자학습 
- Teachable Machine -google 사용
---------------------------
## 모델 파일 변환
- google colab 에서 상기의 convert.ipynb파일 실행 후 openCV용 모델 파일로 변환 
----------------------------
## 필기체 입력 기능 
- 사용자의 마우스 입력의 마지막 좌표를 저장해 현재 좌표와 라인을 이어서 실시간으로 그리는 함수
- ![image](https://user-images.githubusercontent.com/105347300/205650161-ad97a274-67c8-42c1-aea4-bce1ae2670c5.png)
-         else if (event == EVENT_MOUSEMOVE) { //필기체 입력 그리기
            if (flags & EVENT_FLAG_LBUTTON) {
                if (x < 1300 && y >= 100) { //구간 설정
                    line(img, ptOld, Point(x, y), Scalar(255), 10); //그리기
                    imshow("img", img); //영상 출력
                    ptOld = Point(x, y); //마지막 좌표 저장

                }
            }
        }
---------------------------
## =(등호)버튼 (객체 레이블링 기능)
- 사용자의 필기체 문자열입력 구간을 레이블링해 객체를 찾아내고, 객체의 좌표를 벡터에 저장하는 함수
- ![image](https://user-images.githubusercontent.com/105347300/205651154-f45aefa9-1030-4f10-b05e-34bd2a964165.png)
-     void Object_Recognition(Mat img, vector<Rect>& r) {   //필기체 입력구간 객체 인식, 레이블링 하는 함수
        Mat labels, stats, centroids; //객체 레이블링에 필요한 변수 생성
        int cnt = connectedComponentsWithStats(img, labels, stats, centroids);// 객체 레이블링

        for (int i = 0; i < cnt; i++)      //바운딩 박스 정보 vector<Rect>r에 저장
        {
            int* p = stats.ptr<int>(i); //i행 단위 정보 추출
            r.push_back(Rect(p[0]-30, p[1]-30, p[2]+60, p[3]+60)); // 바운딩 박스 정보에 여백을 포함한 크기를 벡터에 저장
        }

        Rect tmp; // 레이블링된 객체를 벡터r에 저장된 순서를 바꾸기위한 빈 객체 생성
        for (int i = 1; i < r.size(); i++)      // 벡터 r의 저장된 순서를 x좌표가 작은순부터 저장되도록 정렬
        {
            for (int j = 0; j < r.size() - i; j++) 
            {       // x좌표가 더 작은 객체가 작은 인덱스를 가지도록 설정
                if (r[j].x > r[j + 1].x) {// 객체의 x좌표 비교
                    tmp = r[j]; //x좌표가 큰 객체를대입
                    r[j] = r[j + 1]; //x좌표가 더 작은 객체 대입해 위치 변경
                    r[j + 1] = tmp; //대입
                }
            }
        }

    }
--------------------------
 ## 문자 인식 기능
 - opencv용 모델파일을 불러와 블롭객체에 넣고, 이 블롭 객체를 그대로 네트워크 입력으로 설정하고, 순방향으로 실행해 예측 결과 행렬을 얻어 문자 추론  
 - ![image](https://user-images.githubusercontent.com/105347300/205651819-8fdf0d52-da90-49e6-a7d7-fdd95da8e5d0.png)

-     void tm_machine(Mat dst, vector<Rect>& r) //문자인식 기능 함수
    {
        for (int i = 1; i < r.size(); i++) //객체 인식
        {
            if (r[i].width < 100 && r[i].height < 100) { //객체 높이와 넓이가 100이하이면 소수점 처리
                message += "."; // 예측결과 넣기
                cout << "예측 결과 : 소수점 ." << endl;
            }
            else { //소수점이 아니면 문자 인식 수행
                Mat img =dst(r[i]); //레이블링된 객체 이미지 추출

                vector<String> classNames = { "0","1","2","3","4","5","6","7","8","9","/","+","-","x","(","A","N","S" }; //클래스 이름 
                // Load network
                Net net = readNet("frozen_model.pb"); //모델 파일 불러오기
                if (net.empty()) { cerr << "Network load failed!" << endl; } //에러처리
                // Load an image
                if (img.empty()) { //에러처리
                    cerr << "Image load failed!" << endl;
                }
                // Inference
                Mat predict; //예측 이미지
                cvtColor(img, predict, COLOR_GRAY2RGB); //채널변경

                Mat inputBlob = blobFromImage(predict, 1 / 127.5, Size(224, 224), -1, 0); //블롭 객체 생성
                net.setInput(inputBlob);//네트워크 입력으로 설정
                Mat prob = net.forward(); //네트워크를 실행
                // Check results & Display

                double maxVal; // 최대값 저장할 변수
                Point maxLoc; //최대값 위치 저장할 변수
                minMaxLoc(prob, NULL, &maxVal, NULL, &maxLoc);
                if (classNames[maxLoc.x] == "A" || classNames[maxLoc.x] == "N" || classNames[maxLoc.x] == "S") { //문자처리
                    if (classNames[maxLoc.x] == "S") { //s가 인식되면 
                        message += to_string(ans); //ans값 대입
                    }
                }
                else if (classNames[maxLoc.x] == "(") { //괄호 처리
                    if (cnt == 0) { // 여는 괄호
                        message += classNames[maxLoc.x]; // ( 넣기!!!!!!!!!
                        cnt = 1;
                    }
                    else if (cnt == 1) {//닫는 괄호이면
                        message += ")"; // ( 넣기!!!!!!!!!
                        cnt = 0;
                    }
                }
                else {
                    message += classNames[maxLoc.x]; // 예측결과 넣기!!!!!!!!!
                }
                String str = classNames[maxLoc.x] + format("(%4.2lf%%)", maxVal *
                    100);
                cout << "예측 결과" << str << endl;//예측 결과 출력
                
                img = Scalar(0, 0, 0); //창 지우기

            }

        }

    }
-----------------------------
## 계산 기능
- 문자 연산자에따른 두개의 값을 연산하는 함수
- ![image](https://user-images.githubusercontent.com/105347300/205653707-e1d2d47b-1852-40a6-829e-a562af4124ec.png)
- 문자열을 후위 표기법으로 만들기 기능 함수
- ![image](https://user-images.githubusercontent.com/105347300/205653749-f4c4f497-2e65-4b65-a800-671565eb94bc.png)
- 후위표기법을 계산하는 함수
- ![image](https://user-images.githubusercontent.com/105347300/205653770-695d5fd3-6d6e-471e-99bf-9a12bb0b0734.png)
