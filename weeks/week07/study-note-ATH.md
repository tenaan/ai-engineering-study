- [블로그 링크](https://armugona.tistory.com/entry/AIE-Ch8-%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B-%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%A7%81)
---
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>데이터셋 엔지니어링의 목표 : 최고의 모델을 학습할 수 있는 데이터셋을 만드는 것</b></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델 중심 AI model-centric AI : 모델 자체를 개선해서 AI 성능을 올리는 방식, 새로운 아키텍처를 설계하거나, 모델을 더 크게 만들거나, 새로운 학습 기법을 개발하는 방식</p>
<p data-ke-size="size16">데이터 중심 AI data-centric AI : 데이터를 개선해서 AI 성능을 올리는 방식, 새로운 데이터 처리 기법을 개발하고 고품질 데이터셋을 만들어서 더 적은 자원으로도 더 좋은 모델을 학습시키는 방식</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>딥러닝 초기에는 같은 데이터셋으로 최대한 좋은 모델을 만들려했지만, 최근에는 같은 모델이 주어졌을때, 해당 모델이 최고 성능을 낼 수 있는 데이터셋을 만드는 것</li>
<li>데이터 중심 경진대회, 데이터 중심 벤치마크</li>
<li>모델 중심, 데이터 중심으로 연구 방향을 잡는 것은 유용하지만 실제로 의미있는 기술 발전을 위해서는 모델과 데이터 개선 모두에 투자해야 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">이 장에서는 사후 학습 데이터에 집중함</p>
<hr contenteditable="false" data-ke-type="horizontalRule" data-ke-style="style6" />
<h2 data-ke-size="size26">데이터 큐레이션</h2>
<p data-ke-size="size16">나쁜 데이터는 모델의 편향과 환각을 키워 모델을 망가트리고 자원이 낭비될 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>데이터 큐레이션 data curation</b></p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델이 어떻게 학습하는지, 학습에 도움이 되는 자원이 무엇인지 이해하는 분야</li>
<li><b>어떤 데이터가 필요한 지는 하려는 일과 모델에게 무엇을 가르치고 싶은지에 달려 있음</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>자기 지도 학습 파인튜닝 : 데이터 시퀀스, 지도 파인튜닝 : (지시) 응답 형식 데이터, 선호도 파인튜닝 : (지시, 선호 응답, 비선호 응답) 형식, 보상 모델 학습 : 선호도 파인튜닝과 동일 혹은 ((지시, 응답), 점수) 형식</li>
</ul>
</li>
<li>학습 데이터는 모델이 학습해야할 행동을 보여줘야함 : 복잡한 행동을 가르치기는 더 어려움
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>CoT</b> : 다른 지시 데이터셋에 비해 CoT 데이터셋을 만들기 어려움</li>
<li><b>도구 사용</b> : 도구 사용 예시를 보여주면 도구 사용 능력을 키울 수 있음, 도구 사용 데이터는 보통 도메인 전문가를 활용 (하지만 사람이 만든 주석이 AI에겐 최적이 아닐 수 있음), 해당 작업을 수행하는데 필요한 일련의 행동으로 구성됨, 특별한 형식이 필요할 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>(도구 사용 시 AI가 한 턴에 여러 메시지를 만들어야 할 수도 있고, 각 메시지는 다른 곳으로 보내짐, e.g. 코드 인터프리터에 하나의 메시지를 보내고 사용자에게 다른 메시지 보내기, 라마 3 연구자들은 각 메시지의 출처와 목적지를 표시하는 메시지 헤더와 사람과 AI의 턴이 어디서 시작하는지 알려주는 특별한 종료 토큰으로 구성된 멀티 메시지 채팅 형식을 설계함)</li>
</ul>
</li>
<li>대화 인터페이스가 있는 애플리케이션 : 싱글 턴 데이터가 필요한지, 멀티 턴 데이터가 필요한지, 둘다 필요한지</li>
</ul>
</li>
</ul>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터 큐레이션은 모델이 나쁜 행동을 잊게 하려고 <b>기존 데이터를 없애는 것도 포함됨</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>e.g. 학습 데이터에서 요청하지 않은 제안을 포함한 주석 예시</li>
</ul>
</li>
<li>학습 단계가 달라져도 다른 데이터 조합이 필요함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>데이터 큐레이션의 전체적인 세 가지 기준</b></p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>데이터 품질 data quality</b></li>
<li><b>데이터 커버리지 data coverage</b></li>
<li><b>데이터 양 data quantity</b></li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 품질</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>적은 양의 고품질 데이터 &gt;</b> 많은 양의 노이즈가 있는 데이터 (관련없음, 일관성없음)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터 예시가 너무 적어도 상용모델만큼 강건하지는 않음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>고품질 데이터의 특성</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>관련성 Relevant</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델이 학습하려는 작업과 관련이 있어야 함</li>
</ul>
</li>
<li><b>작업 요구사항 부합 aligned with task requirement</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>주석은 작업의 요구사항에 부합해야 함, 사용자가 원하는 것이어야 함</li>
<li>사실적 일관성이 필요하다면 주석이 사실적으로 정확해야 함, 창의성이 필요하면 창의적이어야 함, 점수 뿐만아니라 근거를 요구하면 근거도 포함되어야 함, 간결한 답을 요구하면 주석이 간결해야 함</li>
</ul>
</li>
<li><b>일관성 consistent</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>주석의 예시끼리, 주석자들 간에 일관되어야 함</li>
</ul>
</li>
<li><b>올바른 형식 correctly formatted</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모든 예시는 모델이 기대하는 형식을 따라야 함, 불필요한 형식 토큰은 모델 학습을 방해함</li>
</ul>
</li>
<li><b>충분한 고유성 unique</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>고유한 예시가 필요함, 중복은 편향을 만들고 데이터 오염을 일으킬 수 있음</li>
</ul>
</li>
<li><b>규정 준수 compliant</b><b></b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터는 모든 관련 내부 및 외부 정책을 지켜야 함, 학습에 사용할 수 없는 데이터가 있다면 데이터에 들어가선 안 됨</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 커버리지</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>데이터 다양성 data diversity</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>학습 데이터는 모델이 풀어야 할 문제들의 모든 범위를 포괄해야 함</li>
<li><b>다양한 사용 패턴을 담은 데이터</b>를 확보하는 것이 모델이 좋은 성능을 내는 핵심</li>
<li>좋은 커버리지를 확보하려면 데이터 다양성이 필수적임</li>
</ul>
</li>
<li>짧은 지시, 자세한 지시 모두 포함되어야 함</li>
<li>질의에 오타가 있다면 오타가 포함된 예시를 넣어야 함&nbsp;</li>
<li>여러 프로그래밍 언어를 다루면 사용자들이 관심있어하는 프로그래밍 언어들이 포함되어야 함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>애플리케이션마다 다양성의 종류가 다름</li>
</ul>
</li>
<li>챗봇 같은 범용 용도의 경우 파인튜닝 데이터가 다양해야 하고, 광범위한 주제와 말하기 패턴을 담아야 함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>라마 3의 성능 향상은 <b>주로 데이터 품질과 다양성 개선, 늘어난 학습 규모에 의해 이뤄짐&nbsp;</b>(아키텍처 면에서는 이전 버전들과 크게 다르지 않음)</li>
<li>각 단계에서 실제로 포함되는 도메인의 종류와 비중이 다름 (세부 주제는 포함하지 않고 상위 수준의 도메인만 보여줌)</li>
<li>사후 학습에서는 표에는 없지만 토큰 수나 턴 수와 같은 것들도 고려, 이때 합성 데이터가 사용되므로 사람이 생성한 데이터와 AI가 생성한 데이터의 비율 또한 중요한 요소로 고려됨</li>
<li>사전 학습과 지도 파인튜닝에서 수학, 추론, 코드 토큰을 모두 합치면 학습 데이터의 절반을 차지함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>라마 3 연구자들은 고품질 코드와 수학 데이터로 모델을 어닐링 annealing (학습률을 점진적으로 낮추면서 코드와 수학 데이터를 점진적으로 늘려가며 모델을 학습시킴) 하면 주요 벤치마크에서 모델 성능을 높일 수 있다고 밝힘</li>
<li>고품질 코드와 수학 데이터가 자연어 텍스트보다 모델의 추론 능력을 키우는 데 더 효과적이라는 통념을 뒷받침</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1280" height="485" alt="image" src="https://github.com/user-attachments/assets/90333a69-373a-429c-9a36-8e3ee59fb741" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>올바른 데이터 조합은 어떻게 결정?
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>실제 애플리케이션 사용 패턴을 맞춰 데이터 조합을 선택</li>
<li>실험을 통해 최적의 데이터 조합 찾기</li>
</ul>
</li>
<li>특성이 다른 세 개의 데이터셋으로 70억 파라미터 언어 모델을 학습시킨 실험
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>고품질, 다양하지 않음 / 저품질, 다양함 / 고품질, 다양함</li>
</ul>
</li>
</ul>
<img width="1280" height="766" alt="image" src="https://github.com/user-attachments/assets/d8e8b7f9-ba2b-4d9d-980b-89a5c6a7bfa1" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 양</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>수백만 개의 예시가 있다면 처음부터 모델을 학습시키는 것이 더 좋을 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><span style="color: #333333; text-align: start;">학습 데이터가 많을 경우<span> </span></span>사전 학습된 모델 뒤에 파인튜닝 하는 것이 더 나쁠 수도 있음 : <b>경화 ossification</b>라는 현상
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>사전 학습이 모델 가중치를 열려서 파인튜닝 데이터에 잘 적응하지 못하게 만들 수 있음</li>
<li>작은 모델일 수록 경화 현상에 취약함</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">데이터 품질과 다양성 외에도 필요한 데이터의 양을 결정하는 요소 세 가지</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>파인튜닝 기법</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>전체 파인튜닝은 최고 성능을 낼 수 있지만, PEFT 방법보다 몇 배나 많은 데이터가 필요함</li>
<li>(지시, 응답) 쌍이 수만 개에서 수백만 개 있다면 전체 파인튜닝 해볼 만함</li>
<li>수백 개, 수천 개 밖에 없으면 PEFT가 가장 효과적</li>
</ul>
</li>
<li><b>과제 복잡성</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>간단한 과제는 훨씬 적은 데이터로 충분함</li>
</ul>
</li>
<li><b>기본 모델의 성능</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>기본 모델이 원하는 성능에 가까울수록 필요한 예시가 적음 (사전 학습에서는 큰 모델이 더 많은 학습 데이터를 필요로 함)</li>
</ul>
</li>
</ul>
<img width="1280" height="923" alt="image" src="https://github.com/user-attachments/assets/7f5be0b9-33c0-4c96-bdca-42d76c678daa" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터가 적다면, 고급 모델에 PEFT</li>
<li>데이터가 많다면, 더 작은 모델로 전체 파인튜닝</li>
</ul>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>대규모 데이터셋 구축 전, 잘 만들어진 소규모 데이터셋으로 시작해서 파인튜닝이 모델을 개선할 수 있는지 확인해 보는 것이 좋음, 소규모 데이터셋으로 명확한 개선이 보이면, 데이터를 더 추가하면 성능이 향상될 가능성이 높다는 신호</li>
<li>여러가지 요소가 있지만, 대부분 50~100개의 예시로 파인튜닝하면 성능 개선을 확인할 수 있을 것</li>
<li>작은 데이터셋으로 실험해보면 <b>앞으로 데이터가 얼마나 더 필요한지 추정</b>할 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>데이터셋 일부분으로 모델을 파인튜닝하고 데이터셋 크기에 따른 성능 변화를 그래프로 그려보기 (25%, 50%, 100%)</li>
<li>데이터셋이 커질수록 성능이 가파르게 올라간다면 데이터를 두 배로 늘렸을 때 성능도 크게 좋아짐, 성능 증가폭이 없으면 늘려도 효과 미미
  <img width="1280" height="648" alt="image" src="https://github.com/user-attachments/assets/0f8abfa3-cd6b-4f32-bd69-30ab3f4b7480" />

<li>예시가 많을 수록 모델 성능이 좋아짐, But 예시의 다양성도 중요
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>다양성 : 과제 종류 (요약, 질의응답 등), 주제 다양성 (패션, 금융, 기술 등), 출력 형식 (JSON 출력이나 예/아니오 응답 등)</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1280" height="618" alt="image" src="https://github.com/user-attachments/assets/2644c678-338f-4229-ae01-8aaf8256de45" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>품질이 낮거나 관련성이 떨어지는 데이터로 파인튜닝</b>한 다음 고품질 데이터로 파인튜닝하면 필요한 고품질 데이터의 양을 줄일 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>자기 지도 학습 $\rightarrow$ 지도 학습 (법률 문서로 자기 지도 학습 후 (질의, 응답) 쌍으로 추가 파인튜닝</li>
<li>관련성 낮은 데이터 <span style="color: #333333; text-align: start;">$\rightarrow$<span> 관련성 높은 데이터 (트윗 감정 분류 모델로 파인튜닝 후, 제품 감정 분류 추가 파인튜닝)</span></span></li>
<li><span style="color: #333333; text-align: start;"><span>합성 데이터 <span style="color: #333333; text-align: start;">$\rightarrow$<span>&nbsp; 실제 데이터 (But 이 방법은 제대로 하기 어려움, 서로 다른 두 번의 파인튜닝 작업을 하면서 그 사이의 전환을 조율 해야함, 더 많은 컴퓨팅 자원을 사용하고도 고품질 데이터만으로 파인튜닝했을 때보다 못한 모델을 만들 수도)</span></span></span></span></li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 수집과 주석</h3>
<p data-ke-size="size16">데이터 수집의 목표 : 사용자 프라이버시를 존중하고 규정을 지키면서 필요한 품질과 다양성을 갖춘 충분한 크기의 데이터셋을 만드는 것</p>
<p data-ke-size="size16">$\rightarrow$ 공개 데이터 수집, 독점 데이터 구매, 데이터 주석 작업, 데이터 합성 등</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">가장 중요한 데이터 소스 :&nbsp;<b>자체 애플리케이션에서 나오는 데이터 (사용자 콘텐츠, 사용자 로그 데이터, 사용자 피드백 등)</b></p>
<p data-ke-size="size16">중요하게 여기는 데이터의 분포와의 일치를 다른 데이터 소스에서 달성하기 매우 어려움</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">자체 데이터 구축 전 사용 가능한 데이터셋 확인 필요, 대부분은 여러 데이터를 조합해야 함</p>
<div data-ke-type="moreLess" data-text-more="더보기" data-text-less="닫기"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content">
<p data-ke-size="size16"><b>공개 데이터셋 리소스</b></p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>허깅페이스, 캐글</li>
<li>구글 Dataset Search : 훌륭하지만 많이 알려지지 않음</li>
<li>정부 기관들의 오픈 데이터, Data.gov, data.gov.in</li>
<li>미시간 대학교 사회연구소 ICPSR</li>
<li>UC 어바인 ML 저장소</li>
<li>OpenML</li>
<li>Open Data Network</li>
<li>클라우드 서비스 제공업체도 소규모 오픈 데이터셋 모음을 제공함 : AWS Open Data 등</li>
<li>ML 프레임워크가 제공하는 작은 데이터셋들 (텐서플로 데이터셋 등)</li>
<li>일부 평가 도구들은 PEFT 파인튜닝에 충분한 평가 벤치마크 데이터셋 제공 : Eleuther AI의 lm-evaluation-harness</li>
<li>Stanford Large Network Dataset Collection : 그래프 데이터셋 저장소</li>
</ul>
</div>
</div>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파인튜닝을 위한 자체 데이터 주석 작업은 어려운 것</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>명확한 주석 가이드라인을 만들기 어려움</li>
<li>4장에서 다룬 평가 데이터용 가이드라인과 같음</li>
</ul>
<figure id="og_1768467798307" contenteditable="false" data-ke-type="opengraph" data-ke-align="alignCenter" data-og-type="article" data-og-title="[AIE] Ch.4 AI 시스템 평가하기" data-og-description="GitHub - tenaan/ai-engineering-study: 『AI 엔지니어링』 스터디 (2025.11~2026.01)『AI 엔지니어링』 스터디 (2025.11~2026.01). Contribute to tenaan/ai-engineering-study development by creating an account on GitHub.github.comAI 엔지니어" data-og-host="armugona.tistory.com" data-og-source-url="https://armugona.tistory.com/entry/AIE-Ch4-AI-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%8F%89%EA%B0%80%ED%95%98%EA%B8%B0#%ED%8F%89%EA%B0%80%20%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8%20%EC%84%A4%EA%B3%84%ED%95%98%EA%B8%B0-1" data-og-url="https://armugona.tistory.com/entry/AIE-Ch4-AI-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%8F%89%EA%B0%80%ED%95%98%EA%B8%B0" data-og-image="https://scrap.kakaocdn.net/dn/djaDqy/dJMb895W4wY/QhiyIjecLqOYuKblsShQQ0/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/b4xP1c/dJMb88FYlVC/9Sk7vtmxrOLYWSTv6MMWq1/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/wuiq1/dJMb81fL5uH/omXfAZNjTDjtTXVet1E3rk/img.png?width=1107&amp;height=1233&amp;face=0_0_1107_1233"><a href="https://armugona.tistory.com/entry/AIE-Ch4-AI-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%8F%89%EA%B0%80%ED%95%98%EA%B8%B0#%ED%8F%89%EA%B0%80%20%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8%20%EC%84%A4%EA%B3%84%ED%95%98%EA%B8%B0-1" target="_blank" rel="noopener" data-source-url="https://armugona.tistory.com/entry/AIE-Ch4-AI-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%8F%89%EA%B0%80%ED%95%98%EA%B8%B0#%ED%8F%89%EA%B0%80%20%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8%20%EC%84%A4%EA%B3%84%ED%95%98%EA%B8%B0-1">
<div class="og-image" style="background-image: url('https://scrap.kakaocdn.net/dn/djaDqy/dJMb895W4wY/QhiyIjecLqOYuKblsShQQ0/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/b4xP1c/dJMb88FYlVC/9Sk7vtmxrOLYWSTv6MMWq1/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/wuiq1/dJMb81fL5uH/omXfAZNjTDjtTXVet1E3rk/img.png?width=1107&amp;height=1233&amp;face=0_0_1107_1233');">&nbsp;</div>
<div class="og-text">
<p class="og-title" data-ke-size="size16">[AIE] Ch.4 AI 시스템 평가하기</p>
<p class="og-desc" data-ke-size="size16">GitHub - tenaan/ai-engineering-study: 『AI 엔지니어링』 스터디 (2025.11~2026.01)『AI 엔지니어링』 스터디 (2025.11~2026.01). Contribute to tenaan/ai-engineering-study development by creating an account on GitHub.github.comAI 엔지니어</p>
<p class="og-host" data-ke-size="size16">armugona.tistory.com</p>
</div>
</a></figure>
<hr contenteditable="false" data-ke-type="horizontalRule" data-ke-style="style6" />
<h2 data-ke-size="size26">데이터 증강 및 합성</h2>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>데이터 증강 data augmentation</b> : 기존 데이터로 새로운 데이터 만들기 (이미지 뒤집기 등)</li>
<li><b>데이터 합성 data synthesis</b> :&nbsp; 실제 데이터의 특성을 모방하는 데이터를 생성 (봇의 움직인 패턴에 대한 데이터), 실제가 아님</li>
</ul>
<p data-ke-size="size16">처음엔 테스트용 가짜 데이터 생성하는데 사용됨</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 합성을 하는 이유</h3>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>데이터 양 늘리기&nbsp;</b>: 데이터를 대규모로 만들 수 있음</li>
<li><b>데이터 커버리지 늘리기</b> : 특성 특징을 가진 데이터를 생성해 성능 개선, 특정 행동을 하도록&nbsp;</li>
<li><b>데이터 품질 향상&nbsp;</b>: 사람의 근본적인 한계로 인해 때로는 AI가 만든 데이터가 나을 수 있음</li>
<li><b>프라이버시 문제 해결</b> : 의료, 보험 등 민감한 실제 정보 대신 합성 데이터 사용</li>
<li><b>모델 증류&nbsp;</b>: 다른 모델의 행동을 모방하는 모델을 학습시키거나, 비슷한 성능의 더 작은 모델 만드는 것</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">전통적인 데이터 생성 기법</h3>
<p data-ke-size="size16">사람이 생성하는 것과 달리 알고리즘으로 데이터를 생성하는 것 : <b>절차적 생성 procedural generation</b></p>
<p data-ke-size="size16">전통적으로 규칙 기반 방식, 시뮬레이션 방식이 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>규칙 기반&nbsp;</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>미리 정해둔 규칙과 템플릿을 사용 (템플릿을 만들어두고 난수 생성기로 필드를 채우기)<br />
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>특정 형식을 따르는 문서 만들기</li>
</ul>
</li>
<li>기존 데이터에 간단한 변형을 가해 새로운 데이터 생성 (이미지의 경우 뒤집기, 회전, 자르기 등, 텍스트의 경우 유이어 대체 등)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터의 편향을 줄이는 데도 사용할 수 있음</li>
<li>섭동 Perturbation : 기존 데이터에 노이즈를 넣어서 새로운 데이터를 생성하는 것 (픽셀 하나만 바꿔도 잘못 분류될 수도 있음) $\rightarrow$ 이런 섭동 데이터를 악용할 수도 있음, 섭동된 데이터로 모델을 학습시켜 성능을 향상시키고 공격에 견고하게 만들기, 텍스트에도 사용가능 (무작위 단어 교체), 이미지 데이터는 더 정교한 알고리즘을 사용해서 증강할 수 있음 (데이터의 숨은 편향 줄이기)</li>
</ul>
</li>
</ul>
</li>
<li><b>시뮬레이션</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>가상으로 시뮬레이션 : 현실에서 일어나기 힘든 사건의 데이터를 생성하는데 유용함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>자율주행 시뮬레이션</li>
<li>로보틱스용 학습데이터 가상 환경에서 시뮬레이션</li>
</ul>
</li>
<li>시뮬레이션이 아무리 정교해도 실제 세계를 단순화한 것일 뿐, 시뮬레이션에서 학습한 알고리즘을 실제 세계에 적용하는 데 초점을 맞춘 분야 : Sim2Real</li>
<li>모델에게 도구 사용법 가르치는 데이터 생성에서도 쓰임</li>
<li>합성 데이터를 AI에 넣어서 더 다양한 가능성들로부터 학습할 수 있게 해줌</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">AI 기반 데이터 합성&nbsp;</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>강력한 AI를 활용해 시뮬레이션 분야에서 할 수 있는 일
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>API 호출 시뮬레이션 : 여러 API와 상호작용하는 모델 학습</li>
<li>체스 시뮬레이션 : AI 플레이어와 경기 진행, <b>셀프 플레이</b></li>
<li>일반 에이전트 : AI끼리 서로 다른 전략으로 협상하게 (셀프 플레이)</li>
<li>AI의 바꿔쓰기, 번역 능력으로 기존 데이터셋 늘리기&nbsp;
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>자원이 부족한 언어 번역</li>
<li>역번역 back-translation</li>
<li>프로그래밍 언어 번역&nbsp;</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">합성 데이터는 사전 학습보다 <b>사후 학습</b>에 훨씬 자주 쓰임 : 새로운 지식 합성보다 다른 형식으로의 합성이 더 쉬움, 지시 데이터와 선호도 데이터를 포함한 사후 학습 데이터가 보통 만들기 가장 힘듦</p>
<p data-ke-size="size16">주요 과제는 모델의 편향을 고려하는 것 : 위치 편향 등 (위치를 바꿔서 두 번 물어봤을 때 같은 답을 선택했을 때만 유효한 조합으로 인정하는 등의 해결책)</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">지시 데이터 합성</h4>
<p data-ke-size="size16">AI로 지시나 응답, 혹은 둘다 만들기</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>지시 생성 : 활용 사례를 충분히 다루는 지시를 만들기 위해서 데이터셋에 포함하고 싶은&nbsp;<b>주제, 키워드, 지시 유형 목록</b>으로 시작할 수 있음, 항목마다 일정 수의 지시를 만들기, 템플릿 세트로 시작해서 템플릿 당 일정 수의 예시를 만드는 방법, 주제 목록과 템플릿 모두 AI로 만들 수 있음</li>
<li>응답 생성 : 지시 하나당 하나 또는 여러개의 응답 만들기</li>
</ul>
<img width="1280" height="377" alt="image" src="https://github.com/user-attachments/assets/d3bb377e-e400-4dc2-878f-53d2a6096b03" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>지시 데이터 합성하는 방법들
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>AI도 고품질의 긴 응답을 생성하기 어려움, 환각을 일으킬 가능성이 높음 : 사람이 만든 응답과 AI가 만든 지시 함께 사용하기
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>역지시 reverse instruction : 위키백과 글 등 긴 고품질 콘텐츠를 가져와 AI로 그런 콘텐츠를 유도할 수 있는 프롬프트를 생성하는 것, 환각을 피하며 고품질의 지시 데이터를 얻을 수 있음, 수동으로 주석을 단 데이터를 추가하지 않고도 역지시를 사용해 더 강력한 모델을 개발할 수 있음</li>
<li><a href="https://arxiv.org/abs/2304.08460" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2304.08460</a>, <a href="https://arxiv.org/abs/2308.06259" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2308.06259</a>, <a href="https://arxiv.org/abs/2309.05447" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2309.05447</a></li>
</ul>
</li>
</ul>
</li>
<li>라마 3 : 지시 데이터 합성의 훌륭한 사례 연구
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>AI를 사용해 다야한 주제를 다루는 프로그래밍 문제 설명 대량 생성</li>
<li>문제와 프로그래밍 언어가 주어지면 해결책 생성 (좋은 프로그래밍의 일반 규칙과 CoT 추론 포함하여 품질 향상)</li>
<li>생성된 데이터의 품질을 포장하기 위해 엄격한 정확성 분석과 오류 수정 과정을 거침
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>생성된 코드를 파서와 린터로 누락된 임포트나 초기화되지 않은 변수 같은 문법 오류 잡아내기, 단위 테스트를 통한 런타임 실행 오류 잡아내기, 해결책이 어떤 단계에서든 실패하면 모델에 코드를 수정하라고 프롬프트를 주고 (앞서 사용된 모든 피드백 포함됨) 모든 검사를 통과한 예시만 최종 지도 파인튜닝 데이터셋에 포함</li>
</ul>
</li>
<li>코드 번역, 코드역번역, 코드 생성 세 가지 방법을 모두 결합
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>문제 설명 생성, 해결책 생성, 단위 테스트 후 합성된 코드의 오류를 수정하라고 시킴 $\rightarrow$ 생성된 코드를 다양한 프로그래밍 언어로 번역한 후 테스트를 통과하지 못하면 걸러냄 <span style="color: #333333; text-align: start;">$\rightarrow$<span> 코드 설명과 문서 작성을 포함해서 코드에 대한 대화를 생성한 후 역번역 검증을 통과하지 못하는 설명과 문서 걸러냄</span></span></li>
</ul>
</li>
</ul>
</li>
</ul>
<figure id="og_1768470163579" contenteditable="false" data-ke-type="opengraph" data-ke-align="alignCenter" data-og-type="website" data-og-title="The Llama 3 Herd of Models" data-og-description="Modern artificial intelligence (AI) systems are powered by foundation models. This paper presents a new set of foundation models, called Llama 3. It is a herd of language models that natively support multilinguality, coding, reasoning, and tool usage. Our " data-og-host="arxiv.org" data-og-source-url="https://arxiv.org/abs/2407.21783" data-og-url="https://arxiv.org/abs/2407.21783v3" data-og-image="https://scrap.kakaocdn.net/dn/cGAzOo/dJMb8U8M8hu/zPJpe3OMzEeCLndea2Srr1/img.png?width=1200&amp;height=700&amp;face=0_0_1200_700,https://scrap.kakaocdn.net/dn/hmHv7/dJMb85vIkhf/lBk4kPZdsktJlPlCCkUnZk/img.png?width=1000&amp;height=1000&amp;face=0_0_1000_1000"><a href="https://arxiv.org/abs/2407.21783" target="_blank" rel="noopener" data-source-url="https://arxiv.org/abs/2407.21783">
<div class="og-image" style="background-image: url('https://scrap.kakaocdn.net/dn/cGAzOo/dJMb8U8M8hu/zPJpe3OMzEeCLndea2Srr1/img.png?width=1200&amp;height=700&amp;face=0_0_1200_700,https://scrap.kakaocdn.net/dn/hmHv7/dJMb85vIkhf/lBk4kPZdsktJlPlCCkUnZk/img.png?width=1000&amp;height=1000&amp;face=0_0_1000_1000');">&nbsp;</div>
<div class="og-text">
<p class="og-title" data-ke-size="size16">The Llama 3 Herd of Models</p>
<p class="og-desc" data-ke-size="size16">Modern artificial intelligence (AI) systems are powered by foundation models. This paper presents a new set of foundation models, called Llama 3. It is a herd of language models that natively support multilinguality, coding, reasoning, and tool usage. Our</p>
<p class="og-host" data-ke-size="size16">arxiv.org</p>
</div>
</a></figure>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">데이터 검증</h4>
<p data-ke-size="size16">다른 AI 결과물 평가와 같은 방식으로 기능적 정확성, AI 평가자를 사용해 데이터 품질을 검증함 (기능적 정확성으로 검증할 수 없는 합성 데이터는 AI 검증기AI verifier (AI 평가자, 특화된 채점기 등)를 사용하는게 일반적임)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">사람들은 검증할 수 있는 데이터를 합성하는 경향이 있음 (e.g. 코딩: 기능적으로 평가 가능)</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>검증 문제를 구성하는 방법
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>AI 검증기가 생성된 예시에 1~5점까지 점수를 매기거나 예시를 좋음 또는 나쁨으로 분류, 파운데이션 모델에 품질 요구사항을 설명하고 데이터 예시가 이런 요구사항을 만족하는지 판단하게 함</li>
</ul>
</li>
<li>사실적 불일치 탐지 기법으로 환각 가능성이 높은 예시들 걸러내기</li>
<li>합성 데이터와 실제 데이터를 얼마나 구별하기 어려운지로 합성 데이터의 품질을 측정
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>AI 콘텐츠 탐지기를 학습시켜서 AI가 생성한 데이터를 찾아내도록, 구별하기 쉬우면 합성 데이터가 좋지 않은 것</li>
</ul>
</li>
<li>생성된 각 예시의 주제를 탐지하는 모델을 만들어서 상관없는 주제의 예시들을 제거</li>
<li>이상 탐지를 사용해서 이상값 식별 (이상값 예시들은 품질이 낮을 수 있음)</li>
<li>휴리스틱
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>반복적인 예시, 너무 길거나 짧음, 같은 지시지만 다른 응답을 가진 예시, 출력이 입력을 그대로 반복하는 예시</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">AI 생성 데이터의 한계</h3>
<p data-ke-size="size16">AI가 생성한 데이터가 사람이 생성한 데이터를 완전히 대체하기는 어려움</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>품질의 차이</b></li>
<li><b>피상적 모방 superficial imitation의 한계</b></li>
<li><b>모델 성능 저하 가능성</b></li>
<li><b>데이터의 계보가 불분명해지는 문제</b></li>
</ul>
<h4 data-ke-size="size20">품질 관리</h4>
<p data-ke-size="size16">Garbage in garbage out</p>
<p data-ke-size="size16">품질을 검증할 수 없다면 사용하기 망설여짐</p>
<p data-ke-size="size16">데이터를 평가할 신뢰할 만한 방법과 지표가 필요함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">피상적 모방</h4>
<p data-ke-size="size16">모방을 통해 얻는 성능은 겉보기에만 좋아보일 수 있음</p>
<p data-ke-size="size16">정확성과 학습 데이터 범위를 벗어한 과제에 대한 일반화에서 어려움을 겪을 수 있음 $\rightarrow$ 환각을 일으키게 할 수 있음</p>
<p data-ke-size="size16">추론 능력을 개선하려면 기본 모델의 품질 향상에 집중해야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">모델 성능 저하 가능성</h4>
<p data-ke-size="size16">AI가 생성한 데이터로 모델이 얼마나 학습할 수 있는지도 확실하지 않음&nbsp;</p>
<p data-ke-size="size16">일부 연구에 따르면 AI가 생성한 데이터를 반복적으로 학습에 사용하면 모델에 돌이킬 수 없는 결함이 생기고 시간이 지나면서 성능이 떨어진다고 함</p>
<p data-ke-size="size16"><b>모델 붕괴 model collapse </b>: LLM을 포함한 모델들에서 이런 현상이 발생함, 사전 학습, 사후 학습 모두에서 일어날 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>AI 모델이 확률이 높은 사건을 생성할 가능성이 높고, 여러번 반복하면서 확률이 낮은 사건은 생성된 데이터에서 적게 나타남, 시간이 지나며 일반적인 사건을 출력하며 희귀한 사건은 잊어버리게 됨</li>
</ul>
<p data-ke-size="size16">학습 데이터셋 전체가 합성 데이터면 모델 붕괴를 피할 수 없지만, 합성 데이터를 실제 데이터와 섞으면 피할 수 있다고 주장 (어떤 비율로 섞어야 하는지에 대한 명확한 답은 주지 않음)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">또한 AI가 생성한 데이터는 편향을 계속 퍼뜨릴 수도 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">불분명한 데이터 계보</h4>
<p data-ke-size="size16">AI 모델은 학습 데이터의 영향을 받고 사용자도 모르게 그 내용을 그대로 출력할 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델 X를 이용해 학습시킬 데이터를 만들었는데, 모델 X가 저작권을 위반한 데이터로 학습됐다면 학습시킬 모델도 저작권을 위반할 수 있음</li>
<li>벤치마크 B로 학습된 모델인지 모르고 벤치마크 B로 해당 모델을 평가</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">모델 증류</h3>
<p data-ke-size="size16"><b>Model distillation, knowledge distillation 모델 증류</b>는 작은 모델이 큰 모델을 모방하도록 학습시키는 방법</p>
<p data-ke-size="size16">전통적으로 모델 증류의 목표 : 배포용으로 더 작은 모델을 만드는 것, 교사와 비슷한 성능을 내면서도 더 작고 빠른 학생 모델 만들기</p>
<p data-ke-size="size16">학생 모델은 DistillBERT처럼 처음부터 학습시킬 수도 있고 알파카처럼 사전 학습된 모델에서 파인튜닝할 수도 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>DistillBERT : BERT 모델 크기는 40% 줄이면서도 언어 이해 능력의 97%를 유지하고 60% 더 빠름</li>
<li>알파카 : 1,750억 파라미터 모델 test-davinci-003이 생성한 예시로 라마-7B 파인튜닝, 교사 모델 크기의 4%에 불과하면서도 비슷하게 동작함</li>
</ul>
<p data-ke-size="size16">모든 모델을 증류할 수 있는 것은 아님 : 많은 모델 라이선스가 자신의 출력을 다른 모델, 경쟁 모델 학습에 사용하는 것을 금지함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">합성 지시 데이터는 LoRA 같은 어댑터 기반 기법과 함께 사용됨</p>
<p data-ke-size="size16">합성 데이터로 학습한다고 해서 모두 모델 증류인 것은 아니며, 모델 증류는 교사 모델의 성능이 학생의 목표가 된다는 뜻</p>
<p data-ke-size="size16">But, 합성 데이터를 사용해서 교사보다 더 크고 강력한 학생 모델을 학습시키는 것도 가능함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>역지시를 이용한 모델 부트스트래핑</li>
<li>엔비디아의 네모트론-4 (3,400억 파라미터 기본 모델 사전 학습 후 560억 파라미터 MoE 모델인 Mixtral-8x7B-Instruct-v0.1이 생성한 지시와 선호도 데이터로 파인튜닝, 그 결과 Nemotron-4-340B는 교사 모델보다 좋은 성능을 보임)</li>
<li>라마3 : 더 뛰어난 모델이 생성한 데이터로 학습하면 모델 성능을 크게 높일 수 있지만, 자신이 만든 데이터로 무턱대고 학습하는 것은 모델 성능을 개선하지 못하고 오히려 떨어뜨릴 수 있다고 함 (품질 검증이 필요)</li>
</ul>
<hr contenteditable="false" data-ke-type="horizontalRule" data-ke-style="style6" />
<h2 data-ke-size="size26">데이터 처리</h2>
<p data-ke-size="size16">데이터 처리 단계에 대해 다룸</p>
<p data-ke-size="size16">데이터셋 세부 사항을 공개한 모델 논문을 읽으며 연구자들이 어떻게 데이터를 큐레이션하고, 생성하고, 처리했는지에 대한 좋은 팁들이 많이 담겨있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">데이터가 많으면 이런 처리 단계 하나하나가 몇 시간에서 며칠씩 걸릴 수 있음&nbsp;</p>
<p data-ke-size="size16"><b>처리 과정의 효율성을 높이는 팁</b></p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>데이터 처리 단계는 시간과 컴퓨팅 자원을 절약할 수 있는 순서라면 어떤 순서로든 진행해도 됨
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>데이터 중복 제거보다 저품질 데이터 정제가 오래걸리면 중복 제거를 먼저 수행, 중복 제거가 저품질 데이터 필터링보다 오래걸리면 필터링 먼저</li>
</ul>
</li>
<li>모든 데이터에 스크립트를 적용하기 전에 처리 스크립트가 제대로 작동하는지 확인하기 위해 항상 테스트를 진행</li>
<li>데이터를 원본에서 바로 수정하지 않기, 원본 데이터의 사본 반드시 보관
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>이후에 본인이나 다른 팀이 다른 애플리케이션에서 같은 데이터를 다르게 처리해야할 수 있음</li>
<li>스크립트의 버그가 원본 데이터 자체를 망가뜨릴 수 있음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 검사</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>데이터 살펴보고 데이터 품질 파악 : 데이터 기본 정보와 통계 확인 (데이터가 어디서 왔는지, 어떻게 처리됐는지, 다른 용도로는 어떻게 사용됐는지)</li>
<li>토큰 분포, 입력 길이, 응답 길이 등 그래프로 확인 (특별한 토큰이 쓰였는지, 데이터의 주제와 언어 분포를 구할 수 있는지, 주제와 언어들이 과제와 얼마나 관련이 있는지)</li>
</ul>
<img width="1280" height="804" alt="image" src="https://github.com/user-attachments/assets/2128b86a-5882-43c4-8b23-5f3cb9161306" />
<img width="1280" height="476" alt="image" src="https://github.com/user-attachments/assets/213bed2e-7842-4ff1-8ee1-ea3b8accf2a5" />

같은 지시 세트에 대한 GPT-3와 4 생성 결과 차이 비교, 데이터 평가 뿐만 아니라 모델 평가에도 유용함

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>이러한 분포들을 데이터 출처, 시간, 주석자 등으로 나눠서 그려보기
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>더 길거나 짧은 응답, 더 높거나 낮은 점수를 받는 질의 패턴, 이상값, 이상값의 원인, 이상값 처리 등</li>
<li>점수가 정규분포를 따라야 한다면 모든 주석자의 점수가 정규분포를 따르는지, 어떤 주석자들은 훨씬 짧은 응답을 주거나 높은 점수 쪽으로 편향되는 경향이 있다는 걸 발견하면 어떻게 처리할지</li>
</ul>
</li>
<li>각 예시에 주석이 여러 개 있다면 주석자들 간의 의견 차이를 계산하기 (주석이 다른 예시를 확인하고 갈등 해결)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">데이터를 직접 보는 것 : 매우 가치 있는 활동, 데이터 탐색 도구가 많이 있지만 반드시 실제로 데이터를 살펴보고 예시들이 말이 되는지 확인해야 함, 주석이 신뢰할 만한지, 응답의 사실과 맞는지, 예시들이 얼마나 고유한지, 같은 질의인데 다른 응답이거나 같은 응답이거나 다른 질의를 가지거나 등</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 중복 제거</h3>
<p data-ke-size="size16">중복된 데이터는 데이터 분포를 왜곡하고 모델에 편향을 만들 수 있음, 테스트 세트 자체에 오염을 일으킬 수도</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>앤트로픽 연구 : 데이터의 0.1%를 100번 반복하면 나머지 90%의 학습 토큰이 고유해도 8억 파라미터 모델의 성능이 4억 파라미터 모델 수준으로 떨어질 수 있음을 보여줌</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">중복의 형태는 다양하며 일부는 찾아내기 어려움</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>전체 문서 중복</b> : 같은 문서가 여러번 나타나는 것</li>
<li><b>문서 내 중복</b> : 같은 단락이 한 문서에서 두 번 나타나는 것</li>
<li><b>문서 간 중복</b> : 같은 유명한 인용구가 여러 문서에 나타나는 것</li>
</ul>
<p data-ke-size="size16">무엇은 중복으로 볼지도 정의에 따라 다름 : 문서 수준, 단락 수준, 문장 수준, 토큰 수준, 중복으로 간주되려면 완전히 같아야 하는지 80% 겹치면 되는지, 같은 항목을 가지지만 순서가 다르면 중복이 아닌지 등</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>데이터 중복을 제거하는 방법들</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>쌍대 비교</b> : (3장 내용) 정확한 일치, n-gram 유사도, 퍼지 매칭, 의미적 유사도 점수 (큰 데이터셋에서 비용 많이 듦)</li>
<li><b>해싱</b> : 예시들을 서로 다른 버킷으로 해싱하고 같은 버킷에 속하는 예시들 중에서만 확인 (MinHasg, Bloom filter)</li>
<li><b>차원 축소</b> : 먼저 데이터셋의 차원을 줄인 다음 쌍대 비교를 함 (6장에서 다룬 벡터 검색 기법들 활용 가능)</li>
<li>중복 제거에 도움이 되는 라이브러리들 : dupeGuru, Dedupe, datasketch, TextDistance, TheFuzz, deduplicate-text-datasets</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 정리 및 필터링</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>불필요한 토큰 형식 제거 : 불필요한 마크다운, HTML 태그 등 제거 (HTML 태그로 학습시킬게 아니라면)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>제거했더니 모델 정확도 20% 증가, 입력 토큰 길이 60% 감소</li>
</ul>
</li>
<li>PII, 민감한 데이터, 저작권이 있는 데이터, 유해한 데이터 등 정책에 맞지 않는 모든 것 제거</li>
<li>저품질 데이터 찾아내고 제거 (이 단계에서 데이터를 직접 보는 것이 중요함, 저품질 데이터를 찾아내는 휴리스틱 패턴 발견)</li>
<li>필요한 것보다 많은 데이터가 있더나 사용할 여유가 없다면 데이터를 추가로 걸러낼 수도 있음 : 중요도 샘플링 (각 학습 예시의 중요도를 잘 평가할 방법이 있는지에 효과가 있을지 달려있음, 데이터 가지치기 지표를 찾으면 자원 비용 크게 줄일 수 있음)</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">데이터 형식 맞추기</h3>
<p data-ke-size="size16">파인튜닝할 모델이 기대하는 형식에 맞춰야 함, 각 모델은 특정 토크나이저를 사용하며, (5장 내용) 특정 채팅 템플릿 형식의 데이터를 기대함, 데이터를 잘못된 채팅 템플릿으로 만들면 버그 생길 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>지도 파인튜닝 : (지시, 응답), 지시 : (시스템 프롬프트, 사용자 프롬프트)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>프롬프트 엔지니어링에서 파인튜닝으로 넘어갔다면, 파인튜닝에 쓰는 지시가 프롬프트 엔지니어링에서 쓰던 지시와 다를 수 있음 : 파인튜닝할 떄는 지시에 과제 설명이나 예시가 필요 없음, 학습 예시가 충분하다면 모델이 예시들만 봐도 과제에서 어떻게 동작해야 하는지 학습할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>파인튜닝하고 나면 예시없이 더 간단한 프롬프트를 쓸 수 있음 : 지시의 입력 토큰이 걱정된다면 파인튜닝이 비용을 줄이는 방법이 될 수 있음</li>
</ul>
</li>
<li>파인튜닝 데이터 형식이 다르면 성능에 영향을 줄 수 있으므로, 어떤 형식이 가장 좋은지 실험해봐야 함</li>
<li>파인튜닝된 모델을 사용할 때는 프롬프트가 파인튜닝 데이터 형식과 일치하는지 확인해야함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
