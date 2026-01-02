[블로그 링크](https://armugona.tistory.com/entry/AIE-Ch6-RAG%EC%99%80-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8)

---

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">AI 모델도 컨텍스트가 부족할 때 실수를 하거나 환각을 일으킬 가능성이 높음</p>
<p data-ke-size="size16">컨텍스트는 각 질의에 따라 달라짐</p>
<p data-ke-size="size16">프롬프트 엔지니어링은 좋은 지시를 내리는 방법이었다면, 이번 장에서는 적절한 컨텍스트를 구성하는 방법을 위주로</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>RAG 검색 증강 생성</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델이 외부 데이터 소스에서 관련 정보를 검색할 수 있도록 함</li>
<li>컨텍스트 구성하는데 주로 사용됨</li>
</ul>
</li>
<li><b>에이전트</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델이 웹 검색이나 뉴스 API 같은 도구를 사용해 정보를 수집할 수 있도록 함</li>
<li>현실 세계와 상호작용하며 작업의 많은 부분을 자동화할 수 있음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">RAG</h2>
<p data-ke-size="size16">외부 메모리 소스(DB, 사용자의 이전 채팅 세션, 인터넷)에서 관련 정보를 검색하여 모델의 생성을 향상시키는 기술</p>
<img width="1204" height="750" alt="image" src="https://github.com/user-attachments/assets/d46eee02-0112-4ad1-8533-06af5410bff7" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모든 질의에 동일한 컨텍스트를 사용하지 않고, 질의에 특화된 컨텍스트를 만들어내는 기술</li>
<li>사용자 데이터 관리에도 유용</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>컨텍스트 제한 극복 (긴 컨텍스트 길이를 가져도 RAG는 필요하다)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델의 컨텍스트 길이가 아무리 길어도, 더 긴 컨텍스트가 필요한 애플리케이션이 생김</li>
<li>사용 가능한 데이터의 양은 시간이 지나며 계속 증가함</li>
<li>새로운 데이터를 생성, 추가하지만 삭제는 잘 하지 않음</li>
</ul>
</li>
<li>긴 컨텍스트를 처리할 수 있다고 해도 반드시 그걸 잘 활용하는 것은 아님
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>컨텍스트가 길수록 모델은 잘못된 부분에 집중할 가능성이 높아짐</li>
<li>추가되는 컨텍스트는 추가 비용을 발생시킴</li>
<li>RAG로 관련성이 높은 정보만 사용할 수 있게 해서 입력 토큰 수를 줄이고 모델의 성능을 향상시킴</li>
</ul>
</li>
<li>cf. 클로드 모델에 대해서 지식 베이스가 200,000 토큰 (500페이지) 미만이라면 전체 지식 베이스를 프롬프트에 포함시켜도 된다고 조언함 (2024년 기준), 모델 개발사가 제공하는 지침을 참고해서 RAG를 쓸지, 롱 컨텍스트를 쓸지 결정하는 것이 좋음</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">RAG 아키텍처</h3>
<img width="709" height="650" alt="image" src="https://github.com/user-attachments/assets/474498f7-a169-4b2d-9da2-a507f1554230" />

<p data-ke-size="size16">오늘날의 RAG 시스템에서는 검색기와 생성 모델을 따로 학습시키는 경우가 많으며, 이미 만들어진 모델을 활용해 RAG 시스템을 구축함</p>
<p data-ke-size="size16">RAG 시스템 전체를 처음부터 끝까지 파인튜닝하면 성능이 더 좋아질 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>검색기의 품질</b>이 중요함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>검색기의 주요 기능 두가지 <b>색인화 indexing, 질의 query</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>색인화 : 나중에 빠르게 검색할 수 있도록 데이터를 처리하는 작업</li>
<li>질의 : 관련 데이터를 검색하기 위해 시스템에 요청을 전송하는 과정</li>
</ul>
</li>
</ul>
</li>
<li>문서가 너무 길면 관리하기 쉬운 <b>청크 Chunk</b>로 분할할 수 있음 (책에서는 청크도 문서로 지칭함)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">검색 알고리즘</h3>
<p data-ke-size="size16">전통적인 검색 시스템을 위해 개발된 알고리즘을 RAG에도 사용할 수 있음</p>
<p data-ke-size="size16">검색은 주어진 질의에 대한 문서들을 관련성을 기준으로 순위를 매기는 방식으로 작동함</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>용어 기반 검색</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>키워드를 사용하는 <b>어휘적 검색 lexical retrieval</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>문제점
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>많은 문서에서 해당 용어가 등장할 수 있으며, 모든 문서를 컨텍스트로 추가할 만큼 충분한 공간이 없을 수 있음, 해당 용어가 더 자주 등장하는 문서만 선택하는 방법으로 주로 해결함 (용어 빈도수 TF)</li>
<li>프롬프트가 길고 다양한 용어들을 포함할 수 있음, 이때 중요한 용어들을 식별할 방법이 필요함, 많은 문서에 등장하는 용어는 중요하지 않은 용어일 가능성이 높음, 용어가 나타나는 문서의 수가 적을 수록 중요도가 높아지는 개념을 사용해 해결함 (역문서 빈도 IDF, 클수록 중요한 용어)</li>
<li>TF-IDF는 앞의 두 가지를 결합한 알고리즘</li>
</ul>
</li>
</ul>
</li>
<li><b>엘라스틱 서치</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>역색인 inverted index</b>라는 데이터 구조를 활용함, 용어를 문서로 매핑하는 사전
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>용어가 주어졌을때 관련 문서들을 빠르게 찾을 수 있음</li>
<li>TF-IDF 점수 계산 시 유용하게 쓰임</li>
</ul>
</li>
</ul>
</li>
<li><b>오카피 BM 25</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>베스트 매칭 알고리즘의 25번째 세대</li>
<li>TF-IDF를 개선한 점수 계산 방식</li>
<li>문서 길이를 고려해 용어 빈도 점수를 정규화함, 긴 문서일수록 특정 용어를 포함할 가능성이 높고, 용어 빈도 값도 높아짐</li>
<li>BM25+, BM25F 등 널리 사용됨, 현대적인 검색 알고리즘과 비교하는 기준점</li>
</ul>
</li>
<li>n-gram 중복을 가지고 문서를 검색하기도 함</li>
</ul>
</li>
<li><b>임베딩 기반 검색</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>문서의 <b>의미</b>가 질의와 얼마나 가까운지를 기준으로 순위를 매기는 방식으로 <b>의미 기반 검색 semantic retrieval</b> 이라고도 함</li>
<li>색인화 : 원본 데이터 <b>청크를 임베딩</b>으로 변환하고, 이렇게 만들어진 임베딩을 저장하는 DB를 <b>벡터 데이터베이스</b>라고 부름
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터 저장은 쉽지만 벡터 검색은 어려움, 빠르고 효율적인 검색이 가능한 방식으로 색인화되고 저장되어야 함</li>
<li>보통 최근접 이웃 검색 문제로 접근함, 주어진 질의에 대해 k개의 가장 가까운 벡터를 찾는 것 (k-NN), 결과가 정확하지만 많은 계산량이 필요하고 느림, 작은 데이터셋에서 사용</li>
<li>큰 데이터셋에서는 근사 최근접 이웃 ANN 알고리즘으로 벡터 검색을 수행함</li>
<li>벡터 검색 라이브러리 : <b>FAISS, ScaNN, Annoy, Hnswlib 등</b></li>
</ul>
</li>
</ul>
</li>
</ul>
<<img width="1203" height="1039" alt="image" src="https://github.com/user-attachments/assets/584620ba-f681-45cd-9ce5-c37b53e418bc" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>보통 벡터 DB는 벡터를 버킷, 트리, 그래프로 구성함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>각각 다른 휴리스틱을 사용해 비슷한 벡터들이 저장 공간에서 서로 가까이 위치하도록 만듦</li>
<li>양자화나 희소하게 만들어서 연산에 필요한 컴퓨팅 자원을 절약함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>대표적인 벡터 검색 알고리즘 : 지역 민감 해싱 LSH, 계층적 탐색이 가능한 소규모 세계 HNSW, 제품 양자화, 역파일 색인 IVF, 근사 최근접 이웃 탐색 Annoy, 공간 분할 트리 및 그래프 SPTAG, 근사 최근접 이웃을 위한 빠른 라이브러리 FLANN 등</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">검색 알고리즘 비교</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>용어 기반 검색</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>빠름, 용어 추출은 임베딩 생성보다 빠르고 용어를 포함하는 문서로의 매핑은 knn 보다 계산 비용이 적을 수 있음</li>
<li>별도의 설정없이도 잘 작동함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>성능 향상을 위해 조정할 수 있는 구성 요소가 적다는 것을 의미하기도 함</li>
</ul>
</li>
</ul>
</li>
<li><b>임베딩 기반 검색</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>용어 기반 검색보다 좋은 성능을 낼 수 있음</li>
<li>검색기의 품질은 데이터의 품질로 평가할 수 있음, 지표: <b>컨텍스트 정밀도(컨텍스트 관련성)와 컨텍스트 재현율</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>컨텍스트 정밀도 :&nbsp;</b>검색된 모든 문서 중에서 실제로 질의와 관련된 문서의 비율은 얼마인지</li>
<li><b>컨텍스트 재현율 :&nbsp;</b>질의와 관련된 모든 문서 중에서 실제로 검색된 문서의 비율은 얼마인지
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>지표 계산을 위해 테스트용 질의 목록과 각 질의에 해당하는 문서 집합으로 평가 세트를 구성</li>
</ul>
</li>
<li>검색된 문서의 순위가 중요하다면, <b>정규화된 할인 누적 이득 NDCG, 평균 정밀도 MAP, 평균 역순위 MRR 같은 지표</b>를 사용할 수 있음</li>
</ul>
</li>
<li>임베딩 자체의 품질 평가도 필요
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>비슷한 문서들이 벡터 공간에서 가깝게 위치하면 좋은 임베딩으로 봄</li>
<li>특정 작업에 얼마나 효과적인지로도 평가됨 (검색, 분류, 클러스터링 등)</li>
</ul>
</li>
<li>지연시간 : RAG의 지연시간은 주로 출력 생성에서 발생, 질의 임베딩 생성과 벡터 검색에 따른 지연시간은 전체 RAG 지연에 비하면 미미할 수 있으나 추가 지연 시간이 있다면 사용자 경험에 영향을 줄 수 있음</li>
<li>임베딩 생성 비용 : 데이터가 자주 변경되어 자주 임베딩을 다시 만들어야 한다면 문제가 될 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>어떤 벡터 DB를 쓰느냐에 따라서 벡터 저장, 검색 질의 비용 차이가 있을 수 있음</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1192" height="425" alt="image" src="https://github.com/user-attachments/assets/5183c980-ffbc-4874-9856-7c4a29bee37b" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>색인화와 질의 사이의 적절한 균형이 중요함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>색인이 상세할수록 검색 과정은 정확해지지만 색인화 과정은 느려지도 메모리가 많이 필요함, 구축 시간도 오래걸리고 저장공간도 많이 차지함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>HNSW 같은 상세한 색인은 높은 정확도와 빠른 질의 시간을 제공하지만, 구축하는데 상당한 시간과 메모리를 필요로 함</li>
<li>LSH 같은 단순한 색인은 생성하는데 빠르고 메모리가 적게 들지만 질의 속도가 느리고 정확도도 떨어짐</li>
</ul>
</li>
<li>ANN 알고리즘 비교할때 균형을 고려하기 위한 네 가지 주요 지표 (ANN 벤치마크)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>재현율</b> : 알고리즘이 찾은 최근접 이웃의 비율</li>
<li><b>초당 질의 수 (QPS)</b> : 알고리즘이 초당 처리할 수 있는 질의 수, 트래픽이 많은 애플리케이션에서 중요</li>
<li><b>구축 시간</b> : 색인을 구축하는데 필요한 시간, 이 지표는 특히 색인을 자주 업데이트해야 하는 경우 중요</li>
<li><b>색인 크기</b> : 알고리즘이 생성한 색인의 크기, 확장성과 저장 공간 요구사항 평가</li>
</ul>
</li>
</ul>
</li>
<li>RAG 시스템의 품질은 구성 요소와 전체 시스템 모두에서 평가해야함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>검색 품질 평가, 최종 RAG 출력 결과 평가, 임베딩 기반 검색 사용 시 임베딩 품질 평가</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">검색 알고리즘 결합하기</h4>
<p data-ke-size="size16"><b>하이브리드 검색</b> : 용어 기반 검색과 임베딩 기반 검색을 결합하는 것</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>재순위화 reranking</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>용어 기반 시스템으로 후보군을 추리고 knn으로 후보 중에서 최상의 결과를 찾기</li>
</ul>
</li>
</ul>
<p data-ke-size="size16"><b>앙상블 기법</b>으로도 결합할 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>여러 검색기를 동시에 써서 후보를 가져오고, 다양한 순위들을 하나로 결합해 최종 순위를 생성
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>역순위 퓨전 RRF
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>검색기가 매긴 순위에 따라 각 문서에 점수를 부여함, 1/1 =1 (1위), 1/2 = 0.5 (2위)</li>
<li>최종 점수는 모든 검색기에서 받은 점수의 합</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">검색 최적화</h3>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>청킹 전략</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>특정 단위를 기준으로 문서를 동일한 길이의 청크로 나누는 전략
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>문자, 단어, 문장, 단락 같은 단위를 사용</li>
<li>각 청크가 최대 청크 크기 안에 들어올 때까지 점점 작은 단위를 사용해 분서를 재귀적으로 분할할 수도 있음</li>
<li>특정 문서 유형에 따라 청킹 전략을 사용할 수도 있음 (프로그래밍 언어용, Q&amp;A 질의 응답 쌍, 언어)</li>
</ul>
</li>
<li>겹침 없이 청크로 나누면 중요한 컨텍스트에서 끊겨 정보가 손실될 수 있음, 청크 간 겸침을 통해 경계 정보가 최소한 하나의 청크에 포함되도록 보장 (2048자 청크라면 20자 겹침 설정 등)</li>
<li>청크 크기는 생성 모델의 컨텍스트 길이 제한을 넘지 않아야 함</li>
<li>생성 모델의 토큰화된 토큰 기준으로 문서를 청킹하는 전략도 있음</li>
<li>어떤 전략을 선택하든,&nbsp;<b>청크 크기</b>가 중요함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>크기가 작을 수록 다양한 정보를 담을 수 있음, 모델의 컨텍스트에 더 많은 청크를 넣을 수 있음, 너무 작으면 정보 손실, 계산 부담 증가 (임베딩 기반 검색 시, 청크 크기를 절반으로 줄이면 필요한 임베딩 공간이 두배, 질의 속도 느려짐 우려)</li>
<li>실험을 통해 적절한 청크 크기를 찾아야 함</li>
</ul>
</li>
</ul>
</li>
<li><b>재순위화</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>검색기가 생성한 초기 문서 순위를 재순위함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델의 컨텍스트에 맞추거나 입력 토큰 수를 줄이기 위해 검색된 문서 수를 줄여야 할 때 유용함</li>
</ul>
</li>
<li>시간 기준으로 최신 데이터에 높은 가중치를 부여하는 전략</li>
<li>컨텍스트 재순위화는 항목의 정확한 위치가 덜 중요함</li>
</ul>
</li>
<li><b>질의 재작성</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>질의 재구성, 질의 정규화, 질의 확장이라고도 함</li>
<li>사용자 의도를 반영한 질의 재작성이 필요한 경우가 있음 (연속된 대화의 정보를 필요로 하는 질문)
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>새롭게 작성한 질의는 별도의 컨텍스트 없이도 의미가 명확해야 함</li>
</ul>
</li>
<li>다른 AI 모델로 질의 재작성을 수행할 수도 있음</li>
<li>질의에서 필요로하는 정보가 없다면 재작성 모델이 잘못된 응답을 제공하는 것보다는 답을 찾을 수 없다고 명확히 알려야 함</li>
</ul>
</li>
<li><b>컨텍스트 검색</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>각 청크에 관련 컨텍스트를 추가해 필요한 청크를 더 쉽게 검색할 수 있도록 하는 것
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>태그나 키워드 같은 메타데이터로 청크를 보강하는 것</li>
<li>청크에 응답할 수 있는 관련 질의들을 추가할 수도 있음</li>
</ul>
</li>
<li>문서가 여러 청크로 나뉘면 일부 청크를 검색기가 그 내용을 이해하는 데 필요한 컨텍스트가 부족할 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>원본 문서의 제목이나 요약 같은 정보를 각 청크에 추가할 수 있음</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1200" height="663" alt="image" src="https://github.com/user-attachments/assets/dd934f91-cdac-4eb3-ade8-2b236ff3e702" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">검색 솔루션 평가 시 고려해야할 주요 사항들</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>어떤 검색 방식을 지원하는가, 하이브리드 검색을 지원하는가</li>
<li>벡터 DB라면 어떤 임베딩 모델과 벡터 검색 알고리즘을 지원하는가</li>
<li>데이터 저장량과 트래픽 측면에서 얼마나 확장 가능한가, 트래픽 패턴에 적합한가</li>
<li>데이터를 색인화하는 데 얼마나 시간이 걸리는가, 한 번에 얼마나 많은 데이터를 대량으로 처리할 수 있는가</li>
<li>다양한 검색 알고리즘에 대한 질의 지연 시간은 어느 정도인가</li>
<li>관리형 솔루션인 경우, 가격 체계는 어떻게 되는가, 저장된 문서/벡터 양에 따라 가격이 책정되는지, 아니면 검색 요청 횟수에 따라 책정되는지</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">텍스트를 넘어선 RAG</h3>
<p data-ke-size="size16">외부 데이터 소스는 여러 형식이 혼합된 멀티모달 데이터나 표 형태의 데이터일 수 있음</p>
<h4 data-ke-size="size20">멀티모달 RAG</h4>
<p data-ke-size="size16">생성 모델이 멀티모달 데이터를 처리할 수 있다면 컨텍스트도 여러 형태의 데이터를 활용할 수 있음</p>
<img width="1203" height="724" alt="image" src="https://github.com/user-attachments/assets/1343cebe-c49e-4c05-8859-8d6e96ba426c" />

<p data-ke-size="size16">이미지에 제목, 태그, 캡션 같은 메타데이터가 있다면 이를 사용해 검색할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">이미지 내용을 기반으로 검색하고 싶다면 이미지와 질의를 비교할 방법이 필요함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>질의가 텍스트 형태라면, 이미지와 텍스트 모두를 벡터로 변환할 수 있는 멀티모달 임베딩이 필요함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">표 형식 데이터를 활용한 RAG</h4>
<p data-ke-size="size16">표 형식 데이터 tabular로 컨텍스트를 보강하는 과정은 일반적인 RAG 워크플로우와 상당히 다름</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>Text-to-SQL : 사용자 질의와 제공된 테이블 스키마를 기반으로 어떤 SQL 쿼리가 필요한지 결정
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>사용 가능한 테이블이 너무 많아 모든 스키마를 모델 컨텍스트에 담을 수 없다면, 각 질의에 어떤 테이블을 사용할지 예측하는 중간 단계가 필요할 수 있음&nbsp;</li>
<li>최종 응답 생성 모델과 동일한 생성 모델로 수행하거나 Text-to-SQL에 특화된 별도의 모델을 사용할 수 있음</li>
</ul>
</li>
<li>SQL 실행 : 만들어진 SQL 쿼리를 실행</li>
<li>응답 생성 : SQL 실행 결과와 사용자 질의를 토대로 응답 생성</li>
</ul>
<img width="1199" height="763" alt="image" src="https://github.com/user-attachments/assets/8530da94-3c7a-4d48-9beb-44b3b48cf405" />

<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">에이전트</h2>
<p data-ke-size="size16">AI 에이전트는 아직 이를 정의하고, 개발하고, 평가하기 위한 확립된 이론적 프레임워크가 없는 신생 분야</p>
<p data-ke-size="size16">에이전트의 능력을 결정짓는 두 가지 핵심요소인 도구, 계획 수립에 대해 알아봄</p>
<p data-ke-size="size16">에이전트는 새로운 개념이지만 모델의 자기 비평, CoT, 구조화된 출력 등 앞에서 다룬 개념들을 토대로 만들어짐</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">에이전트 개요</h3>
<p data-ke-size="size16">에이전트는 자신의 환경을 인식하고, 그 환경에서 행동할 수 있는 모든 것을 말함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>작동하는 환경</b>과 <b>수행할 수 있는 행동들</b>로 정의됨</li>
<li>AI 에이전트가 수행할 수 있는 행동 집합들은 접근할 수 있는 도구에 의해 확장됨</li>
<li>환경은 에이전트가 잠재적으로 사용할 수 있는 도구를 결정함</li>
<li>도구 목록은 활동할 수 있는 환경을 제한함</li>
</ul>
<img width="1194" height="561" alt="image" src="https://github.com/user-attachments/assets/d471e9ac-7071-4972-b71b-5622b9c7b037" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>AI 에이전트는 사용자가 입력으로 제공되는 작업을 수행하기 위해 만들어짐
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>AI : 작업과 환경에서 받은 피드백을 포함한 정보를 처리하는 두뇌 역할, 작업을 달성하기 위한 행동 순서를 계획, 작업이 완료되었는지 판단</li>
</ul>
</li>
<li>에이전트가 강력한 모델이 필요한 두 가지 이유
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>누적되는 오류 : 에이전트는 작업을 수행하기 위해 여러 단계를 거쳐야 함, 단계 수가 증가할수록 전체 정확도는 감소함</li>
<li>더 큰 위험성 : 도구를 사용할 수 있게 되면서 더 영향력있는 작업을 수행할 수 있음, 실패할 경우 더 심각한 결과를 초래함</li>
</ul>
</li>
<li>사용할 수 있는 도구와 계획 수립에 따라 주어진 환경에서 에이전트의 성공이 달라짐</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">도구</h3>
<p data-ke-size="size16">꼭 외부 도구에 접근할 필요는 없지만, 외부 도구 없이는 에이전트의 능력이 제한적일 수밖에 없음</p>
<p data-ke-size="size16">모델은 보통 한 가지 행동만 수행할 수 있음 (텍스트 생성, 이미지 생성 등)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>도구 tool</b>은 에이전트에게 환경을 인식하는 능력 (읽기), 환경에 변화를 주는 능력을 부여함 (쓰기)</p>
<p data-ke-size="size16">세 가지 도구 유형 :<b> 지식 증강, 능력 확장, 에이전트가 환경에 직접 변화를 줄 수 있는 도구들</b></p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">지식 증강</h4>
<p data-ke-size="size16">에이전트의 지식을 증강해주는 도구들</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>텍스트 검색기, 이미지 검색기, SQL 실행기 등</li>
<li>내부 인물 검색, 재고 API, 슬랙 메세지 검색, 이메일 읽기 도구 등
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>이 도구들 중 상당수는 조직의 비공개 프로세스와 정보로 모델을 강화함, 도구는 인터넷과 같은 공개 정보에도 접근할 수 있게 해줌</li>
</ul>
</li>
<li>웹 브라우징&nbsp;
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>검색 API, 뉴스 API, 깃허브 API, X, 링크드인, 레딧과 같은 소셜 미디어 API 등 인터넷에 접근하는 모든 도구를 아우르는 의미</li>
<li>최신 정보를 참조하는 장점이 있지만 인터넷의 유해한 콘텐츠에 노출될 위험도 있음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">능력 확장</h4>
<p data-ke-size="size16">모델의 고유한 한계를 보완하는 도구들</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>계산기, 캘린더, 시간대 변환기, 단위 변환기, 번역기, 코드 인터프리터 (코드 주입 공격의 위험이 따름)</li>
<li>텍스트/이미지 전용 모델을 멀티모달로 만들 수 있음</li>
</ul>
<p data-ke-size="size16"><b>도구 사용이 단순한 프롬프팅이나 파인튜닝보다 모델의 성능을 크게 향상시킬 수 있음</b></p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">쓰기 행동</h4>
<p data-ke-size="size16">쓰기 행동을 통해 더 많은 일을 할 수 있게 됨</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>SQL: 테이블 변경, 삭제 / 이메일 답장 / 송금 거래</li>
</ul>
<p data-ke-size="size16">AI에게 우리 삶을 자동으로 변경할 수 있는 능력을 부여하는 것은 두려운 일임</p>
<p data-ke-size="size16">시스템의 능력이 중요하지만 보안 조치에 대한 신뢰도 중요함, 악의적으로 조작되어 해로운 행동을 하지 않도록 철저히 방어해야 함</p>
<p data-ke-size="size16">많은 모델 제공업체는 함수 호출 function calling을 통해 모델이 다양한 도구를 사용할 수 있게 지원하고 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">계획 수립</h3>
<p data-ke-size="size16">파운데이션 모델 에이전트의 핵심 : 작업 해결 모델, 작업은 목표와 제약 조건으로 정의됨</p>
<p data-ke-size="size16">복잡한 작업은 계획 수립이 필요함, 작업을 달성하는 데 필요한 단계를 정리한 로드맵을 만드는 과정 (어려움)</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">계획 수립 개요</h4>
<p data-ke-size="size16">주어지 작업을 분해하는 방법은 많지만 모든 방법이 성공적인 결과로 이어지지는 않으며, 올바른 해결책 중에서도 더 효율적인 것이 존재함</p>
<p data-ke-size="size16">하나의 프롬프트에서 계획 수립과 실행을 하게할 수도 있지만, 목표 달성이 불가능하고 오래걸리는 계획을 세울 경우 무의미한 단계들을 실행할 수 있음, 낭비</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>계획 수립과 실행은 분리해야 함</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>계획을 만들어달라고 요청한 뒤, 검증된 후에만 실행
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>휴리스틱을 사용해 검증할 수 있음 (e.g. 계획은 구글 검색을 필요로하는데 에이전트가 구글 검색 사용을 못함, n단계보다 많은 모든 계획은 제거)</li>
<li>AI 평가자를 사용해 계획 검증</li>
<li>계획이 부적절하다고 평가되면 다른 계획을 생성하도록 요청
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>계획 생성 요소, 계획 검증 요소, 계획 실행 요소 : 각 구성 요소를 에이전트로 본다면, 이는 멀티 에이전트 시스템이라고 할 수 있음</li>
<li>계획을 차례대로 생성하는 대신 여러 계획을 동시에 만들고 평가자에게 선택하도록 요청해서 과정을 빠르게 진행할 수 있음, 지연 시간과 비용사이의 균형점을 찾아야 함</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1263" height="589" alt="image" src="https://github.com/user-attachments/assets/b18fa9a5-cae1-4fc0-869c-7360b970f52e" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>계획 수립을 위해서는 작업 뒤에 있는 <b>의도를 이해하는 것</b>이 필요함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>의도 분류는 별도의 프롬프트나 작업을 위해 학습된 분류 모델을 사용해 수행할 수 있음</li>
<li>의도 분류 메커니즘은 멀티 에이전트 시스템의 또 다른 에이전트로 볼 수 있음</li>
<li>의도를 파악하면 적절한 도구 선택에 도움이 됨</li>
</ul>
</li>
<li>계획 생성, 검증, 실행 세 단계를 처리하는 과정을 개선하고 위험을 줄이기 위해 사람이 개입할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>각 작업에 대해서 에이전트의 자동화 수준을 명확히 정의해야 함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">계획자 역할을 하는 파운데이션 모델</h4>
<p data-ke-size="size16">많은 연구자는 자기회귀 언어 모델을 기반으로 한 모델은 계획을 세울 수 없다고 봄</p>
<p data-ke-size="size16">그럴듯해 보일 수 있지만 실행 단계에서 여러 문제와 오류를 일으킬 수 있음</p>
<p data-ke-size="size16"><b>계획 수립은 본질적으로 탐색 문제&nbsp;</b></p>
<p data-ke-size="size16">탐색 과정은 종종 <b>백트래킹</b>이 필요하지만 자기회귀 언어 모델은 이 능력이 없기에 계획을 세울 수 없다는 주장도 있음 (하지만 꼭 그렇지만은 않음)</p>
<p data-ke-size="size16">계획 수립에 취약하다는 사례가 있어도 올바르게 사용을 못한건지 근본적인 문제인지 명확하지 않음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">계획 수립에 취약한 이유는 필요한 도구를 제대로 갖추지 못했기 때문일 수도 있음</p>
<p data-ke-size="size16">효과적인 계획을 세우려면 행동의 잠재적 결과도 알아야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">계획 생성</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>가장 간단한 방법 : 프롬프트 엔지니어링</li>
<li>행동 순서와 관련된 파라미터 모두 AI 모델이 생성하기 때문에 환각이 발생할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>계획 수립 능력을 향상시키기 위한 방법
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>더 많은 예시가 포함된 프롬프트</li>
<li>도구와 파라미터에 대해 더 자세히 설명</li>
<li>복잡한 함수를 나누기 등 함수를 단순하게 만들기</li>
<li>강력한 모델 사용</li>
<li>계획 생성을 위해 파인튜닝</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">함수 호출</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>도구 사용 방식
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>도구 목록 생성, 에이전트가 사용할 수 있는 도구 지정</li>
<li>특정 API를 사용하고 싶다면 해당 API의 문서를 확인하는 것이 좋음&nbsp;</li>
</ul>
</li>
<li>정의된 에이전트가 어떤 도구를 사용할지와 필요한 파라미터를 자동으로 생성함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>파라미터 값들이 정확한지 반드시 검토해야 함</li>
</ul>
</li>
</ul>
<img width="1258" height="891" alt="image" src="https://github.com/user-attachments/assets/d5b41f48-240a-4ee4-94cb-7de5e52f5a66" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">계획의 세부성</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>계획 과 실행 사이에 절충점이 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>절충을 피하기 위한 방법: 계층적으로 계획 세우기 (큰 그림의 계획 세우고 작은 단위 계획 세우기)</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>정확한 도구 이름 사용, 특정 도구 목록 기반으로 계획을 세우도록 모델을 파인 튜닝한 경우, 다시 학습이 필요함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>도구 API 변화에 유연하게 대응하기 위해 함수 이름보다는 일반적인 자연어를 사용해 계획을 만듦
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>현재 날짜 가져오기, 지난주 베스트셀러 제품 조회, 제품 정보 조회, 쿼리 생성, 응답 생성</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">복잡한 계획</h4>
<p data-ke-size="size16">앞에서 본 예시는 모두 순차적 형태로 제어 흐름 (행동이 실행되는 순서)의 한 유형</p>
<p data-ke-size="size16">에어진트 프레임워크 평가 시 어떤 제어 흐름을 지원하는지 확인해야 함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>순차 실행</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>작업 B가 작업 A에 의존하는 경우가 많음</li>
</ul>
</li>
<li><b>병렬 실행</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>A와 B를 동시에 실햄</li>
</ul>
</li>
<li><b>If 조건문</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>이전 단계의 결과에 따라 작업 B나 C로 분기</li>
</ul>
</li>
<li><b>For 반복문</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>특정 조건이 충족될 때까지 작업 A를 반복</li>
</ul>
</li>
</ul>
<img width="1181" height="747" alt="image" src="https://github.com/user-attachments/assets/799921fc-1e1c-442c-a6d3-d169090c7725" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">성찰 및 오류 수정</h4>
<p data-ke-size="size16">계속해서 평가하고 조정하며 성공 가능성을 높여야 함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>성찰</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>사용자 질의를 받은 후 요청이 실현 가능한지 검토</li>
<li>초기 계획이 타당한지 확인</li>
<li>실행 단계 이후 올바른 방향으로 진행되고 있는지 점검</li>
<li>계획이 실행된 후 작업이 제대로 완료되었는지 확인</li>
</ul>
</li>
<li>ReAct : 추론과 행동의 교차 방식이 에이전트의 일반적인 패턴이 됨
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>추론 : 계획 수립과 성찰을 모두 포함하는 개념</li>
<li>각 단계에서 자신의 생각을 설명 (계획), 행동을 취한 뒤 관찰 결과를 분석 (성찰) 과정을 반복함</li>
</ul>
</li>
<li>멀티 에이전트 : 한 에이전트는 계획, 행동 / 다른 에이전트는 결과 평가</li>
<li>리플렉션 : 성찰이 두 모듈로 나뉨
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>평가자, 무엇이 잘못되었는지 과정을 짚어보는 성찰 모듈</li>
<li>궤적 : 계획을 지칭하는 단어</li>
</ul>
</li>
</ul>
<img width="1266" height="1144" alt="image" src="https://github.com/user-attachments/assets/e81761be-04a5-46f2-ad6a-f04287c0005d" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">도구 선택</h4>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>도구는 작업 성공에 중요한 역할을 함</li>
<li>최상의 도구 세트를 선택하는 방법은 없음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>도구가 많을 수록 능력이 향상되지만, 그만큼 효율적으로 사용하기 어려워짐</li>
</ul>
</li>
<li>도구 선택에도 실험과 분석이 필요함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>여러 조합으로 에이전트 성능 비교</li>
<li>특정 도구 젝 시 성능이 얼마나 떨어지는지 연구 수행 (Ablation study)</li>
<li>에이전트가 자주 실수하는 도구를 찾아 교체</li>
<li>도구 호출 분포 시각화</li>
</ul>
</li>
</ul>
<img width="1280" height="691" alt="image" src="https://github.com/user-attachments/assets/860dd687-29ec-4704-a068-9dce2f877348" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>도구 전환 연구</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>도구 X를 사용한 후 도구 Y를 호출할 확률은 얼마나 될지</li>
<li>도구가 자주 함께 사용되는 패턴이 보인다면 둘을 하나의 강력한 도구로 합칠 수 있음</li>
<li>이런 패턴을 인식하면 초기 도구들을 조합해 더 복잡한 도구를 점진적으로 만들어낼 수 있음</li>
</ul>
</li>
</ul>
<img width="708" height="986" alt="image" src="https://github.com/user-attachments/assets/ec96c056-6a7c-44a6-807d-dae4bd917c68" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>스킬 매니저</b> : 에이전트가 습득한 기술을 추적하고 사용할 수 있게 해주는 스킬 매니저
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>새로 생성된 스킬이 유용하다고 파단하면 스킬 라이브러리에 추가</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">에이전트 실패 유형과 평가</h3>
<p data-ke-size="size16">계획 수립, 도구 실행, 효율성 관련된 실패를 겪을 수 있음</p>
<p data-ke-size="size16">에이전트의 <b>실패 유형</b>을 파악하고 얼마나 자주 발생하는지 측정해 평가해야함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>실패 유형 벤치마크 : 버클리 함수 호출 리더보드, AgentOps 평가 도구, TravelPlanner 벤치마크</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">계획 수립 실패</h4>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>도구 사용 실패&nbsp;</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>유효하지 않은 도구</li>
<li>유효한 도구, 유효하지 않은 파라미터</li>
<li>유효한 오두, 잘못된 파라미터 값</li>
</ul>
</li>
<li><b>목표 달성 실패&nbsp;</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>작업을 해결하지 못하거나 제약조건을 따르지 않음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>시간은 중요하지만 종종 간과되는 제약 조건</li>
</ul>
</li>
</ul>
</li>
<li><b>성찰 오류</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>실제로는 작업을 완료하지 못했는데 완료했다고 확신하는 것</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">계획 수립 실패에 대해 평가하기 위한 방법으로 각 예시를 (작업, 도구 목록) 쌍으로 구성된 계획 수립 데이터셋 만들기</p>
<p data-ke-size="size16">k개의 계획을 생성한 뒤, 다음 지표를 계산</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>생성된 모든 계획 중 유효한 계획의 비율은?</li>
<li>주어진 작업에 대해 유효한 계획을 얻기 위해 평균적으로 몇 개의 계획을 생성해야 하는지</li>
<li>모든 도구 호출 중 유효한 호출의 비율은</li>
<li>유효하지 않은 도구가 호출되는 빈도는</li>
<li>유효한 도구가 유효하지 않은 파라미터로 호출되는 빈도는</li>
<li>유효한 도구가 잘못된 파라미터 값으로 호출되는 빈도는</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">도구 실패</h4>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>도구 자체가 잘못된 출력을 내놓는 경우</li>
<li>적절한 도구를 가지고 있지 않은 경우</li>
</ul>
<p data-ke-size="size16">도구 실패는 도구마다 다르게 나타나며, 개별적으로 테스트 되어야 함</p>
<p data-ke-size="size16">특정 분야에서 자주 실패한다면, 도구가 부족하기 때문일 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">효율성</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>유효한 계획을 세우더라도 비효율적일 수 있음</li>
<li>효율성 평가 지표 요소
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>작업을 완료하는데 평균적으로 몇 단계가 필요한가</li>
<li>작업을 완료하는 데 평균적으로 얼마의 비용이 드는가</li>
<li>각 행동은 보통 얼마나 시간이 거릴는가, 시간이나 비용이 많이 드는 행동이 있는가</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">메모리</h2>
<p data-ke-size="size16">메모리 시스템은 모델의 능력을 크게 향상시킬 수 있음</p>
<p data-ke-size="size16">메모리 : 모델이 정보를 저장하고 활용할 수 있게 하는 방식</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>RAG : 검색된 정보로 컨텍스트 보강, 이를 처리 하기 위해 메모리 활용
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>대화가 계속될수록 더 많은 정보를 가져와 점점 커질 수 있음</li>
</ul>
</li>
<li>에이전트 : 지시, 예시, 컨텍스트, 도구 목록, 계획, 도구 출력, 성찰 등을 저장하기 위해 메모리가 필요함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>AI 모델의 메모리 종류
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>내부 지식</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델이 학습 데이터에서 얻은 모든 지식을 저장하고 있으므로 그 자체로 하나의 기억 시스템임</li>
<li>모델을 업데트하지 않는 한 변하지 않음</li>
</ul>
</li>
<li><b>단기 메모리</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델의 컨텍스트도 하나의 기억 시스템으로 봄, 이전 대화 내용을 컨텍스트에 추가하면 응답 생성 시 이 정보를 활용</li>
<li>작업이 끝나면 사라짐</li>
<li>접근 속도는 빠르지만 용량이 한정되어 있고, 현재 작업에서 가장 중요한 정보를 저장하는 데 사용</li>
</ul>
</li>
<li><b>장기 메모리</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>검색을 통해 접근할 수 있는 외부 데이터 소스</li>
<li>모델 자체를 업데이트 하지 않고도 수정 삭제 가능</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="779" height="674" alt="image" src="https://github.com/user-attachments/assets/23e054aa-e2d4-48c6-a544-dc2c8c0f24f5" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>필수적인 정보는 학습이나 파인튜닝으로 내부 지식으로 만들기</li>
<li>현재 진행 중인 작업이나 대화에 바로 필요한 특정 상황의 정보는 단기 메모리</li>
<li>거의 필요하지 않은 정보는 장기 메모리&nbsp;</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>AI 모델을 위한 메모리 관리 도구, 메모리 시스템 추가 시 얻는 이점들
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>세션 내 정보 과부하 관리</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>작업 실행 과정에서 새로운 정보의 양이 최대 컨텍스트 길이를 초과하면, 넘치는 정보를 장기 메모리 시스템에 저장할 수 있음</li>
</ul>
</li>
<li><b>세션 간 정보 유지</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>대화 기록 유지</li>
</ul>
</li>
<li><b>모델의 일관성 향상</b></li>
<li><b>데이터 구조 무결성 유지</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>구조화된 데이터를 저장할 수 있는 메모리 시스템으로 무결성 유지를 도움</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>AI를 위한 메모리 시스템의 두 가지 핵심 기능
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>메모리 관리</b> : 어떤 정보를 단기 및 장기 메모리에 저장할지 관리</li>
<li><b>메모리 검색 </b>: 장기 메모리에서 작업과 관련된 정보 검색</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>메모리 관리
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>추가와 삭제라는 두 가지 작업으로 이루어짐</li>
<li>외부 메모리 저장 공간은 상대적으로 저렴하고 쉽게 확장할 수 있지만, 단기 메모리는 최대 컨텍스트 길이에 의해 제한되므로 무엇을 추가하고 무엇을 삭제할지 전략이 필요함</li>
</ul>
</li>
<li>장기 메모리는 단기 메모리의 넘치는 정보를 저장하는데 활용될 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델에 입력되는 컨텍스트는 단기 메모리와 장기 메모리에서 검색된 정보로 구성됨</li>
<li>단기 메모리 용량은 컨텍스트 중 장기 메모리에서 가져온 정보에 얼마나 많은 공간은 할당하는지에 따라 결정됨</li>
</ul>
</li>
<li>메모리 관리는 모든 데이터 시스템의 기본, 메모리를 효율적으로 사용하기 위한 전략이 존재<br />
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>선입선출 FIFO</li>
<li>중복을 감지하고 줄이기
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>대화의 요약을 사용</li>
<li>요약을 얻은 후, 요약에서 놓친 핵심 정보와 기존 메모리를 결합해 새로운 메모리를 구성하는 연구 존재
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>문장과 요약의 각 문장에 대해, 어떤 문장을 새 메모리에 추가할지 결정하는 분류기 개발</li>
</ul>
</li>
</ul>
</li>
<li>성찰 기반 접근법
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>행동을 한 후 에이전트에게 두 가지를 수행하도록 요청
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>방금 생성된 정보에 대해 성찰</li>
<li>새로운 정보가 메모리에 추가되어야 하는지, 기존 메모리에 병합되어야 하는지, 다른 정보가 특히 오래되었거나 새로운 정보와 모순되어서 다른 정보로 대체해야 하는지 결정</li>
</ul>
</li>
</ul>
</li>
<li>모순되는 정보를 처리할 때, 최신 정보를 유지하는 방식을 선호하거나 AI 모델에게 어떤 것을 유지할지 판단하도록 요청, 모순된 정보가 에이전트를 혼란스럽게 할 수 있지만, 다양한 관점에서 문제를 바라보는데 도움이 될 수도 있음</li>
</ul>
</li>
</ul>
