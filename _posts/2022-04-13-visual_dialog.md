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

# 1. Introduction
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
• We propose a new AI task: Visual Dialog, where a machine must hold dialog with a human about visual content. 

우리는 새로운 AI 작업인 Visual Dialog를 제안합니다. 여기서 기계는 시각적 콘텐츠에 대해 사람과 대화를 해야 합니다.

• We develop a novel two-person chat data-collection protocol to curate a large-scale Visual Dialog dataset (VisDial). Upon completion1, VisDial will contain 1 dialog each (with 10 question-answer pairs) on ∼140k images from the COCO dataset [32], for a total of ∼1.4M dialog question-answer pairs. When compared to VQA [6], VisDial studies a significantly richer task (dialog), overcomes a ‘visual priming bias’ in VQA (in VisDial, the questioner does not see the image), contains free-form longer answers, and is an order of magnitude larger.

우리는 대규모 Visual Dialog 데이터 세트(VisDial)를 선별하기 위해 새로운 2인 채팅 데이터 수집 프로토콜을 개발합니다. 완료되면, VisDial은 COCO 데이터 세트[32]의 ~140k 이미지에 대해 각각 1개의 대화(10개의 질문-답변 쌍 포함)를 포함하여 총 ~140만 개의 대화 질문-답변 쌍을 포함합니다. VQA[6]와 비교할 때 VisDial은 훨씬 더 풍부한 작업(대화 상자)을 연구하고 VQA의 '시각적 프라이밍 편향'을 극복하고(VisDial에서는 질문자가 이미지를 볼 수 없음) 자유 형식의 긴 답변을 포함하고 규모가 더 큽니다.

• We introduce a family of neural encoder-decoder models for Visual Dialog with 3 novel encoders– Late Fusion: that embeds the image, history, and question into vector spaces separately and performs a ‘late fusion’ of these into a joint embedding.– Hierarchical Recurrent Encoder: that contains a dialoglevel Recurrent Neural Network (RNN) sitting on top of a question-answer (QA)-level recurrent block. In each QA-level recurrent block, we also include an attentionover-history mechanism to choose and attend to the round of the history relevant to the current question.– MemoryNetwork: that treats each previous QA pair as a ‘fact’ in its memory bank and learns to ‘poll’ the stored facts and the image to develop a context vector. We train all these encoders with 2 decoders (generative and discriminative)– all settings outperform a number of sophisticated baselines, including our adaption of state-ofthe-art VQA models to VisDial. 

3개의 새로운 인코더가 있는 Visual Dialog용 신경 인코더-디코더 모델 제품군을 소개합니다. 후기 융합: 이미지, 기록 및 질문을 벡터 공간에 별도로 포함하고 이들의 '후기 융합'을 공동 포함으로 수행합니다. – 계층적 반복 인코더: 질문-답변(QA) 수준의 순환 블록 위에 있는 대화 수준의 순환 신경망(RNN)을 포함합니다. 각 QA 수준 반복 블록에는 현재 질문과 관련된 히스토리 라운드를 선택하고 관심을 기울이기 위한 어텐션 오버 히스토리 메커니즘도 포함됩니다. – MemoryNetwork: 메모리 뱅크에서 각 이전 QA 쌍을 '팩트'로 취급합니다. 저장된 사실과 이미지를 '폴링'하여 컨텍스트 벡터를 개발하는 방법을 배웁니다. 우리는 2개의 디코더(생성 및 판별)를 사용하여 이러한 모든 인코더를 훈련합니다. 모든 설정은 VisDial에 대한 최첨단 VQA 모델의 적응을 포함하여 여러 정교한 기준선을 능가합니다.

• We propose a retrieval-based evaluation protocol for Visual Dialog where the AI agent is asked to sort a list of candidate answers and evaluated on metrics such as meanreciprocal-rank of the human response.

우리는 AI 에이전트가 후보 답변 목록을 정렬하도록 요청하고 인간 응답의 meanreciprocal-rank와 같은 메트릭에 대해 평가하는 Visual Dialog에 대한 검색 기반 평가 프로토콜을 제안합니다.

• Weconduct studies to quantify human performance.

우리는 인간의 성과를 정량화하기 위해 연구를 수행합니다.

• Putting it all together, on the project page we demonstrate the first visual chatbot!

이 모든 것을 종합하면 프로젝트 페이지에서 첫 번째 시각적 챗봇을 시연합니다!

#2. Related Work

**Vision and Language.** A number of problems at the intersection of vision and language have recently gained prominence – image captioning [15, 16, 27, 62], video/movie
description [51, 59, 60], text-to-image coreference/grounding [10, 22, 29, 45, 47, 50], visual storytelling [4, 23], and
of course, visual question answering (VQA) [3, 6, 12, 17,
19, 37–39, 49, 69]. However, all of these involve (at most) a
single-shot natural language interaction – there is no dialog.
Concurrent with our work, two recent works [13, 43] have
also begun studying visually-grounded dialog.

비전과 언어. 시각과 언어의 교차점에서 이미지 캡션[15, 16, 27, 62], 비디오/영화와 같은 여러 문제가 최근에 두드러졌습니다.
설명 [51, 59, 60], 텍스트-이미지 상호 참조/접지 [10, 22, 29, 45, 47, 50], 시각적 스토리텔링 [4, 23] 물론, 시각적 질문 답변(VQA) [3, 6, 12, 17, 19, 37–39, 49, 69]. 그러나 이들 모두는 (최대) 단일 샷 자연어 상호 작용 – 대화가 없습니다. 우리의 작업과 동시에 두 개의 최근 작업 [13, 43]이 시각적 기반 대화를 연구하기 시작했습니다.

**Visual Turing Test.** Closely related to our work is that of
Geman et al. [18], who proposed a fairly restrictive ‘Visual
Turing Test’ – a system that asks templated, binary questions. In comparison, 1) our dataset has free-form, openended natural language questions collected via two subjects
chatting on Amazon Mechanical Turk (AMT), resulting in
a more realistic and diverse dataset (see Fig. 5). 2) The
dataset in [18] only contains street scenes, while our dataset
has considerably more variety since it uses images from
COCO [32]. Moreover, our dataset is two orders of magnitude larger – 2,591 images in [18] vs ∼140k images, 10
question-answer pairs per image, total of ∼1.4M QA pairs

Gemanet al. 상당히 제한적인 '시각적
Turing Test' – 템플릿화된 이진 질문을 하는 시스템입니다. 이에 비해 1) 데이터 세트에는 두 가지 주제를 통해 수집된 자유 형식의 개방형 자연어 질문이 있습니다.
Amazon Mechanical Turk(AMT)에서 채팅하여
보다 현실적이고 다양한 데이터 세트(그림 5 참조). 2)
[18]의 데이터 세트에는 거리 장면만 포함되지만 데이터 세트는
의 이미지를 사용하기 때문에 훨씬 더 다양합니다.
코코 [32]. 또한, 우리의 데이터 세트는 [18]의 2,591개 이미지와 ~140k개의 이미지, 10개의 크기로 2배 더 큽니다.
이미지당 질문-답변 쌍, 총 ~140만 QA 쌍

**Text-based Question Answering.** Our work is related
to text-based question answering or ‘reading comprehension’ tasks studied in the NLP community. Some recent
large-scale datasets in this domain include the 30M Factoid Question-Answer corpus [52], 100K SimpleQuestions
dataset [8], DeepMind Q&A dataset [21], the 20 artificial
tasks in the bAbI dataset [65], and the SQuAD dataset for
reading comprehension [46]. VisDial can be viewed as a
fusion of reading comprehension and VQA. In VisDial, the
machine must comprehend the history of the past dialog and
then understand the image to answer the question. By design, the answer to any question in VisDial is not present in
the past dialog – if it were, the question would not be asked.
The history of the dialog contextualizes the question – the
question ‘what else is she holding?’ requires a machine to
comprehend the history to realize who the question is talking about and what has been excluded, and then understand
the image to answer the question.

우리의 작업은 관련이 있습니다
NLP 커뮤니티에서 공부하는 텍스트 기반 질문 답변 또는 '독해' 작업까지. 최근 일부
이 도메인의 대규모 데이터 세트에는 30M Factoid Question-Answer corpus[52], 100K SimpleQuestions가 포함됩니다.
데이터 세트[8], DeepMind Q&A 데이터 세트[21], 20개의 인공
bAbI 데이터 세트 [65]의 작업 및 SQuAD 데이터 세트
독해력 [46]. VisDial은 다음과 같이 볼 수 있습니다.
독해력과 VQA의 융합. VisDial에서는
기계는 과거 대화의 역사를 이해해야 하고
그런 다음 이미지를 이해하여 질문에 답하세요. 설계상 VisDial의 모든 질문에 대한 답변은
과거 대화 – 그렇다면 질문이 묻지 않습니다.
대화의 역사는 질문을 맥락화합니다.
'그녀는 또 무엇을 들고 있습니까?'라는 질문에 기계가 필요합니다.
역사를 이해하여 질문이 말하는 대상과 제외된 대상을 파악한 다음 이해
질문에 답하는 이미지입니다.

**Conversational Modeling and Chatbots.** Visual Dialog is the visual analogue of text-based dialog and conversation modeling. While some of the earliest developed chatbots were rule-based [64], end-to-end learning based approaches are now being actively explored [9,14,26,31,53,54,61]. A recent large-scale conversation dataset is the Ubuntu Dialogue Corpus [35], which contains about 500K dialogs extracted from the Ubuntu channel on Internet Relay Chat (IRC). Liu et al. [33] perform a study of problems in existing evaluation protocols for free-form dialog. One important difference between free-form textual dialog and VisDial is that in VisDial, the two participants are not symmetric– one person (the ‘questioner’) asks questions about an image that they do not see; the other person (the ‘answerer’) sees the image and only answers the questions (in otherwise unconstrained text, but no counter-questions allowed). This role assignment gives a sense of purpose to the interaction (why are we talking? To help the questioner build a mental model of the image), and allows objective evaluation of individual responses.

Visual Dialog는 텍스트 기반 대화 및 대화 모델링의 시각적 유사체입니다. 초기에 개발된 챗봇 중 일부는 규칙 기반이었지만[64] 현재 종단 간 학습 기반 접근 방식이 활발히 연구되고 있습니다[9,14,26,31,53,54,61]. 최근 대규모 대화 데이터 세트는 Ubuntu Dialogue Corpus[35]로, IRC(Internet Relay Chat)의 Ubuntu 채널에서 추출된 약 500K 대화를 포함합니다. Liu et al. [33] 자유 형식 대화에 대한 기존 평가 프로토콜의 문제에 대한 연구를 수행합니다. 자유 형식 텍스트 대화와 VisDial의 한 가지 중요한 차이점은 VisDial에서 두 참가자가 대칭이 아니라는 것입니다. 한 사람('질문자')이 보이지 않는 이미지에 대해 질문합니다. 다른 사람('답변자')은 이미지를 보고 질문에만 답합니다(다른 방식으로 제한되지 않은 텍스트로, 그러나 반대 질문은 허용되지 않음). 이 역할 할당은 상호작용에 목적 의식을 제공하고(왜 우리가 이야기하고 있습니까? 질문자가 이미지의 정신적 모델을 구축하는 것을 돕기 위해) 개별 응답에 대한 객관적인 평가를 허용합니다.
