# 시간을 나타내는 Class
시간을 관리하다보면 마주하는 class가 3가지 있었다       
LocalDateTime, OffsetDateTime, zonedDateTime 이다

## LocalDateTime
`yyyy-MM-ddTHH:mm:ss.SSS`의 형태를 가지고 있음
> 2025-09-24T00:12:50.123

하위에는 Date와 Time이 존재하며 시차 정보가 별도로 존재하지 않는다      
따라서 다른 위치에 있는 서버에서 동시에 저장하게 될 경우 다른 시간으로 판명되고, 같은 시간으로 판단할 수 없다
> 나라별 시차 고려 등이 불가능

## OffsetDateTime
`yyyy-MM-ddTHH:mm:ss.SSSZ`의 형태를 갖는다
> 2025-09-24T00:12:50.123+09:00

하위에는 Date, Time 뿐만 아니라 offset 정보가 포함된다      
offset은 시차 정보로 LocalDateTime은 반영할 수 없던 것을 반영하게 되며, 시차를 계산할 수 있게 된다

이는 GMT를 기준으로 하는데, GMT는 Greenwich Mean Time으로 세계의 시간 표준이다.
> 런던의 그리니치 왕립 천문대를 기준으로 하는 시간, 경도가 0인 본초자오선을 기준으로 지구의 모든 시간대 계산

## ZonedDateTime
`yyyy-MM-ddTHH:mm:ss.SSSZ`의 형태를 갖는다
> 2025-09-24T12:20:00+09:00[Asia/Seoul]

OffsetDateTime과 비슷해 보이지만 ZonedDateTime에서는 지역 정보까지 포함된다

## Kotlin의 LocalDateTime
kotlinx에서도 LocalDateTime을 지원하고 있다

[공식 문서](https://kotlinlang.org/api/kotlinx-datetime/kotlinx-datetime/kotlinx.datetime/-local-date-time/-local-date-time.html)

KMM을 고려하는 프로젝트라면 Kotlinx의 LocalDateTime을 사용해야 한다     