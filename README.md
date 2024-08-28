# JST Shibboleth IdP installer
## 1.사전준비
#### 1.1 VM서버
- 기본사양 메모리4G이상, Disk50GB이상, CPU4코어이상 

#### 1.2 OS설치
- Ubuntu 20.04.5 LTS(Focal Fossa) 
- https://mirror.kakao.com/ubuntu-releases/focal/ubuntu-20.04.5-live-server-amd64.iso
 
#### 1.3 IP 설정
- DMZ구간에 사설IP사용할 경우 공인IP로 1:1 NAT 구성 필요 DNS주소에는 공인IP 여야함

#### 1.4 방화벽 개방 
|순번|(출발지) **포트**|(목적지) **포트**|
|:------:|:---:|:---:|
|1|(ANY) **ANY**|(IdP IP) **443**|
|2|(IdP IP) **ANY**|(DB IP) **1521_오라클,3306_MySQL,MSSQL 기타 DB 포트**|


#### 1.5 DNS 등록 
- 호스명 idp 또는 sidp 추천
- 학교도메인.jst.ac.kr (myuniv.jst.ac.kr) 

#### 1.6 docker 설치
    sudo timedatectl set-timezone Asia/Seoul
    ls -al /etc/localtime
    sudo apt update
    sudo apt upgrade
    cd $HOME
    curl -fsSL https://get.docker.com -o get-docker.sh 
    sudo sh get-docker.sh
    sudo usermod -aG docker $USER
    sudo chmod 666 /var/run/docker.sock    
    docker version
    ip a
- TimeZone 확인 : Asia/Seoul
- docker 버전 확인(Docker Engine - Community Client,Server 버전: 20.10.22) 

#### 1.7 docker-compose 설치 및 버전확인
    cd $HOME
    sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    docker-compose --version
- docker-compose 버전 확인   

#### 1.8 보유중인 와일드카드 공인 인증서 업로드 및 변환
    mkdir $HOME/cert
    cd $HOME/cert
    cat chain2.crt chain1.crt CA.crt > chainCA.pem
    openssl pkcs12 –export –inkey [your_domain].key –in [your_domain].crt –certfile chainCA.pem –out idp-browser.p12 –passout pass:12345
    openssl pkcs12 -info -in idp-browser.p12
- (1라인) 홈디렉토리 아래에 cert 폴더 생성한 후 cert폴더에 기관의 인증서 복사<br>
  개인키: [your_domain].key<br>
  서버인증서: [your_domain].crt<br>
  CA 및 체인인증서: CA.crt, chain1.crt chain2.crt<br>
- (2라인) 홈디렉토리 아래 cert 폴더로 이동 
- (3라인) CA 및 체인인증서를 하나의 인증서(chainCA.pem)로 생성하는 방법
- (4라인) 보유중인 인증서들을 이용하여 .p12로 변환, '-passout pass:12345'는 인증서 비밀번호 
- (5라인) 생성된 idp-browser.p12 상태 확인

#### 1.9 Github에서 idP installer 다운로드(사용자의 github 계정에 접근권한 승인받은 후 사용 가능)
    cd $HOME
    git clone https://github.com/jst/shibboleth-idp.git   
    Username: YOUR_USERNAME
    Password: YOUR_TOKEN

- 위 코드를 터미널(SSH Client Program)에서 진행하고자 할 때 개인 엑세스 토큰 발급받아서 진행해야 함.
- (필수) https://www.github.com 에 로그인 > 개인 엑세스 토큰 만들기(아래 [사이트] 참조)
- [사이트] https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
  <img size="80%" src="https://user-images.githubusercontent.com/71858454/211704580-5a27ceaa-fd9c-4a47-8085-fa0a89e118ad.png"/>
  생성버튼 클릭후 필수 사항 3가지<br>
  (1)Note: 토큰 이름 <br>
  (2)Expiration: 사용기간등록<br>
  (3)Available scopes: 아래 그림 참조
  <img src="https://docs.github.com/assets/cb-43299/images/help/settings/token_scopes.gif"/><br>
  <img src="https://docs.github.com/assets/cb-10912/images/help/settings/generate_token.png"/>
- 발급받은 토큰 복사 > ID 입력 > 암호입력(발급받은 토큰 붙여넣기)
- *토큰을 계속 사용하기 위해 토큰을 텍스트파일(my_token.txt)에 저장하여 관리*
#### 1.10 공인인증서 변환 파일 복사
    cd $HOME/shibboleth-idp/
    cp $HOME/cert/idp-browser.p12 ./shibboleth-idp/opt/shibboleth-idp/credentials/
    ls shibboleth-idp/opt/shibboleth-idp/credentials/
- 위 코드를 진행하여 idp-browser.p12 확인 
## 2.환경설정
    cd $HOME/shibboleth-idp
    mv .env-sample .env
    vi .env
- 환경설정 파일 이름 변경 및 내용 편집
##    
    REST_IMAGE_NAME=docker.iugj.ac.kr/rest-oracle:latest
    HOST=3.39.178.32
    DB_PORT=49161
    SERVICE=XE
    DB_USER=DBUSER
    DB_PASSWORD=qlcrkfka1#
    
    AUTH_QUERY=SELECT * FROM USERS WHERE USERNAME=:USERID AND PASSWORD=DBMS_CRYPTO.HASH(UTL_RAW.CAST_TO_RAW(UPPER(:PASSWORD)), 2)   

    ATTRIBUTE_QUERY=SELECT USERNAME as "uid", USERNAME || '@myuniv.ac.kr' AS eduPersonPrincipalName, USERTYPE || '@myuniv.ac.kr' AS eduPersonScopedAffiliation, FIRST_NAME || LAST_NAME as "displayName", 'Department of Mechanical Engineering' as "organizationalUnitName", 'Bitgaram       Univ.' as "organizationName", EMAIL as "mail", 'student' as "eduPersonAffiliation", '987654' as "employNumber" FROM USERS WHERE USERNAME=:USERID
    
    IDP_SCOPE=jst.ac.kr
    IDP_HOST_NAME=myuniv.jst.ac.kr
    IDP_ENTITYID=https://myuniv.jst.ac.kr/idp/shibboleth
    IDP_SSL_PASSWORD=xxxxxx
    IDP_ORG_DISPLAYNAME=빛가람대학교
    IDP_ORG_HOMEPAGE=https://www.univ.ac.kr
    IDP_FORGOT_PASSWORD_URL=https://www.univ.ac.kr/Find/Password
    IDP_SUPPORT_URL=https://www.univ.ac.kr/HelpDesk

- **테스트 환경구성**을 위해 **=** 이후 부분을 소속기관에 맞게 편집
##
    cd $HOME/shibboleth-idp/shibboleth-idp/opt/shibboleth-idp/edit-webapp/images/logo/
    wget [이미지URL]  또는 sftp로 업로드
    
- **기관의 IdP 로그인 메인화면 로고 파일 업로드**<br>
  이미지 크기(가로x세로): 960 x 220<br>
  이미지 포멧: png<br>
  이미지 이름: [IDP_SCOPE]_logo.png<br>
  (Sample)IdP 로그인 메인화면 이미지<br>
  <img width="80%" src="https://user-images.githubusercontent.com/71858454/211687034-90ffa65a-c049-4695-b89d-2d52233ad9e7.png"/><br>

## 3.설치(환경파일 변경시에도 수행) 및 재기동
    cd $HOME/shibboleth-idp
    docker-compose up -d --build idp

## 4.컨테이너 상태 보기
    docker ps

## 5.컨테이너 로그 보기
    docker-compose logs
    docker-compose logs idp    
- (1라인)모든 컨테이너 로그 보기
- (2라인)idP 컨테이너의 로그만 보기


## 6.메타데이터 확인
    https://youre_domain.jst.ac.kr/idp/shibboleth

## 7.메타데이터 등록
- 위의 메타데이터를 다운로드 받아 
- https://registry.iugj.ac.kr/rr3 웹사이트에 등록

## 8.테스트 및 검증
- https://dev.jst.ac.kr/simplesamlphp
- [**Authentication**] 탭 > **Test configured authentication sources** > **default-sp** 클릭하여 소속대학 선택 후 인증 및 속성값 검증
