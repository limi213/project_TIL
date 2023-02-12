# :pushpin: batch & cron


프로젝트에서 특정 타입으로 사용자들의 랭킹을 구해야 한다.

이를 위해서 cal_log, rank 테이블을 만들었다.

cal_log는 

| cal_log_sq | user_id | rank_cd | value | start_dt | end_dt | reg_tm | mod_tm |
|---|---|---|---|---|---|---|---|
| PK | FK/RESTRICT | NN	| NN | NN |	NN | NN | | 	
|SMALLINT |	SMALLINT | SMALLINT |	SMALLINT | DATE |	DATE | TIMESTAMP | TIMESTAMP |
|AI | RankType |	|  |	 |	 | CURRENT_TIMESTAMP |  |

다음과 같이 설계되었고

rank 는

| rank_no | user_id | ranking | rank_cd | reg_tm | mod_tm |
|---|---|---|---|---|---|
| PK | FK/RESTRICT | NN	| NN | NN | | 	
|SMALLINT |	SMALLINT | SMALLINT |	SMALLINT | TIMESTAMP | TIMESTAMP |
|AI |  | 1~100	| RankType | CURRENT_TIMESTAMP |  |

다음과 같이 설계되었다.

cal_log에는 매일매일 하루 누적, 일주일 누적의 값들이 랭킹 타입마다 쌓이게 된다.

rank는 매일 자정 00:00에 cal_log 중 각 타입의 100위까지의 데이터를 가져온다.

이를 위해서 배치 파일을 작성하고 우분투 서버에서 cron을 사용하여 스케줄러를 작동하도록 하였다.


## :triangular_flag_on_post: batch 파일 작성

batch는 일괄처리를 위해서 묶는 것으로 명령어 집합 파일이라고 생각하면 된다.

프로젝트에서는 cal_log에 새로운 데이터를 삽입하고, rank에 데이터를 삭제 후 삽입하는 과정을 mysql script로 작성한 파일을 실행해야 한다.

따라서 아래 그림 내용과 같이 작성해주었다.

![배치](https://user-images.githubusercontent.com/81286029/218315118-b9ee777d-8dac-4833-9f5d-f7792c2b1e53.png)

사용자와 암호, 데이터베이스를 설정한 후 scipt를 실행하면 된다.


## :truck: cron 작성

cron은 리눅스 계열에서 제공하는 시간 기반 잡 스케줄러이다.

따라서 개발자가 신경쓰지 않고 프로그램이 알아서 실행할 수 있도록 설정할 수 있다.

```
crontab -e
```
명령어를 통해 크론을 생성할 수 있다.

![cron](https://user-images.githubusercontent.com/81286029/218315283-78630b57-5b87-4665-9ad2-a91751049f0b.png)

다음과 같이 vi 에디터를 사용하여 내용을 작성하면 된다.

이때 crond를 설정하는 규칙이 있다.

> 분 시 일 월 요일 명령어

순으로 작성하면 되고 * 를 사용하면 매번 반복을 의미한다.

매일 00:00분에 실행해야 하기 때문에 0 0 * * * 로 설정하였고 위에서 작성한 batch 파일을 실행하도록 했다.

만일의 경우를 대비하여 로그를 작성하였다.





