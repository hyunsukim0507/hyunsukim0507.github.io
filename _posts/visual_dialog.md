---
layout : single
title : "Visual Dialog"
---

# Abstract 

We introduce the task of Visual Dialog, which requires an AI agent to hold a meaningful dialog with humans in natural, conversational language about visual content. Specifically, given an image, a dialog history, and a question about the image, the agent has to ground the question in image, infer context from history, and answer the question accurately. Visual Dialog is disentangled enough from a specific downstream task so as to serve as a general test of machine intelligence, while being grounded in vision enough to allow objective evaluation of individual responses and benchmark progress. We develop a novel two-person chat data-collection protocol to curate a large-scale Visual Dialog dataset (VisDial). VisDial v0.9 has been released and contains 1 dialog with 10 question-answer pairs on ∼120k images from COCO, with a total of ∼1.2M dialog questionanswer pairs.

Visual Dialog의 작업을 소개합니다. 이것은 AI 에이전트가 시각적 콘텐츠에 대해 자연스럽고 대화적인 언어로 인간과 의미 있는 대화를 해야 합니다. 구체적으로, 이미지, 대화기록 및 이미지에 대한 질문이 주어지면 에이전트는 이미지에 질문의 근거를 두고 기록에서 맥락을 추론하고 질문에 정확하게 대답해야 합니다. Visual Dialog는 기계 지능의 일반적인 테스트 역할을 하기 위해 특정 다운스트림 작업과 충분히 얽혀있으면서 개별 응답 및 벤치마크 진행 상황을 객관적평가할 수 있을 만큼 충분히 비전에 기반을 둡니다. 우리는 대규모 Visual Dialog 데이터 세트를 선별하기 위해 새로운 2인 채팅 데이터 수집 프로토콜을 개발합니다. visDial v0.9가 출시되었으며 COCO의 ~120k 이미지에 대한 10개의 질문-답변 쌍이 있는 1개의 대화 상자와 총 ~120만 개의 대화 질문 대답 쌍이 포함되어 있습니다.

We introduce a family of neural encoder-decoder models for Visual Dialog with 3 encoders – Late Fusion, Hierarchical Recurrent Encoder and Memory Network– and 2 decoders (generative and discriminative), which outperform a number of sophisticated baselines. We propose a retrieval-based evaluation protocol for Visual Dialog where the AI agent is asked to sort a set of candidate answers and evaluated on metrics such as mean-reciprocal-rank of human response. We quantify gap between machine and human performance on the Visual Dialog task via human studies. Putting it all together, we demonstrate the first ‘visual chatbot’! Our dataset, code, trained models and visual chatbot are available on https://visualdialog.org.

3개의 인코더가 있는 Visual Dialog용 신경 인코더-디코더 모델 제품군을 소개합니다. Late Fusion, 계층적 순환 인코더 및 메모리 네트워크, 그리고 2개의 디코더(생성 및 판별)는 정교한 기준선을 능가합니다.우리는 검색을 제안합니다. 우리는 AI 에이전트가 일련의 후보 답변을 정렬하고 인간 응답의 평균 역수와 같은 메트릭에 대해 평가하는 프로토콜을 제안합니다. 
우리는 인간 연구를 통해 Visual Dialog 작업에서 기계와 인간의 성능 사이의 격차를 정량화합니다. 이를 종합하여 최초의 '비주얼 챗봇'을 시연합니다! 데이터세트, 코드, 훈련된 모델 및 시각적 챗봇은 https://visualdialog.org에서 사용할 수 있습니다.

# Introduction
Weare witnessing unprecedented advances in computer vision (CV) and artificial intelligence (AI)– from ‘low-level’ AI tasks such as image classification [20], scene recognition [63], object detection [34]– to ‘high-level’ AI tasks such as learning to play Atari video games [42] and Go [55], answering reading comprehension questions by understanding short stories [21, 65], and even answering questions about images [6,39,49,71] and videos [57,58]!

What lies next for AI? We believe that the next generation of visual intelligence systems will need to posses the ability to hold a meaningful dialog with humans in natural language about visual content. 

우리는 차세대 시각 지능 시스템이 시각 콘텐츠에 대해 자연 언어로 인간과 의미 있는 대화를 나눌 수 있는 능력이 필요하다고 믿습니다.

Applications include: 
• Aiding visually impaired users in understanding their surroundings [7] or social media content [66] (AI: ‘John just uploaded a picture from his vacation in Hawaii’, Human: ‘Great, is he at the beach?’, AI: ‘No, on a mountain’). 
• Aiding analysts in making decisions based on large quantities of surveillance data (Human: ‘Did anyone enter this room last week?’, AI: ‘Yes, 27 instances logged on camera’, Human: ‘Were any of them carrying a black bag?’),
• Interacting with an AI assistant (Human: ‘Alexa– can you see the baby in the baby monitor?’, AI: ‘Yes, I can’, Human: ‘Is he sleeping or playing?’). 
• Robotics applications (e.g. search and rescue missions) where the operator may be ‘situationally blind’ and operating via language [40] (Human: ‘Is there smoke in any room around you?’, AI: ‘Yes, in one room’, Human: ‘Go there and look for people’).

Despite rapid progress at the intersection of vision and language– in particular, in image captioning and visual question answering (VQA)– it is clear that we are far from this grand goal of an AI agent that can ‘see’ and ‘communicate’. In captioning, the human-machine interaction consists of the machine simply talking at the human (‘Two people are in a wheelchair and one is holding a racket’), with no dialog or input from the human. While VQAtakesasignificant step towards human-machine interaction, it still represents only a single round of a dialog– unlike in human conversations, there is no scope for follow-up questions, no memory in the system of previous questions asked by the user nor consistency with respect to previous answers provided by the system (Q: ‘How many people on wheelchairs?’, A: ‘Two’; Q: ‘How many wheelchairs?’, A: ‘One’). 

특히 이미지 캡션과 VQA(시각적 질문 답변) 분야에서 시각과 언어의 교차점에서 급속한 발전에도 불구하고 '보고', '소통'할 수 있는 AI 에이전트의 이 원대한 목표와는 거리가 멀다는 것이 분명합니다. 캡션에서 인간-기계 상호 작용은 인간의 대화나 입력 없이 단순히 인간에게 말하는 기계로 구성됩니다('두 사람은 휠체어에 있고 한 사람은 라켓을 잡고 있습니다').
VQA는 인간-기계 상호 작용을 향한 중요한 단계를 취하지만 여전히 대화의 한 라운드만 나타냅니다. 인간 대화와 달리 후속 질문의 범위가 없으며 시스템에 사용자가 묻는 이전 질문에 대한 메모리나 일관성이 없습니다. 시스템에서 제공한 이전 답변과 관련하여(Q: '휠체어를 탄 사람은 몇 명입니까?', A: '둘', Q: '휠체어가 몇 개입니까?', A: '하나')

As a step towards conversational visual AI, we introduce a novel task– Visual Dialog– along with a large-scale dataset, an evaluation protocol, and novel deep models.

대화형 시각적 AI를 향한 단계로 대규모 데이터 세트, 평가 프로토콜 및 새로운 심층 모델과 함께 새로운 작업인 Visual Dialog를 소개합니다.

**Task Definition.** The concrete task in Visual Dialog is the following– given an image I, a history of a dialog consisting of a sequence of question-answer pairs (Q1: ‘How many people are in wheelchairs?’, A1: ‘Two’, Q2: ‘What are their genders?’, A2: ‘One male and one female’), and a natural language follow-up question (Q3: ‘Which one is holding a racket?’), the task for the machine is to answer the question in free-form natural language (A3: ‘The woman’). This task is the visual analogue of the Turing Test.

Visual Dialog의 구체적인 작업은 다음과 같습니다. 이미지 I, 일련의 질문-답변 쌍으로 구성된 대화 이력(Q1: '얼마나 많은 사람들이 휠체어에 타고 있습니까?', A1: '2', Q2: '그들의 성별은 무엇입니까?', A2: '남성 1명, 여성 1명'), 자연어 후속 질문(Q3: '라켓을 들고 있는 사람은 누구입니까?'), 기계의 임무는 대답하는 것입니다. 자유 형식의 자연어로 된 질문(A3: '여자'). 이 작업은 튜링 테스트의 시각적 유사체입니다.

Consider the Visual Dialog examples in Fig. 2. The question ‘What is the gender of the one in the white shirt?’ requires the machine to selectively focus and direct attention to a relevant region. ‘What is she doing?’ requires co-reference resolution (whom does the pronoun ‘she’ refer to?), ‘Is that a man to her right?’ further requires the machine to have visual memory (which object in the image were we talking about?). Such systems also need to be consistent with their outputs– ‘How many people are in wheelchairs?’, ‘Two’, ‘What are their genders?’, ‘One male and one female’– note that the number of genders being specified should add up to two. Such difficulties make the problem a highly interesting and challenging one.

그림 2의 Visual Dialog 예를 고려하십시오. '흰 셔츠를 입은 사람의 성별은 무엇입니까?'라는 질문은 기계가 관련 영역에 선택적으로 초점을 맞추고 주의를 집중하도록 요구합니다. '그녀는 무엇을 하고 있나요?'는 공동 참조 해상도(대명사 '그녀'는 누구를 가리킵니까?)가 필요하고, '그 여자는 오른쪽에 있는 남자인가요?'는 더 나아가 기계에 시각적 기억(이미지에서 어떤 물체가 있었는지)을 요구합니다. 우리 얘기?). 이러한 시스템은 또한 '휠체어를 탄 사람이 몇 명입니까?', '2인', '성별은 무엇입니까?', '남성 1명과 여성 1명'과 같은 출력과 일치해야 합니다. 지정되는 성별의 수는 다음과 같아야 합니다. 2개까지 추가합니다. 그러한 어려움은 문제를 매우 흥미롭고 도전적인 문제로 만듭니다.

**Why do we talk to machines?** Prior work in language-only (non-visual) dialog can be arranged on a spectrum with the following two end-points:
언어 전용(비시각적) 대화에서 이전 작업은 다음 두 끝점이 있는 스펙트럼에서 정렬할 수 있습니다.

goal-driven dialog (e.g. booking a flight for a user) ←→ goal-free dialog (or casual ‘chit-chat’ with chatbots). The two ends have vastly differing purposes and conflicting evaluation criteria. Goal-driven dialog is typically evaluated on task-completion rate (how frequently was the user able to book their flight) or time to task completion [14,44]– clearly, the shorter the dialog the better. In contrast, for chit-chat, the longer the user engagement and interaction, the better. For instance, the goal of the 2017 $2.5 Million Amazon Alexa Prize is to “create a socialbot that converses coherently and engagingly with humans on popular topics for 20 minutes.”

목표 기반 대화(예: 사용자를 위한 항공편 예약) ←→ 목표 없는 대화(또는 챗봇을 사용한 캐주얼한 '잡담'). 두 가지 목적은 크게 다른 목적과 상충되는 평가 기준을 가지고 있습니다. 목표 기반 대화는 일반적으로 작업 완료율(사용자가 항공편을 예약할 수 있었던 빈도) 또는 작업 완료 시간[14,44]으로 평가됩니다. 분명히 대화는 짧을수록 좋습니다. 반면, 잡담의 경우 사용자 참여 및 상호 작용이 길수록 좋습니다. 예를 들어, 2017년 250만 달러 Amazon Alexa Prize의 목표는 "인기 있는 주제에 대해 20분 동안 인간과 일관되고 매력적으로 대화하는 소셜봇을 만드는 것"입니다.

We believe our instantiation of Visual Dialog hits a sweet spot on this spectrum. It is disentangled enough from a specific downstream task so as to serve as a general test of machine intelligence, while being grounded enough in vision to allow objective evaluation of individual responses and benchmark progress. The former discourages task engineered bots for ‘slot filling’ [30] and the latter discourages bots that put on a personality to avoid answering questions while keeping the user engaged [64].

~~우리는 Visual Dialog의 인스턴스화가 이 스펙트럼에서 최적의 지점에 도달했다고 믿습니다. 특정 다운스트림 작업에서 충분히 분리되어 기계 지능의 일반적인 테스트 역할을 하는 동시에 개별 응답 및 벤치마크 진행 상황을 객관적으로 평가할 수 있도록 비전에 기반을 두고 있습니다. 전자는 '슬롯 채우기'[30]를 위해 taskengineered 봇을 권장하지 않으며, 후자는 사용자의 참여를 유지하면서 질문에 대답하지 않기 위해 개성을 부여하는 봇을 권장하지 않습니다[64].~~

**Contributions.** We make the following contributions: 
• We propose a new AI task: Visual Dialog, where a machine must hold dialog with a human about visual content. • We develop a novel two-person chat data-collection protocol to curate a large-scale Visual Dialog dataset (VisDial). Upon completion1, VisDial will contain 1 dialog each (with 10 question-answer pairs) on ∼140k images from the COCO dataset [32], for a total of ∼1.4M dialog question-answer pairs. When compared to VQA [6], VisDial studies a significantly richer task (dialog), overcomes a ‘visual priming bias’ in VQA (in VisDial, the questioner does not see the image), contains free-form longer answers, and is an order of magnitude larger.
• Weintroduce a family of neural encoder-decoder models for Visual Dialog with 3 novel encoders– Late Fusion: that embeds the image, history, and question into vector spaces separately and performs a ‘late fusion’ of these into a joint embedding.– Hierarchical Recurrent Encoder: that contains a dialoglevel Recurrent Neural Network (RNN) sitting on top of a question-answer (QA)-level recurrent block. In each QA-level recurrent block, we also include an attentionover-history mechanism to choose and attend to the round of the history relevant to the current question.– MemoryNetwork: that treats each previous QA pair as a ‘fact’ in its memory bank and learns to ‘poll’ the stored facts and the image to develop a context vector. We train all these encoders with 2 decoders (generative and discriminative)– all settings outperform a number of sophisticated baselines, including our adaption of state-ofthe-art VQA models to VisDial. 
• We propose a retrieval-based evaluation protocol for Visual Dialog where the AI agent is asked to sort a list of candidate answers and evaluated on metrics such as meanreciprocal-rank of the human response.
• Weconduct studies to quantify human performance.
• Putting it all together, on the project page we demonstrate the first visual chatbot!
