# docker_guide
# docker 이용

1. docker 설치
    
    **curl -fsSL [https://get.docker.com/](https://get.docker.com/) | sudo sh**
    
2. **docker version**으로 정상적으로 설치되었는지 확인.

3. docker pull로 이미지 당겨오기
    
    [Release Notes :: CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)
    
    먼저, 위 사이트에서 cuda 버전과 이에 대응하는 nvidia-driver의 버전에 대해 호환 여부를 체크해야함.
    
    그렇지 않으면, 만들어 놓은 이미지를 다른 서버에서 사용할 때 torch.cuda.is_available()하면 false발생으로 쿠다를 잡지 못하는 문제가 발생함.
    
    가이아의 경우 nvidia driver 버전이 465 이므로, 위 사이트에서 cuda 11.4와는 호환이 안되는 것을 알 수 있음, 따라서 cuda 11.3 이하로 설치하면 됨.
    
    [Docker Hub](https://hub.docker.com/r/nvidia/cuda)
    
    위 사이트에서 원하는 이미지 버전을 검색해서 pull한다.
    
    base/runtime/devel로 나누어져있는데, devel이 용량이 크다는 단점이 있으나 패키지 호환성이 좋음.
    
    (ex. pip install pesq는 runtime 버전에서 설치 안되는 것을 확인함)
    
    **docker pull nvidia/cuda:11.1-devel-ubuntu20.04**
    
4. container 생성 및 접속
    
    **docker run —name [컨테이너이름] -it —gpus all nvidia/cuda:11.1-devel-ubuntu20.04 /bin/bash**
    
    —gpus all을 해줘야 gpu를 사용할 수 있음.(덕분에 nvidia-docker 사용안해도됨)
    
    —name은 컨테이너 이름 지정할 수 있게 함.
    
5. 아나콘다(가상환경) 설치
    - 다운로드
        
        아래 인터넷 아나콘다 공식 홈피에서 리눅스 다운로드 주소복사하기 
        
        [https://www.anaconda.com/products/individual#download-section](https://www.anaconda.com/products/individual#download-section)
        
        링크 하단에 64-Bit (x86) Installer 주소복사.
        
        **apt-get update**
        
        **apt-get install wget**
        
        **wget 다운로드링크복사(shift+insert키)**
        
    - 아래 명령어로 home 에 아나콘다 설치
        
        **bash Ananconda ~** 
        
        설치가이드 참고) [https://m.blog.naver.com/cjh226/220919371679](https://m.blog.naver.com/cjh226/220919371679)
        
6. source .bashrc 입력해서 계정 앞에 (base) 붙는 거 확인.

7. conda 가상환경 생성
    
    conda create -n <원하는 환경명> python=3.8
    
    conda activate <원하는 환경명>
    
8. 원하는 패키지 설치, 환경 구성 (pip install ~~ 등)
    - pytorch 설치
        
        [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)
        
        위 사이트에서 아래와 같이 선택 후 복사 붙여넣기.
        
        반드시 가이아기준인 CUDA 11로 설치할 것.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fd097bfb-9e66-4fc1-a761-4692c7590247/Untitled.png)
        
        **conda install pytorch torchvision torchaudio cudatoolkit=11.1 -c pytorch -c nvidia**
        
        설치완료 후 아래 명령어로 잘 설치되었는지 확인.
        
        콘솔창에 **Python** 입력 후
        
        ```
        >>> import torch
        >>> torch.cuda.is_available()
        True
        >>> torch.cuda.current_device()
        0
        >>> torch.cuda.get_device_name(0)
        'GeForce GTX 1060 6GB'
        ```
        

- 그 밖에 원하는 패키지, 라이브러리 설치

1. commit (컨테이너로부터 이미지 다시 작성함)
    
    지금까지 컨테이너에서 설치한 패키지, 라이브러리 등을 다시 이미지로 만들어야함. 
    
    **docker commit <컨테이너이름> [이미지명(repository:tag)]**
    
    (만약, 도커 허브를 이용하고 싶으면 이미지 명은 도커 허브랑 동일하게 태깅해야 push됨)
    
2. 이미지 저장
    
    [[Docker] 이미지 생성 (commit, export, import, save, load) & docker system prune](https://nomad-programmer.tistory.com/305)
    
    **docker image save -o <저장하고자할 .tar이름> [이미지명(repository:tag)]**
    
    이러면 내가 만든 이미지가 .tar 형태로 저장됨
    

11. 다른 pc에서 이미지 load

**docker load -i <.tar명>**

### 참고) 도커 허브 이용

- Push
    
    commit할 때 도커 허브에 있는 Repository 명과 docker image의 repository와 동일해야하고, tag는 내가 임의로 정하면됨.
    
    (직접 tag 변경해줘도됨, docker image tag 기존repository:tag 변경할repository:tag)
    
    **docker push [이미지명(repository:tag)]**
    
- Pull
    
    **docker pull [이미지명(repository:tag)]**
    
    permission denied 에러발생 시 아래 수행 후 로그아웃-로그인
    
    ```
    $ sudo usermod -a -G docker $USER
    $ sudo service docker restart
    ```
    
    private 이미지이면 로그인 필수
    

참고했던 사이트)



---------------------------------------------------------------------------------------------------------------------


# docker 기본 설명

[초보를 위한 도커 안내서 - 설치하고 컨테이너 실행하기](https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html)

- 컨테이너 실행 확인
    
    docker ps :  실행중인 컨테이너 확인
    
    docker ps -a : 실행중이지 않은 컨테이너를 포함한 정보 확인
    
- 컨테이너 삭제
    
    docker rm -f [container name]
    
    docker rm -v $(docker ps -a -q -f status=exited)
    
- 이미지 삭제
    
    docker rmi [image name]
    
- 컨테이너 시작/중지
    
    docker start [container name]
    
    docker stop [container ID]
    
- run(컨테이너 생성)
    
    → 우분투 이미지 다운로드(pull)-컨테이너 생성(create)-실행(start)-접속(exec) 과정을 한 번에 함
    
    docker run -t -i -p [외부포트:내부포트] —name [컨테이너 이름] [이미지이름] /bin/bash
    
    (-t, -i 는 터미널 입력 옵션, -p는 호스트와 컨테이너의 포트 연결)
    
- pull (이미지 다운로드)
    
    docker pull ubuntu:20.04(tag)
    
- create (컨테이너 생성)
    
    docker create -t -i -p 8888:8888 —name test ubuntu:20.04 /bin/bash
    
    이러면 docker ps -a 에서 Created 상태가 됨.
    
- start(컨테이너 실행)
    
    스타트 하면 컨테이너가 실행되어 docker ps 에 올라감 
    
    docker start [컨테이너 이름]
    
- exec(컨테이너로 접속)
    
    docker exec -it [NAMES] /bin/bash
    
    (-it 는 터미널 키보드 입력 옵션)
    

- 도커 컨테이너 환경 구축

apt-get update 

apt-get wget

wget conda다운로드주소

vim ~/.bashrc (아나콘다 path 설정인데, 이거 안했는데도 터미널 다시 새창으로 켜니까 (base) 생성됨)

docker container export sanghun -o testdocker.tar

→ 이미지 타르 파일 만들어짐




