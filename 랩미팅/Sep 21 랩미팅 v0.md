 일반 synthetic network traffic generation 분야.

## Lightweight GenAI for Network Traffic Generation
: Fidelity, Augmentation, and Classification
저자: Giampaolo Bovenzi (Italy, Pegaso Telematic University, citations 1197)
arXiv:2603.25507v2 [cs.NI] (25 Aug 2026)

![[Screenshot 2026-09-22 at 3.55.49 PM.png]]
#### 주제
"we investigate lightweight Generative AI architectures for practical NTG"
	생성형 Ai 활용한 NTG(Network Traffic Generation)생성

"we synthesize compact flow-level traffic representations derived from early packet-header information, enabling transformer-based, state-space, and diffusion models with only a few million parameters."
	초기 패킷 헤더정보에서 나온 간단한 flow 정도의 traffic 표현을 합성
	1-2백만개의 파라미터로 다양한 ML 기법 구현가능

- 각 네트워크 flow의 첫번째 packet header field에서 파생된 컴팩트한 traffic 표현 생성 (Packet Length + DIRection)
  ~> 1-2백만개의 파라미터 내에서 transformer, SSM, DM 같은 GenAI Model의 구성을 학습가능
- 생성된 representation (Packet Length, DIRection), 다시 traffic matrix 형태로 복원되어 classification에 활용됨

#### 실험 수치
-* 공개 데이터셋: Mirage-2019(40개 모바일 앱), CESNET-TLS22-80 (80개 네트워크 서비스)
평가: 
- 합성데이터로만 학습된 분류기: 실제 트래픽에서 최대 87%의 F1-score 를 달성
	실제 데이터 학습 대비 4% 격차

기존과 비교: 
- 거대 네트워크 파운데이션 모델 -> 경량 GenAI, 1-2백만개 파라미터로 모델 학습
- raw packet byte 생성에 초점 -> Application network 지문을 구성하는 경량 트래픽 특징 합성
	- Packet = PL(Payload length) + DIR(direction - server <-> client 방향) 로 compact representation

#### Flow pipeline
Modular NTG pipeline (powered nu lightweight GenAI models)
![[Screenshot 2026-09-22 at 3.38.40 PM.png]]
1. training phase: 실제 network trace를 전처리해 GenAI Model 학습

2. generation phase: 위 내용 활용, 어떤 application의 traffic을 만들지 조건 설정(`<CLASS>`), data synthesize (합성)
- 네트워크 클래스를 지정하는 `<CLASS>` 토큰 프롬프트를 사용해 조건화됨. 
- 각자 고유 형식으로 합성 샘플 생성. 
	- DM(Diffusion Models) 은 2D image, Transformer 및 SSM은 token sequence 
- 합성 샘플을 하위 NTC 작업에 활용하기 위해 역매핑 단계 거침.

![[Screenshot 2026-09-22 at 3.39.05 PM 1.png]]

#### 평가항목
i) synthetic traffic fidelity (합성 트래픽 충실도) : packet sequence 구조 - 실제 traffic
ii) synthetic-only training for privacy-preserving NTC (개인정보 보호를 위한 NTC 합성 데이터 전용학습)
	-* Network Traffic Classification
- CESNET-TLS22-80 데이터셋에서 LLamA 합성 데이터로 학습한 RF가 real test data로 테스트한 결과 최대 성능 87.9 달성. 
![[Pasted image 20260922161145.png]]

**iii) data augmentation under low-data regimes (데이터 부족 환경에서 데이터 증강법)**
- 전체 label이 지정된 dataset 으로 학습되어서 합성 샘플 생성에 사용됨
- RF 통해 소수의 실제 샘플과 합성 데이터, 함께 사용하여 학습.
	~> 네트워크 운영자가 제한된 label data를 보안하기 위해 사전 학습된 GenAI model을 활용하는 시나리오 반영.

iv) computational efficiency (계산효율성)

#### 논문_Synthetic Network Traffic Generation
다음 주 동안 읽기.

---




#### 목표
고객의 웹서비스를 테스트해보기 위해 임의로 트래픽을 생성한다. 
트래픽을 생성하는 측면에서 ZAP이 중요하다. 
트래픽을 생성할 때 LLM, AI 기술을 활용해서 생성한다.
고객의 웹서비스, DB등의 트래픽을 미리 만들어서 취약점을 시험해본다.

트래픽 생성 시, ZAP은 패턴(parameter 파악- active rule 적용, 변형 페이로드 생성- 서버에 요청보내 반응파악)을 통한 traffic generation 과, crawling을 통한 tree site 생성으로 진행된다. 
우리는 AI 기술을 접목해서 traffic generation을 진행한다. 
#### Idea
AI를 통해 traffic 을 생성하고- AI를 통해 생성한 ruleset 에 넣어서- 새 ruleset을 생성함
-> 계속해서 ruleset에 input 으로 넣음. feedback을 주어서 loop를 돌림

위험할 거라고 예상했었던 traffic 을 Ai가 생성한 ruleset에 넣어봐서 ruleset이 잘 거르는지 여부 검증가능
(위험을 대비하기 위해 생성한 ruleset이 실제로 차단이 되는지 여부를 test 가능함)

**ZAP의 트래픽 생성 기능을 기존에 정규식 등으로 하드코딩해둔 것에서, LLM이나 AI를 통해 생성하는 기능으로 업그레이드 해보자**

--progesi 논문 참고
토큰 단위로 쪼개서 다양한 변형과 통일된 규칙을 생성한다
의미 단위로 각 SQLi item을 나누고, 이를 막을 수 있는 rule을 만든다 
LLM이 놓칠 수 있는 의미 단위를 파악할 수 있다. 

--WebSpotter
토큰 단위
#### 방향성

ZAP의 트래픽 생성 기능에 AI를 접목할 수 있는 방안 찾기
Web payload generation 예시들을 다양하게 찾아보기 (LLM, AI, argument, ...)
저번 랩미팅때 발표한 논문들 읽어보기

