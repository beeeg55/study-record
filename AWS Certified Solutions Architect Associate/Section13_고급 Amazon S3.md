# Section13. 고급 Amazon S3

Date: June 10, 2023

스토리지 클래스 간 데이터 이동

- 자신보다 아래에 있는 모든 수준으로 이동 가능
- 객체에 자주 액세스
- 객체 옮기는 방법
    - 수동
    - 수명 주기 규칙(lifecycle rules)을 이용하여 자동화
        - 전환작업
            - 다른 스토리지 클래스로 객체를 전환하도록 구성
            - 예시 : 60일 후 standard ia 로 이동하도록 설정, 6개월 후에 glacier에 아카이빙 되도록 설정
        - 만료작업
            - 일정 기간 만료되면 객체를 삭제
            - 예시 : 365일 이후 액세스 로그 파일을 삭제, 버저닝을 활성화한 경우 이저버전의 파일을 삭제, 완료되지않은 멀티파트 업로드는 삭제
        
        → 규칙에는 특정 접두사를 사용하여 전체 버킷이나 버킷일부, 특정 객체에만 적용 가능
        

스토리지 클래스 분석

- Amazon s3 analytics 를 이용해서 결정
    - standard ia, standard에 대해 추천해줌(그외는 x)
    - 버킷에 관한 권장 사항과 통계를 csv로 제공해줌
    - 매일 업데이트
    - 분석 결과까지 24-48시간 소요
    - lifecycle rule을 세우거나 개선방향에 도움이 된다
    

요청자 지불

- 일반적으로는 버킷 소유자가 버킷과 관련된 모든 s3 스토리지 및 데이터 전송 비용을 지불
- 설정을 통해 요청자가 다운로드 비용을 담당하게 할 수 있다 (소유자가 스토리지 소유에 대한 비용은 담당, 네트워킹 비용을 요청자가 담당하는것임)
- 대량이 데이터넷을 다른 계정으로 이동시킬때 유용
- AWS에서 인증을 받은 요청자만이 가능 (요청자가 익명이어서는 안됨)

S3 이벤트 알림

- S3에서 발생하는 이벤트에 반응이 가능하다.
- s3 이벤트 예시 : 객체 생성, 삭제, 복원, 복제
- 사용 : sns, sqs, lambda함수, amazon eventbridge 등에 알림을 보낼때
    - amazon eventbridge
        - 모든 이벤트를 이곳에 알림보내며 이곳에 규칙을 설정하여 18개가 넘는 aws 서비스에 이벤트 알림 보내기 가능
        - 고급필터링 옵션 더 다양함(메가데이터, 객체 크기 등)
        - 여러 수신지에 보내기 가능
        - 이벤트 보관, 안정적 전송 등 추가기능 지원
- 원하는 만큼 이벤트 생성 가능
- 초대 1분까지 알림 가는데 걸림

s3 성능

- s3는 요청이 많으면 자동 확장(지연시간 100-200ms)
- 버킷내에서 prefix 당 초당 3500개의 put/copy/post/delete과 5500개의 get,head 요청을 지원함
- prefix수에 제한이없다

최적화 방법 

- 멀티파트 업로드
    - 추천 100mb이상
    - 필수 5GB이상
    - 업로드를 병렬화 하여 전송속도를 높여 대역폭을 최대화 할 수 있음
    - 다시 S3에서는 합쳐짐
- s3 전송 가속화
    - 업로드 및 다운로드를 위함
    - 파일을 자신과 가까운 aws엣지 로케이션으로 전송해서 전송속도를 높이고 데이터를 대상 리전에 있는 s3버킷으로 pirvate aws상에서 전달
        
        (엣지로케이션은 리전수 보다 높다)
        
    - 멀티파트 업로드와 같이 사용 가능
- S3 바이트 범위 패치
    - 다운로드 속도를 높이거나 부분만 가져오기 가능
    - 파일에서 특정 바이트 범위를 가져와서 get 요청을 병렬화
    - 특정 바이트 범위를 가져오는데 실패한 경우에도 더 작은바이트에서 재시도하여 복원력이 높음
    

S3 Select & Glacier Select

- S3에서 파일을 검색할때 , 검색한 후 필터링하면 너무많은 데이터를 검색하게됨
    
    → 서버 측에서 미리 필터링할수 있게 도와줌 (속도는 400%빨라지고, 비용은 80%줌)
    
- SQL문엣 행, 열을 사용하여 필터링
- 네트워크 전송이 줄어들어 데이터 검색과 필터링에 드는 클라이언트 측의 cpu비용 줄어듬
- 간단한 필터링의 경우 이것을 사용하는게 좋다

S3 Batch Operations

- 단일 요청으로 기존 s3객체에서 대량 작업을 수행하는 서비스
- 사용사례 - 객체목록에서 원하는 작업은 무엇이든 수행이 가능함
    - 한번에 S3객체의 메타데이터와 프로퍼티 수정 가능
    - 배치 작업으로 S3버킷간 객체 복사
    - S3 버킷내에 암호화되지않은 모든 객체를 한번에 암호화 가능
    - acl 및 태그 수정 가능
    - glacier에서 한번에 많은 객체 복원 가능
    - lambda함수를 호출해 S3 배치 오프레이션의 모든 객체에서 사용자 지정 작업 수행 가능
- 작업은 객체의 목록, 수행할 작업 옵션 매개변수로 구성
    - 객체목록은 S3 Inventory 라는 기능을 사용하여 가져오고 S3 SELECT를 사용해 객체를 필터링한다.
- 재시도를 관리 할 수 있고 진행상황을 추적하고 작업 완료 알림을 보내고 보고서 생성 등을 할 수 있다.