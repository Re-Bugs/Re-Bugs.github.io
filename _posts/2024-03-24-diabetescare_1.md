---
layout: single
title:  "처방전 인식을 위한 OCR API"
categories: diabetes_care
redirect_from:
  - /diabetes_care
---
# OCR 기능이란?
OCR은 '광학 문자 인식'(Optical Character Recognition)의 약자이다.<br>
이 기술은 이미지에서 글자를 인식하여 편집 가능한 텍스트로 변환하는 과정을 말한다.<br>
예를 들어, 스캔된 문서나 사진 속의 텍스트를 디지털 형태로 변환할 때 사용된다.<br>
<br>
OCR 기능을 직접 구현할 능력과 시간이 부족했기에 **<u>Google Cloud Platform에서 제공하는 OCR API를 사용하기로 결정</u>**하였다.

<br>

**<u>이를 이용하면 아래와 같은 기능을 구현할 수 있다.</u>** 

1. 식사 후 약을 복용해야할 때, 약을 먹어야 한다는 리마인더 기능

2. 약물이 모두 소진되기 이전에, 투약 일수를 기반으로 다시 처방받아야 함을 사용자에게 알림

<br>

# Google Cloud Platform OCR API

![GCP_OCR]({{site.url}}/images/2024-03-23-r2r_1/GCP_OCR.png)

Google Cloud Platform의 OCR API는 Google Cloud Vision API의 일부로 제공된다. 

이 API는 이미지 내의 텍스트를 강력하고 정확하게 인식할 수 있게 해주며, 문서 스캔과 이미지 기반 데이터 추출에 매우 유용하다.<br>

이에 대한 Google Cloud 설정과 스프링 부트에서 간단히 적용하는 방법은 [여기](https://rebugs.tistory.com/638)에 정리해 두었다.<br>

<br>

이 프로젝트에선 처방전 사진을 인식해야하기 때문에 처방전을 찍어 테스트하였다.<br>

<br>



# OCR 테스트 결과

![OCR_TEST]({{site.url}}/images/2024-03-24-diabetescare_1/OCR_TEST.png)

(처방전 전체를 촬영하였으나, 개인정보를 위해 일부를 잘라냈음.)<br>

위 처방전 이미지에서의 텍스트를 추출한 내용은 아래와 같다.

```tex
※환자가 요구하면 질병분류기호를 적지 않습니다.
처방 의약품의 명칭 및 코드
울메택정 10㎎
(641601990)
에보프림연질캡술 40mg
(664600720)
에나폰정 5(
(657202640)
'포시가정 10mg
(650700840)
리피토정 20mg
(073400330)
<1회 투약량은 청구단위 입니다>
1회
1일
여
투약량
총
투약일수 구분코드
본인
2
부담률
용
법
1.0000
60
2.0000
2
60
(하루1회) 아침식후 30분에
D
(하루 2회) 아침/저녁 식후 30분에
B
1.0000
30
(하루1회) 저녁 식후 30분에
DE
1.0000 1
60
60
(하루1회) 아침식후 30분에
D
1.0000
①
09
60
(하루1회) 아침식후 30분에
D
조제시
본인부담
주사제 처방명세(원내조제□, 원외처방)
참고사항
구분기호
1.0000 1
6
집에서 투약하세요.
투제오주솔로스타 300단위/mL
(652000951)
1.0000 1
8
집에서 투약하세요.
```



위의 텍스트를 기반으로 **<u>정규 표현식으로 원하는 데이터를 추출</u>**하였다.

결과는 아래와 같다.

![OCR_TEST3]({{site.url}}/images/2024-03-24-diabetescare_1/OCR_TEST3.png)



아래의 코드는 약품 코드를 정규 표현식으로 추출하는 메서드이다.

```java
//약품 코드를 추출하는 메서드
private List<String> extractNineDigitNumbers(String input) {
    List<String> numbers = new ArrayList<>();
    // 정규 표현식: '(' 뒤에 오는 9자리 숫자를 ')' 전까지 찾음
    Pattern pattern = Pattern.compile("\\((\\d{9})\\)");
    Matcher matcher = pattern.matcher(input);

    // 매칭되는 모든 숫자를 찾아 리스트에 추가
    while (matcher.find()) {
        numbers.add(matcher.group(1));
    }

    return numbers;
}
```



이러한 방식으로 약품 코드, 하루 중 복용 시간대, 복용 횟수, 투약 일수를 추출할 수 있었다.<br>

