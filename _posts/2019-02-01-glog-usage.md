---
title: glog 사용
published: true
categories: cpp
tags: c++ glog
---

#### 환경 
- 테스트는 다음 OS에서 진행되었다.  
- Linux RHEL-6.9-01 2.6.32-696.el6.x86_64 

#### 설치방법

1. gflags
    - gflags 와 연동하려는 경우 미리 설치해야 한다. (연동하는 편이 더 편하다.)
    - 설치 순서
        > [cmake 설치]   
        > a. https://cmake.org/download 에서 최신 파일 다운로드  
        > b. 압축해제 tar -zxvf cmake-3.13.3.tar.gz  
        > c. cd cmake-3.13.3  
        > d. ./bootstrap  
        > e. make  
        > f. make install 

        > [gflags 설치]  
        > a. git clone https://github.com/gflags/gflags.git  
        > b. cd gflags  
        > c. mkdir build && cd build  
        > d. ccmake ..  
        > e. option 설정 화면이 나오는데 그냥 디폴트로 사용하면 된다.    
        > f. make  
        > g. make test  
        > f. make install 

2. glog 
    - 설치 순서  
        > [automake, autoconf, libtool 설치]  
        > 다음 파일들을 설치한다. (버전은 상황에 맞게 한다.)  
        > automake-1.11.1-4.el6.noarch  
        > autoconf-2.63-5.1.el6.noarch  
        > libtool-2.2.6-15.5.el6.x86_64

        > [glog 설치]  
        > git clone https://github.com/google/glog.git  
        > cd glog  
        > ./autoget.sh && ./configure && make && make install 

#### 로그 생성 

1. 파일 명명 규칙  
    > 로그 파일은 설정된 디렉토리에 다음과 같이 생성된다.  
    > [program name].[host name].[user name].log.[severity level].[date]-[time].[pid]  

2. 파일 종류 
    > 각 severity level 별로 파일이 생성된다.  
    > 따라서 4가지 파일이 생성된다.  

    ```bash
    # pgm name = glog_1, host name = RHEL-6.9-01, user name = huefax 인 경우
    # 4가지 파일이 생성되며 각각에 대하여 심볼도 생성된다. 
    glog_1.RHEL-6.9-01.huefax.log.ERROR.20190131-163233.17423
    glog_1.RHEL-6.9-01.huefax.log.FATAL.20190131-163239.17423
    glog_1.RHEL-6.9-01.huefax.log.INFO.20190131-163233.17423
    glog_1.RHEL-6.9-01.huefax.log.WARNING.20190131-163233.17423
    glog_1.ERROR -> glog_1.RHEL-6.9-01.huefax.log.ERROR.20190131-163233.17423
    glog_1.FATAL -> glog_1.RHEL-6.9-01.huefax.log.FATAL.20190131-163239.17423
    glog_1.INFO -> glog_1.RHEL-6.9-01.huefax.log.INFO.20190131-163233.17423
    glog_1.WARNING -> glog_1.RHEL-6.9-01.huefax.log.WARNING.20190131-163233.17423
    ```
3. 파일 내용
    > 각 파일은 자신의 파일보다 같거나 높은 레벨의 로그를 남긴다.   
    > INFO : INFO, WARNING, ERROR, FATAL  
    > WARNING : WARNING, ERROR, FATAL  
    > ERROR : ERROR, FATAL  
    > FATAL : FATAL
    > 디폴트로 ERROR, FATAL은 stderr 에도 출력한다. 

    > 로그 형식은 다음과 같다.  
    > [severity level(IWEF)][mmdd] [hh:mm:ss.uuuuuu] [thread-id] [file:line]] [msg]  
    > I0131 16:32:37.123123 17321 glog_1.cpp:80] Hello info log every 10 times  
    > E0131 16:32:37.464644 17423 glog_1.cpp:89] Hello error log

#### 사용 방법
1. 시작  

    ```cpp
    //! gflags, glog 헤더 파일 
    #include "gflags/gflags.h"
    #include "glog/logging.h"

    int main(int argc, char* argv[]) {

        //! glog 초기화
        google::InitGoogleLogging(argv[0]);

        //! 파라미터로 받은 flag 파싱
        google::ParseCommandLineFlags(&argc, &argv, true);

        //! 로그 기록
        LOG(INFO) << "Hello glog";
    }
    ```

2. 로그 레벨 (Severity Level)
    > INFO, WARNING, ERROR, FATAL (오른쪽으로 갈 수록 증가한다.)  
    > : INFO(0) < WARNING(1) < ERROR(2) < FATAL(3)  
    > FATAL은 로그 기록 후 프로그램을 종료시킨다.  
    > 디폴트로 ERROR, FATAL 은 stderr 에도 기록한다. 

3. Flags 설정
    > gflags 가 설치되어 있다면 설치시 (configure script 실행) 인식한다.  
    > gflags가 없다면 다음과 같이 옵션 설정을 하면 된다.  

    ```bash
    # gflags가 설치되어 있는 경우
    ./application --logstderr=1 

    # gflags가 미설치되어 있는 경우 (환경변수 설정 후 프로세스 실행)
    GLOG_logstderr=1 ./application
    ```

    option명 | type | default | 설명
    :--- | :--- | :--- | --- |
    logtostderr | bool | false | 로그를 stderr로 보낸다.
    stderrthreshold | int | 2 (ERROR) | level >= this 이면 추가로 stderr로도 보낸다.
    minloglevel | int | 0 (INFO) | level >= this 이면 기록한다. 
    log_dir | string | DefaultLogDir() | 로그 디렉토리 
    v | int | 0 | VLOG(m)에서 m <= this 이면 기록한다. --vmodule에 의해 덮어씌워질 수 있다. 
    vmodule | string | "" | 모듈별 verbose level을 나타낸다.
    - | - | - | ex) --vmodule=gfx*=1
    - | - | - | gfx로 시작하는 이름을 가진 모듈은 모두 verbose level을 1로 설정한다.
    - | - | - | ,로 구분해서 여러 개를 한꺼번에 쓸 수 있다. 
    alsologtostderr | bool | false | 로그를 stderr에도 보낸다.
    colorlogtostderr | bool | false | stderr에는 컬러로 보여준다. (터미널이 지원한다면) 
    log_prefix | bool | false | 로그 라인에 각각 prefix를 붙여준다. 단, DLOG, VLOG는 제외한다. 
    logbuflevel | int32 | 0 | level <= this 이면 buffering 하고 level > this 이면 바로 flush 한다. -1 이면 버퍼링하지 않는다. 
    logbufsecs | int32 | 30 | 로그를 buffering할 최대 시간 (초)
    logfile_mode | int32 | 0644 | 로그파일 모드/권한
    log_link | string | "" | 로그 파일과 링크를 추가로 저장할 디렉토리 -> 무조건 현재 디렉토리로만 되어서 사용하기는 어려울 것 같다.
    max_log_size | int32 | 1800 | 단위는 MB이며 초과할 경우 파일을 새로 만들어 준다.
    stop_logging_if_full_disk | bool | false | 디스크가 full이면 디스크로 로그 쓰기를 중지한다. 
    alsologtoemail | string | "" | 로그를 입력된 메일 주소로도 전송한다.
    logemaillevel | int32 | 999 | level >= this 인 로그는 메일로 전송한다.
    logmailer | string | "/bin/mail" |  메일 전송에 사용할 메일 프로그램을 설정한다.

    > flags를 코드 상에서 접근하기 위해서는 FLAGS_* 형식의 변수명을 사용하면 된다.  
    > 대부분의 flags는 설정 즉시 반영이 되지만 FLAGS_log_dir은 google::InitGoogleLogging 호출 전에 설정되어야 한다.

4. 로그 함수  
    로그 함수는 predefined macro를 사용한다.  
    ```cpp
    //! LOG(N) severity가 N인 로그를 기록한다.
    //! LOG_IF(N, condition) condition이 참인 경우 severity가 N인 로그를 기록한다.
    //! LOG_EVERY_N(N, M) M 번째 출력시 severity가 N인 로그를 기록한다. 
    //! LOG_IF_EVERY_N(N, condition, M) condition 이 최초 발생시 또는 이후 M번째 발생시 severity가 N인 로그를 출력한다. 
    //! 다른 종류의 로그도 대부분 이런 형식을 따른다. 
    LOG(INFO) << "Hello INFO";
    LOG_IF(INFO, count % 10 == 0) << "Hello LOG_IF";
    LOG_EVERY_N(INFO, 10) << "Hello counter is " << google::COUNTER;
    LOG_IF_EVERY_N(INFO, count % 10 == 0, 2) << "Hello counter 2 is " << google::COUNTER;

    //! DEBUG 모드인 경우 사용할 수 있는 macro 
    //! DLOG, DLOG_IF, DLOG_EVERY_N, DLOG_IF_EVERY_N
    //! 컴파일시 -D NDEBUG 를 주거나 #define NDEBUG를 해주면 출력되지 않는다.
    DLOG(INFO) << "Hello DLOG";
    DLOG_IF(INFO, count % 10 == 0) << "Hello DLOG_IF";
    DLOG_EVERY_N(INFO, 10) << "Hello counter is " << google::COUNTER;
    DLOG_IF_EVERY_N(INFO, count % 10 == 0, 2) << "Hello counter 2 is " << google::COUNTER; 

    //! VLOG, VLOG_IF, VLOG_EVERY_N, VLOG_IF_EVERY_N
    //! --v, --vmodule 로 설정된 레벨 보다 <= 이면 출력된다.
    //! #define VLOG(verboselevel) LOG_IF(INFO, VLOG_IS_ON(verboselevel))
    //! VLOG는 로그 레벨이 INFO 이다.
    VLOG(1) << "Hello VLOG";
    VLOG_IF(1, count % 10 == 0) << "Hello VLOG_IF";
    VLOG_EVERY_N(1, 10) << "Hello counter is " << google::COUNTER;

    //! RAW_LOG 
    //! stderr로만 출력됨 (락을 걸거나 메모리에 저장하지 않는다.)
    //! 따라서 stderr로 출력되는 조건이 되야만 출력됨 (아래 조건)
    //! FLAGS_logtostderr || severity >= FLAGS_stderrthreshold ||
    //! FLAGS_alsologtostderr || !IsGoogleLoggingInitialized()
    RAW_LOG(ERROR, "Failed foo with %i: %s", status, error);

    //! RAW_DLOG
    //! RAW_LOG와 같으며 NDEBUG이 정의되어 있는 경우만 실행된다.
    RAW_DLOG(ERROR, "Failed foo with %i: %s", status, error);

    //! RAW_VLOG
    //! RAW_LOG와 같으며 단지 INFO 레벨이고 --v, --vmodule로 설정된 verboselevel 기준으로 처리된다. 
    RAW_VLOG(1, "INFO LOG %d", error);

    //! CHECK, CHECK_NE, CHECK_EQ, CHECK_LE, CHECK_LT, CHECK_GE, CHECK_GT, CHECK_NONULL
    //! condtion을 체크하여 원하는 condition이 아니면 FATAL 로그를 출력한다.
    CHECK(fp->Write(x) == 4) << "Write failed";
    CHECK_NE(1, 2) << "The world must be ending!";
    CHECK_EQ(string("abc")[1], 'b') << "How?";
    CHECK_EQ(ptr, NULL);
    CHECK_NOTNULL(ptr);

    //! String 관련 macro
    //! CHECK_STREQ, CHECK_STRNE, CHECK_STRCASEEQ, CHECK_STRCASENE
    //! null, null are equal
    //! not null, null are not equal

    //! 기타 CHECK macro
    //! CHECK_INDEX(I, A) I가 유효한 A의 인덱스인지 여부 확인 <
    //! CHECK_BOUND(B, A) B가 유효한 A의 바운더리인지 여부 확인 <=
    //! CHECK_DOUBLE_EQ 
    //! CHECK_NEAR(val1, val2, margin) val1과 val2가 margin 이내에 있는지 여부 확인

    //! PLOG, PLOG_IF, PCHECK 
    //! LOG*, CHECK와 동일한 기능을 가진다. 단, 원인 로그가 추가된다.
    PCHECK(write(1, NULL, 2) >= 0) << "Write NULL failed";
    //! 출력 : Check failed: write(1, NULL, 2) >= 0 Write NULL failed: Bad address [14]

    //! SYSLOG, SYSLOG_IF, SYSLOG_EVERY_N 
    //! LOG*, CHECK와 동일한 기능을 가진다.
    //! 단, 현재의 로그 출력 이외에 추가로 실행하는 부분이므로 성능에 영향을 줄 수 있다.

    //! GOOGLE_STRIP_LOG 
    //! 다음과 같이 설정을하면 이 설정 미만의 로그는 컴파일 타임에 제외된다.
    #define GOOGLE_STRIP_LOG 1 
    //! 위의 경우는 INFO 로그를 제외시킨다.
    ```
