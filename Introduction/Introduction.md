#Introduction

OpenCV(오픈 소스 컴퓨터 비전 라이브러리: http://opencv.org)는 수 백가지의 컴퓨터 비전 알고리즘을 포함하고 있는 오픈 소스 BSD-라이센스 라이브러리이다. 이 문서는 C 베이스인 OpenCV 1.x API의 반대로서 기본적으로 C++ API인 OpenCV 2.x API를 설명합니다. OpenCV 1.x API는 마지막에 opencv1x.pdf에서 설명한다.
OpenCV는 모듈형 구조를 가지고 있으며, 각각의 공유되거나 정적인 라이브러리를 포함하고 있는 패키지를 의미한다.

 * Core 기능: 기본적인 데이터 구조를 정의하는 소형. 다차원 배열인 Mat을 포함하며 기본적인 함수들은 다른 모든 모듈에 의해 사용된다.

 * Image processing: 선형 및 비선형 이미지 필터링을 포함하는 이미지 처리 모듈. 기하학적인 이미지 변화(리사이즈, 아핀 및 관점 왜곡, 테이블에 기반한 리매핑을 총칭한다), 색 공간 전환, 히스토그램 등

 * Video: 움직임 판단, 배경 빼냄, 객체 추적 알고리즘을 포함하는 비디오 분석 모듈.

 * Calib3d: 기본적인 multiple-view 기하학 알고리즘, 단일 및 스트레오 카메라 눈금화, 객체 포즈 추정, 스트레오 대응 알고리즘, 3D 재건의 요소.
  (multiple-view  위, 아래, 옆에서 봤을 때를 말하는 것 같다)
  
 * feature2d: 가장 두드러진 특징 탐지기, descriptors and descriptor matchers 

 * Objdetect: 미리 정의된 클래스들의 인스터스와 객체들을 검출.

 * Highgui: 간단한 UI 기능을 사용하기 쉬운 인터페이스

 * Video I/O: 비디오 캡처와 비디오 코덱에 사용하기 쉬운 인터페이스

 * Gpu: 다른 OpenCV 모듈들로부터 가속화 gpu 알고리즘

 * FLANN 그리고 Google test wrappers, 파이썬 바인딩 등과 같은 다른 도우미 모듈


문서의 상기 챕터는 각 모듈의 기능을 설명. 하지만 먼저, 반드시 라이브러리 안에서 철처히 사용되는 일반적인 API 개념들에 대해 익숙해져야 한다.


##API Concepts
###cv Namespace

모든 OpenCV 클래스들과 함수들은 cv namespace 안에 들어있습니다. 
그러므로 당신의 코드에서 이 기능에 접근하려면 cv::specifier 이나 using namespace cv 를 이용해야 합니다. 
   ```
  #include "opencv2/core.hpp"
  ...
  cv::Mat H = cv::findHomography(points1, points2, CV_RANSAC, 5);
  ...
```
   
또는
```
  #include "opencv2/core.hpp"
  using namespace cv;
  ...
  Mat H = findHomography(points1, points2, CV_RANSAC, 5 );
  ...
```
현재 또는 미래의 OpenCV 외부 이름은 STL이나 다른 라이브러리들과 충돌이 있을 수 있다. 이런 경우에는 명시적 네임 스페이스를 사용하여 이름 충돌을 해결한다.

```
  Mat a(100, 100, CV_32F);
  randu(a, Scalar::all(1), Scalar::all(std::rand()));
  cv::log(a, a);
  a /= std::log(2.);
```


###Automatic Memory Management
OpenCV는 자동적으로 모든 메모리를 처리한다.


가장 먼저, std::vector, Mat, 그리고 함수들과 메소드에 의해 사용되는 다른 데이터 구조들은 필요할 때 기본 메모리 버퍼들을 할당 해제하는 소멸자가 있다. 이 뜻은 Mat의 경우 항상 소멸자가 할당된 버퍼를 해제하지 않는 것을 의미한다. 데이터를 공유하는 것이 가능하지 고려한다. 소멸자는 메트릭스 데이터 버퍼에 관련된 참조 카운터를 소멸시킨다. 이 버퍼가 할당이 해제된다면 참조 카운터를 0에 이르게 한다. 말하자면 다른 구조체들이 같은 버퍼에 참조하지 않는 경우를 말한다. 유사하게, Mat 복사할 경우, 실제 데이터가 복사되지 않는다. 대신에, 동일 데이터에 다른 소유가 있다는 것을 기억하기 위해 참조 카운터를 증가시킨다. 또한 메트릭스 데이터의 전체를 복사하는 Mat::clone 메소드도 있다. 아래의 예를 보자.
```
  // 8MB의 커다란 메트릭스를 생성한다.
  Mat A(1000, 1000, CV_64F);
     
  // 같은 메트릭스에 대해 다른 헤더를 생성한다.
  // 메트릭스 사이즈에 상관하지 않는 즉각적인 동작이다.
  Mat B = A;
  // A의 3번 째 행에 다른 헤더를 생성시킨다; 어떠한 데이터도 복사하지 않는다.
  Mat C = B.row(3);
  // 이제 메트릭스의 별도의 복사본을 생성한다.
  Mat D = B.clone();
  // B의 5번 째 행을 C에 복사한다. 말하자면 A의 5번 째 행을 A의 3번 째의 행으로 복사시킨다.
  B.row(5).copyTo(C);
  // 이제 A와 B의 데이터를 공유시킨다. 그 후 A의 수정된 버전은 B와 C에 여전히 참조되있다.
  A = D;
  //  이제 빈 메트릭스 B를 만들자(메모리 버퍼에는 참조 되어 있지 않다),
  //  C는 단지 원래의 A에 한 행임에도 불구하고 A의 수정된 버전은 여전히 C를 참조한다.
  B.release();
    
  // 마침내, 전체적으로 C를 복사한다. 결과적으로 크게 수정되었다.
  // 메트릭스는 누가 참조하지 않았기 때문에 할당이 해제될 것이다.
  C = C.clone();
```
Mat과 다른 기본 구조물들이 간단하게 사용할 수 있다는 것을 알 수 있다. 그러나 자동 메모리 관리를 고려하지 않고 고급 클래스들과 사용자 타입들을 생성하는 것은 어떨까요? 그들에게 있어 openCV는  C++ 11에서 std::shared_ptr 과 유사한 ptr 템플릿 클래스를 제공한다. 즉,  plain pointer를 사용하는 대신에 :
```
  T* ptr = new T(...);
```  
너는 사용할 수 있다 :
```  
  Ptr<T> ptr(new T(...));
```
또
  ```
  Ptr<T> ptr = makePtr<T>(...);
```

ptr<T>는 포인터와 연관된 T 인스턴스와 참조 카운터를 캡슐화한다.


##Automatic Allocation of the Output Data

OpenCV는 자동으로 메모리 할당된 것을 해제하며 또한 대부분의 시간동안 output 함수의 매개 변수에 대한 메모리를 자동으로 할당한다. 그리고 만약 함수에 하나 이상의 input 배열들(cv::Mat instances)와 output 배열들이 있을 경우, output 배열들은 자동적으로 할당하거나 재할당된다. output 배열들의 크기나 타입은 input 배열들의 크기와 타입에 따라 결졍된다. 만약 필요로 한다면, 함수는 output 배열을 파악하는데 도움을 얻기 위해서 추가적으로 매개 변수를 필요로 한다.

```
  #include "opencv2/imgproc.hpp"
  #include "opencv2/highgui.hpp"
  
  using namespace cv;
   
  int main(int, char**)  
  {
    VideoCapture cap(0);
    if(!cap.isOpened()) return -1;
      
    Mat frame, edges;
    namedWindow("edges",1);
    for(;;)
    {
        cap >> frame;
        cvtColor(frame, edges, COLOR_BGR2GRAY);
        GaussianBlur(edges, edges, Size(7,7), 1.5, 1.5);
        Canny(edges, edges, 0, 30, 3);
        imshow("edges", edges);
        if(waitKey(30) >= 0) break;  
    }
    return 0;
 }
```