# Docker 활용해서 클러스터 구축하기

- Docker를 활용해서 master 1개와 slave 3개로 구성된 클러스터를 구축
- 클러스터 매니저는 Yarn, 파일시스템은 HDFS를 사용하여 구축하고, 사용하는 application은 MR 또는 Spark
**
하둡 클러스터 내에서 마스터들은 슬레이브들이 접속하도록 기다리고, 각 슬레이브들은 자신이 마스터에 등록해서 클러스터에 참여하는 형태

**
설정파일을 통해서 Namenode, ResourceManger의 주소를 입력
namenode
- core-site.xml
resource manager
- yarn-site.xml

## DataNode -> NameNode 등록과정
- DataNode 데몬이 시작되면, 설정 파일에 기재된 fs.defaultFS 주소를 확인하여 NameNode에 접속을 시도
- NameNode는 새로운 DataNode가 접속하면, 블록 저장 상태나 노드정보를 등록하고 heartbeat를 통해 노드 상태를 모니터링
- 등록 성공 시, hdfs 클러스터에 해당 DataNode가 정상적으로 참여됨
- registerDataNode RPC 호출 -> NamenodeRPCServer -> FSNameSystem 등록 관리

## NodeManager -> ResourceManager 등록과정
- NodeManager 데몬이 시작되면, yarn-site.xml에 정의된 ResourceManager 주소로 연결 시도
- ResourceManager는 NodeManager가 보내오는 리소스 정보를 받아들여 "얼마나 자원을 할당할 수 있는지"를 기록하고, NodeManager를 "헬스체크"를 통해 계속 모니터링
- 등록 성공 시, Yarn클러스터에 해당 NodeManager가 정상적으로 참여하며 작업 실행을 담당
- 위와 마찬가지로 RPC를 호출하여 NodeManager와 ResourceManager 통신

## 실험 전 오해 했던 점
- 초기에는 스파크 클러스터를 구축할 때 Hadoop 환경에서 Yarn, Spark master, Spark worker를 모두 활성화 시켜야 하는 줄 알았다
- 하지만 yarn을 클러스터 매니저로 사용하면, spark 애플리케이션의 리소스 할당은 yarn에서 담당하기 때문에 spark master는 활성화 시킬 필요가 없었다.
- spark master와 worker는 spark standalone 모드로 실행될 때만 사용된다.
