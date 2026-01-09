[블로그 링크]()

---
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파인튜닝은 모델의 가중치를 조정하는 방식으로 모델을 변화시킴, 더 맞춤화된 모델을 만들 수 있지만 더 많은 초기 투자가 필요함</p>
<p data-ke-size="size16"><b>언제 파인튜닝을 하고 언제 RAG를 해야하는가?</b></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파운데이션 모델은 규모가 매우 커서 파인튜닝하는 데에도 비용이 많이 들고 구현하기 어려움.</p>
<p data-ke-size="size16">메모리 요구량 감소가 많은 파인튜닝 기법의 핵심 목표가 되었음, 따라서 모델의 메모리 사용량에 영향을 주는 요소들을 이해해야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>Parameter Efficient Finetuning : PEFT&nbsp;</b>의 여러 기법들을 살펴보자.</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파인튜닝은 모델을 직접 학습시키는 과정이므로 ML 지식이 필요함</p>
<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">파인튜닝 개요</h2>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>파인 튜닝</b> : <b>전이 학습 Transfer Learning</b>의 한 방법
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>전이 학습&nbsp;
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>딥러닝 초기부터 학습 데이터가 부족하거나 구하기 어려운 작업에 좋은 해결책이 되어옴</li>
<li><b>표본 효율성 Sample Efficiency</b>를 높여 모델이 더 적은 예시로도 같은 행동을 학습할 수 있게 함</li>
<li>이상적으로 모델이 학습해야 할 많은 부분이 이미 기본 모델에 내재되어 있고, 파인튜닝은 단지 모델의 행동을 다듬는 과정</li>
</ul>
</li>
</ul>
</li>
<li>사전 학습 이후에 모델을 추가로 학습시키는 과정을 통칭하여 파인튜닝이라고 부름</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델 학습은 보통 자기 지도 학습 방식의 사전 학습에서 시작함.</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>지속적 사전 학습 continued pre-training</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>비싼 작업별 데이터로 사전 학습 모델을 파인튜닝하기 전에, 저렴한 관련 분야 데이터로 먼저 자기 지도 학습을 적용해보는 자기 지도 파인 튜닝</li>
</ul>
</li>
<li><b>인필링 파인튜닝 infilling finetuning&nbsp;</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>지도 파인튜닝을 통해 다음 토큰을 예측하거나 빈칸을 채우도록 모델을 파인튜닝</li>
<li>텍스트 편집 및 코드 디버깅 같은 작업에 특히 유용함, 자기회귀 방식으로 사전 학습된 모델도 인필링 파인튜닝이 가능함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">자기 지도 학습을 통해 방대한 지식을 얻지만, 사용자가 원하는 특정 작업에 바로 활용하기는 어려움</p>
<p data-ke-size="size16"><b>지도 파인튜닝</b>을 통해 이런 격차를 줄이는 역할을 함. (입력, 출력) 쌍으로 모델을 학슴하며, 입력은 지시, 출력은 응답이 될 수 있음. 이러한 고품질 지시 데이터를 만드는 것은 전문 지식이 필요할 경우 생성하기 어렵고 비용이 많이 들 수 있음.</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">강화학습을 통한 선호도 파인튜닝, (지시, 선호 응답, 비선호 응답) 형식의 비교 데이터가 필요함</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>컨텍스트 길이를 늘리기 위한 파인튜닝 : 롱 컨텍스트 파인튜닝 long-context finetuning
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>위치 임베딩 조정 같은 모델 구조 수정이 피룡함</li>
<li>롱 시퀀스는 토큰의 가능한 위치가 더 많단느 뜻이며, 위치 임베딩이 이를 처리할 수 있어야 함</li>
<li>다른 파인튜닝보다 더 까다로움, 숏 시퀀스에서 성능이 오히려 떨어질 수도 있음</li>
</ul>
</li>
</ul>
<img width="1088" height="393" alt="image" src="https://github.com/user-attachments/assets/8d9590f8-d342-48cf-a794-868568bd95ec" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">애플리케이션 개발자는 대게 사후 학습된 모델을 파인튜닝하게 됨</p>
<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">파인튜닝이 필요한 경우</h2>
<p data-ke-size="size16">파인튜닝은 많은 데이터와 고사양 하드웨어를 요구하며 ML 전문가를 필요로 함</p>
<p data-ke-size="size16">프롬프트 기반 방법을 충분히 시도한 후에 파인튜닝을 시도하는 것이 일반적임</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">파인튜닝을 해야 하는 이유</h3>
<p data-ke-size="size16">일반 능력과 특정 작업 수행 능력을 모두 향상시키는 데 있음 (JSON이나 YAML 같은 특정 구조의 출력을 생성할 때 효과적)</p>
<p data-ke-size="size16">범용 모델이 특정 작업에서는 성능이 떨어질 수 있음, 모델이 특정 작업에 충분히 학습되지 않았다면, 관련된 자체 데이터로 파인튜닝하는 것이 효과적임 (e.g. SQL 문법이 포함된 데이터로 파인튜닝)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>편향 완화</b> : 기본 모델이 학습 데이터의 특정 편향을 반영한다면 파인튜닝 과정에서 신중하게 선별된 데이터를 사용해 이런 편향을 상쇄할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">작은 모델을 파인튜닝하는 경우가 일반적임 : 메모리 요구량이 적고, 운영 환경에서도 저렴하고 빠르게 사용할 수 있음</p>
<p data-ke-size="size16">특정 작업에 파인튠이된 작은 모델이 같은 작업에서 훨씬 더 큰 기본 모델보다 뛰어난 성능을 보일 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>증류 Distillation</b> : 큰 모델이 생성한 데이터로 작은 모델을 학습시켜 큰 모델처럼 작동하게 만드는 방식</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파운데이션 모델 초기에는 가장 강력한 모델들이 파인튜닝 접근이 제한된 상법용이어서 파인튜닝할만한 경쟁력이 있는 모델이 많지 않았음. 오픈 소스 커뮤니티가 다양한 도메인에 맞춰진 여러 크기의 고품질 모델을 개발하며, 파인튜닝은 실현 가능하고 매력적인 선택지가 되었음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">파인튜닝을 하지 말아야 하는 이유</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>특정 작업을 위해 모델을 파인튜닝하면 그 작업에서는 성능이 향상될 수 있지만, 다른 작업에서는 오히려 성능이 떨어질 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>다양한 프롬프트를 사용해야 하는 애플리케이션에서는 이런 문제가 상당히 답답할 수 있음 (e.g. 제품 추천, 주문 변경, 일반 피드백 세 가지 유형의 질의를 처리할 모델이 필요함)</li>
<li>모든 질의에 대해서 모델을 파인튜닝 하거나, 모든 작업에 대해서 모델이 잘 작동하게 만들기 어렵다면 다른 작업에 별도 모델을 사용할 수 있음, 별도 모델들을 하나로 합쳐 서빙을 쉽게 하고 싶다면 모델 병합을 고려해볼 수 있음</li>
</ul>
</li>
<li>프로젝트 실험을 시작하는 단계라면 파인튜닝은 가장 먼저 시도할 방법이 아님 : 초기투자가 크고 지속적인 관리가 필요함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터가 필요함 : 주석이 달린 데이터는 수동으로 보으는데 시간이 걸리고 비용이 많이 듦, 오슨 소스 데이터와 AI 생성 데이터의 효과는 경우에 따라 크게 다름</li>
<li>모델 학습 방법에 대한 지식이 필요함, 조정할 수 있는 다양한 학습 파라미터를 이해, 학습 과정 모니터링, 문제 발생 시 디버깅 (e.g. 옵티마이저 작동 방식, 적절한 학습률, 필요한 학습 데이터 양, 과적/과소적합 해결 방법, 전체 과정에서 모델을 평가하는 방법 등)</li>
<li>파인튜닝된 모델을 서빙하는 방법 : 호스팅, API 서비스 등, LLM의 추론 최적화는 간단하지 않음</li>
<li>모델을 모니터링하고 유지보수하며 업데이트하기 위한 정책과 예산 수립</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파인튜닝과 프롬프트 실험 모두 체계적인 접근 방식이 필요함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>프롬프트 실험을 진행하며 <b>평가 파이프라인, 데이터 주석 가이드라인, 실험 추적 방법 등을 구축할 수 있으며 이는 파인튜닝의 기반이 됨</b></li>
<li>프롬프트 캐싱 도입 전에 파인튜닝의 큰 장점 중 하나로 토큰 사용을 최적화하는 데 도움이 된다는 것이었음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>매번 프롬프트를 예시에 포함하는 대신 모델을 파인튜닝하여 더 짧은 프롬프트로 같은 결과를 얻을 수 있다는 것</li>
<li><b>프롬프트 캐싱&nbsp;</b>: 반복적인 프롬프트 부분을 저장했다 재사용하는 기술, 위와 같은 장점은 캐싱 도입 이후로 크게 줄어들었지만 여전히 프롬프트와 함께 사용할 수 있는 예시 수는 최대 컨텍스트 길이에 제한을 받는 반면, 파인튜닝에서는 활용할 수 있는 예시 수에 제한이 없음</li>
</ul>
</li>
</ul>
<img width="1081" height="785" alt="image" src="https://github.com/user-attachments/assets/f898780e-24c2-4ee2-85b9-0afb346adf0f" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">파인튜닝과 RAG</h3>
<p data-ke-size="size16">RAG를 적용할지 파인튜닝을 적용할지 :&nbsp;<b>모델 오류의 원인이 정보 부족인지 행동 방식의 문제인지에 따라 달라짐</b></p>
<p data-ke-size="size16"><b>모델이 정보가 부족해서</b>라면 <b>RAG가 효과적임</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델이 정보를 가지고 있지 않은 경우
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>공개 모델은 사용자나 조직의 비공개 정보를 알지 못함</li>
</ul>
</li>
<li>모델의 정보가 오래된 경우</li>
</ul>
<p data-ke-size="size16">시사 문제에 관한 질의, 최신 정보가 필요한 작업에서는<b> RAG</b>가 파인튜닝된 모델보다 더 좋은 성능을 보임</p>
<img width="1081" height="353" alt="image" src="https://github.com/user-attachments/assets/05b9d01a-fd15-4e63-98d8-e4c6f4a54efa" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>모델의 행동 방식에 문제가 있다면 파인튜닝이 효과적임&nbsp;</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델의 출력이 사실 관계는 맞지만 요청한 작업과 전혀 무관한 경우 (e.g. 소프트웨어 프로젝트 기술 명세서에 필요로 하는 세부사항이 없음)</li>
<li>모델이 원하는 출력 형식을 제대로 따르지 못하는 경우 (e.g. HTML 코드 작성을 요청했는데 생성된 코드가 작독하지 않음)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>시맨틱 파싱 semantic parsing (자연어를 JSON 같은 구조화된 형식으로 변환하는 과정)은 모델이 정해진 형식에 맞춰 출력을 생성하는 능력이 작업의 성패를 좌우함, 이 작업에는 종종 파인튜닝이 필요함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>인터넷에서 예시를 찾기 어려운 구문, 덜 유명한 도구의 특수 언어, 복잡한 구문에서 성능이 떨어질 수 있음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>파인튜닝은 형식을 위한 것이고, RAG는 사실을 위한 것</b></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">정보와 행동 측면 모두에 문제가 있다면 RAG부터 시작하는 것이 좋음, 복잡한 벡터 데이터베이스로 바로 뛰어들기 보다는 BM25 같은 단순한 키워드 기반 검색부터 시작하는 것이 좋음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>RAG와 파인튜닝은 상호 배타적이지 않으며 함께 사용하여 성능을 극대화할 수 있음</li>
<li>모든 애플리케이션에 딱 맞는 보편적인 개발 과정은 없음,</li>
</ul>
<img width="1065" height="679" alt="image" src="https://github.com/user-attachments/assets/47ce7b55-7b16-4b91-af0b-fec2863ad734" />

<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">메모리 병목 현상</h2>
<ol style="list-style-type: decimal;" data-ke-list-type="decimal">
<li>파운데이션 모델의 큰 규모로 인해, 추론과 파인튜닝 모두에서 메모리가 주요 병목 지점이 된다. 신경망 학습 방식의 특성상 파인튜닝에 필요한 메모리는 일반적으로 추론보다 훨씬 더 많음</li>
<li>모델의 메모리 사용량에 큰 영향을 미치는 요소는 <b>전체 파라미터 수, 학습 가능한 파라미터 수, 수치 표현 방식</b></li>
<li>학습 가능한 파라미터가 많을수록 메모리 사용량이 늘어남, 학습 가능한 파라미터 수를 줄이면 파인튜닝에 필요한 메모리를 줄일 수 있으며 이것이 바로 PEFT의 핵심 아이디어</li>
<li>양자화는 모델을 더 적은 비트 형식으로 변환하는 기법</li>
<li>추론은 보통 16비트, 8비트, 4비트처럼 가능한 한 적은 비트를 사용해 수행함</li>
<li>학습은 수치 정밀도에 더 민감해서 낮은 정밀도로 모델을 학습하는 것이 어려움, 학습은 주로 혼합 정밀도 방식을 사용하며 일부 연산은 더 높은 정밀도(e.g. 32비트)로, 나머지는 낮은 정밀도(e.g. 16비트 또는 8비트)로 처리함</li>
</ol>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">역전파와 학습 가능한 파라미터</h3>
<p data-ke-size="size16">학습 가능한 파라미터 : 파인튜닝 중에 업데이트될 수 있는 파라미터 (추론 과정에서는 업데이트 안됨)</p>
<p data-ke-size="size16">파인튜닝 중 일부 또는 모든 모델 파라미터가 업데이트될 수 있음, 변경되지 않고 유지되는 파라미터는&nbsp;<b>고정 파라미터</b></p>
<p data-ke-size="size16">학습 가능한 파라미터에 필요한 메모리는 모델 학습 방식에서 비롯됨.</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>역전파&nbsp;</b>학습 단계의 두 단계
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>순방향</b> 패스 : 입력값에서 출력값을 계산하는 과정</li>
<li><b>역방향</b> 패스 : 순방향 패스에서 집계된 신호를 활용해 모델의 가중치를 업데이트하는 과정
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>학습 시에 두 패스가 모두 실행됨</li>
<li>순방향 패스에서 손실을 계산하고, 학습 가능한 파라미터가 실수에 얼마나 기여했는지 그래디언트를 미분을 통해 계산함. 그래디언트를 사용해 학습 가능한 파라미터 값을 조정하며 얼마나 재조정할지는 옵티마이저가 결정함.&nbsp;</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1086" height="548" alt="image" src="https://github.com/user-attachments/assets/d9b42975-2096-412b-ac71-ccf399c8f152" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">메모리 계산</h3>
<p data-ke-size="size16"><b>적절한 하드웨어를 선택</b>하기 위해 <b>모델에 필요한 메모리 양</b>을 미리 아는 것은 유용함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>메모리 사용량은 모델 자체뿐 아니라 작업 부하와 메모리 사용량을 줄이기 위한 다양한 최적화 기법에 따라 달라짐</li>
<li>대략적인 계산식만 다룸</li>
<li>추론과 학습 과정에서 모델 운영에 필요한 메모리 양을 대략적으로 가늠할 수 있도록</li>
</ul>
<p data-ke-size="size16">cf. 추론과 학습이 요구하는 메모리 특성이 다르기 때문에 학습용 칩과 추론용 칩이 서로 다른 방향으로 발전하고 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">추론에 필요한 메모리</h4>
<p data-ke-size="size16">순방향 패스만 실행됨 : 모델 가주이를 위한 메모리가 필요함</p>
<p data-ke-size="size16">N을 모델의 파라미터 수라고 하고, M을 각 파라미터에 필요한 메모리라고 할때 모델 파라미터를 로드하는 데 필요한 메모리는 다음과 같음</p>
<p data-ke-size="size16">$$N \times M$$</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">활성화 값을 위한 메모리도 필요하며, 트랜스포머 모델은 어텐션 메커니즘을 위한 키-값 벡터에도 메모리가 필요함. 활성화 값과 키-값 벡터를 위한 메모리는 시퀀스 길이와 배치 크기에 비례해 증가함</p>
<p data-ke-size="size16">많은 애플리케이션에서 해당 값에 필요한 메모리는 모델 가중치 메로리의 약 20%로 가정할 수 있음. 애플리케이션이 더 긴 컨텍스트나 더 큰 배치 크기를 사용한다면, 실제로 필요한 메모리는 더 많아질 것이며 이 가정에 따르면 모델의 메모리 사용량은 다음과 같음</p>
<p data-ke-size="size16">$$N \times M \times 1.2$$</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">e.g. 130억 파라미터를 가진 모델은 각 파라미터에 2바이트가 필요하다면, 가중치에는 130억 X 2바이트 = 26GB가 필요할 것, 추론에 필요한 메모리는 26GB X 1.2 = 31.2GB</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">메모리 사용량은 크기에 따라 급격히 증가함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">학습에 필요한 메모리</h4>
<p data-ke-size="size16">앞서 논의한 모델 가중치와 활성화를 위한 메모리가 필요하며 추가적으로 그래디언트와 옵티마이저 스테이트를 위한 메모리도 필요함, 이는 학습 가능한 파라미터 수에 비례함.</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>SGD 옵티마이저는 스테이트가 없음, 모멘텀 옵티마이저는 학습 가능한 파라미터당 스테이드를 하나 저장함, Adam 옵티마이저는 학습 가능한 파라미터당 스테이트를 두 개 저장함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">학습에 필요한 메모리는 다음과 같이 계산됨</p>
<blockquote data-ke-style="style2"><b>학습 메모리 = 모델 가중치 + 활성화 + 그래디언트 + 옵티마이저 스테이트</b></blockquote>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">e.g. Adam 옵티마이저로 130억 파라미터 모델의 모든 파라미터를 업데이트한다고 해보면, 학습 가능한 파라미터는 그래디언트 값 하나와 두 개의 옵티마이저 스테이트, 총 3개의 값을 필요로 함. 각 값을 2바이트로 저장한다고 가정하면 그래디언트와 옵티마이저 스테이트에 필요한 메모리는 다음과 같음</p>
<blockquote data-ke-style="style2"><b>130억 $\times$ 3 $\times$&nbsp; 2바이트 = 78GB</b></blockquote>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">학습 가능한 파라미터가 10억 개만 있다면, 필요한 메모리는 크게 줄어듦</p>
<blockquote data-ke-style="style2"><b>10억 $\times$ 3 $\times$&nbsp; 2바이트 = 6GB</b></blockquote>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">이전 공식에서 활성화에 필요한 메모리가 모델 가중치 메모리보다 적다고 가정했으나 실제로는 활성화 메모리가 훨씬 더 클 수 있음, 그래디언트 계산을 위해 활성화를 저장한다면, 활성화 메모리는 모델 가중치 메모리를 훨씬 웃돌 수 있음.</p>
<img width="1108" height="614" alt="image" src="https://github.com/user-attachments/assets/1845e997-570a-4b7e-be08-06e258ba0f74" />

<p data-ke-size="size16">대형 트랜스포머 모델에서 활성화 재계산 감소에 따라 다양한 규모의 메가트론 Megatron 모델에서 활성화 메모리와 모델 가중치 메모리를 비교한 것</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>활성화 메모리를 줄이는 한 가지 방법</b> : 활성화를 아예 저장하지 않는 것</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>활성화를 재사용하기 위해 저장하는 대신, 필요할 때마다 재계산하는 방식 : <b>그래디언트 체크포인팅 gradient checkpointing</b>&nbsp;또는 <b>활성화 재계산 activation recomputation </b>이라고 함</li>
<li>메모리 요구량을 줄여주지만, 재계산으로 인해 학습 시간이 늘어남</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">수치 표현 방식</h3>
<p data-ke-size="size16">모델에서 각 값을 표현하는 데 필요한 메모리는 모델의 전체 메모리 사용량에 직접적인 영향을 미침, 값에 필요한 메모리를 절반으로 줄이면, 모델 가중치에 필요한 메모리도 절반으로 줄어듦</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>수치 표현 방식 이해</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>신경망의 수치 값은 전통적으로 <b>부동소수점</b> 수로 표현됨 : 가장 일반적인 부동소수점 형식은 <b>FP 계열</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>FP32 : </b>부동소수점을 표현하기 위해 32비트(4바이트)를 사용, <b>단정밀도 single precision</b></li>
<li><b>FP64 :&nbsp;</b>64비트(8바이트)를 사용, <b>배정밀도 double precision</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>많은 계산에 사용되지만 메모리 사용량 때문에 신경망에서는 거의 사용되지 않음</li>
</ul>
</li>
<li><b>FP16 :&nbsp;</b>16비트(2바이트)를 사용, <b>반정밀도 half precision</b></li>
<li><b>인기있는 다른 부동소수점 형식 : BF16, TF32</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>BF16 :&nbsp;</b>구글이 TPU에서 AI 성능을 최적화하기 위해 설계함</li>
<li><b>FT32 :</b>&nbsp;엔비디아가 GPU를 위해 설계함</li>
</ul>
</li>
</ul>
</li>
<li><b>정수 표현</b> : 일반적이지는 않지만 점점 인기를 얻고 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>INT8(8비트 정수), INT4(4비트 정수)</li>
</ul>
</li>
<li>각 부동 소수점 형식은 보통 숫자의 부호를 나타내기위해 1비트를 사용하고 나머지 비트는 범위와 정밀도로 나뉨
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>범위 : 범위 비트 수로 표현할 수 있는 값의 범위를 결정함</li>
<li>정밀도 : 숫자를 얼마나 정확하게 표현될 수 있는지 결정함</li>
</ul>
</li>
<li>고정밀도 형식의 숫자를 저정밀도 형식으로 변환하면 정밀도가 낮아짐, 값이 변하거나 오류가 발생할 수도 있음</li>
</ul>
<img width="1089" height="473" alt="image" src="https://github.com/user-attachments/assets/6d143802-65f9-43b0-a24f-74d58756ceff" />
<img width="1089" height="600" alt="image" src="https://github.com/user-attachments/assets/dc1259b3-902e-4075-a679-c459f719b48b" />

<p data-ke-size="size16"><span style="text-align: center;">BF16과 FP16이 같은 비트 수를 가졌지만 BF16은 범위에 더 많은 비트를, 정밀도에 더 적은 비트를 할당함</span></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델을 사용할 때는 반드시 지정된 형식으로 모델을 로드해야 함, 잘못된 수치 형식으로 로드하면 성능이 크게 저하될 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">양자화</h3>
<p data-ke-size="size16"><b>양자화 Quantization</b> : 정밀도를 맞추는 것&nbsp;</p>
<p data-ke-size="size16">모델의 메모리 사용량을 줄이는 저렴ㅎ면서도 매우 효과적인 방법, 구현이 간단하며 다양한 작업과 아키텍처에 두루 적용할 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>저정밀도는 표준 FP32보다 적은 비트를 가진 모든 형식을 의미함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>엄밀히하면 대상 형식이 정수일 때만 양자화라고 해야하지만 실제로는 값을 저정밀도 형식으로 변환하는 모든 기법을 양자화하고 부름</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>무엇을 양자화할 것인가?</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>메모리를 가장 많이 차지하는 요소부터 양자화하는 것이 이상적임</li>
<li>성능 저하없이 적용할 수 있는 부분이 무엇인지에 따라 신중하게 대상을 선택해야 함</li>
<li>추론 중 모델의 메모리 사용량에 큰 영향을 미치는 요소는 모델의 가중치와 활성화 값 : 일반적으로 가중치 양자화가 대체로 더 안정적이고 정확도 손실이 적어 사용됨</li>
</ul>
</li>
<li><b>언제 양자화할 것인가?</b><br />
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>학습 중이나 학습 후에 진행할 수 있음&nbsp;</li>
<li><b>학습 후 양자화 Post Training Quantization (PTQ)</b>는 모델이 완전히 학습된 후에 양자화하는 것, 보통 모델을 직접 학습시키지 않는 AI 애플리케이션 개발자들에게 적합</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">추론 양자화</h4>
<p data-ke-size="size16">딥러닝 초기엔 32비트 FP32를 사용해 학습하고 서빙하는 것이 표준</p>
<p data-ke-size="size16">2010년대 후반부터 16비트 및 더 낮은 정밀도로 모델을 서빙하는 것이 점점 보편화되었음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델은 <b>혼합 정밀도로 서빙</b>될 수 있음, 상황에 맞춰 가능할 때는 값의 정밀도를 낮추고 필요할 때는 높은 정밀도를 유지하는 방식 (e.g. 애플 : 2비트와 4비트 형식을 혼합해 평균 가중치당 3.5 비트 사용, 엔비디아는 4비트 부동소수점 모델 추론을 지원하는 GPU 아키텍처인 블랙웰 발표)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>8비트 이하</b>로 내려가면 수치 표현 방식이 더 복잡해짐</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>FP8 (8비트)과 FP4 (4비트) 같은 미니플로트 형식 중 하나를 사용해 파라미터 값을 부동소수점으로 유지할 수 있음</li>
<li>하지만 더 일반적으로 파라미터 값은<b> INT8이나 INT4 같은 정수 형식으로 변환함</b></li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">효과적인 기법이지만 양자화는 1비트보다 작게 양자화할 수 없음</p>
<p data-ke-size="size16">마이크로소프트에서 소개된 BitNet b1.58은 파라미터당 <b>1.58비트</b>만 필요로 하는 트랜스포머 기반 언어 모델로 파라미터가 39억 개일 때까지는 16비트 라마 2와 비슷한 성능을 보임</p>
<img width="1081" height="469" alt="image" src="https://github.com/user-attachments/assets/28671cf6-0911-44be-a760-a517add25dbb" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">정밀도를 낮추면 메모리 사용량이 줄어들 뿐 아니라 계산 속도도 향상되는 경우가 많음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>더 큰 배치 크기</b>를 사용할 수 있어 모델이 <b>더 많은 입력을 병렬로 처리할 수 있음</b></li>
<li>정밀도 감소는 <b>계산 속도를 높여</b> 추론 지연 시간과 학습 시간을 더욱 단축시킴
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>형식 변환에 필요한 추가 계산이 오히려 시간을 더 소모할 수도 있긴 함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">단점</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>각 변환은 작은 값 변화를 일으키는 경우가 많음, 이 작은 변화들이 모여&nbsp;<b>큰 성능 차이</b>를 만들 수 있음</li>
<li>값이 낮아진 정밀도 형식의 표현 범위를 벗어나 무한대나 임의의 값으로 변환될 수 있음, 품질이 예상보다 더 떨어질 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델 성능에 미치는 영향을 최소화하면서 정밀도를 낮추는 방법은 활발히 연구되는 분야</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>낮은 정밀도 추론은 이제 표준
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>파이토치 PyTorch, 텐서플로 TensorFlow, 트랜스포머 Transformer 같은 주요 프레임워크는 몇 줄의 코드만으로 PTQ를 무료로 제공함</li>
<li>일부 엣지 디바이스는 양자화된 추론만 지원하여 텐서플로 라이트 TensorFlow Lite 및 파이토치 모바일 PyTorch Mobile 같은 온디바이스 추론용 프레임워크도 PTQ를 제공함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">학습 양자화</h4>
<p data-ke-size="size16">PTQ 만큼 보편적이지는 않지만 인기를 얻고 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>추론 과정에서 낮은 정밀도에서도 우수한 성능을 보이는 모델을 만드는 것</b> : 학습 후 양자화 과정에서 모델 품질이 저하될 수 있는 문제를 해결하기 위한 것</li>
<li><b>학습 시간과 비용을 줄이는 것</b> : 메모리 사용량을 줄여 더 저렴한 하드웨어에서 더 큰 모델을 학습할 수 있게, 계산 속도를 높여 비용도 추가로 절감</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>양자화 인식 학습 Quantization-aware training (QAT)</b>은 추론 시 낮은 정밀도에서도 품질이 좋은 모델을 만드는 것이 목표</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>학습 중에 낮은 정밀도 동작을 시뮬레이션하므로, 낮은 정밀도에서도 품질 높은 출력을 생성하도록 학습할 수 있음</li>
<li>계산이 여전히 높은 정밀도로 이루어지므로 모델의 학습 시간을 줄이지는 않음, 오히려 시뮬레이션하는 추가 작업으로 인해 시간이 늘어날 수 있음</li>
</ul>
<p data-ke-size="size16">처음부터 낮은 정밀도로 모델을 학습하도록 하면, 학습/서빙 간의 정밀도 불일치를 없애느 동시에 학습 효율까지 향상시킬 수 있음, 하지만 정밀도를 낮춰 학습하는 것은 역절파 과정이 낮은 정밀도에 더 민감하게 반응하여 어려움</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>혼합 정밀도 Mixed Precision</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>가중치 사본은 높은 정밀도로 유지하지만 그래디언트나 활성화 같은 다른 값은 낮은 정밀도로 유지하는 방식</li>
<li>덜 민감한 가중치 값은 낮은 정밀도로 계산하고 더 민감한 가중치 값은 높은 가중치로 계산할 수 있음</li>
<li><b>자동 혼합 정밀도 Automatic Mixed Precision (AMP)&nbsp;</b>: 기능을 통해 자동으로 설정할 수 있음</li>
</ul>
</li>
<li>학습 단계마다 다른 정밀도 수준을 사용하기
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>높은 정밀도로 학습하되 낮은 정밀도로 파인튜닝</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">파인튜닝 기법</h2>
<h3 data-ke-size="size23">파라미터 효율적 파인튜닝</h3>
<p data-ke-size="size16">전체 파인튜닝의 높은 메모리와 데이터 요구사항 때문에 부분 <b>파인튜닝 Partial finetuning</b>을 시작하게 되었음</p>
<p data-ke-size="size16">모델 파라미터의 일부만 업데이트되는 것&nbsp; (e.g. 마지막 레이어만 파인튜닝)</p>
<p data-ke-size="size16">메모리 사용량을 줄일 수 있지만 파라미터 효율성이 떨어짐, 전체 파인튜닝에 근접한 성능을 내기 위해 여전히 많은 학습 파라미터가 필요함 (전체 파라미터의 약 25%)</p>
<img width="1072" height="682" alt="image" src="https://github.com/user-attachments/assets/2d294d33-0c3d-4e81-9063-4b7d8fd3f402" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>파라미터 효율적 파인튜닝 PEFT</b> : 훨씬 적은 파라미터를 사용하면서도 전체 파인튜닝에 가까운 성능을 달성하기 위한 기법들</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>어댑터 모듈 추가한 후 어댑터만 업데이트 : 3%만으로 전체 파인튜닝과 성능 차이 0.4% 이내
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>단점 : 파인튜닝된 모델의 추론 지연 시간이 늘어남</li>
</ul>
</li>
</ul>
<img width="1086" height="723" alt="image" src="https://github.com/user-attachments/assets/319a347f-3bf6-4d62-af19-7668f959793e" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>PEFT 방법들은 샘플 효율적이기도 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">PEFT 기법들</h4>
<p data-ke-size="size16">크게 두 가지로 나눌 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>어댑터 기반 방법</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>모델 가중치에 추가 모듈을 붙이는 모든 방식</b> (부가적 방법이라고도 함)</li>
<li>LoRA, BitFit, IA3 (효율적인 혼합 작업 배치 전략 덕에 다중 작업 파인튜닝에 특히 유리함), LongLoRA (컨텍스트 길이를 늘리기 위해 어텐션 수정 기법을 결합한 LoRA의 변형)</li>
</ul>
</li>
<li><b>소프트 프롬프트 기반 방법</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>특별한 학습 가능한 토큰을 도입해 모델이 입력을 처리하는 방식을 바꿈</li>
<li>추가 토큰들은 입력 토큰과 함께 모델에 주입됨
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>하트 프롬프트(입력)과의 차이점
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>하드 프롬프트는 사람이 읽을 수 있지만, 소프트 프롬프트는 임베딩 벡터와 비슷한 연속적인 벡터로 사람이 읽을 수 없음</li>
<li>하드 프롬프트는 고정되어 있지만, <b>소프트 프롬프트는 튜닝 과정에서 역전파를 통해 최적화할 수 있어</b> 특정 작업에 맞게 조정할 수 있음</li>
</ul>
</li>
</ul>
</li>
<li>프리픽스 튜닝 prefix-tuning (모든 트랜스포머 레이어의 입력 앞에), P-튜닝, 프롬프트 튜닝 (임베딩된 입력 앞에만) : 소프트 프롬프트가 어디에 삽입되는지가 주요 차이</li>
</ul>
</li>
</ul>
<img width="987" height="543" alt="image" src="https://github.com/user-attachments/assets/01e31149-8694-4105-bf5b-18c6f710a652" />

<p data-ke-size="size16">&nbsp;</p>
<img width="1088" height="681" alt="image" src="https://github.com/user-attachments/assets/93ccf214-83ac-45e3-83b6-4a1d543adff7" />

<p data-ke-size="size16">LoRA의 높은 인기, 소프트 프롬프트는 상대적으로 덜 보편화되어 있지만, 프롬프트 엔지니어링보다 더 많은 커스터마이징을 원하면서도 파인튜닝에 투자하기를 꺼리는 사람들 사이에서 점점 더 관심을 끌고 있는 추세</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">LoRA, low-rank adaptation</h4>
<p data-ke-size="size16">추론 지연 시간을 추가로 발생시키지 않으면서 파라미터를 효과적으로 추가함</p>
<p data-ke-size="size16">기본 모델에 새로운 레이어를 추가하는 대신, <b>LoRA는 원래 레이어와 병합할 수 있는 모듈을 활용함</b></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">LoRA는 <b>개별 가중치 행렬</b>에 적용할 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>특정 가중치 행렬이 주어지면, 이 행렬을 두 개의 더 작은 행렬의 곱으로 분해하고, 작은 행렬들을 업데이트한 후 다시 원래 행렬로 병합하는 방식을 취함</li>
<li>$n \times m$ 차원의 가중치 행렬 $W$가 있다고 할때, LoRA는 다음과 같은 과정으로 작동함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>더 작은 행렬들의 차원 $r$을 선택, 여기서 $r$은 LoRA 랭크라고 부름</li>
<li>두개의 행렬 $ A(n\times r)$와 $B(r \times m)$, 이 행렬의 곱 $W_{AB}$는 원래 행렬 $W$와 동일한 차원을 가짐</li>
<li>$W_{AB}$를 원래 가중치 행렬 $W$에 더해 새로운 가중치 행렬 $W^\prime$를 생성함&nbsp;</li>
<li>모델에서는 $W$ 대신 $W^\prime$을 사용함, 하이퍼파라미터 $\alpha$를 통해 $W_{AB}$가 행렬에 얼마나 영향을 미칠지 조절할 수 있음 $$W^\prime = W + \frac{\alpha}{r}W_{AB}$$</li>
<li>파인튜닝 과정에서 $A, B$의 파라미터만 업데이트하고 W는 변경하지 않고 그대로 유지함&nbsp;</li>
</ul>
</li>
<li><b>저랭크 분해</b> 개념을 토대로 만들어짐
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>원래 행렬은 <b>풀랭크</b>를 가지는 반면 <b>두 개의 작은 행렬은 저랭크 근사치를 표현&nbsp;</b></li>
</ul>
</li>
</ul>
<img width="1088" height="781" alt="image" src="https://github.com/user-attachments/assets/b537a7ab-1298-4e01-b121-3069244dd96a" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">LoRA가 효과적인 이유</h4>
<p data-ke-size="size16">많은 연구에서 LLM이 수많은 파라미터를 가지고 있음에도 실제로는 매우 낮은 내재적 차원을 갖는다는 점을 보여줌</p>
<p data-ke-size="size16">사전 학습 과정이 자연스럽게 모델의 내재적 차원을 줄인다는 사실 또한 밝혀냄</p>
<p data-ke-size="size16"><b>더 큰 모델일수록 사전학습 후에 오히려 더 낮은 내재적 차원을 갖는 경향이 있음</b></p>
<p data-ke-size="size16"><b>이는 사전학습이 다운스트림 작업을 위한 일종의 압축 프레임워크 역할을 한다는 것을 의미</b></p>
<p data-ke-size="size16">LLM이 더 잘 학습될수록, 적은 수의 파라미터와 소량의 데이터만으로도 모델을 효과적으로 파인튜닝하기가 더 쉬워짐</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">많은 연구자들이 <b>저랭크 사전학습</b>은 모델의 파라미터 수를 대폭 줄여 학습 시간과 비용을 크게 절감할 수 있다고 생각했음</p>
<p data-ke-size="size16">연구를 통해 저랭크 분해는 소규모에서는 효과적임이 입증되었음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">저랭크 사전학습을 확장하는 방법을 개발할 가능성이 있으나, 저랭크 분해가 효과적으로 작동할 수 있는 지점까지 모델의 내재적 차원을 충분히 줄이기 위해서는 여전히 풀랭크 사전 학습이 필요함</p>
<p data-ke-size="size16">저랭크 학습 전환 전 얼마만큼의 풀랭크 학습이 필요한지 연구한다면 매우 흥미로운 결과를 얻을 수 있을 것</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">LoRA 구성</h4>
<p data-ke-size="size16">어떤 행렬에 LoRA를 적용하는지와 모델의 아키텍처에 LoRA의 효율성이 좌우됨</p>
<p data-ke-size="size16">다른 아키텍처에 LoRA를 적용한 연구들이 있지만 LoRA는 주로 트랜스포머 모델에 사용되고 있음</p>
<p data-ke-size="size16">보통 <b>어텐션 모듈의 쿼리, 키, 값, 출력 투영 행렬 ($W_q, W_k, W_v, W_o$)</b>에 적용됨</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>GPT-3-175B를 파인튜닝할 때 학습 가능한 파라미터를 1,800만 개로 설정 (전체 파라미터의 0.01%), 이 파라미터로 여러 방식의 LoRA를 적용할 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>랭크가 8인 하나의 행렬</li>
<li>랭크가 4인 두 개의 행렬</li>
<li>랭크가 2인 네 개의 모든 행렬 (해당 모델 차원은 12,288, 이 방식으로 하면 레이어마다 (12,288 * 2 * 2) * 4 = 195,608의학습 가능한 파라미터, 전체 18,874,368개의 학습 가능한 파라미터)</li>
</ul>
</li>
<li><b>랭크가 2인 네 개의 모든 행렬에 LoRA를 적용했을 때 최고의 성능</b>을 얻었다고 밝힘</li>
<li>어텐션 행렬 중 두 개만 선택해야 하는 상황이라면, <b>쿼리와 값 행렬을 택하는 것이 일반적으로 가장 효과적</b>이라고 제안함</li>
<li>피르포워드 행렬을 포함해 더 많은 가중치 행렬에 LoRA를 적용할수록 더 좋은 결과를 얻을 수 있음</li>
</ul>
<img width="1085" height="322" alt="image" src="https://github.com/user-attachments/assets/6613cad6-c5c4-4ba3-875c-a140abe9ad56" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>피드포워드 기반 LoRA, 어텐션 기반 LoRA는 서로 보완적인 역할을 할 수 있지만, <b>메모리 제약이 있는 환경에서는 보통 어텐션 LoRA가 더 효과적이라고 함</b></li>
<li>랭크에 따라 성능이 달라지긴 하지만, 4에서 64 정도의 작은 r 값만으로도 대부분의 활용 사례에서 충분함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>r 값을 늘려도 파인튜닝 성능이 개선되지 않음을 발견함, 일부는 r값이 너무 높으면 과적합 때문에 성능이 떨어질 수 있다고 주장함 (아닌 경우도 있긴 함)</li>
</ul>
</li>
<li>$\alpha$ 값은 실무에서는 $\alpha : r$ 비율을 보통 1:8에서 8:1 사이로 설정하지만 경우에 따라 다름, 여러 실험을 거쳐 찾아야 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">LoRA 어댑터 서빙</h4>
<p data-ke-size="size16">LoRA로 파인튜닝된 모델을 서빙하는 방법 두 가지</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델 서빙 전 LoRA 가중치 A, B를 원본 모델에 <b>미리 병합해서 새로운 행렬 $W^{\prime}$을 만듦</b>, 추론할 때 추가 연산이 필요하지 않아서 지연 시간도 늘어나지 않음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>서빙할 LoRA 모델이 하나뿐인 경우 좋음</b></li>
</ul>
</li>
<li>서빙 시 <b>W, A, B 가중치를 각각 따로 유지. 추론 과정에서 A와 B를 W에 병합</b>해야 하므로 지연 시간이 늘어남
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>멀티 LoRA 서빙인 경우 좋음</b></li>
<li>저장 공간 대폭 절약,&nbsp; 풀랭크 행렬 $W^{\prime}$을 모델마다 저장하지 않고 작은 행렬들만 저장
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>작업을 바꿀때도 필요한 LoRA 어댑터만 로딩하면 되므로 로딩 시간을 크게 줄일 수 있음</li>
<li>A와 B를 분리해서 유지하면 지연 시간이 늘어나지만 최소화하는 최적화 기법들이 있음</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1016" height="488" alt="image" src="https://github.com/user-attachments/assets/abe7b416-d385-4b9f-a37a-4c290cb7fe65" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>멀티 LoRA 서빙을 활용하면 전문화된 여러 모델을 쉽게 결합할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>여러 작업을 처리하는 강력한 모델 하나 대신, 작업마다 LoRA 어댑터를 따로 만들 수 있음</li>
</ul>
</li>
<li>모듈성 덕분에 어댑터를 쉽게 공유하고 재사용할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>공개된 파인튜닝된 LoRA 어댑터들을 사용할 수 있음, 이는 사전 학습 모델을 사용하는 것과 같음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>LoRA의 단점</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>전체 파인튜닝만큼 강력한 성능을 제공하지 못한다는 점</li>
<li>모델의 구현을 수정해야하므로 아키텍처에 대한 이해와 코딩 기술이 필요해 전체 파인튜닝보다 더 어렵다는 단점도 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>이런 문제는 덜 대중적인 기본 모델에서만 발생</li>
</ul>
</li>
<li>PEFT, Axolotl, unsloth, LitGPT 같은 PEFT 프레임워크들은 대부분 인기 있는 기본 모델에 대해서 LoRA를 별도 설정 없이 바로 사용할 수 있게 지원함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">양자화된 LoRA</h4>
<p data-ke-size="size16">다양한 LoRA 변형들이 개발됨</p>
<p data-ke-size="size16">LoRA 어댑터가 사용하는 메모리는 모델 가중치에 비해 매우 작고, <b>LoRA 파라미터를 줄여봤자 전체 메모리 사용량은 거의 줄어들지 않음</b></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">LoRA의 파라미터수를 줄이는 것보다는, 파인튜닝할 때 모델의 가중치, 활성화, 그래디언트를 양자화하는 편이 메모리를 훨씬 효과적으로 절약할 수 있음&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>QLoRA : 양자화된 LoRA</b></p>
<p data-ke-size="size16">기존 LoRA 논문에서는 파인튜닝 시 모델 가중치를 16비트로 저장함, QLoRA는 4비트로 저장했다가 순전파와 역전파를 계산할 때만 다시 BF16으로 역양자화해서 사용</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">QLoRA가 사용하는 4비트 형식은 NF4 NormalFloat-4로 사전 학습된 가중치는 대개 중앙값이 0인 정규 분포를 따른다는 사실을 활용해 값을 양자화함</p>
<p data-ke-size="size16">4비트 양자화에 더해 QLoRA는 특히 롱 시퀀스 길이에서 GPU 메모리가 부족할 때 CPU와 GPU 사이에 데이터를 자동으로 전송하는 <b>페이징 최적화 도구 paged optimizer</b>를 사용함</p>
<p data-ke-size="size16"><b>과나코 Uanaco </b>: QLoRA 연구자들이 라마 7B부터 65B까지 다양한 모델을 4비트 모드로 파인튜닝한 것</p>
<img width="1080" height="558" alt="image" src="https://github.com/user-attachments/assets/51efcc18-a42a-45f2-a3e3-86b3c08da187" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>한계 :&nbsp;</b>NF4 양자화 과정에 비용이 많이 든다는 점, 양자화하고 되돌리는 과정에서 시간이 더 걸려 학습시간이 늘어날 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">QA-LoRA, ModuLoRA, IR-QLoRA 등 관련 연구들이 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">모델 병합과 다중 작업 파인튜닝</h3>
<p data-ke-size="size16"><b>모델 병합</b> : 여러 모델을 결합하여 맞춤형 모델을 만드는 방법</p>
<p data-ke-size="size16">파인튜닝만으로는 얻을 수 없는 더 큰 유연성을 제공함 (병합 전에 각각 파인튜닝해도 됨)</p>
<p data-ke-size="size16">병합한 모델을 추가 파인튜닝할 필요는 없지만, 대부분의 경우 파인튜닝을 거치면 성능이 더 좋아짐, 파인튜닝 없이도 모델 병합 자체는 GPU없이 할 수 있기 때문에 개인 모델 개발자들에게 매력적인 선택지</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델 병합의 핵심은 개별 모델들을 따로따로 쓰는 것보다 <b>훨씬 유용한 하나의 통합 모델</b>을 만드는 데 있음&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>성능</b> 면에서 각기 다른 강점을 가진 두 모델을 합쳐 더 뛰어난 성능을 보이는 단일 모델로 병합할 수 있음</li>
<li><b>메모리 사용량</b>을 줄여서 비용을 절약할 수 있음, 더 적은 파라미터로 두 작업을 모두 처리하는 하나의 모델로 통합 (어댑터 기반 모델에서 특히 효과적, 같은 기본 모델에서 파생된 두 파인튜닝 모델의 각각의 어댑터를 하나로 합침)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>다중 작업 파인튜닝 </b>사례에서 모델 병합이 빛을 발함</p>
<p data-ke-size="size16">모델 병합 기법 없이 여러 작업에 대해 모델을 파인튜닝하려면 다음과 같은 방법들 중 선택해야함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>동시 파인튜닝</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>모든 작업에 대한 예시를</b> <b>하나의 데이터셋</b>에 담아서 <b>모델이 모든 작업을 동시에 학습</b>하는 방법</li>
<li>여러 작업을 동시에 학습하는 것이 보통 더 어렵기 때문에, 더 많은 데이터와 더 많은 학습이 필요함</li>
</ul>
</li>
<li><b>순차 파인튜닝</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>각 작업을 하나씩 순차적으로 파인튜닝</li>
<li>신경망의 <b>재앙적 망각 catastrophic forgetting</b>&nbsp;현상에 취약함</li>
</ul>
</li>
<li>서로 다른 작업에 대해 각각 파인튜닝을 병렬로 진행한 다음, 서로 다른 모델들을 병합하는 <b>모델 병합</b> : 각 작업을 개별적으로 파인튜닝해 더 특화된 성능을 얻고, 순차적 학습이 아니기 때문에 재앙적 망각 문제도 훨씬 적음</li>
<li>메모리 용량이 제한되어 있는 <b>온디바이스 배포</b>에서도 유용함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>서로 다른 작업을 위한 모델 여러 개를 기기에 욱여넣는 대신, 이 모델들을 하나로 합쳐서 <b>메모리는 훨씬 적게 쓰면서도 여러 작업을 수행할 수 있는 하나의 모델로 병합</b></li>
<li>온디바이스 배포는 인터넷 연결이 제한되거나 신뢰할 수 없는 경우 필요함, 추론 비용을 크게 줄일 수 있음 (사용자 기기가 처리할 수 있는 연산이 많아질수록, 데이터 센터 비용도 줄어듦)</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>모델 병합</b>은 <b>연합 학습 federated learning</b>을 수행하는 한 가지 방법 : 여러 기기가 별도의 데이터로 같은 모델을 학습시킴</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>여러 기기에 모델 X를 배포하면, 각 기기의 X가 온디바이스 데이터를 사용해 <b>독립적으로 학습</b>할 수 있음</li>
<li>시간이 흐르면 <b>서로 다른 데이터로 학습된 X 버전</b>들이 여러개 생기게 됨</li>
<li>이런 버전들을 모두 합쳐 (조금씩 바뀐 모델들의 파라미터를 평균내거나 해서) 모든 구성 모델의 학습을 포함하는 하나의 <b>새로운 기본 모델</b>을 만들어 낼 수 있음</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">여러 모델을 합쳐서 더 좋은 성능을 얻으려는 시도는 <b>모델 앙상블 방법 model ensemble method</b>에서 출발함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>개별 학습 알고리즘 하나로는 불가능한 더 나은 예측 성능을 얻기 위해 <b>여러 학습 알고리즘을 결합</b>하는 기법</li>
<li><b>모델 병합</b>은 <b>구성 모델의 파라미터를 섞어서 만드는 방식</b>, 앙상블은 구성 모델을 놔두고 출력만 결합하는 방식</li>
</ul>
<img width="1088" height="539" alt="image" src="https://github.com/user-attachments/assets/6cfaa51a-d8fd-4602-b6c8-7abe1cc8c0db" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델 병합 기법은 실험 단계인 것들이 많아, 큰 틀에서만 살펴보자</p>
<img width="1071" height="586" alt="image" src="https://github.com/user-attachments/assets/3d6bb2db-f4a3-423b-bb1a-e0948f59296b" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20"><b>합산</b></h4>
<p data-ke-size="size16">구성 모델들의 <b>가중치 값들을 더하는 방식</b></p>
<h4 data-ke-size="size20">선형 결합</h4>
<p data-ke-size="size16">평균과 가중 평균을 모두 포함</p>
<p data-ke-size="size16">모델 A, B가 주어졌을 때, 이들의 가중 평균은 다음과 같음</p>
<img width="343" height="119" alt="image" src="https://github.com/user-attachments/assets/fd7bd454-c887-4c3a-9554-6abbdf2b223f" />

<img width="1069" height="333" alt="image" src="https://github.com/user-attachments/assets/ed466a0e-5733-456a-977e-136fe7d5d987" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>모델 수프 Model soup</b> : 여러 파인튜닝된 모델의 가중치를 통째로 평균내면 추론 시간은 그대로 두면서도 정확도를 향살시킬 수 있다는 것을 보여줌</p>
<p data-ke-size="size16">어댑터 같은 특정 구성 요소만 선형으로 결합해서 모델을 병합하는 방식이 더 일반적임</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">어떤 모델 세트든 선형적으로 결합할 수 있지만, <b>같은 기본 모델에서 파인튜닝된 모델들끼리 결합</b>할 때 효과가 가장 좋음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>작업 벡터 task vector 개념으로 선형 결합을 설명할 수 있음, 델타 파라미터라고도 불림, LoRA로 파인튜닝했다면 LoRA 가중치로 작업 벡터를 만들 수 있음</li>
<li>작업 벡터가 있으면 <b>작업 산술 task arithmetic</b>을 할 수 있음 (벡터를 더해 결합, 특정 능력 제거, 편향이나 사생활 침해 소지가 있는 민감한 기능 등 원치 않는 동작을 제거)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">아키텍처나 크기가 같다면 선형 결합은 간단함, 구성 요소가 다른 모델들에도 적용할 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>한 모델의 특정 레이어가 더 크다면, 둘 중 하나 또는 둘 다 동일한 차원으로 투영하여 결합</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">구면 선형 보간법 (SLERP)</h4>
<p data-ke-size="size16"><b>보간 interpolation</b> : 모델 병합의 측면에서 모르는 값이 병합될 모델, 알고있는 값이 구성 모델. 선형 결합도 보간 기법 중 하나</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>병합할 각 구성 요소를 구 위의 점이라 생각하고, 두 벡터를 병합할 때 구의 표현을 따라 두 점 사이의 최단거리를 구함. 벡터를 병합한 결과는 최단 경로상의 어떤 한 점이 됨, 어떤 지점이 될지는 보간인자에 따라 결정됨</li>
<li>두 개의 벡터에 대해서만 정의되어 있으며 세 개 이상을 병합하고 싶다면 SLERP를 순차적으로 수행하면 됨</li>
</ul>
<img width="1106" height="423" alt="image" src="https://github.com/user-attachments/assets/1738dd36-aea9-4fb7-a8b9-865c853ee8ab" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">불필요한 파라미터 가지치기</h4>
<p data-ke-size="size16">파인튜닝을 하는 동안 조정되는 파라미터는 많지만, 대부분은 미미하며 성능에 크게 도움이 되지 않음</p>
<p data-ke-size="size16"><a href="https://arxiv.org/abs/2306.01708" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2306.01708</a> : 작업 벡터의 상당 부분을 <b>재설정 resetting해도 성능이 거의 떨어지지 않음을 증명</b></p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>재설정 resetting : 파인튜닝된 파라미터를 원래 값으로 되돌리는 것, 작업 벡터 파라미터를 0 (작업 벡터 = 파인튜닝된 모델 - 기본 모델)
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>불필요한 파라미터들이 개별 모델에서는 해가 되지 않지만, 여러 모델 병합 시 문제가 될 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>작업 벡터를 병합하기 전에 불필요한 파라미터 가지치기</li>
<li>병합할 모델이 많을 수록 중요</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1081" height="385" alt="image" src="https://github.com/user-attachments/assets/3adb236f-ddb5-4558-8e4f-9ebc3fe67a33" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">레이어 쌓기</h4>
<p data-ke-size="size16">여러 모델에서 서로다른 레이어를 가져와 쌓아올리는 방법</p>
<p data-ke-size="size16"><b>패스스루 passthrought, 프랑켄머징 frankenmerging</b></p>
<p data-ke-size="size16">기존에 없던 고유한 아키텍처와 파라미터 수를 가진 모델을 만들 수 있음</p>
<p data-ke-size="size16">합산과 달리 제대로 된 성능을 내려면 보통 추가 파인튜닝이 필요</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>골리앗-120B : 파인튜닝된 라마 2-70B 모델 두 개, Xwin과 Euryale를 합쳐서 만들어짐</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>전문가 혼합 (MoE) 모델</b>을 학습할 때도 활용할 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/2212.05055">https://arxiv.org/abs/2212.05055</a></li>
<li>&nbsp;MoE를 밑바닥부터 학습시키는 대신, 사전학습된 모델을 가져와 특정 레이어나 모듈을 여러개 복사</li>
<li>라우터를 추가해 각 입력을 가장 적합한 복사본으로 내보냄</li>
<li>병합된 모델과 라우터를 추가 학습</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<img width="1089" height="701" alt="image" src="https://github.com/user-attachments/assets/c04b4c60-d6df-469c-a8a2-c089407e178d" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>투게더 AI : 오픈 소스 모델 여섯 개를 조합해 <b>에이전트 혼합 mixture-of-agents</b>을 만들었음, 일부 벤치마크에서 GPT-4o에 필적하는 성능 달성</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>모델 업스케일링 : </b>적은 자원으로 더 큰 모델을 만드는 방법을 연구하는 분야
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>Depthwise scaling</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>사전 학습된 모델 복사</li>
<li>두 복사본을 병합하는데, 일부 레이어는 합치고 (두 레이어를 하나로), 나머지는 그냥 쌓음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>어떤 레이어를 합칠지는 목표 모델 크기에 맞춰서 결정 (SOLAR 10.7B는 32개 레이어를 가진 7B 파라미터 모델로 10.7B 모델을 만들었음, 16개 레이어를 합쳐서 최종적으로 32 * 2 - 16 = 48개 레이어가 되도록 함)</li>
</ul>
</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1085" height="621" alt="image" src="https://github.com/user-attachments/assets/534f89fb-950e-4b81-ae31-fb148935fdd9" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">연결</h4>
<p data-ke-size="size16">그냥 연결하는 방법 : 병합된 구성 요소의 파라미터 개수는 모든 구성 요소의 파라미터를 다 합친 것과 같음</p>
<img width="1044" height="514" alt="image" src="https://github.com/user-attachments/assets/a7e92e8e-32e2-45be-920c-cf6e4303154c" />


<p data-ke-size="size16">모델을 각각 따로 서빙하는 것과 비교해 메모리 사용량이 전혀 줄어들지 않음</p>
<p data-ke-size="size16">성능이 더 좋아질 수는 있지만, 늘어나는 파라미터 수에 비해 그 정도 성능 향상은 가치가 없을 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">파인튜닝 전술</h3>
<p data-ke-size="size16">실용적인 파인튜닝 노하우&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">파인튜닝 프레임워크와 기본 모델</h4>
<p data-ke-size="size16">파인튜닝에서 결정해야 할 3가지</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>기본 모델</b></li>
<li><b>파인튜닝 방법</b></li>
<li><b>파인튜닝 프레임워크</b></li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">기본 모델</h4>
<p data-ke-size="size16">프로젝트에 따라 시작 모델이 제각각임</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">두 가지 개발 방향</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>진행 경로 progression path</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>가장 저렴하고 빠른 모델로 파인튜닝 코드를 테스트</li>
<li>가지고 있는 데이터를 확인해보기 위해, 중간급 모델을 파인튜닝, 파인튜닝에 사용되는 데이터를 늘림에 따라 학습 손실이 떨어지지 않으면 뭔가 잘못된 것</li>
<li>최고로 성능이 좋은 모델로 몇 가지 실험을 더 돌려서 성능을 어디까지 끌어올릴 수 있는지 확인</li>
<li>좋은 결과가 나오면 모든 모델로 학습을 진행해, 비용 대비 성능 현황을 파악, 활용 사례에 적합한 모델 선택</li>
</ul>
</li>
<li><b>증류 경로 distillation path</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>작은 데이터셋으로 감당할 수 있는 가장 강력한 모델을 학습시킴<br />
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>기본 모델이 이미 강력하므로, 좋은 성능을 달성하기 위해 소량의 데이터만 필요함</li>
</ul>
</li>
<li>이 파인튜닝된 모델을 사용해 더 많은 학습 데이터를 생성</li>
<li>새로운 데이터셋으로 더 저렴한 모델을 학습시킴</li>
</ul>
</li>
<li>프롬프트 엔지니어링 실험 결과에서 이해한 다양한 모델들의 동작 방식을 바탕으로 개발 계획을 세워야 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">파인튜닝 방법</h4>
<p data-ke-size="size16">LoRA 같은 어댑터 기법은 비용 효율적이지만 전체 파인튜닝만큼 성능이 나오지는 않음 $\rightarrow$ LoRA로 시작 후, 전체 파인튜닝 시도</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">데이터 양에 따라서 파인튜닝 방법이 달라질 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>전체 파인튜닝은 최소 수천 개의 예시, 대개 훨씬 더 많이 필요함</li>
<li>PEFT 방법들은 훨씬 적은 데이터로도 괜찮은 성능을 낼 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>수백 개 정도의 작은 데이터셋이 있다면 LoRA가 나을 수도</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파인튜닝된 모델이 몇 개나 필요한지, 어떻게 서빙하고 싶은지 생각해 봐야 함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>어댑터 방법은 전체 모델 하나만 서빙하면 되지만, 전체 파인튜닝은 전체 모델을 여러개 서빙해야 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">파인튜닝 프레임워크</h4>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>파인튜닝 API 사용
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>데이터 올리기와 기본 모델 선택을 하면 파인튜닝된 모델을 받을 수 있음</li>
<li>여러 업체에서 제공하고 편리하지만 API가 지원하는 기본 모델로만 작업할 수 있다는 한계가 있음</li>
<li>최적의 파인튜닝을 위한 세부 설정이 노출되지 않음</li>
</ul>
</li>
<li>LlaMA-Factory, unsloth, PEFT, Axolotl, LitGPT : 여러 파인튜닝 프레임워크 중 하나를 사용해 파인튜닝할 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>다양한 파인튜닝 방법 지원 (특히 어댑터 기반)</li>
</ul>
</li>
<li>전체 파인튜닝 : 깃허브에 오픈 소스 학습 코드가 제공된 기본 모델들 (Llama Police에 파인튜닝 프레임워크, 모델 저장소에 대한 광범위한 최신 정보가 저장되어 있음)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">직접 파인튜닝을 하려면 필요한 컴퓨팅 자원을 직접 준비해야 함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>어댑터 기반 방법만 사용 : 중간 등급의 GPU로도 대부분의 모델에 충분</li>
<li>더 많은 컴퓨팅이 필요한 경우, 클라우드 업체와 매끄럽게 연동되는 프레임워크를 선택할 수 있음</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">분산 학습 프레임워크 : 딥스피드, PyTorch Distributed, Colo ssalAI</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">파인튜닝 하이퍼파라미터</h4>
<p data-ke-size="size16">여기서는 자주 나오는 중요한 것들 몇가지를 소개, 구체적인 활용 사례에 맞는 하이퍼파라미터는 기본 모델이나 파인튜닝 프레임워크 문서에서 찾아보기</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">학습률 learning rate</h4>
<p data-ke-size="size16">학습 단계에서 모델의 파라미터가 얼마나 빨리 변할지 결정 (보폭)</p>
<p data-ke-size="size16">모든 상황에 통하는 최적 학습률은 없음 : 1e-7에서 1e-3 사이에서 여러가지 실험 필요</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>흔히 쓰는 방법으로 사전 학습 단계 끝에서 사용한 학습률에 0.1에서 1사이의 값을 곱함</li>
<li>손실 곡선을 단서를 얻기
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>변동이 심하다면 학습률이 너무 큰 것</li>
<li>안정적이지만 오래걸리면 너무 작은 것</li>
</ul>
</li>
<li>학습 과정 중에 변경하면서도 쓸 수 있으며, 이를 결정하는 알고리즘을 학습률 스케쥴이라고 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">배치크기</h4>
<p data-ke-size="size16">모델이 가중치 업데이트할 때 각 단계에서 몇 개의 예시로 학습할지 결정함, 크기가 너무 작으면 학습이 불안정해질 수 있음</p>
<p data-ke-size="size16">배치 크기가 클수록 모델이 학습 예시들을 더 빠르게 처리할 수 있으나 많은 메모리를 필요로 함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>비용과 효율성 사이의 딜레마</li>
<li><b>그래디언트 누적 gradient accumulation</b> : 배치마다 모델 가중치를 업데이트하는 대신, 여러 배치에 걸쳐 그래디언트를 모아두었다가 신뢰할 수 있는 그래디언트가 쌓이면 모델 가중치를 업데이트
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>역전파를 위해 들고있는 중간 계산값(레이어별 중간 출력값)이 메모리를 가장 많이 잡아먹음, GA 시 이 값들을 버릴 수 있음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">컴퓨팅 비용이 크게 중요한 요소가 아니라도, 다양한 배치 크기를 실험해서 어떤 것이 최고의 성능을 내는지 확인하는 것이 좋음</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">에폭 수</h4>
<p data-ke-size="size16">각 학습 예시가 몇 번 학습되는지를 결정</p>
<p data-ke-size="size16">작은 데이터셋이 큰 데이터셋보다 더 많은 에폭이 필요할 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>수백만 개면 1~2 에폭, 수천개면 4~10 에폭 후에도 성능이 좋아질 수 있음</li>
</ul>
<p data-ke-size="size16">학습 손실과 검증 손실의 차이를 보고 결정할 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>둘다 꾸준히 감소하면 에폭 수를 늘려서 (더 많은 데이터를 확보해서) 학습하면 성능이 개선될 수 있음</li>
<li>검증 손실은 증가한다면 과적합이므로 에폭 수를 줄여보는게 좋음</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">프롬프트 손실 가중치</h4>
<p data-ke-size="size16">지시 파인튜닝에서 에시가 프롬프트와 응답으로 구성되며, 학습 과정에서 프롬프트와 응답 모두 모델 손실에 영향을 줄 수 있음</p>
<p data-ke-size="size16">하지만 추론 과정에서는 사용자가 프롬프트를 제공하고 모델은 응답만 생성하면 됨, 응답 토큰을 더 중요하게 여겨서 학습해야 함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>프롬프트 모델 가중치</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>응답에 비해 프롬프트가 손실 계산에 얼마나 반영될지 결정함</li>
<li>100% $\rightarrow$ 프롬프트와 응답이 손실 계산에 동등하게 반영되며, 모델을 둘 모두 똑같이 학습하는 것을 의미함</li>
<li>0$ <span style="color: #333333; text-align: start;">$\rightarrow$<span> 모델은 응답에서만 학습</span></span></li>
<li><span style="color: #333333; text-align: start;"><span>기본적으로 10%로 설정함, 프롬프트에서도 일부 정보를 학습하지만 주로 응답에서 학습해야 함</span></span></li>
</ul>
</li>
</ul>
