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

---

- [블로그 링크](https://armugona.tistory.com/entry/AIE-Ch9-%EC%B6%94%EB%A1%A0-%EC%B5%9C%EC%A0%81%ED%99%94?category=1273755)
  
#10장 
<h2 data-ke-size="size26">AI 엔지니어링 아키텍처</h2>
<p data-ke-size="size16">가장 단순한 아키텍처에서 시작해 더 많은 구성요소를 추가하며 볼 것</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">가장 단순한 형태 : 애플리케이션이 질의를 받아 모델로 보내는 것</p>
<img width="795" height="330" alt="image" src="https://github.com/user-attachments/assets/1d9822db-4b26-43c3-8366-57165c47ba86" />

<p data-ke-size="size16">모델 API 상자는 서드파티 API와 자체 호스팅 모델 모두 가리킴</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">추가할 수 있는 구성요소들</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>정보 수집을 위해 외부 데이터 소스와 도구에 접근할 수 있게 해서, 모델에 입력되는 컨텍스트 보강</li>
<li>시스템과 사용자 보호를 위한 가드레일 도입</li>
<li>복잡한 파이프라인 지원, 보안을 강화하기 위해 모델 라우터와 게이트웨이를 추가</li>
<li>캐싱을 통해 지연 시간과 비용 최적화</li>
<li>시스템 성능 극대화를 위해 복잡한 로직과 실행 기능 추가</li>
</ul>
<p data-ke-size="size16">모니터링 monitoring, 관찰 가능성 observability, 오케스트레이션 orchestration에 대해서도 다룰 예정</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">1.컨텍스트 보강</h3>
<p data-ke-size="size16">검색 메커니즘 : 텍스트 검색, 이미지 검색, 표 형태 데이터 검색 등</p>
<p data-ke-size="size16">도구 : 웹 검색, 뉴스, 날씨, 이벤트 등 API 사용해서 자동으로 정보 수집</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>컨텍스트 구성 context construction</b> 은 파운데이션 모델을 위한<b> 특성 공학 feature engineering </b>과 같음</p>
<p data-ke-size="size16">: 모델이 출력을 생성하는 데 필요한 정보를 제공하는 것, 컨텍스트 구성이 시스템의 출력 품질에 핵심적인 역할</p>
<p data-ke-size="size16">따라서 대부분의 모델 API 제공 업체가 파일 업로드 등 기능을 제공하지만 컨텍스트 구성을 지원하는 방식이 제각각임</p>
<p data-ke-size="size16">전문 RAG 솔루션이라면 벡터 DB 용량이 허용하는 만큼 문서를 무제한으로 올릴 수 있지만 범용 API는 문서 몇 개만 올릴 수 있음</p>
<p data-ke-size="size16">프레임 워크마다 검색 알고리즘, 청크 크기 같은 검색 설정이 다름, 솔루션마다 도구 사용에서도 도구 지원 범위, 여러 함수 병렬 실행, 오래걸리는 작업 처리 등 다름</p>
<img width="1150" height="508" alt="image" src="https://github.com/user-attachments/assets/e235bb6c-b51a-42a1-8ae2-c99e4b224f33" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">2. 가드레일 도입하기</h3>
<p data-ke-size="size16"><b>가드레일 Guardrail</b> : 위험을 줄이고 애플리케이션 제공자와 사용자를 보호하는 역할을 함, 위험에 노출될 수 있는 모든 지점에 가드레일을 배치해야 함</p>
<p data-ke-size="size16">일반적으로 <b>입력 가드레일, 출력 가드레일</b>로 나눔&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">입력 가드레일</h4>
<p data-ke-size="size16">두 가지 유형의 위험을 막아줌</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>위부 API로 개인정보가 유출되는 것</li>
<li>시스템을 망가뜨릴 수 있는 악성 프롬프트 실행</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">5장에서 프롬프트 해킹 방법과 방어 기법을 다뤘으나, 이 기법은 위험을 줄일 수는 있지만 모델이 응답을 만드는 고유한 방식과 사람이 저지르는 실수 때문에 완전히 없앨 수 없음</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>외부 API로 개인정보가 유출되는 위험 : 데이터를 조직 외부로 보내야 하는 외부 모델 API를 사용할 때의 문제<br />
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>직원이 회사 기밀이나 사용자 개인정보를 프롬프트에 복사해서 서드파티 API로 전송</li>
<li>애플리케이션 개발자가 회사 내부 정책과 데이터를 애플리케이션의 시스템 프롬프트에 넣는 경우</li>
<li>도구가 내부 데이터베이스에서 개인정보를 가져와서 컨텍스트에 추가하는 경우
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>이러한 잠재적인 유출을 완벽하게 막을 방법은 없으나 가드레일을 통해 줄이고자 함</li>
<li>민감한 데이터를 자동으로 탐지하는 여러 상용 도구 중 하나를 사용하면 됨 : 어떤 민감한 데이터를 탐지할지는 직접 정해야 함 (개인정보(주민번호, 전화번호, 계좌번호), 사람 얼굴, 회사 지적 재산이나 기밀 정보와 관련된 특정 키워드/문구)</li>
</ul>
</li>
</ul>
</li>
<li>민감 데이터 탐지 도구는 AI를 사용해 잠재적으로 민감할 수 있는 정보를 식별함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>특정 문자열이 실제 집 주소와 유사한지 판단</li>
</ul>
</li>
<li>민감한 정보가 포함된 것으로 확인되면, 질의 전체를 차단하거나 / 민감한 정보만 제거</li>
</ul>
<img width="1085" height="822" alt="image" src="https://github.com/user-attachments/assets/f1b3ee8e-f9e9-4caf-a61c-ff5ed67d1962" />

<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">출력 가드레일</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>출력 실패 탐지</li>
<li>다양한 실패 유형을 처리하는 정책 명시</li>
</ul>
<p data-ke-size="size16">실패 양상은 애플리케이션마다 다름</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>품질과 보안</b>에서 자주 보는 실패 사례</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>품질
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>예상한 출력 형식을 따르지 않는 잘못된 형식의 응답 (JSON을 예상했으나 유효하지 않은 JSON 생성)</li>
<li>환각</li>
<li>수준이 낮은 응답, 결과물의 질이 매우 나쁜 경우</li>
</ul>
</li>
<li>보안
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>인종차별적이거나 성적인 콘텐츠 또는 불법적인 활동을 담은 유해한 응답</li>
<li>개인정보나 민감한 정보가 들어있는 응답</li>
<li>원격 도구나 코드 실행을 유발하는 응답</li>
<li>자사나 경쟁사에 대해 잘못 설명해서 브랜드에 위험을 초래하는 응답</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">보안 측정 시에는 오거부율 지표 false refusal rate도 확인해야 함 : 보안을 너무 강하게 적용하면 괜찮은 요청까지 차단하여 사용자의 작업을 방해하고 불편을 초래함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">많은 실패는 간단한 재시도 로직으로완화할 수 있음: 응답이 비어있으면 N번 다시 시도, 형식이 잘못됐다면 올바른 형식이 나올때까지 $\rightarrow$ 지연 시간과 비용이 늘어날 수 있음, 재시도를 할 때마다 API를 한 번 더 호출해야 함&nbsp;</p>
<p data-ke-size="size16">$\rightarrow$ 지연 시간을 줄이기 위해 호출을 병렬로 처리할 수도 있음</p>
<p data-ke-size="size16">까다로운 요청은상담원에서 전달하기 등 대화를 언제 사람에게 넘길지 결정하기 위한 모델을 사용하기도 함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">가드레일 구현</h4>
<p data-ke-size="size16">가드레일의 트레이드 오프 : <b>신뢰성과 지연 시간</b>의 트레이드 오프</p>
<p data-ke-size="size16">지연 시간을 줄이기 위한 스트림 완성 모드(실시간으로 생성된 토큰을 전달stream하는 방식)에서는 출력 가드레일이 제대로 작동하지 않을 수 있음&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">얼마나 많은 가드레일을 구현할 지는 모델을 자체 호스팅하는지, 서드파티 API를 쓰는지에 따라 달라짐&nbsp;</p>
<p data-ke-size="size16">서드파티 API를 사용하면 제공업체에서 기본으로 가드레일을 제공해 구현해야 할 가드레일의 수를 줄일 수 있음, 모델을 자체 호스팅하면 요청을 외부로 보낼 필요가 없어서 여러 유형의 입력 가드레일에 대한 필요성이 줄어듦</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">애플리케이션이 실패할 수 있는 지점이 매우 다양하므로 가드레일도 다양한 레벨에서 구현할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">사용할 수 있는 가드레일 솔루션 : 메타의 퍼플 라마, 엔비디아의 네모 가드레일, 애저의 파이릿, 애저의 AI 콘텐츠 필러, 퍼스펙티브 API, 오픈AI의 콘텐츠 조정 AI 등</p>
<p data-ke-size="size16">입력과 출력의 위험이 서로 겹치는 부분이 많아 가드레일 솔루션은 입력과 출력을 모두 보호하는 기능을 제공함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">가드레일 추가 시 아키텍처</p>
<img width="1145" height="621" alt="image" src="https://github.com/user-attachments/assets/9716ce6a-355f-4efd-b046-bb13b3a9421a" />

<p data-ke-size="size16">스코어러 Scorer(평가기)는 보통 생성 모델보다 작고 빠르지만 AI 기반인 경우가 많음, 출력 가드레일 상자에 들어가도 됨</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">3. 모델 라우터와 게이트웨이 추가</h3>
<p data-ke-size="size16">여러 모델을 서빙하는데 따르는 복잡성과 비용을 관리하기 위한 라우터와 게이트웨이가 필요함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">라우터</h4>
<p data-ke-size="size16">모든 질의에 하나의 모델만 사용하는 대신, 질의 유형별로 각기 다른 솔루션을 사용할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">장점</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>특정 질의에 대해서 범용 모델보다 성능이 더 좋을 수 있는 특화 모델을 사용할 수 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>기술적인 문제 해결에 특화된 모델과 요금 청구에 특화된 모델</li>
</ul>
</li>
<li>비용 절약
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>모든 질의에 비싼 모델을 쓰지 않고 단순한 질의는 저렴한 모델로</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">라우터는 사용자가 무엇을 하려 하는지 예측하는 <b>의도 분류기 intent classifier</b>로 구성됨</p>
<p data-ke-size="size16">예측된 의도를 바탕으로 질의를 적절한 솔루션으로 보냄</p>
<p data-ke-size="size16">시스템이 범위를 벗어난 대화에 빠지는 것을 막을 수 있음 : 질의가 부적절하다고 판단해 응답을 거절하거나 애매한 질의 감지</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델이 다음에 무엇을 할지 결정하는 데 도움을 주는 종류의 라우터 : 여러 작업이 가능한 에이전트의 경우 다음에 어떤 행동을 해야할지 예측하는 <b>다음 행동 예측기 next-action predictor</b>의 역할을 할 수 있음, 메모리 시스템을 갖춘 경우, 모델이 메모리 계층의 어느 부분에서 정보를 가져와야 할지 예측, 첨부된 문서 정보에 의존할지 질의에 대해 인터넷 검색할지 결정</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">의도 분류기와 다음 행동 예측기는 파운데이션 모델을 기반으로 구현할 수 있음 : GPT-2, BERT, 라마 7B같은 작은 언어 모델을 의도 분류기로 활용, 작은 분류기를 처음부터 만들기도 $\rightarrow$ <b>라우터는 빠르고 저렴해야 함</b>, 여러개를 사용해도 지연 시간과 비용이 크게 늘어나지 않도록</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">컨텍스트 한계가 있는 모델로 질의를 라우팅할 땐 질의의 컨텍스트를 그에 맞게 조정해야할 수 있음 : 4k 컨텍스트 한계가 있는 모델로 보낼 예정인 1,000토큰 짜리 질의가 있을때 8,000토큰의 컨텍스트를 웹 검색으로 가져왔다고 가정, 모델에 맞게 질의의 컨텍스트를 잘라내거나 더 큰 컨텍스트를 가진 모델로 질의를 라우팅</p>
<p data-ke-size="size16">&nbsp;</p>
<img width="1135" height="719" alt="image" src="https://github.com/user-attachments/assets/562d031e-95ac-4df3-8fd9-a159e82642b9" />

<p data-ke-size="size16">라우터를 다른 모델과 함께 묶으면 모델을 더 쉽게 관리할 수 있음</p>
<p data-ke-size="size16">라우팅의 위치는 유연하게 정할 수 있지만(라우팅-검색, 검색-라우팅도 가능), 보통은 라우팅-검색-생성-스코어링(평가)의 흐름이 일반적임&nbsp;</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">게이트웨이</h4>
<p data-ke-size="size16">조직이 다양한 모델과 통합되고 안전한 방식으로 상호작용할 수 있게 해주는 중간 계층</p>
<p data-ke-size="size16">가장 기본적인 기능 : 자체 호스팅 모델과 상용 API 뒤에 있는 모델을 포함한 다양한 모델에 통합 인터페이스를 제공하는 것, 코드 유지보수가 더 쉬워짐 $\rightarrow$ 모델 API가 변경되더라도 이 API에 의존하는 모든 애플리케이션을 업데이트할 필요 없이 게이트웨이만 업데이트하면 됨</p>
<img width="907" height="538" alt="image" src="https://github.com/user-attachments/assets/17422ae5-b519-412c-a1ad-fdcb4494bc13" />

<p data-ke-size="size16">가장 단순한 형태의 모델 게이트웨이는 통합 래퍼</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모델 게이트웨이는 <b>접근 제어 access control, </b><b>비용 관리 cost management</b> 기능을 제공함</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>API에 접근하려는 모든 사람에게 쉽게 유출될 위험이 있는 조직 토큰을 직접 주는 대신, 중앙에서 접근을 통제할 수 있는 모델 게이트웨이라는 단일 접근 지점을 만들어 그곳에 접근 권한을 부여</li>
<li>사용자나 애플리케이션이 어떤 모델에 접근해야 하는지 명시하는 세분화된 접근 제어를 구현할 수도 있음</li>
<li>게이트웨이는 API 호출 사용량을 모니터닝하고 제한해서 남용을 방지하고 비용을 효과적으로 관리할 수 있음</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">속도 제한이나 API 실패를 극복하기 위한 <b>폴백 정책 fallback policy</b>을 만드는 데 활용</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>주요 API를 사용할 수 없을 때, 게이트 웨이는 요청을 대체 모델로 보내거나, 잠시 기다린 후 재시도하거나, 다른 방식으로 실패를 원활하게 처리할 수 있음</li>
<li>애플리케이션이 중단 없이 원활하게 작동하도록 보장할 수 있음</li>
</ul>
<p data-ke-size="size16">모든 요청과 응답이 게이트웨이를 거치게 되므로, 게이트웨이는 로드 밸런싱, 로깅, 분석 같은 다른 기능을 구현하기에 가장 적합한 장소 (일부는 캐싱이나 가드레일을 제공하기도 함)</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">포트키의 AI 게이트웨이, MLflow AI 게이트웨이, 웰스심플의 LLM 게이트웨이, 트루파운드리, 콩, 클라우드플레어 등 사용할 수 있음</p>
<img width="1167" height="753" alt="image" src="https://github.com/user-attachments/assets/ffe2fb96-afe2-4fdb-8a32-1a7bf847974c" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">4. 캐시로 지연 시간 줄이기</h3>
<p data-ke-size="size16">시스템 캐싱 메커니즘</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>완전 일치 캐싱 exact caching</b></li>
<li><b>시맨틱 캐싱 semantic caching</b></li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">완전 일치 캐싱</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>정확히 같은 항목이 요청될 때만 캐시된 항목을 사용
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>벡터 검색이 중복되는 것을 피하기 위해 임베딩 기반 검색에서도 사용됨 : 들어온 질의가 이미 벡터 검색 캐시에 있다면 캐시된 결과를 가져옴, 없으면 벡터 검색 수행 후 결과를 캐시에 저장</li>
<li>CoT처럼 여러 단계를 포함하거나 검색, SQL 실행, 웹 검색처럼 시간이 오래 걸리는 동작이 포함된 질의에 특히 매력적임</li>
</ul>
</li>
<li>빠른 검색을 위해 인메모리 저장소를 사용해 구현할 수 있으나 인 메모리 저장소는 용량이 제한적임
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>속도와 저장 용량의 균형을 맞추기 위해 PostgreSQL, 레디스 Redis같은 데이터베이스나 계층형 저장소를 사용해 캐시를 구현할 수 있음</li>
</ul>
</li>
<li>캐시 크기 관리 및 성능 유지에는 제거 정책이 중요함
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>LRU least recently used : 가장 최근에 사용된 것 제거</li>
<li>LFU least frequently used : 가장 적게 사용된 것 제거</li>
<li>FIFO first in first out : 가장 먼저 들어온 것부터 제거</li>
</ul>
</li>
<li>다시 호출될 가능성이 얼마나 높은지에 따라 캐시를 얼마나 오래 보관할지 결정됨 : 질의를 캐시에 저장해야 할지 예측하는 별도의 분류기를 학습시키기도 함</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">시맨틱 캐싱</h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>들어온 질의와 캐시된 항목이 완전히 똑같지 않더라도 의미적으로만 비슷하면 캐시된 항목을 사용함</li>
<li>비슷한 질의를 재사용하면 캐시 적중률이 높아지고 잠재적으로 비용도 줄일 수 있지만, 모델의 성능을 저하시킬 수도 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>두 질의가 유사한지 판단할 수 있는 방법이 확실히 있을 때만 제대로 작동함 $\rightarrow$ 의미적 유사도를 활용
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>질의마다 임베딩 생성, 벡터 검색으로 현재 질의 임베딩과 가장 높은 유사도 점수를 가진 캐시된 임베딩을 찾고, 특정 유사도 임곗값보다 높으면 캐시된 질의와 유사한 것으로 판단하고 캐시된 결과를 반환함, 그렇지 않으면 현재 질의 처리 후 결과와 임베딩을 캐시에 저장</li>
<li>캐시된 임베딩을 저장하기 위한 벡터 DB가 필요함</li>
</ul>
</li>
</ul>
</li>
<li>시맨틱 캐싱 단점
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>제대로 작동하려면 좋은 품질의 임베딩, 안정적인 벡터 검색, 신뢰할 수 있는 유사도 측정 모두 필요함</li>
<li>적절한 유사도 임곗값을 설정하기도 까다로움, 잘못 판단 시 응답이 틀릴 수 있음</li>
<li>벡터 검색이 포함되어 있어 시간이 오래걸리고 연산량도 많음, 벡터 검색의 속도와 비용은 캐시에 저장된 임베딩의 크기에 달려있음</li>
</ul>
</li>
<li>캐시 적중률이 높을 때 가치 있지만 복잡하므로 추가하는데 효율성, 비용, 성능 위험을 꼼꼼히 검토해야 함</li>
</ul>
<img width="1153" height="805" alt="image" src="https://github.com/user-attachments/assets/a1e404a7-5266-466e-83f7-dc2a0601cbc8" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">5. 에이전트 패턴 추가</h3>
<p data-ke-size="size16">지금까지는 각 질의가 순차적인 흐름을 따랐지만, 루프, 병렬 실행, 조건부 분기 등으로 더 복잡해질 수 있음 (6장)</p>
<img width="950" height="682" alt="image" src="https://github.com/user-attachments/assets/b700accc-fb8e-41c0-ab1c-4c5274a55880" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>출력 생성 후 작업을 완료하지 못했다 판단해 더 많은 정보를 수집하기 위해 또 다른 검색을 수행, 원래 응답과 새로 검색한 컨텍스트를 합쳐서 같은(혹은 다른)모델에 다시 넣기</li>
<li>모델의 출력은 이메일 작성, 주문하기 등 <b>쓰기 작업 호출</b>에 사용될 수도 있음
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>쓰기 작업 권한은 최대한 신중하게 줘야함</li>
</ul>
</li>
</ul>
<img width="953" height="647" alt="image" src="https://github.com/user-attachments/assets/bc6e8bd3-7cf5-4da3-8de2-85bfbd328e89" />

<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">6. 모니터링과 관찰 가능성</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>관찰 가능성 observability</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>제품을 설계할 때부터 핵심에 두어야 함, 제품이 복잡해질수록 중요해짐</li>
<li>모든 소프트웨어 엔지니어링 분야에서 널리 쓰이는 방법</li>
<li>검증된 모범 사례와 상용 및 오픈 소스 솔루션을 갖춘 산업</li>
<li>책에서는 파운데이션 모델 기반 애플리케이션에서만 나타나는 고유한 문제와 기법들에 초점을 맞춤</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">모니터링의 목표는 평가의 목표와 같음 : 위험을 줄이고 (개선과 비용 절감 등의)기회를 발견, 책임 소재 명확히</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>위험 : 애플리케이션 실패, 보안 공격, 드리프트</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">관찰 가능성 수준을 평가할 수 있는 세가지 지표</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>평균 탐지 시간 mean time to detection MTTD : 문제가 생겼을 때, 감지하는데 얼마나 걸리는가</li>
<li>평균 응답 시간 mean time to reponse MTTR : 감지한 후, 해결되는데 얼마나 걸리는가</li>
<li>변경 실패율 change failure rate CFR : 수정이나 롤백이 필요한 실패를 일으키는 변경이나 배포의 비율, 현재 CFR을 모른다면 플랫폼을 더 관찰 가능하도록 재설계해야할 때</li>
</ul>
<p data-ke-size="size16">CFR이 높다고 해서 반드시 모니터링 시스템이 나쁘다는 것은 아니지만, 잘못된 변경사항이 배포되기 전에 잡힐 수 있도록 평가 파이프라인 점검이 필요함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">평가 지표는 모니터링 지표로 이어져야 함 : 평가 단계에서 좋았다면 모니터링에서도 좋은 성능을 보여야함, 모니터링 중에 발견된 문제는 평가 파이프라인에 발견되어야 함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">지표</h4>
<p data-ke-size="size16">지표 자체는 목적이 될 수 없음</p>
<p data-ke-size="size16">어떤 실패 유형을 잡고 싶은지, 실패 중심으로 지표를 설계하는 것이 중요</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">파운데이션 모델의 개방형 출력은 다양한 문제들을 발생시킴</p>
<p data-ke-size="size16">지표 설계에는 분석적 사고, 통계 지식, 창의성이 필요함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">어떤 지표를 추적할지는 애플리케이션마다 다름</p>
<p data-ke-size="size16">여러 유형의 모델 품질 지표 (4~6장), 계산하는 다양한 방법 (3, 5장)</p>
<p style="color: #333333; text-align: start;" data-ke-size="size16"><b><br />복습</b></p>
<p style="color: #333333; text-align: start;" data-ke-size="size16">추적하기 가장 쉬운 유형 $\rightarrow$ 형식 실패 (유효하지 않은 JSON 등, 이때 쉽게 고칠 수 있는 것이 얼마나 있는지 추적해야 함)</p>
<p style="color: #333333; text-align: start;" data-ke-size="size16">개방형 생성 <span style="color: #333333; text-align: start;">$\rightarrow$<span> 사실 일관성, 간결성, 창의성, 긍정성 (AI 평가자 사용)</span></span></p>
<p style="color: #333333; text-align: start;" data-ke-size="size16"><span style="color: #333333; text-align: start;"><span>안전 <span style="color: #333333; text-align: start;">$\rightarrow$<span>&nbsp; 관련 지표 추적, 입출력에서 개인정보나 민감한 정보 탐지, 가드레일이 얼마나 자주 작동하는지, 시스템이 얼마나 자주 응답을 거부하는지, 이상한 질의 탐지</span></span></span></span></p>
<p style="color: #333333; text-align: start;" data-ke-size="size16"><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>모델 품질 <span style="color: #333333; text-align: start;">$\rightarrow$<span> 사용자의 자연어 피드백, 대화신호로 추론 (생성 중단 빈도, 턴 수, 입력당 평균 토큰 수, 출력 당 평균 토큰 수, 출력 토큰 분포), 길이 관련 지표는 지연 시간과 비용 추적에도 중요함</span></span></span></span></span></span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span></span></span></span></span></span></span></p>
<p style="color: #333333; text-align: start;" data-ke-size="size16">&nbsp;</p>
<p style="color: #333333; text-align: start;" data-ke-size="size16"><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>애플리케이션 파이프라인의 구성 요소마다의 고유한 지표</span></span></span></span></span></span></p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>RAG 애플리케이션에서는 검색 품질 평가를 위한 컨텍스트 관련성, 컨텍스트 정확도 사용</li>
<li>벡터 DB는 데이터를 색인화하는 데 필요한 저장 공간의 양과 데이터를 쿼리하는 데 걸리는 시간으로 평가</li>
</ul>
<p data-ke-size="size16">여러 지표 간 상관관계, 일일 활성 사용자 daily active user (DAU), 세션 지속 시간, 구독 수 같은 비즈니스 핵심 지표와의 상관관계를 측정하는 것이 유용함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">지연 시간 추적은 사용자 경험 이해에 필요함 (9장) <span style="color: #333333; text-align: start;">$\rightarrow$<span> TTFT, TPOT, 총 지연 시간</span></span></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><span style="color: #333333; text-align: start;"><span>비용 추적 <span style="color: #333333; text-align: start;">$\rightarrow$<span>&nbsp; 질의 수, 초당 토큰 수 (TPS) 같은 입출력 토큰의 양</span></span></span></span></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>지표 계산 시 표본 검사 or 전수 검사 <span style="color: #333333; text-align: start;">$\rightarrow$<span>&nbsp; 시스템 요구사항과&nbsp; 가용 자원에 따라 달라짐, 적절히 조합해 균형 잡힌 모니터링 전략 만들기</span></span></span></span></span></span></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>지표 계산 시 사용자, 릴리즈 버전, 프롬프트/체인 버전, 프롬프트/체인 타입, 시간같은 관련 기준으로 세분화하면 성능 변화를 이해하고 특정 문제를 찾아내는 데 도움이 됨</span></span></span></span></span></span></p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20"><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>로그와 트레이스</span></span></span></span></span></span></h4>
<p data-ke-size="size16"><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>지표는 속성와 이벤트를 나타내는 수치적 측정값</span></span></span></span></span></span></p>
<p data-ke-size="size16"><b><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>로그</span></span></span></span></span></span></b><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>는&nbsp;<b>추가만 가능한&nbsp;</b>append-only 이벤트 기록 <span style="color: #333333; text-align: start;">$\rightarrow$<span> 지표가 5분 전에 문제가 생겼다고 알려주지만 정확히 무슨 일이 일어났는지는 모름, 로그의 에러와 지표를 연관지어서 올바른 문제 식별 확인</span></span></span></span></span></span></span></span></p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">빠른 탐지를 위한 빠른 지표 계산</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">로깅의 원칙 : 모든 것을 로깅</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델 API 엔드포인트, 모델 이름, 샘플링 설정(온도, top-p, top-k, 중단 조건 등), 프롬프트 템플릿 등 모든 설정</li>
<li>사용자 질의, 모델에 전송된 최종 프롬프트, 출력, 중간 출력, 도구 호출, 도구 출력</li>
<li>구성 요소의 시작과 끝, 충돌 발생 시점
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>로그를 기록할 땐 시스템의 어느 부분에서 왔는지 알 수 있도록 태그와 ID를 부여해야함</li>
</ul>
</li>
<li>로그의 양이 빠르게 증가할 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>자동화된 로그 분석과 로그 이상탐지를 위한 AI로 구동</li>
<li>수동으로 처리하기 어렵지만 직접 살펴보는 것은 유용함</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">로그는 개별 이벤트들을 따로 기록한 것</p>
<p data-ke-size="size16"><b>트레이스</b>는 관련된 이벤트들을 하나로 연결해서 하나의 트랜잭션이나 프로세스의 완전한 타임라인을 만든 것, 처음부터 끝까지 각 단계가 어떻게 연결되는지 보여줌 <span style="color: #333333; text-align: start;">$\rightarrow$<span> 각 단계에 걸린 시간과 관련 비용까지 함께 보여줘야 함</span></span></p>
<img width="753" height="878" alt="image" src="https://github.com/user-attachments/assets/ca304b9c-22c0-41b4-a8e8-49bcc47901cd" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">트레이스 활용 시, 이상적으로 각 질의가 시스템을 거쳐가며 변화하는 과정을 단계적으로 추적할 수 있어야 함, 질의가 실패했을 때 어느 단계에서 문제가 발생했는지 짚어내기 위함</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">드리프트 감지</h4>
<p data-ke-size="size16">시스템의 구성 요소가 많을 수록 바뀔 수 있는 것들도 많아짐</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>시스템 프롬프트 변경</b></li>
<li><b>사용자 행동 변화</b></li>
<li><b>기반 모델 변경</b></li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">AI 파이프라인 오케스트레이션</h3>
<p data-ke-size="size16">AI 애플리케이션은 시간이 지날 수록 상당히 복잡해질 수 있음&nbsp;</p>
<p data-ke-size="size16">오케스트레이터 : 복잡함을 해소하기 위해 서로 다른 구성 요소들의 상호작용 방식을 정의, 엔드투엔드 파이프라인을 구성, 구성 요 간에 데이터가 원활하게 흐르도록 보장</p>
<p data-ke-size="size16">&nbsp;</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>구성 요소 정의 compontnets definition</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>시스템이 어떤 요소들을 쓰는지 오케스트레이터에게 알려주기</li>
<li>다양한 모델, 검색을 위한 외부 데이터 소스, 시스템이 사용할 수 있는 도구</li>
<li>모델 게이트웨이를 사용하면 모델을 더 쉽게 추가할 수 있음</li>
<li>평가나 모니터링 도구</li>
</ul>
</li>
<li><b>체이닝 chaining (파이프라이닝 pipelining)</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>함수를 조합하는 것</li>
<li>여러 다른 함수(구성 요소)들을 하나로 엮음</li>
<li>질의를 받는 순간부터 작업을 완료할 때까지 시스템이 수행하는 단계들을 오케스트레이터에게 알려줌</li>
</ul>
</li>
<li>구성 요소들 사이에서 데이터를 전달하는 역할임
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>현재 단계의 출력이 다음 단계에서 예상하는 형식인지 확인해주는 도구들 제공</li>
<li>구성 요소 실패나 데이터 불일치 오류로 데이터 흐름이 끊어질때 사용자에게 알려줘야 함</li>
</ul>
</li>
<li>에어플로 Airflow, 메타플로 Metaflow 같은 일반적인 워크플로 오케스트레이터와 다름</li>
</ul>
<p data-ke-size="size16">요구사항이 엄격한 애플리케이션의 파이프라인을 설계할 때는 가능한 한 많은 작업을 병렬로 처리하면 좋음 : 라우팅 구성 (질의를 어디로 보낼지) 요소와 개인정보 제거 구성 요소가 있다면 둘을 동시에 처리할 수 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">AI 오케스트레이션 도구 : 랭체인, LlamaIndex, Flowise, Langflow, Haystack 등, 검색과 도구 사용은 일반적인 애플리케이션 패턴 $\rightarrow$ RAG 및 에이전트 프레임우크도 오케스트레이션 도구 역할을 함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">처음에는 도구 없이 만들어 볼 것 : 외부 도구는 복잡성을 더함, 시스템을 이해하고 디버깅하기 어렵게 함</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">오케스트레이터 평가 시 명심해야 할 세 가지</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>통합과 확장성</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>현재 사용중이거나 앞으로 도임할 수도 있는 구성 요소들을 지원하는지 평가해야 함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>라마 모델을 쓰고 싶다면 오케스트레이터가 지원하는지 확인해야 함</li>
<li>오케스트레이터의 확장성 고려</li>
</ul>
</li>
</ul>
</li>
<li><b>복잡한 파이프라인 지원</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>분기, 병렬 처리, 오류 처리 같은 고급 기능을 지원하는 오케스트레이터는 복잡성을 효율적으로 관리하는 데 도움이 될 것, 지원 여부 확인이 필요함</li>
</ul>
</li>
<li><b>사용 편의성, 성능, 확장성</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>사용자 친화성 : 직관적인 API, 포괄적인 문서, 강력한 커뮤니티 지운</li>
<li>애플리케이션, 개발자, 트래픽 수가 증가하면서 효과적으로 확장될 수 있는지 확인</li>
</ul>
</li>
</ul>
<hr contenteditable="false" data-ke-type="horizontalRule" data-ke-style="style6" />
<h2 data-ke-size="size26">사용자 피드백</h2>
<p data-ke-size="size16">사용자 피드백 : 애플리케이션 성능 평가, 개발 방향에 정보 제공, 독점 데이터, 경쟁 우위의 원천</p>
<p data-ke-size="size16">모델 개인화부터 앞으로 나올 모델 학습에도 사용될 수 있음, 민감한 데이터를 활용할 때 주의하기, 사용자는 자신의 데이터가 어떻게 사용되는지 알 권리가 있음</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">대화형 피드백 추출</h3>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>명시적 피드백 explicit feedback</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>좋아요/싫어요, 추천/비추천, 별점 평가, 질의에 대한 예/아니오</li>
</ul>
</li>
<li><b>암시적 피드백 implicit feedback</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>행동에서 추론한 정보
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>e.g. 애플리케이션이 추천한 제품을 구매</li>
</ul>
</li>
<li>애플리케이션의 특성에 따라 달라짐</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">대화형 인터페이스는 피드백을 주기 더 쉽게 만듦 $\rightarrow$ AI가 사용자와 대화하는 것 자체가 성능과 선호도에 대한 피드백</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">사용자 피드백의 활용</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>평가 : 애플리케이션을 모니터링할 지표 도출</li>
<li>개발 : 향후 모델 학습이나 개발 방향 안내</li>
<li>개인화 : 각 사용자에게 맞게 애플리케이션을 개인화</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16"><b>암시적 대화형 피드백 implicit conversational feedback </b>: 사용자의 메세지 내용, 의사소통 패턴에서 추론 가능, 꼼꼼한 데이터 분석과 사용자 연구 필요<b></b></p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">자연어 피드백</h4>
<p data-ke-size="size16">사용자와의 대화에서 추출한 내용</p>
<p data-ke-size="size16">자연여 피드백의 예시들</p>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>조기 종료
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>생성을 중간에 멈추거나 종료, 그만하라고 말하기, 에이전트 방치 등</li>
</ul>
</li>
<li>오류 교정
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>후속 질의에 아니요 등&nbsp;</li>
<li>오류를 고치기 위해 다른 방식으로 질의를 표현하려는 시도를 휴리스틱이나 ML 모델을 사용해 탐지</li>
<li>사용자가 모델이 잘못 행동한 경우 지적할 수 있음</li>
<li>행동 교정 피드백은 에이전트를 더 좋은 행동으로 유도하는 에이전트 활용 사례에서 흔함</li>
<li>일부 애플리케이션은 사용자가 모델 응답을 직접 편집할 수 있도록 함 (사용자가 무엇을 선호하는지 알 수 있게 해주는 데이터)</li>
</ul>
</li>
<li>불평
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>출력 고정없이 그냥 불평</li>
<li>어떤 지점에서 사용자가 마음에 안드는지 이해하여 봇을 개선해야 함</li>
</ul>
</li>
</ul>
<img width="1181" height="728" alt="image" src="https://github.com/user-attachments/assets/131ba317-2ffc-4342-8905-97bf3f0f829d" />

<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>감정
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>부정적인 감정 : 좌절, 실망, 조롱 등
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>e.g. 콜센터에서 사용자 목소리 추적</li>
</ul>
</li>
<li>모델 응답에서 추론 : 응답 거부율</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">기타 대화형 피드백&nbsp;</h4>
<p data-ke-size="size16">메세지 대신 행동에서 얻을 수 있는 피드백</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>재생성
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>맘에 안들거나 비교용</li>
<li>생성같은 창의적인 요청에서 흔함</li>
<li>구독 기반 앱보다 사용량 기반 과금 앱에서 의미가 더 클 수 있음</li>
</ul>
</li>
<li>대화 관리
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>삭제, 이름 변경, 공유, 북마크 등</li>
</ul>
</li>
<li>대화 길이
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>대화당 턴 수</li>
<li>긍정인지 부정인지는 앱마다 다름
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>재미를 위한 거라면 긴 대화는 대화를 즐기고 있다는 뜻, 생산성을 목표로 하는 경우 비효율적</li>
</ul>
</li>
</ul>
</li>
<li>대화 다양성
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>고유 토큰이나 주제 개수로 측정 가능</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">명시적 피드백은 해석하기 더 쉽지만 사용자에게 추가적인 노력을 요구 : 사용자가 적은 애플리케이션에서 특히 드물게 나타날 수 있음, 응답 편향의 문제 (불만족한 사용자들이 불평할 가능성이 더 높음)</p>
<p data-ke-size="size16">암시적 피드백은 많지만 노이즈도 그만큼 많음 : 해석하기 어려울 수 있음 $\rightarrow$ 사용자 자체 연구가 중요</p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23">피드백 설계</h3>
<h4 data-ke-size="size20">피드백을 수집하는 시점</h4>
<p data-ke-size="size16">수집이 사용자의 흐름을 방해해서는 안됨</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">사용자 피드백이 특히 유용할 수 있는 상황</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>처음 시작할 때
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>사용자를 위한 초기 앱의 동작을 보정 calibrate 하는데 도움이 됨</li>
<li>음성 어시스턴트, 언어 학습 앱, 얼굴 인식 등 일부 앱에서는 보정이 꼭 필요함, 다른 앱에서는 제품 사용에 마찰을 일으킬 수 있기 때문에 선택 사항으로 하는 것이 좋음</li>
</ul>
</li>
<li>문제가 생겼을 때
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>환각, 정당한 요청 차단, 문제가 되는 이미지 생성, 응답 지연 등 문제에 대해 피드백을 남길 수 있어야 함</li>
<li>이상적으로는 제품이 실수를 해도 사용자들이 작업을 완수할 수 있어야 함
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>e.g. 잘못 분류시 재분류, 사용자가 AI와 협력, 사람과 협력</li>
</ul>
</li>
</ul>
</li>
</ul>
<img width="967" height="1014" alt="image" src="https://github.com/user-attachments/assets/a0adc743-401f-4a72-adb4-738be28ae3d8" />

<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델의 신뢰도가 낮을 때
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>모델 자체가 특정 행동에 대해 확신하지 못할 때, 피드백을 요청해서 신뢰도를 높일 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>e.g. 짧은 요약 vs 긴 요약 <span style="color: #333333; text-align: start;">선택권 주기<span> </span></span>$\rightarrow$ 선호도 파인튜닝에 사용 가능, 노이즈가 많은 투표로 이어질 수도 있음 (그래서 일부는 응답의 시작 부분만 보여줌), <span style="color: #333333; text-align: start;"><span>&nbsp;</span>사진 정리 시 동일인물인지 확인받기</span></li>
</ul>
</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">애플리케이션은 기본적으로 좋은 결과를 내놓아야 한다&nbsp; $\rightarrow$ 긍정적/부정적 피드백 둘다 요청하는 것을 권장하지 않음 (애플 입장), 궁극적으로 만족하면 계속 씀</p>
<p data-ke-size="size16">놀라운 것을 경험했을 때 피드백을 줄 수 있는 옵션이 있어야 한다는 주장 $\rightarrow$ 시간내서 칭찬할만큼 좋아하는게 뭔지 알 수 있음</p>
<p data-ke-size="size16">사용자가 귀찮아할 수도 있으니 사용자가 많다면 1%에게만 요청을 보여주자 (사용자 비율 작을 수록 편향 위험 커짐)</p>
<p data-ke-size="size16">&nbsp;</p>
<h4 data-ke-size="size20">피드백 수집 방법</h4>
<p data-ke-size="size16">사용자의 워크플로에 자연스럽게 녹아들어야 함, 쉽게 무시할 수도 있어야 함, 좋은 피드백을 제공하도록 유도하는 인센티브 필요</p>
<img width="1185" height="552" alt="image" src="https://github.com/user-attachments/assets/92ae2f66-f893-4afb-aa07-82db7cb02d82" />
<img width="1198" height="829" alt="image" src="https://github.com/user-attachments/assets/ed29ebe1-20b1-47f8-a1c7-4a9b5b426bb2" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">챗PGT나 클로드같은 독립형 애플리케이션은 사용자의 일상 워크플로에 통합되어 있지 않아 기존 도구에 내장된 제품만큼 고품질 피드백을 수집하기 어려움, 피드백만으로도 제품 분석에 도움이 될 수 있음 $\rightarrow$ But 깊이있는 분석을 위해서는 이전 5~10개의 대화같은 주변 컨텍스트가 필요</p>
<p data-ke-size="size16">$\rightarrow$ 이런 컨텍스트에는 개인 식별 정보가 포함될 수도 있어 명시적으로 동의하지 않으면 컨텍스트를 얻는 것이 불가능함 $\rightarrow$ 서비스 약관에 분석과 제품 개선을 위해 사용자 데이터에 접근할 수 있다는 조항을 넣음</p>
<p data-ke-size="size16">조항이 없는 경우엔 사용자들에게 최근 상호작용 데이터를 기부(공유)해달라고 요청</p>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">사용자에게 피드백이 어떻게 사용되는지 설명하면 더 많고 더 좋은 피드백 주도록 동기부여 가능</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>제품 개인화, 일반적인 사용량 통계 수집, 새로운 모델 학습</li>
<li>개인정보 보호를 우려한다면 모델 학습에 사용되지 않거나 기기 외부로 나가지 않음을 알려야 함</li>
<li>불가능한 것 요구하지 않기 (모르겠다는 선택지)</li>
<li>사용자의 이해를 돕기 위해 선택지에 아이콘이나 툴팁 추가, 헷갈리게 할 수 있는 디자인은 피하기</li>
</ul>
<img width="1171" height="679" alt="image" src="https://github.com/user-attachments/assets/5ccfb9ab-c6e0-4458-8664-1fccf9f76b00" />

<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size16">사용자 피드백을 공개로 할지 비공개로 할지도 결정해야 함 $\rightarrow$ 사용자 행동, 사용자 경험, 피드백 품질에 큰 영향을 줄 수 있음, 비공개 환경에서 솔직해지고 더 좋은 품질의 신호를 얻을 수 있음 But, 비공개 신호는 발견 가능성 discoverability, 설명 가능성 explainability를 감소시킬 수 있음 (e.g. 좋아요를 숨기고 팔로우하는 사람들의 좋아요 기반으로 트윗 추천&nbsp; <span style="color: #333333; text-align: start;">$\rightarrow$ 왜 특정 트윗이 뜨는지 이해 불가)</span></p>
<p data-ke-size="size16">&nbsp;</p>
<h3 data-ke-size="size23"><span style="color: #333333; text-align: start;">피드백의 한계</span></h3>
<h4 data-ke-size="size20"><span style="color: #333333; text-align: start;">편향</span></h4>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li><b>관대함 편향 leniency bias</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>실제보다 긍정적으로 평가</li>
<li>부정적으로 적으면 이유를 적어야 하니까 : 사용자 피드백에 추가적인 작업을 하도록 하면 안되는 이유
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>보통 5점을 줘야한다는 압박을 느끼고 문제가 있으면 4점을 줌 : 우버 운전기사 평균이 4.8, 4.6 미만이면 퇴출될 위기에 처함, 사용자 평점의 분포를 살펴보는 것이 중요함</li>
<li>더 세밀한 피드백을 원하면 숫자 대신 문장 선택지를 보여줘서 낮은 점수가 주는 부정적인 느낌 줄이기</li>
</ul>
</li>
</ul>
</li>
<li><b>무작위성</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>신중하게 생각할 동기가 부족해서 아무렇게나 피드백을 줆</li>
</ul>
</li>
<li><b>위치 편향</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>일반적으로 첫번째 선택</li>
<li>위치에 따른 제안의 실제 성공률 계산하는 모델 만들어서 줄이기</li>
</ul>
</li>
<li><b>선호도 편향</b>
<ul style="list-style-type: disc;" data-ke-list-type="disc">
<li>영향을 주는 다른 여러 편향</li>
<li>최근성 편향 recency bias : 두 개 비교 시 마지막 응답 선호</li>
</ul>
</li>
</ul>
<p data-ke-size="size16">&nbsp;</p>
<p data-ke-size="size18">퇴화 피드백 루프</p>
<p data-ke-size="size18">사용자 피드백은 기본적으로 불완전함 : 보여준 것에 대한 피드백만 얻을 수 있음</p>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li><b>퇴화 피드백 루프 degenerate feedback loop&nbsp;</b>
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>사용자 피드백이 모델의 동작을 정하는 시스템의 경우 생길 수 있음</li>
<li>예측 자체가 피드백에 영향을 주고 피드백이 다시 모델의 다음 버전에 영향을 주면서 초기 편향이 점점 심해지는 현상
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>e.g. A랑 B랑 비슷한데 초기에 순위가 높았던게 계속 인기를 얻고 순위를 유지하는 현상</li>
<li>노출 편향, 인기 편향, 필터 버블</li>
</ul>
</li>
<li>제품의 초점과 사용자층을 바꿀 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>인종차별, 성차별, 선정적 콘텐츠 선호 등 편향을 키울 수 있음</li>
</ul>
</li>
<li>대화형 에이전트를 사용자 피드백에 따라 동작을 수정하면 거짓말쟁이로 만들 수 있음
<ul style="list-style-type: circle;" data-ke-list-type="circle">
<li>정확하거나 유익한 것보다는 사용자가 듣고 싶어하는 응답을 하도록 (아첨)</li>
</ul>
</li>
</ul>
</li>
</ul>
