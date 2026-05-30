# Active Object Localization with Deep Reinforcement Learning

Estimated total speaking time: about 20 minutes  
English slide-by-slide presentation: about 16 minutes  
Final Korean recap: about 4 minutes

## English Presentation Script

### Slide 1 - Title: Active Object Localization with Deep Reinforcement Learning

On this first slide, please focus on the sentence on the left: "Localization is not only scoring boxes. It can be a sequence of actions." This is the core thesis of the presentation.

The traditional view is shown as "Before": evaluate many candidate regions. The new view in this paper is shown as "This paper": move one box through a learned policy. So the main question for today is written at the bottom right: can an object detector act instead of evaluating many boxes?

In this presentation, I will explain how the paper changes object localization from passive region evaluation into policy-guided sequential search.

### Slide 2 - Presentation Outline

This slide shows the order of the presentation. I will follow the five icons from left to right.

First, I will explain the existing localization approach, which depends on many candidate regions. Second, I will define the problem: this search is inefficient and mostly passive. Third, I will explain the new reinforcement learning idea, where box movement becomes an action. Fourth, I will discuss the experiments, especially efficiency and accuracy. Finally, I will evaluate the paper by discussing its strengths and limitations.

This structure also follows the analysis format: problem and motivation, proposed method, experimental results, and the paper's advantages and limitations.

### Slide 3 - Before This Paper: Localization as Region Evaluation

On slide 3, the pipeline on the right shows the conventional localization process. It starts from an image, generates candidate boxes, extracts features, classifies or ranks the boxes, and then produces the final detection.

The image illustration on the left shows why this can become expensive. There are many possible boxes in one image. The system has to consider many locations, sizes, and shapes before it can decide which box is correct.

So in this older pipeline, the detector is mainly an evaluator. It receives many candidate boxes and asks, "Which candidate box is correct?" The detector does not really decide where to look next. It mostly scores what has already been proposed.

### Slide 4 - What Is This Paper Competing Against?

Slide 4 shows three important competing ideas.

The first one is sliding window. This method scans many positions across the image. It is simple, but it can be exhaustive because it checks too many possible windows.

The second one is region proposal. This method first generates candidate boxes that may contain objects. It is more efficient than full sliding window, but the detector becomes dependent on the proposal stage.

The third one is one-shot regression. This method predicts or refines the box directly. It is fast, but it does not show a real search process.

The paper positions itself between these approaches. It wants the detector to search actively, but without evaluating a huge static list of candidate regions.

### Slide 5 - Problem: Too Many Regions, Too Little Decision-Making

This slide states the problem most clearly. On the left, the old question is: "Which candidate box is correct?" On the right, the new question is: "What action should the agent take next?"

This difference is the main conceptual shift of the paper. The authors are not only trying to improve a classifier. They are changing the task itself. Object localization becomes sequential decision-making.

In reinforcement learning terms, the detector becomes an agent. It observes the current state, chooses an action, receives a reward, and learns a policy. Here, the state is the current visual region. The action is a transformation of the bounding box. The reward tells the agent whether the box moved closer to the true object.

So the motivation is efficiency and active decision-making. The detector should not passively wait for many candidate boxes. It should decide how to move the current box.

### Slide 6 - Main Idea: Let the Bounding Box Move

Slide 6 introduces the main idea visually. In the paper figure, the agent starts from a broad region, attends to a sequence of regions, and finally stops with a trigger action.

The three blocks on the right are useful for remembering the method. The start point is the whole scene or a broad region. The loop is "observe, action, transformed box." The end point is the trigger action, which means the agent believes the box is tight enough.

The important point is that the box is not just a final output. In this paper, the box is something the agent controls. It can move, expand, shrink, and change shape. Localization becomes a path, not just a single prediction.

### Slide 7 - Method Loop: Observe, Act, Transform

Slide 7 gives the reinforcement learning loop.

First, the current box is the state. This is what the agent observes. Second, the CNN plus Q-network works as a value estimator. It estimates which action is useful in the current state. Third, the agent selects an action, which is a box transformation. Fourth, the new box is produced, and the reward is calculated from the IoU change.

The red stop button in the center is the trigger. If the agent selects this trigger, the search stops and the current box becomes the detection result.

This loop is repeated until the trigger is selected. So, instead of scoring thousands of independent boxes, the method creates a controlled sequence of box transformations.

### Slide 8 - What Can the Agent Do?

Slide 8 explains the action space. The paper defines eight box transformation actions plus one trigger action.

The actions cover four degrees of freedom. The agent can move the box horizontally or vertically. It can scale the box bigger or smaller. It can adjust the aspect ratio. And finally, it can trigger detection.

This action set is discrete, not continuous. That means the agent does not directly predict exact coordinates. Instead, it learns a search policy over a fixed set of possible movements.

This design makes the reinforcement learning problem easier to train, but it also creates a limitation. Because the actions are discrete, the search may be less precise for very small objects or highly cluttered scenes.

### Slide 9 - Reward: Did the Box Get Closer?

Slide 9 explains the reward design. The key measure here is IoU, which means Intersection over Union. IoU measures how much the predicted box overlaps with the ground-truth box.

The slide shows three steps. First, compare the current box with the ground truth. Second, apply an action and get a new box. Third, check whether the IoU improved.

If IoU improves, the agent receives a positive reward, plus one. If IoU decreases, the agent receives a negative reward, minus one. If the trigger is correct, the agent receives a larger terminal reward, plus three.

This reward design is simple and intuitive. It directly teaches the agent that good actions are actions that move the box closer to the object. It also teaches the agent when to stop.

### Slide 10 - Why DQN? Because the State Is Visual

Slide 10 explains why the method uses DQN, or Deep Q-Network.

The state is visual, so the model needs image features. In the figure, the current region is warped to a 224-pixel input. Then a pre-trained CNN extracts fc6 features. These features form a 4096-dimensional observation vector.

The slide also shows action history. This is important because localization is sequential. If the agent only sees the current crop, it may not know how it arrived there. The action history gives the model a small memory of previous movements.

The output is Q-values for nine possible actions. A Q-value is the expected long-term value of choosing one action in the current state. The agent chooses the action with the best value and continues the search.

So the method can be understood as a Markov Decision Process. The state is visual features plus action history. The action changes the box. The reward comes from IoU improvement. The learned policy decides the next movement.

### Slide 11 - What Should the Experiments Prove?

Slide 11 changes from method to experiment. The experiments need to prove three things.

First is efficiency. Can the method localize an object with fewer evaluated regions? Second is accuracy. Does it still reach good boxes? Third is policy contribution. Is learned search better than naive or random search?

These three questions are important because the paper is not only claiming high accuracy. It is claiming that localization can be done through an efficient learned search process.

### Slide 12 - Baselines Are Not Decoration

Slide 12 explains why the baselines matter.

Sliding window tests whether active search can avoid exhaustive scanning. Region proposal tests whether the learned policy can reduce dependence on external candidate generation. Random or heuristic search tests whether the learned policy is meaningful. Detection baselines test whether efficiency destroys accuracy.

So this slide tells us how to read the experiments. The baselines are not just names in a table. Each one checks a specific part of the paper's claim.

### Slide 13 - Detection Result: Use the Actual Table

Slide 13 shows the main detection result table on Pascal VOC 2007.

The proposed method, shown as Ours AAR, reaches 46.1 MAP. Regionlets reaches 40.2 MAP. DetNet reaches 30.5 MAP. R-CNN reaches 54.2 MAP.

At first, we can see that R-CNN is stronger in absolute MAP. However, the important interpretation is more careful. R-CNN relies on object proposals, while this paper focuses on active localization with a learned policy. Therefore, the proposed method should not be presented as the best detector overall. It should be presented as a competitive active localization method compared with non-proposal CNN localization baselines.

The main contribution signal is not only the MAP number. It is the combination of reasonable accuracy and an active search formulation.

### Slide 14 - Efficiency Result: How Many Actions?

Slide 14 is the strongest slide for the efficiency claim.

The median number of actions is 11. That means a typical successful detection needs roughly 11 processed regions. The mean is 25.6 actions across 5,147 detections.

This is very different from methods that evaluate many candidate boxes. The agent does not scan the whole image blindly. It follows a directed path and stops when it thinks the box is good enough.

So the real contribution is not "best absolute AP." The real contribution is active localization with very few evaluated regions.

### Slide 15 - Qualitative Result: What the Agent Actually Sees

Slide 15 shows qualitative examples. Each row shows the attended regions and selected actions before the final trigger.

This slide helps us understand what the agent is doing. The search path is image-dependent. It is not a fixed sliding-window sweep. The agent changes its movement according to the visual content of the image.

The strength is that the model can produce an interpretable sequence of attention. We can see how the box moves toward the object.

But the limitation is also shown here. Because the movements are discrete, the agent can still miss small objects or objects in cluttered scenes. This is why the slide says the best reading is: this is a 2015 efficiency argument for learned active search, not a modern detector replacement.

### Slide 16 - Final Takeaway

The final slide summarizes the whole paper.

Previous localization methods evaluated many candidate regions. This paper reformulates localization as policy-guided box transformation. The result is efficient active search, but it is limited by class-specific policy, discrete actions, and simplified reward.

So my final interpretation is this. The paper is valuable because it changes how we think about localization. Instead of treating object detection as only a scoring problem, it shows that a detector can act, move, and decide when to stop.

Thank you for listening.

## 마지막 한글 전체 요약 발표문

지금부터는 PPT를 처음부터 다시 한글로 요약해서 설명하겠습니다.

1번 슬라이드의 핵심 문장은 "Localization is not only scoring boxes. It can be a sequence of actions."입니다. 즉, 객체 위치 검출은 많은 후보 박스에 점수를 매기는 일만이 아니라, 박스를 움직이는 행동들의 순서로 볼 수 있다는 뜻입니다. 이 발표의 중심 질문은 "객체 검출기가 많은 박스를 평가하는 대신 직접 행동할 수 있는가?"입니다.

2번 슬라이드는 발표 순서입니다. 기존 방식, 문제점, 강화학습 기반 새 아이디어, 실험, 그리고 장점과 한계를 차례로 설명합니다.

3번과 4번 슬라이드는 기존 방식을 설명합니다. 기존 위치 검출은 이미지에서 후보 박스를 많이 만들고, 특징을 추출하고, 박스를 분류하거나 순위를 매긴 뒤 최종 검출을 선택했습니다. 슬라이딩 윈도우는 너무 많이 훑어야 하고, 리전 프로포절은 후보 생성 단계에 의존하며, 원샷 회귀는 탐색 과정이 약합니다.

5번 슬라이드에서 문제가 명확해집니다. 기존 질문은 "어떤 후보 박스가 맞는가?"였지만, 이 논문의 질문은 "에이전트가 다음에 어떤 행동을 해야 하는가?"입니다. 그래서 객체 위치 검출을 순차적 의사결정 문제로 바꿉니다.

6번부터 10번 슬라이드는 제안 방법입니다. 에이전트는 넓은 영역에서 시작해 현재 박스를 관찰하고, CNN과 Q-network로 행동 가치를 계산한 뒤, 박스를 움직이거나 크기를 바꾸거나 비율을 조정합니다. 박스가 충분히 좋다고 판단되면 trigger action으로 멈춥니다. 보상은 IoU 변화로 계산합니다. IoU가 증가하면 +1, 감소하면 -1, 올바른 trigger는 +3을 받습니다. DQN은 현재 이미지 영역의 CNN fc6 특징과 행동 이력을 사용해 9개 행동의 Q-value를 예측합니다.

11번과 12번 슬라이드는 실험이 무엇을 증명해야 하는지 보여줍니다. 핵심 질문은 세 가지입니다. 적은 영역만 보고도 찾을 수 있는가, 정확도는 유지되는가, 학습된 정책이 무작위나 휴리스틱보다 의미 있는가입니다. 그래서 슬라이딩 윈도우, 리전 프로포절, 랜덤 탐색, 검출 베이스라인이 각각 다른 주장을 검증합니다.

13번 슬라이드에서는 Pascal VOC 2007 결과를 봅니다. 제안 방법인 Ours AAR은 46.1 MAP이고, Regionlets는 40.2, DetNet은 30.5, R-CNN은 54.2입니다. R-CNN이 절대 성능은 더 높지만, 이 논문의 핵심은 최고 MAP가 아니라 능동적 탐색 방식으로 경쟁력 있는 검출을 했다는 점입니다.

14번 슬라이드는 효율성 결과입니다. 중앙값 기준으로 11번의 행동, 평균적으로 25.6번의 행동만 사용했습니다. 이것은 많은 후보 박스를 전부 평가하는 방식과 달리, 에이전트가 방향성 있게 탐색한다는 것을 보여줍니다.

15번과 16번 슬라이드는 장점과 한계입니다. 장점은 위치 검출을 강화학습 문제로 명확하게 바꿨고, 적은 행동으로 탐색하며, 박스 이동 과정을 시각화할 수 있다는 점입니다. 한계는 행동이 이산적이라 작은 객체나 복잡한 장면에서 정밀도가 떨어질 수 있고, 2015년 논문이므로 현대 검출기를 대체하는 모델로 보기는 어렵다는 점입니다.

결론적으로 이 논문은 객체 검출을 "많은 후보 박스 평가"에서 "정책이 박스를 움직이며 찾는 과정"으로 바꿔서 설명한 연구입니다. 정확도만 보면 최고는 아니지만, 능동적이고 효율적인 탐색이라는 관점에서 의미 있는 논문입니다.
