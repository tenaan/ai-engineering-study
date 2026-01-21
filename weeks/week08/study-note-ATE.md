- [블로그 링크](https://armugona.tistory.com/entry/AIE-Ch9-%EC%B6%94%EB%A1%A0-%EC%B5%9C%EC%A0%81%ED%99%94?category=1273755)
---
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델이 아무리 좋아도 너무 느리면 예측 결과가 더 이상 쓸모 없어질 수 있음, 너무 비싸도 투자 대비 효과가 떨어져 이용 가치가 없어짐</p>
<p data-ke-size="size16">모델을 빠르고 저렴하게 만드는 방법을 알아보자&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">추론 최적화는 <b>모델, 하드웨어, 서비스 수준에서 접근</b>할 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델 수준: 모델 크기를 줄이거나 더 효율적인 아키텍처 개발 (어텐션 메커니즘 연산 병목 제거한 아키텍처)</li>
<li>하드웨어 수준: 더 성능이 좋은 하드웨어 설계</li>
<li>서비스 수준: 추론 서비스는 주어진 하드웨어에서 모델을 실행해 사용자 요청을 처리함, 특정 하드웨어에 최적화된 모델을 위한 기법들을 활용할 수 있음, 사용량과 트래픽 패턴을 고려해 리소스를 효율적으로 배분해 지연 시간과 비용 줄이기</li>
</ul>
<p data-ke-size="size16">모델과 서비스 수준의 최적화 관련된 내용이 주를 이루며 AI 가속기도 간단히 살펴보며, 성능 지표와 트레이드오프도 다룸</p>
<hr contenteditable="false" data-ke-type="horizontalRule" data-ke-style="style6" />
<h2 data-ke-size="size26">추론 최적화 이해하기</h2>
<p data-ke-size="size16">추론에 대한 전반적인 설명과 앞으로 쓸 용어 정리</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">추론 개요</h3>
<p data-ke-size="size16"><b>추론 서버</b> : 운영 환경에서 모델 추론을 실행하는 구성 요소</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>여러 모델들을 호스팅하고 필요한 하드웨어를 사용할 수 있음</li>
<li>요청이 들어오면 적절한 모델을 실행하고 결과를 사용자에게 반환</li>
<li>추론 서비스라는 더 큰 시스템의 한 부분 (추론 서비스는 요청을 받아 적절한 곳으로 보내고, 추론 서버에 도달하기 전에 전처리하는 역할도 담당)</li>
</ul>
<img width="1280" height="757" alt="image" src="https://github.com/user-attachments/assets/c5b530b4-d383-48e6-8407-428492069a16" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">오픈AI나 구글에서 제공하는 모델 API들 : 추론 서비스</p>
<p data-ke-size="size16">이런 서비스만 활용하면 이번 장에서 나오는 기법들을 직접 구현할 일은 없으나, 직접 모델을 호스팅한다면 추론 서비스를 개발하고 최적화하고 유지보수하는 일을 모두 해야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">연산 병목</h4>
<p data-ke-size="size16">추론 서버는 담당하는 추론 작업의 연산 병목을 해결하도록 설계해야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>주요 연산 병목 : 연산 제약, 메모리 대역폭 제약</b></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>연산 제약 compute-bound</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>작업을 끝내는 데 걸리는 시간이 연산량에 따라 결정되는 경우</li>
</ul>
</li>
<li><b>메모리 대역폭 제약 memory bandwidth-bound</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>시스템 내부의 데이터 전송 속도에 제약을 받는 경우</li>
<li>메모리와 프로세서 간의 데이터 전송 속도가 핵심적인 병목 지점, CPU 메모리에 데이터를 저장하고 GPU에서 모델을 학습한다면 CPU에서 GPU로 데이터를 옮기는 데 시간이 오래 걸릴 수 있음, 논문에서는 그냥 메모리 제약이라고 부르기도 함 (OOM 상황을 메모리 제약이라고 하기도 함)</li>
</ul>
</li>
</ul>
<div data-ke-type="moreLess" data-text-more="더보기" data-text-less="닫기"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content">
<p data-ke-size="size16">OOM out-of-memory 메모리 부족 오류, 메모리 대역폭이 아니라 메모리 용량 때문에 제한되는 상황을 가리킬 때 메모리 재약이라고 하기도 함</p>
<p data-ke-size="size16">이런 문제는 작업을 작게 나누면 해결되는 경우가 많음</p>
<p data-ke-size="size16">GPU 메모리가 부족해서 모델 전체를 GPU에 올릴 수 없다면, 모델을 GPU 메모리와 CPU 메모리에 나누어 저장할 수 있음 $\rightarrow$ 연산이 느려지지만 데이터 전송이 충분히 빠르다면 큰 문제가 되지 않음</p>
<p data-ke-size="size16">메모리 용량 제한도 실제로는 메모리 대역폭 문제일 수 있음</p>
</div>
</div>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">수학적으로는 <b>산술 강도 arithmetic intensity</b>로 연산이 어느 쪽 제약을 받는지 구분할 수 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>산술 강도 : 메모리 1바이트에 접근할 때 몇 번의 산술을 하는지를 뜻함</li>
<li>엔비디아 엔사이드 NVIDIA Nsight 같은 프로파일링 도구를 쓰면 루프라인 차트 roofline chart로 작업이 연산 제약인지 메모리 대역폭 제약인지 볼 수 있음, 하드웨어 성능 분석에서 자주 쓰이는 방법</li>
</ul>
<img width="1280" height="699" alt="image" src="https://github.com/user-attachments/assets/669e76ab-876d-4b7a-9635-ae6f54d5fa89" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>최적화 기법마다 해결하는 병목이 다름
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>연산 제약 작업은 더 많은 칩에 분산시키거나 연산 성능이 더 좋은 칩을 활용해서 속도를 높일 수 있음</li>
<li>메모리 대역폭 제약 작업은 대역폭이 더 넓은 칩을 활용해서 속도를 높일 수 있음</li>
</ul>
</li>
<li>모델 구조와 작업 종류에 따라 연산 병목도 다르게 나타남
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>이미지 생성 모델 추론은 보통 연산 제약, 자기회귀 언어 모델 추론은 보통 메모리 대역폭 제약</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">언어 모델 추론 예시</p>
<p data-ke-size="size16">트랜스포머 기반 언어 모델 추론은 프리필과 디코딩으로 나뉨</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b> 프리필</b> : 모델이 입력 토큰들을 병렬로 처리, 한번에 처리할 수 있는 토큰 개수는 하드웨어가 정해진 시간에 실행할 수 있는 연산량에 달려 있음, <b>연산 제약</b></li>
<li><b>디코딩 </b>: 모델이 출력 토큰을 한 번에 하나씩 생성, 큰 행렬들(모델 가중치)을 GPU에 불러오는 작업을 포함함, 하드웨어가 얼마나 빨리 데이터를 메모리로 불러올 수 있는지에 따라 제한됨, <b>대역폭 제약</b><b></b></li>
</ul>
<img width="1280" height="432" alt="image" src="https://github.com/user-attachments/assets/83caa461-2219-487b-aeee-bbfbf3ea5916" />

<p data-ke-size="size16">프리필과 디코딩은 연산 방식이 달라 <b>실제 운영 환경에서는 따로 분리해서 다른 머신에서 돌리는 경우가 많음</b></p>
<p data-ke-size="size16">LLM 추론 서버에서 병목이 어디서 생기는지는 컨텍스트 길이, 출력 기링, 배치 요청 방식에 따라 달라짐</p>
<p data-ke-size="size16">컨텍스트가 길면 메모리 대역폭 제약이 생기지만 뒷부분에서 다루는 최적화 기법으로 제거할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">온라인과 배치 추론 API</h3>
<p data-ke-size="size16">모델 제공업체들이 온라인과 배치, 두 종류의 추론 API를 제공함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>온라인 API는 <b>지연 시간</b>을 최적화, 요청이 들어오면 바로 처리함 (지연 시간에 큰 영향을 주지 않는 선에서는 요청을 모아서 처리할 수 있음)</li>
<li>배치 API는 <b>비용을 최적화</b>함, <b>높은 처리량</b>, 애플리케이션에 지연 시간이 그리 중요하지 않다면 배치 API로 보내서 더 효율적으로 처리할 수 있음, 요청을 모아서 한 번에 처리하거나 저렴한 하드웨어를 쓰는 등 다양한 최적화 기법을 쓸 수 있음 (초나 분 단위가 아니라 시간 단위로 오래 걸림)
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>추론 서비스를 운영할 때, 추론 API를 온라인과 배치로 나누면 정말 빨라야 하는 요청을 우선 처리하는데 도움이 됨, 추론 서버가 지연 시간 저하없이 초당 최대로 처리할 수 있는 요청보다 더 많은 요청이 들어온다면, 이상적으로 덜 급한 요청을 배치 API로 보내면 서비스가 온라인 API 요청부터 먼저 처리할 수 있음</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">API는 보통 완성된 응답을 통째로 반환하지만 자기회귀 디코딩에서 모델이 응답을 완성하는데 오랜 시간이 걸릴 수 있음 $\rightarrow$ 온라인 API가 스트리밍 모드를 제공함, 토큰이 생성되는 대로 하나씩 보내주는 방식, 사용자에게 보여주기 전 응답을 평가할 수 없어 사용자가 나쁜 응답을 볼 가능성이 커짐</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파운데이션 모델의 배치 API는 기존 ML의 배치 추론과 다름</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>기존 ML : 온라인 추론은 요청이 들어온 후에 예측을 연산함, 배치 추론은 요청이 들어오기 전에 예측을 미리 연산해둠</li>
<li>파운데이션 모델의 배치 추론은 사용자가 요청을 보낸 후에 계산이 시작되지만 나중에 한꺼번에 처리하는 것
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>기존 ML은 입력값을 미리 계산할 수 있지만 파운데이션 모델은 입력값의 경우의 수가 무한대</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">추론 성능 지표</h3>
<h4 data-ke-size="size20">지연 시간, TTFT, TPOP</h4>
<p data-ke-size="size16">지연 시간 : 사용자가 질의를 보낸 시점부터 완전한 응답을 받기까지 걸리는 시간</p>
<p data-ke-size="size16">자기 회귀 생성 방식 (특히 스트리밍 모드)에서는 전체 지연 시간을 여러 지표로 나눌 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>첫 토큰까지 걸리는 시간 TTFT</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>질의를 보낸 후 첫 토큰이 나오기까지 걸리는 시간, 프리필 단계에 해당하며 입력 길이에 따라 달라짐</li>
</ul>
</li>
<li><b>출력 토큰당 시간 TPOT</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>첫 토큰 이후 각 토큰이 얼마나 빨리 생성되는지 측정</li>
<li>약 120ms 또는 초당 6~8개 토큰이면 대부분 충분함</li>
</ul>
</li>
<li><b>토큰 간 시간과 토큰 간 지연 시간 : </b><b>TBT와 ITL</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>출력 토큰 사이 사이 걸리는 시간 측정</li>
</ul>
</li>
<li><b>전체 지연 시간</b> : TTFT + TPOT * (출력 토큰 수)
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>전체 지연 시간이 같아도 각 시간이 달라지면 사용자 경험도 달라짐</li>
<li>어떤 걸 최적화할지는 사용자 연구가 필요함</li>
<li>사용자가 경험하는 TTFT와 TPOT는 모델 관점과 다를 수 있음 : CoT나 에이전트 방식에서 모델이 내부적으로 중간 단계를 거칠 때 더욱
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>일부 팀은 공개까지의 시간 time to publish 라는 지표를 사용함</li>
<li>여러 단계를 거칠 경우 사용자는 마지막 단계에서 생성된 최종 출력의 첫 토큰만 봄, TTFT가 훨씬 길어짐</li>
</ul>
</li>
</ul>
</li>
<li>지연 시간은 여러 값들의 분포로 나타나기 때문에 평균만 보면 잘못 판단할 수 있음 (TTFT이 10개 요청이 있을 때)&nbsp;
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>어떤 경우든 원인을 찾아봐야함, 요청이 많으면 평균을 왜곡하는 이상치가 생기기 마련임</li>
<li>지연 시간을 백분위수로 보는 것이 도움이 됨, p90/95/99를 일반적으로 살펴봐야 함</li>
<li>TTFT 값을 입력 길이별로 그래프로 그려보기</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">처리량과 굿풋</h4>
<p data-ke-size="size16"><b>처리량 throughput</b>은 추론 서비스가 <b>모든 사용자와 요청을 통틀어서 초당 몇 개의 출력 토큰을 만들어낼 수 있는지</b>를 측정함</p>
<p data-ke-size="size16">입력 토큰 처리(프리필)과 출력 토큰 생성(디코딩)은 병목 지점이 다르고, 최신 추론 서버에서는 둘을 분리해서 처리하는 경우가 많아 입력과 출력 처리량을 따로 연산해야 함</p>
<p data-ke-size="size16">보통 처리량이라 하면 출력 토큰을 의미함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">처리량은 추로 tokens/s (TPS)로 측정함, 여러 사용자에게 서비스한다면 tokens/s/user로 사용자가 늘어날 때 시스템이 어떻게 확장되는지 평가하기도 함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">처리량을 정해진 시간 동안 완료한 요청 개수로 측정할 수도 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>초당 요청 수 PRS</b></li>
<li>파운데이션 모델 기반 애플리케이션은 요청 하나가 완료되는 데 몇 초 씩 걸릴 수 있어 <b>분당 완료 요청 수 (RPM)</b>을 많이 씀</li>
<li>추론 서비스가 동시 요청을 어떻게 처리하는지 알 수 있음 : 너무 많은 요청을 보내면 서비스를 제한하기도 함</li>
</ul>
<p data-ke-size="size16">처리량은 연상 비용과 연결됨 : 처리량이 높을 수록 비용은 낮다</p>
<p data-ke-size="size16">요청당 총 비용은 프리필 비용과 디코딩 비용을 합한 것</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">좋은 처리량은 모델, 하드웨어, 작업에 따라 달라짐 : 작은 모델과 고성능 칩은 보통 처리량이 높음, 입력과 출력 길이가 일정한 작업이 길이가 들쭉날쭉한 작업보다 최적화하기 쉬움</p>
<p data-ke-size="size16">비슷한 조건이라도 처리량을 직접 비교하는 건 대략적일 뿐이므로, <b>요청당 비용 같은 지표로 추론 서버의 효율성을 비교하</b>는게 낫다</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">AI 애플리케이션에도 지연 시간과 처리량 사이의 트레이드오프가 있음</p>
<p data-ke-size="size16">링크드인 AI 팀에 따르면 TTFT와 TPOT를 조금 희생하면 처리량을 2~3배까지 올릴 수 있는 경우가 많음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">이러한 트레이드오프 때문에 추론 서비스를 처리량과 비용만으로 판단하면 사용자 경험이 나빠질 수 있음</p>
<p data-ke-size="size16">$\rightarrow$ LLM 애플리케이션에 맞게 변형한 지표인 <b>굿풋 goodput</b>이라는 지표에 집중</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>굿풋</b> :<b> 소프트웨어 수준 목표 (SLO)를 만족하는 초당 요청 개수를 측정</b></p>
<p data-ke-size="size16">애플리케이션에 있는 목표가 TTFT 최대 200ms, TPOT 최대 100ms 일때, 추론 서비스가 분당 100개 요청을 완료할 수 있지만 100개 중 30개만 SLO를 만족한다면 이 서비스의 굿풋은 30 RPM</p>
<img width="1256" height="794" alt="image" src="https://github.com/user-attachments/assets/c2001f56-7aad-4ac8-9a50-c71204485acf" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">활용률, MFU, MBU</h4>
<p data-ke-size="size16"><b>활용률 utilization</b> 지표는 리소스가 얼마나 효율적으로 사용하고 있는지 측정 : 전체 사용 가능한 용량 중 실제로 쓰이고 있는 비율을 말함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>nvidia-smi가 보여주는 지표 중 하나가 GPU 활용률, GPU가 실제로 작업을 처리하는 시간의 비율을 의미함</li>
<li>GPU 클러스터에서 10시간 동안 추론을 실행했는데 그중 5시간 동안 GPU가 실제로 작업을 처리했다면, GPU 활용률은 50%
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>하지만 작업을 처리하고 있다고 해서 효율적으로 하고 있다는 뜻은 아님 : 초당 100개 연산이 가능한 GPU가 초당 1개만 연산하고 있어도 활용률은 100%로 나옴</li>
<li>실제로 이 GPU 활용률 지표는 별로 도움이 되지 않음</li>
</ul>
</li>
<li><b>MFU model FLOP/s utilization </b>: 머신이 할 수 있는 모든 연산 중에서 정해진 시간에 실제로 몇 개를 하고 있는지 보여주는 지표
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>시스템이 최고 FLOP/s로 동작할 때 달성할 수 있는 <b>이론상 최대 처리량 대비</b>, <b>실제 처리량 (tokens/s)이 어느 정도인지를 나타내는 효율 지표</b></li>
</ul>
</li>
<li><b>MBU model bandwith utilization&nbsp;</b>: 사용 가능한 메모리 대역폭 중 실제 쓰이는 비율을 측정함, 메모리 대역폭도 효율적으로 활용되어야할 비싼 자원
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>LM 추론에 사용되는 메모리 대역폭 연산 방법 : $파라미터수 \times 파라미터당 바이트 \times tokens/s$</li>
<li>MBU 연산 방법 : $(파라미터수 \times 파라미터당 바이트 \times&nbsp; tokens/s) / (이론적 대역폭)$</li>
<li>e.g. 70억 파라미터 모델을 FP16(파라미터당 2바이트)으로 사용해서 초당 100토큰을 얻는다면, 사용되는 대역폭
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>$7B \times 2 \times 100 = 1400GB/s$</li>
<li>양자화가 중요한 이유, 파라미터당 바이트 수가 적을수록 모델이 대역폭을 덜 소모함</li>
</ul>
</li>
<li>이론적 메모리 대역폭 2TB/s인 A100-80GB 기준 MBU는
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>$(1400GB/s) / (2TB/s) = 70%$</li>
</ul>
</li>
</ul>
</li>
<li>처리량과 MBU, 처리량과 MFU 사이 관계는 일반적으로 선형이므로 처리량으로 MBU와 MFU를 표현하기도 함</li>
<li>좋은 MFU와 MBU 수준은 모델, 하드웨어, 작업에 따라 다름&nbsp;
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>연산 제약 작업은 MFU가 높고 MBU가 낮음</li>
<li>대역폭 제약 작업은 MFU가 낮고 MBU가 높음</li>
</ul>
</li>
<li>학습은 추론보다 작업 패턴이 예측 가능해 더 효율적인 최적화(배치 처리 개선 등)를 적용할 수 있음 $\rightarrow$ 학습 MFU가 보통 추론 MFU보다 높음</li>
<li>추론에서 프리필 단계는 연산 위주, 디코딩 단계는 메모리 대역폭 위주 $\rightarrow$ 프리필 떄의 MFU가 디코딩 떄의 MFU보다 높음</li>
<li>집필 시점에서 MFU 50% 이상을 일반적으로 좋다고 보지만 하드웨어에 따라 쉽지 않을 수 있음</li>
</ul>
<img width="1149" height="346" alt="image" src="https://github.com/user-attachments/assets/84d06070-32a6-4606-a972-ea1739555a60" />
<img width="1137" height="771" alt="image" src="https://github.com/user-attachments/assets/532ce7fb-a650-4c67-88e1-1474b778a3a8" />

<p data-ke-size="size16">서로 다른 하드웨어에서 라마 2-70B를 FP16으로 추론할 때의 MBU, 수치가 떨어지는 이유는 사용자가 늘어나면 연산 부하가 커져서 병목이 대역폭에서 연산으로 옮겨지기 때문</p>
<p data-ke-size="size16">같은 하드웨어에서 비슷한 작업을 할때 활용률이 높다면 보통 서비스가 점점 효율적으로 바뀌고 있다는 뜻</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">AI 가속기</h3>
<h4 data-ke-size="size20">가속기의 정의</h4>
<p data-ke-size="size16">가속기는 <b>특정 종류의 연산 작업을 빠르게 처리하도록 만들어진 칩</b></p>
<p data-ke-size="size16">AI 가속기는 AI 작업 전용으로 설계됨, 가장 널리 쓰이는 가속기는 GPU&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>CPU : 범용 작업용</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>CPU는 강력한 코어 몇 개를 가지고 있음 (고급 소비자용 머신은 최대 64개 코어)</li>
<li>많은 CPU 코어가 멀티스레드 작업을 효과적으로 처리할 수는 있지만 복잡한 순차 프로세스 처리 같은 높은 단일 스레드 성능이 필요한 작업에 특히 뛰어나다</li>
</ul>
</li>
<li><b>GPU : 병렬 처리용</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>수천 개의 작고 상대적으로 약한 코어를 가지고 있음</li>
<li>그래픽 렌더링이나 ML 처럼 다량의 작은 독립적인 연산으로 나눌 수 있는 작업에 최적화되어 있음, 대부분 ML 작업은 병렬화에 매우 용이한 행렬 곱셈 연산이 주를 이룸</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">엔비디아 GPU, AMD 최신 GPU, 구글 TPU, 인텔 하바나 가우디, 그래프코어의 IPU, 그로크의 LPU, 세라브라스의 QPU 등 많은 가속기가 등장했고 계속해서 더 많은 칩들이 나오고 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">많은 칩이 학습과 추론을 모두 처리할 수 있지만, 최근 떠오르는 중요한 흐름 중 하나는 <b>추론 전용 칩</b>임</p>
<p data-ke-size="size16">보편적으로 사용되는 시스템에서는 추론 비용이 학습 비용을 넘어설 수 있음, 이미 배포되어 운영 중인 AI 시스템은 ML 관련 비용의 최대 90%를 추론이 차지하는 것으로 나타남</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">학습은 보통 처리량에 집중하는 반면 추론은 지연 시간을 줄이는 것이 목표임</p>
<p data-ke-size="size16">추론용으로 설계된 칩들은 대용량 메모리보다는 낮은 정밀도와 더 빠른 메모리 접근에 최적화됨</p>
<p data-ke-size="size16">(애플의 Neural Engine, AWS의 Inferentia, 메타의 MTIA, 구글의 엣지 TPU, 엔비디아의 Jetson Xavier (뒤 두개는 엣지 컴퓨터용 칩))</p>
<p data-ke-size="size16">특정 모델 아키텍처에 특화된 칩도 존재 (트랜스포머 전용 칩)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">하드웨어 아키텍처마다 메모리 레이아웃과 특화된 연산 유닛 compute unit이 다름, 특정 데이터 유형에 최적화되어 있음</p>
<img width="1158" height="356" alt="image" src="https://github.com/user-attachments/assets/c74f977b-4ca0-4118-9ae3-973e43109c27" />

<p data-ke-size="size16">한 칩 안에 여러 데이텅 유형에 최적화된 다양한 연산 유닛이 함께 들어있을 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">특정 하드웨어에서 모델을 효율적으로 돌리려면 그 하드웨어 메모리 구조와 연산 방식을 고려해야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">칩의 사양에서 중요한 핵심 특성 : <b>연산 성능, 메모리 크기, 대역폭, 전력 소모&nbsp;</b>(뒤의 내용에서 GPU를 예시로 알아봄)</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">연산 성능</h4>
<p data-ke-size="size16">칩이<b> 정해진 시간에 수행할 수 있는 연산 수</b>로 측정 : 많이 쓰이는 지표 FLOP/s (FLOPS) 초당 부동 소수점 연산 횟수 측정</p>
<p data-ke-size="size16">실제 FLOP/s와 이론적인 최고 FLOP/s 사이 비율이 하나의 활용도 지표가 됨</p>
<img width="932" height="366" alt="image" src="https://github.com/user-attachments/assets/f395ac99-d7c5-4f01-9acc-56427af8cdbe" />

<p data-ke-size="size16">7장의 수치 표현 방식 확인</p>
<figure id="og_1768497756372" contenteditable="false" data-ke-type="opengraph" data-ke-align="alignCenter" data-og-type="article" data-og-title="[AIE] Ch.7 RAG와 에이전트" data-og-description="GitHub - tenaan/ai-engineering-study: 『AI 엔지니어링』 스터디 (2025.11~2026.01)『AI 엔지니어링』 스터디 (2025.11~2026.01). Contribute to tenaan/ai-engineering-study development by creating an account on GitHub.github.comAI 엔지니어" data-og-host="armugona.tistory.com" data-og-source-url="https://armugona.tistory.com/entry/AIE-Ch7-RAG%EC%99%80-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8?category=1273755#%EC%88%98%EC%B9%98%20%ED%91%9C%ED%98%84%20%EB%B0%A9%EC%8B%9D-1" data-og-url="https://armugona.tistory.com/entry/AIE-Ch7-RAG%EC%99%80-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8" data-og-image="https://scrap.kakaocdn.net/dn/V9F71/dJMb9g44EbT/EZtAatuW6UqpfNxNnAAhrk/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/blMiBl/dJMb9dHhxcv/O3RsLAyeOXIsLJb3qpw9tk/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/NAtaX/dJMb9b3LB0w/7zKFwaOBYBONkTJhDUQHZk/img.png?width=1065&amp;height=679&amp;face=0_0_1065_679"><a href="https://armugona.tistory.com/entry/AIE-Ch7-RAG%EC%99%80-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8?category=1273755#%EC%88%98%EC%B9%98%20%ED%91%9C%ED%98%84%20%EB%B0%A9%EC%8B%9D-1" target="_blank" rel="noopener" data-source-url="https://armugona.tistory.com/entry/AIE-Ch7-RAG%EC%99%80-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8?category=1273755#%EC%88%98%EC%B9%98%20%ED%91%9C%ED%98%84%20%EB%B0%A9%EC%8B%9D-1">
<div class="og-image" style="background-image: url('https://scrap.kakaocdn.net/dn/V9F71/dJMb9g44EbT/EZtAatuW6UqpfNxNnAAhrk/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/blMiBl/dJMb9dHhxcv/O3RsLAyeOXIsLJb3qpw9tk/img.png?width=800&amp;height=800&amp;face=0_0_800_800,https://scrap.kakaocdn.net/dn/NAtaX/dJMb9b3LB0w/7zKFwaOBYBONkTJhDUQHZk/img.png?width=1065&amp;height=679&amp;face=0_0_1065_679');">&nbsp;</div>
<div class="og-text">
<p class="og-title" data-ke-size="size16">[AIE] Ch.7 RAG와 에이전트</p>
<p class="og-desc" data-ke-size="size16">GitHub - tenaan/ai-engineering-study: 『AI 엔지니어링』 스터디 (2025.11~2026.01)『AI 엔지니어링』 스터디 (2025.11~2026.01). Contribute to tenaan/ai-engineering-study development by creating an account on GitHub.github.comAI 엔지니어</p>
<p class="og-host" data-ke-size="size16">armugona.tistory.com</p>
</div>
</a></figure>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">메모리 크기와 대역폭</h4>
<p data-ke-size="size16">GPU는 수많은 코어가 병렬로 작동하기 때문에, 메모리에서 코어들로 데이터를 계속해서 옮겨야 함 : 전송속도가 중요</p>
<p data-ke-size="size16">거대한 가중치 행렬과 학습 데이터를 다루는 AI 모델에서도 데이터 전송이 매우 중요함</p>
<p data-ke-size="size16">$\rightarrow$ GPU 메모리는 CPU 메모리보다 더 넓은 대역폭과 더 낮은 지연 시간이 필요함, GPU 메모리는 더 고급 메모리 기술이 필요함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>CPU는 2D 구조인 DDR SDRAM을 사용</li>
<li>GPU(특히 고급형)은 3D 적층 구조인 HBM을 많이 사용</li>
</ul>
<p data-ke-size="size16">가속기의 메모리는 크기와 대역폭으로 평가됨</p>
<p data-ke-size="size16">GPU 같은 가속기는 세 단계 메모리와 상호작용함</p>
<img width="866" height="446" alt="image" src="https://github.com/user-attachments/assets/7e0ee009-fb02-442d-bdef-1f3f37a3f2a1" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>CPU 메모리 (DRAM)</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>가속기는 보통 CPU와 함께 사용되어 CPU 메모리(시스템 메모리, 호스트 메모리, CPU DRAM)에 접근할 수 있음</li>
<li>대역폭이 가장 낮고, 데이터 전송 속도는 25GB/s에서 60GB/s 정도, 크기는 다양함, 일반 노트북 16GB~64GB, 고급 워크스테이션 1TB</li>
</ul>
</li>
<li><b>GPU 고대역폭 메모리 (HBM)</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>GPU 전용 메모리, CPU 메모리보다 빠르게 접근하기 위해 GPU 가까이에 둚</li>
<li>훨씬 높은 대역폭 제공, 256GB/s에서 1.5TB/s 이상, 대량 데이터 전송과 높은 처리량 작업을 효율적으로 처리하는 데 필요함, 소비자용 GPU는 24~80GB의 HBM</li>
</ul>
</li>
<li><b>GPU 온칩 SRAM</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>칩 안에 바로 들어 있는 메모리, 가장 자주 쓰는 데이터와 지시를 거의 바로 접근할 수 있게 저장함</li>
<li>SRAM으로 만들어진 L1과 L2 캐시가 포함됨, 일부는 L3 캐시도 포함, 이 캐시들은 레지스터 파일과 공유 메모리 같은 다른 부품들과 함께 더 큰 온칩 메모리의 일부가 됨</li>
<li>전송 속도 10TB/s 를 넘음, 크기는 40MB 이하</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">GPU 최적화의 상당 부분은 이 메모리 계층 구조를 어떻게 최대한 활용하느냐에 달려 있음</p>
<p data-ke-size="size16">집필 시점에서는 파이토치, 텐서플로와 같은 인기 프레임워크들은 아직 메모리 접근을 세밀하게 제어할 수 없음</p>
<p data-ke-size="size16">CUDA, 오픈AI의 트리톤, ROCm 같은 GPU 프로그래밍 언어에 관심을 갖는 이유 (ROCm은 CUDA에 맞서는 AMD의 오픈 소스 대안)</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">전력 소모</h4>
<p data-ke-size="size16">칩은 연산할 때 트랜지스터를 사용함, 각 연산은 트랜지스터가 켜졌다 꺼졌다하면 이뤄지며 이 과정에서 에너지가 필요함</p>
<p data-ke-size="size16">GPU는 수십억 개의 트랜지스터가 들어있음 (A100은 540억 개, H100은 800억 개)</p>
<p data-ke-size="size16">수십억 개의 트랜지스터가 빠르게 상태를 바꾸며 엄청난 양의 에너지를 소비하고 상당한 열을 발생시킴, 냉각 시스템에도 전기가 필요해 데이터센터 전체 에너지 소비가 늘어남</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">데이터 센터의 에너지 소모는 환경에 엄청난 영향을 줄 수 있음, 컴퓨팅 확장에서 전력이 가장 큰 걸림돌</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">가속기는 최대 전력 소모량이나 대체 지표인 열 설계 전력 TDP로 전력 소모를 표시함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>최대 전력 소모량 : 칩이 최대 부하 상태에서 뽑아낼 수 있는 최대 전력</li>
<li>TDP : 칩이 일반적인 작업을 할 때 냉각 시스템이 방출해야 하는 최대 열을 나타냄, 전력 소비를 정확히 측정한 건 아니지만 예상 전력 소모량을 나타내는 지표</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">&nbsp;cf. 가속기 선택</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>연산 중심 작업이라면 FLOP/s가 높은 칩을 찾는 것이 좋음</li>
<li>메모리 중심 작업이라면 대역폭이 넓고 메모리 용량이 큰 칩에 투자하는 것이 훨씬 편함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>이 하드웨어로 원하는 작업을 실행할 수 있는가</li>
<li>실행하는 데 얼마나 걸리는가</li>
<li>비용이 얼마나 드는가</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h2 data-ke-size="size26">추론 최적화</h2>
<img width="1139" height="650" alt="image" src="https://github.com/user-attachments/assets/59575792-7949-41f4-9ff2-25006b6dd908" />

<h3 data-ke-size="size23">모델 최적화</h3>
<p data-ke-size="size16">모델 자체를 수정하는 방식으로 효율성을 높이는 것을 목표로 함 $\rightarrow$ 모델의 성능이 바뀔 수 있음</p>
<p data-ke-size="size16">트랜스포머 아키텍처 + 자기회귀 언어 모델 구성 요소를 포함하고 있음 $\rightarrow$ 추론을 리소스 집약적으로 만드는 <b>모델 크기, 자기회귀 디코딩, 어텐션 메커니즘</b></p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">모델 압축</h4>
<p data-ke-size="size16">모델 크기를 줄이는 여러 기법, 모델이 작아지면 속도도 빨라질 수 있음 (e.g. 앙자화, 증류)</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>프루닝 pruning</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>큰 모델 안에 전체 모델의 동작을 담을 수 있는 파라미터의 부분집합이 존재할 것이라는 것을 핵심 아이디어로 함</li>
<li>두 가지 뜻이 있음
<ol style="list-style-type: decimal;" data-ke-list-type="decimal">
<li>&nbsp;신경망 노드 전체를 제거 - 아키텍처를 바꾸고 파라미터 개수 줄이기</li>
<li>예측에 도움이 안 되는 파라미터 찾아서 0으로 - 0이 아닌 파라미터 개수만 줄이기 (모델이 희소해짐)</li>
</ol>
</li>
<li>집필 시점에서는 실제로 프루닝을 쓰는 경우가 많지 않음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>원본 모델 아키텍처에 대한 이해가 필요하여 어려움, 다른 방법보다 성능 향상도 적은 경우가 많음</li>
<li>모든 하드웨어 아키텍처가 희소행렬 계산을 빠르게 하도록 설계되어 있지 않음</li>
</ul>
</li>
</ul>
</li>
<li><b>가중치 전용 양자화 weight-only quantization</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>가장 인기 있는 방법</li>
<li>쓰기 쉽고, 많은 모델에서 바로 작동, 모델 정밀도를 32비트에서 16비트로 줄이면 메모리 사용량이 절반으로 줄어듦</li>
<li>값 하나당 1비트보다 더 낮출 수 없다는 한계</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">자기회귀 디코딩 병목 현상 극복하기</h4>
<p data-ke-size="size16">토큰을 하나씩 차례로 생성, 여러 모델 API 업체에서 출력 토큰은 입력 토큰의 2~4배 비용이 들고, 출력 토큰이 지연 시간에 미치는 영향이 큼&nbsp;</p>
<p data-ke-size="size16">$\rightarrow$ 자기회귀 생성 과정을 개선하면 사용자 경험이 크게 향상될 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">추측 디코딩 speculative decoding</h4>
<p data-ke-size="size16">성능이 낮은 모델(초안 모델 or 제안 모델)을 사용해서 토큰 시퀀스를 생성한 다음, 목표 모델(원래 사용하려했던 모델)이 검증하는 방식</p>
<img width="1148" height="705" alt="image" src="https://github.com/user-attachments/assets/7a472df4-cebc-4c3c-b536-6ffc3de7ee42" />

<p data-ke-size="size16">초안 토큰이 하나도 수락되지 않으면, 목표 모델이 생성한 하나의 토큰만 만들어 냄</p>
<p data-ke-size="size16">모든 초안 토큰이 수락되면, K+1개 (K개는 초안 모델이, 1개는 목표 모델이) 만들어 냄</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>검증을 병렬화</b>할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>초안 모델은 생성 시간이 매우 짧음, 목표 모델이 검증하는 시간이 직접 생성하는 시간보다 짧음</li>
<li>디코딩 연산을 프리필 방식으로 바꾸는 효과</li>
</ul>
</li>
<li>예측하기 쉬운 토큰들은 약한 초안 모델도 잘 맞춤, 토큰 수락률이 높아</li>
<li>디코딩은 메모리 대역폭에 제약을 받아서 디코딩 과정에서 보통 남은 FLOP을 검증에 활용할 수 있음</li>
</ul>
<p data-ke-size="size16">비교적 구현하기 쉽고 모델 품질을 바꾸지 않기 때문에 큰 주목을 받고 있음 $\rightarrow$ vLLM, TensorRT-LLM, llama.cpp 같은 유명 추론 프레임워크에도 이미 들어가 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">참조 기반 추론</h4>
<p data-ke-size="size16">응답할 때 입력의 토큰들을 참조해야 하는 경우가 많음, 문서 내용을 그대로 인용하거나 원본 코드의 대부분을 재사용하거나</p>
<p data-ke-size="size16">반복되는 토큰을 새로 생성하게 하는 대신, 입력을 바로 복사해서 생성 속도를 높이자는 아이디어</p>
<p data-ke-size="size16">추측 디코딩과 비슷하지만 초안 토큰을 입력에서 가져오는 점에서 차이가 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>디코딩 단계에서 컨텍스트에서 가장 관련성 높은 텍스트 구간을 찾아내는 알고리즘</b>을 개발하는 것이 핵심 과제
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>가장 간단한 방법은 현재 토큰들과 일치하는 텍스트 구간을 찾기</li>
</ul>
</li>
<li>추가 모델이 필요없지만 검색 시스템, 코딩, 멀티 턴 대화처럼 컨텍스트와 출력 사이에 상당한 중복이 있는 생성 시나리오에서만 유용함</li>
</ul>
<img width="1280" height="846" alt="image" src="https://github.com/user-attachments/assets/6a347c5f-1434-493f-bd0a-ef2be90470fc" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">병렬 디코딩</h4>
<p data-ke-size="size16">순차적 의존성을 없애는 것을 목표로 함</p>
<p data-ke-size="size16">$x_{i+1}$이 뭔지 모르는 상태에서 $x_{i+2}$를 생성</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>기존 시퀀스만 알아도 다음 몇 개 토큰을 충분히 예측할 수 있는 경우가 많기 때문</li>
</ul>
<p data-ke-size="size16">룩어헤드 Lookahead 디코딩처럼 같은 디코더로 생성하거나, 메두사 Medusa 처럼 다른 디코딩 헤드들로 생성할 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>메두사에서는 원본 모델에 여러 디코딩 헤드를 추가하는데, 각 헤드는 특정 위치의 미래 토큰을 예측하도록 학습된 작은 신경망 레이어 (원본 모델이 $x_{i+1}$ 을 예측한다면 k번째 헤드는 $x_{i+k+1}$ 예측</li>
<li>헤드들은 원본 모델과 함께 학습되지만 원본 모델은 동결됨</li>
<li>HGX H200 GPU에서 라마 3.1 토큰 생성을 최대 1.9배 향상시켰다고 주장</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>토큰들이 순차적으로 생성되지 않기 때문에, 서로 잘 맞는지 확인하기 위한 <b>검증</b>이 필요함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>병렬 디코딩에서 가장 중요한 <b>검증과 통합</b></li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>룩어헤드 디코딩은 야코비 방법 Jacobi method을 사용해서 생성된 토큰을 검증함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>K개 미래 토큰이 병렬로 생성됨</li>
<li>K개 토큰이 컨텍스트와 일관성이 있는지, 서로 의미가 통하는지 검증</li>
<li>하나 이상의 토큰이 검증에 실패하면, K개 토큰 전체를 합치는 대신 모델은 실패한 토큰만 다시 만들거나 고침</li>
</ul>
</li>
<li>메두사는 트리 기반 어텐션 메커니즘을 사용해서 토큰을 검증함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>메두사 헤드는 확정된 단어 하나가 아닌 여러개를 생성, 이 후보들은 트리 구조로 구성된 다음 가장 유망한 조합을 선택하는데 사용됨</li>
</ul>
</li>
</ul>
<img width="1247" height="953" alt="image" src="https://github.com/user-attachments/assets/2058829d-91c0-43e4-8c0f-4399f2ccb3bf" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">어텐션 메커니즘 최적화</h4>
<p data-ke-size="size16">다음 토큰을 생성하기 위해서는 이전 모든 토큰의 키, 값 벡터가 필요함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>토큰 $x_t$를 생성하려면 토큰 $x_1, x_2, ..., x_{t-1}$의 키와 값 벡터가 필요함</li>
</ul>
<p data-ke-size="size16">토큰 $x_{t+1}$을 생성할 때 토큰 $x_1, ... x_{t-1}$의 키와 값 벡터를 다시 연산하는 대신, 이전 단계에서 연산한 벡터들을 재사용함 $\rightarrow$ 가장 최큰 토큰 $x_t$의 벡터만 새로 연산하면 됨</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>재사용을 위해 키와 값 벡터를 저장하는 캐시를 <b>KV 캐시</b>라고 함, 새로 연산된 키와 값 벡터는 KV 캐시에 추가됨
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>KV 캐시는 추론 시에만 사용된다</li>
</ul>
</li>
</ul>
<img width="1237" height="650" alt="image" src="https://github.com/user-attachments/assets/5bdbe281-d8f4-4f1a-8d69-67eb1a2c8a2c" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>토큰 하나를 생성하려면 이전 모든 토큰과의 어텐션 점수를 연산해야함 $\rightarrow$ 어텐션 연산량은 시퀀스 길이에 따라 제곱에 비례해서 증가함, KV 캐시 크기는 시퀀스 길이에 따라 선형적으로 증가함</li>
<li>KV 캐시 크기는 배치 크기가 클수록 더 커짐
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>멀티헤드 어텐션을 가진 500B+ 모델에서 배치 크기 512, 컨텍스트 길이 2048일때 KV캐시는 3TB, 모델 가중치의 3배에 달함</li>
</ul>
</li>
<li>KV 캐시 크기는 하드웨어 저장 공간에 의해 제한됨, 롱 컨텍스트를 처리하는 애플리케이션에 병목 현상을 일으킴.</li>
<li>캐시 크기가 크면 메모리에 로드하는데에도 시간이 걸림</li>
<li>캐시 크기 연산 : 2 * 배치 크기 * 시퀀스 길이 * 트랜스포머 레이어 개수 * 모델 차원 * 캐시 수치를 저장하는데 필요한 메모리 (FP16, FP 32 등)</li>
<li>컨텍스트가 길어질수록 값이 엄청나게 커질 수 있음</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">어텐션 메커니즘 재설계</h4>
<p data-ke-size="size16">모델 아키텍처를 직접 변경하기 때문에 학습 또는 파인튜닝 단계에서만 적용할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>로컬 윈도우 어텐션</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/2004.05150v2" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2004.05150v2</a>&nbsp;</li>
<li>새 토큰 생성 시 이전 모든 토큰에 어텐션을 적용하는 대신, 인접한 고정된 크기의 윈도우 안에 있는 토큰에만 어텐션 적용</li>
<li>유효 시퀀스 길이가 고정된 윈도우 크기로 줄어듦</li>
<li>로컬 윈도우 어텐션과 글로벌 어텐션을 섞어 쓸 수도 있음 : 로컬 윈도우 어텐션은 가까운 맥락, 글로벌 어텐션은 문서 전체에서 작업에 필요한 정보 찾기</li>
</ul>
</li>
<li><b>크로스 레이어 어텐션 / 멀티 쿼리 어텐션</b>&nbsp;
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/1911.02150?ref=research.character.ai">https://arxiv.org/abs/1911.02150?ref=research.character.ai</a>&nbsp;</li>
<li><a href="https://arxiv.org/abs/2405.12981?ref=research.character.ai">https://arxiv.org/abs/2405.12981?ref=research.character.ai</a></li>
<li>&nbsp;키-값 쌍 개수를 줄여서 KV 캐시의 메모리 사용량을 줄임</li>
<li>크로스 레이어 어텐션 : 인접한 레이어들끼리와 키-값 벡터 공유. 레이어 3개가 같은 키-값 벡터를 공유하면 KV 캐시가 3분의 1로 줄어듦</li>
<li>멀티 쿼리 어텐션 : 쿼리 헤드들끼리 키-값 벡터를 공유
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>8개의 쿼리 헤드가 있다면 키, 값은 1개씩</li>
</ul>
</li>
</ul>
</li>
<li><b>그룹 쿼리 어텐션</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/2305.13245">https://arxiv.org/abs/2305.13245</a> &nbsp;</li>
<li>&nbsp;멀티 쿼리 어텐션 발전시킨 방법</li>
<li>모든 쿼리 헤드에 하나의 키-값 쌍 세트만 사용하는 대신, 쿼리 헤드들을 더 작은 그룹으로 나누고 같은 그룹 안에서만 키-값 쌍을 공유</li>
<li>쿼리 헤드 수와 키-값 쌍 수 사이에서 더 유연하게 조절 가능함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>8개 쿼리 헤드에서 2의 그룹으로 나누면 키, 값은 2개씩</li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">AI 챗봇 애플리케이션 Character.AI에서는 평균 대화에 180개 메시지 대화 기록이 쌓인다고 밝힘</p>
<p data-ke-size="size16">$\rightarrow$ 시퀀스가 길다, 추론 처리량의 주요 병목은 KV 캐시</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">KV 캐시 크기 최적화</h4>
<p data-ke-size="size16">KV 캐시 관리 방식은 추론 시 메모리 병목 현상 완화, 롱 컨텍스트를 다루는 애플리케이션에서 더 큰 배치 크기를 가능하게 함, KV 캐시를 줄이고 관리하는 많은 기법들이 개발되고 있음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>vLLM</b> : <b>페이지드 어텐션 Paged Attention&nbsp;</b>도입
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/2309.06180" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2309.06180</a></li>
<li>KV 캐시를 불연속적인 블록, Page 단위로 나누어 메모리 단편화를 줄임, 버려지던 공간 제거로 메모리 활용 극대화</li>
<li>여러 사용자의 요청이 동일한 메모리를 유연하게 공유하여 LLM 서빙 효율 개선 및 메모리 관리 최적화 (토큰 Throughput 높임)</li>
</ul>
</li>
<li><b>KV 캐시 양자화</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/2401.18079">https://arxiv.org/abs/2401.18079</a></li>
<li><a href="https://arxiv.org/abs/2403.05527">https://arxiv.org/abs/2403.05527</a> &nbsp;</li>
<li>KV 벡터를 INT8이나 INT4로 변환하여 저장</li>
</ul>
</li>
<li><b>적응형 KV 캐시 압축</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://arxiv.org/abs/2310.01801" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2310.01801</a>&nbsp;</li>
<li>&nbsp;문장이 길어질 때 어텐션 점수가 낮아 중요도가 떨어지는 토큰을 실시간으로 식별</li>
<li>중요하지 않은 토큰의 KV를 삭제하거나 압축하여 캐시 크기를 일정 수준 이하로 <b>동적으로</b> 유지</li>
<li>핵심적인 문맥 정보를 놓치지 않으면서 불필요한 계산 줄이기</li>
</ul>
</li>
<li><b>선택적 KV 캐시</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><a href="https://proceedings.mlsys.org/paper_files/paper/2024/hash/48fecef47b19fe501d27d338b6d52582-Abstract-Conference.html">https://proceedings.mlsys.org/paper_files/paper/2024/hash/48fecef47b19fe501d27d338b6d52582-Abstract-Conference.html</a></li>
<li>이후 추론 과정에서 참조될 가능성이 높은 핵심 정보만 골라 캐싱</li>
<li>특정 레이어나 특정 토큰의 KV 값만 선택적으로 유지</li>
<li>추론 지연 시간 Latency 최적화</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">어텐션 연산을 위한 커널 작성</h4>
<p data-ke-size="size16">메커니즘 설계를 바꾸거나 저장 공간 최적화 대신, 어텐션 점수를 어떻게 연산하는지 살펴보고 연산을 더 효율적으로 만드는 방법을 찾음</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>플래시 어텐션</b> : 어텐션 연산에 최적화된 커널
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>트랜스포머 기반 모델에서 많이 사용되는 여러 연산을 하나로 융합해서 더 빠르게 실행되도록 함</li>
</ul>
</li>
</ul>
<img width="1116" height="438" alt="image" src="https://github.com/user-attachments/assets/4fedc5c8-f720-4e12-a213-d7efde671ff7" />

<h4 data-ke-size="size20">커널과 컴파일러&nbsp;</h4>
<p data-ke-size="size16"><b>커널 kernel</b>: GPU나 TPU 같은 특정 하드웨어 가속기에 맞춰 최적화한 특수 코드, 연산량이 많고 반복적으로 실행해야 하는 작업을 처리하도록 만들어짐, 병렬로 실행해 가속기 성능을 최대한 끌어냄</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>행렬 곱셈, 어텐션 연산, 합성곱 연산 등 일반적인 AI 연산은 하드웨어에서 전용 커널을 가지고 있음</li>
</ul>
<p data-ke-size="size16">커널 작성을 위해서는 하드웨어 아키텍처를 깊게 이해해야 함</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>캐시, 글로벌 메모리, 공유 메모리, 레지스터 등&nbsp;</li>
<li>메모리 계층 구조에서 레벨 간의 데이터 접근 및 이동 방식</li>
</ul>
<p data-ke-size="size16">커널은 CUDA, 트리톤, ROCm과 같은 저수준 프로그래밍 언어로 작성됨 $\rightarrow$ 어려움</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">연산 속도를 높이는 데 자주 쓰이는 네 가지 일반적인 기법들</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>벡터화</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>루프나 중첩 루프에서 데이터를 하나씩 처리하는 대신, 메모리에서 연<b>속으로 배치된 여러 데이터를 동시에 처리</b></li>
<li>I/O 연산이 적어져 지연 시간이 줄어듦</li>
</ul>
</li>
<li><b>병렬화</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>입력 배열을 여러 코어나 스레드에서 <b>동시에 처리</b>할 수 있는 독립적인 덩어리로 나누어 연산 속도 높임</li>
</ul>
</li>
<li><b>루프 타일링 tiling</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>하드웨어의 메모리 레이아웃과 캐시에 맞게 루프의 데이터 접근 순서를 최적화</li>
<li>하드웨어에 따라 달라지며 효율적인 CPU 타일링 패턴이 GPU에서는 잘 작동하지 않을 수 있음</li>
</ul>
</li>
<li><b>연산자 융합</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>여러 연산자를 단일 패스로 결합하여 불필요한 메모리 접근 피함</li>
<li>e.g. 두 개의 루프가 동일한 배열에 작동한다면, 루프 하나로 합쳐서 데이터 읽기 및 쓰기 횟수를 줄임</li>
<li>앞서 다른 기법들은 여러 모델에 폭넓게 쓸 수 있지만, 연산자 융합은 모델의 특정 연산자와 아키텍처를 더 깊이 알아야 함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>커널은 새로운 하드웨어 아키텍처가 나올 때마다 새로운 커널을 개발해야 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델 스크립트 : 해당 모델을 실행하기 위해 해야 할 일련의 연산들 정의
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>로어링 lowering </b>: 특정 하드웨어에서 코드를 실행하기 위해 해당 하드웨어와 호환되는 언어로 변환하는 과정</li>
<li><b>컴파일러</b> : 특정 하드웨어에서 실행되도록 코드를 로어링하는 도구
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>ML 모델과 하드웨어 사이를 연결</li>
<li>로어링 과정에서 연산들을 더 빠르게 돌아가는 전용 커널로 바꿈</li>
</ul>
</li>
</ul>
</li>
<li>파이토치 팀은 다음 최적화 과정을 거쳐서 라마-7B의 처리량을 개선함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>torch.compile로 모델을 더 효율적인 커널로 컴파일</li>
<li>모델 가중치를 INT8로 양자화</li>
<li>모델 가중치를 INT4로 한 번 더 양자화</li>
<li>추측 디코딩 추가
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>But, 품질에 얼마나 영향을 주는지는 아직 정확하게 밝혀지지 않음</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="783" height="716" alt="image" src="https://github.com/user-attachments/assets/b4254054-f187-4fa7-8f0d-612021652ab3" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">컴파일러는 아파치 TVM이나 MLIR 같은 독립적인 도구일 수 있고, ML과 추론 프레임워크에 통합되기도 함 (torch.compile, XLA, TensorRT)</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">추론 서비스 최적화</h3>
<p data-ke-size="size16">서비스 수준 최적화 기법은 리소스 관리에 집중함</p>
<p data-ke-size="size16">한정된 리소스와 계속 변하는 작업량이 있을 때, 목표는 지연 시간과 비용을 최적화하도록 작업량에 리소스를 효율적으로 배분하는 것</p>
<p data-ke-size="size16">서비스 수준 기법은 모델을 수정하지 않고 출력 품질도 바꾸지 않아야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">배치 처리</h4>
<p data-ke-size="size16">추론 서비스는 여러 요청을 동시에 받을 수 있음</p>
<p data-ke-size="size16">비슷한 시간에 들어온 요청을 묶어서 배치 처리하여 서비스 처리량을 크게 높일 수 있음</p>
<p data-ke-size="size16">배치처리를 똑똑하게 하면 지연 시간에 미치는 영향을 최소화할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>정적 배치 처리 static batching</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>정해진 수의 입력을 하나의 배치로 묶음 (자리 다 차면 출발하는 버스랑 비슷)</li>
<li>배치의 마지막 요청이 도착할때까지 지연됨</li>
</ul>
</li>
<li><b>동적 배치 처리 dynamic batching</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>각 배치에 최대 대기 시간 설정 (e.g. 배치 크기 4, 최대 대기 시간 100ms, 둘 중 하나 일어나면 처리)</li>
<li>배치가 항상 꽉 차 있지 않을 수 있어 연산 자원 낭비일 수 있음</li>
</ul>
</li>
<li>기본적인 배치 처리 구현은 배치 안의 모든 요청이 끝나야 응답을 반환할 수 있음, 요청마다 걸리는 시간이 다를 수 있음, 짧은 요청에 불필요한 지연 발생</li>
<li><b>연속 배치 처리 continuous batching</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>배치의 응답이 끝나는 대로 바로 사용자에게 반환</li>
<li>배치를 고정하지 않고 매 순간 계산이 필요한 토큰만 골라서 배치를 구성</li>
<li>배치에서 요청이 끝나고 응답이 반환된 후, 그 자리에 다른 요청을 배치에 추가해서 배치 처리가 계속 이어지게 함</li>
<li>아래 그림은 인플라이트 배치 처리 in-flight batching 이라고 불리는 연속 배치 처리 예시</li>
<li><a href="https://www.usenix.org/conference/osdi22/presentation/yu" target="_blank" rel="noopener&nbsp;noreferrer">https://www.usenix.org/conference/osdi22/presentation/yu</a></li>
</ul>
</li>
</ul>
<img width="1113" height="755" alt="image" src="https://github.com/user-attachments/assets/0df01770-614e-4a09-b6c7-e5b6c4cc937a" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">프리필과 디코딩 분리</h4>
<p data-ke-size="size16">프리필 : 연산 집약적</p>
<p data-ke-size="size16">디코딩 : 대역폭 집약적</p>
<p data-ke-size="size16">$\rightarrow$ 동일한 장비에서 두 작업을 수행하면 비효율적인 리소스 경쟁이 발생해 TTFT와 TPOT 모두 크게 느려질 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>프리필과 디코딩을 다른 인스턴스에 할당하면, 지연 시간 요구사항을 지키면서도 처리되는 요청량을 크게 늘릴 수 있음, 프리필 인스턴스에서 디코딩 인스턴스로 중간 상태를 전송해야 하지만, NVLink 같은 고대역폭 연결을 갖춘 GPU 클러스터에서 통신 오버헤드가 크지 않음을 보여줌
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><a href="https://arxiv.org/abs/2401.11181" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2401.11181</a></li>
<li><a href="https://arxiv.org/abs/2401.09670" target="_blank" rel="noopener&nbsp;noreferrer">https://arxiv.org/abs/2401.09670</a></li>
</ul>
</li>
</ul>
<p data-ke-size="size16">프리필 인스턴스와 디코딩 인스턴스의 비율은 워크로드 특성이나 지연 시간 요구 사항 등에 따라 달라짐 (입력 길이가 길수록 더 많은 프리필 인스턴스가 필요, TTFT와 TPOT 중 무엇을 낮추고 싶은지)</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">프롬프트 캐싱</h4>
<p data-ke-size="size16">여러 프롬프트에는 텍스트의 일부가 겹치는 부분이 있으며 프롬프트 캐시는 이런 겹치는 부분을 저장해둠</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>시스템 프롬프트 : 공통적으로 겹치는 대표적인 부분</li>
<li>긴 문서를 다루는 질의에도 유용 : 긴 문서를 캐시에 저장해 여러 질의에 재사용, 긴 대화에서의 이전 메시지들의 처리 결과 저장</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">프롬프트 캐시는 컨텍스트 캐시, 프리픽스 캐시라고도 불림</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">KV 캐시처럼 프롬프트 캐시도 크기가 꽤 클 수 있으며 메모리 공간을 차지함, 이 기능이 포함된 모델 API를 사용하지 않는 한, 프롬프트 캐싱을 직접 구현하려면 상당한 엔지니어링 노력이 필요</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>구글 제미나이 : 이 기능을 제공, 캐시된 입력 토큰에 일반 입력 토큰 대시 75% 할인 제공, 캐시 저장 공간은 추가 비용 지불 해야함</li>
<li>앤트로픽 : 90% 비용 절감과 최대 75% 지연 감소를 약속하는 캐싱 기능을 제공</li>
</ul>
<img width="1137" height="429" alt="image" src="https://github.com/user-attachments/assets/2ac93588-eba4-4dc0-858e-07eed2c0565b" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">병렬 처리</h4>
<p data-ke-size="size16">병렬 처리 전략은 고성능 컴퓨팅의 핵심</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>데이터 병렬 처리</b></li>
<li><b>모델 병렬 처리</b></li>
<li><b>컨텍스트/시퀀스 병렬 처리 (LLM)</b></li>
</ul>
<p data-ke-size="size16">하나의 최적화 기법에 여러 병렬 처리 전략이 포함될 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>복제 병렬 처리 replica parallelism</b> : 가장 구현하기 간단한 전략, 서비스하려는 모델의 복제본을 여러 개 만들어 작업을 병렬화하는 방식
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>복제본이 많을수록 더 많은 칩이 필요, 모델+복제본+칩의 종류가 많아질수록 문제가 복잡해짐</li>
</ul>
</li>
<li><b>모델 병렬 처리</b> : 하나의 모델을 여러 머신에 분산시키는 방식, 모델을 여러 칩에 배치하는 문제가 훨씬 더 복잡해짐</li>
</ul>
<p data-ke-size="size16">모델 분할 방법</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>텐서 병렬 처리</b> (연산자 내 병렬 처리) : 추론에서 가장 일반적인 접근법
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>한 연산자에 포함된 텐서들을 여러 장치에 걸쳐 분할하여, 해당 연산자를 병렬로 실행될 수 있는 더 작은 조각으로 효과적으로 나눔, 연산 속도 높이기 (가중치와 hidden 차원 분할)</li>
<li>단일 머신에 맞지 않는 큰 모델을 서비스할 수 있게 해줌</li>
<li>지연 시간을 줄이지만, 지연 시간 감소 효과가 추가적인 통신 오버 헤드 때문에 줄어들 수 있음</li>
</ul>
</li>
</ul>
<img width="1124" height="649" alt="image" src="https://github.com/user-attachments/assets/4f06fdf8-1ed2-4e1f-8108-54e9ed455a5a" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>파이프라인 병렬 처리&nbsp;</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모델의 연산을 여러 개의<b> 독립적인 단계</b>로 나누고 각 단계를 다른 장치에 할당하는 것 (레이어 블록 dpeth를 stage로 나눔)</li>
<li>데이터가 모델을 통과하면서 각 단계는 데이터의 한 부분을 처리하고 다른 단계들은 그다음 부분들을 처리하여 연산이 겹치도록 할 수 있음</li>
<li>하나의 배치가 더 작은 <b>마이크로 배치</b>로 분할될 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>마이크로 배치가 처리된 후, 다음 장비에 있는 모델의 다음 부분으로 전달됨</li>
</ul>
</li>
<li>여러 머신에서 큰 모델을 서비스할 수 있게 해주지만, 파이프라인 단계 간의 추가 통신 때문에 각 요청의 총 지연 시간은 증가함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>엄격한 지연 시간 요구사항이 있는 경우, 추론시에는 복제 병렬 처리를 선호함</li>
<li>파이프라인 병렬 처리를 처리량을 높일 수 있어 학습 시에 일반적으로 사용됨</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="1127" height="647" alt="image" src="https://github.com/user-attachments/assets/02d66d17-719a-4d5d-b0b4-607d4b5ca1f8" />

<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>컨텍스트/시퀀스 병렬 처리</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>긴 입력 시퀀스 처리를 더 효율적으로 만들기 위해 개발됨</li>
<li>둘다 Sequence Length 축 분할</li>
<li><b>컨텍스트 병렬 처리 context parallelism</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>입력 시퀀스 자체가 여러 다른 디바이스로 분할되어 같은 레이어 연산을 토큰 축으로 병렬화 (sequence length 축 분할, 긴 컨텍스트 prefill attention 가속용)</li>
<li>e.g. 머신 1이 토큰 0~(L/2 - 1) , 머신 2가 토큰 (L/2)~(L-1)</li>
<li>타겟 연산이 self-attention으로 여러 머신이 attention 결과 공유</li>
</ul>
</li>
<li><b>시퀀스 병렬처리 sequence parallelism</b><br />
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>텐서 병렬 처리와 함께 사용</li>
<li>시퀀스 병렬 처리에서는 Attention 뿐만 아니라 LayerNorm, Dropout 등 레이어 내부 연산 전반에 토큰 축으로 병렬화 (sequence length 축 분할)</li>
<li>TP와 함께 쓸때 주변 연산에서의 중간값들을 토큰 축으로 유지하여 hidden 전체를 모으는 것을 피하고 필요한 경우(LN) 작은 통계값만 통신하여 통신과 메모리 병목을 줄임
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>LN의 경우, 한 토큰 hidden 차원 전체를 보고 평균과 분산을 내야함</li>
</ul>
</li>
</ul>
</li>
<li>큰 모델에 긴 프롬프트라면 주로 텐서 병렬 처리와 컨텍스트 병렬 처리르 함께 쓰며 시퀀스 병렬 처리는 추가적인 최적화 역할 정도</li>
</ul>
</li>
</ul>
