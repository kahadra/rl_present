# Active Object Localization with Deep Reinforcement Learning

Estimated total speaking time: about 20 minutes  
English slide-by-slide presentation: about 17 minutes  
Final Korean recap: about 3 minutes

## English Presentation Script

### Slide 1 - Title: Active Object Localization with Deep Reinforcement Learning

Hello everyone. I am Chunwoong Park, a master's student, and today I will be presenting the paper "Active Object Localization with Deep Reinforcement Learning."

In object localization, the goal is to find where an object is in an image, usually with a bounding box. This paper asks a slightly different question. Instead of predicting the final box at once, can a system move the box step by step until it reaches the object? Reinforcement learning is used to solve localization from that viewpoint.

### Slide 2 - Presentation Outline

I will explain the paper in five parts.

First, I will briefly review existing localization methods. Second, I will compare the main approaches before this paper. Third, I will explain the problem this paper focuses on. Fourth, I will describe the proposed reinforcement learning method. Finally, I will discuss the results, strengths, and limits.

### Slide 3 - Before This Paper: Localization as Region Evaluation

Before this paper, many localization systems treated localization as region evaluation.

The pipeline is simple: create possible regions, extract features, score each region, and keep the best boxes as detections.

This makes sense because the hard problem of finding an object becomes many smaller checks. Candidate boxes define what to check, features describe each region, and the classifier decides which regions look like the target.

So the old pipeline is practical. It turns spatial search into region scoring.

### Slide 4 - What Is This Paper Competing Against?

The paper is related to three common strategies.

Sliding window search checks many positions and scales. Its strength is coverage.

Region proposal methods create a smaller set of likely object regions. Their strength is focus.

One-shot regression predicts or refines the box directly from features. Its strength is directness.

So these methods have different strengths, but all of them are still ways of handling spatial search.

### Slide 5 - Problem: Too Many Regions, Too Little Decision-Making

The limitation is about search behavior.

In sliding window search, the pattern is fixed. In region proposal methods, the detector depends on proposals from another stage. In one-shot regression, the model jumps directly to a box.

So these methods do not fully answer one question: from the current box, what should the system do next?

This paper focuses on that question and turns localization into a decision-making problem.

### Slide 6 - Main Idea: Let the Bounding Box Move

The main idea is to let the bounding box move.

The agent starts from a broad region, often the whole image. It observes the current box and updates the search region step by step.

In this view, the box is not just the final output. It is an object that keeps being updated during the decision process.

That is why the method is called active localization. The system looks at a region and decides where to attend next.

### Slide 7 - Method Loop: Observe, Act, Transform

The method can be understood as a repeated loop: observe, act, and transform.

First, the agent observes the current box. Second, it chooses an action. Third, that action changes the box into a new one. Then the agent observes again.

The loop continues until the agent decides to stop. During training, the agent receives feedback about whether its choices were useful. During testing, it must rely on the policy it has learned.

A good action moves the box closer to the object. A bad action moves it away. This is where reinforcement learning fits.

### Slide 8 - What Can the Agent Do?

The action space uses a fixed and limited set of actions.

The agent can move the box left, right, up, or down. It can make the box larger or smaller. It can adjust the shape of the box. It can also choose the trigger action, which means final detection.

In total, the paper uses eight box transformation actions plus one trigger action.

The model does not predict a completely new box at every step. It chooses how to modify the current box. This makes the search process easy to follow, but it can also limit precision for small objects or fine boundaries.

### Slide 9 - Reward: Did the Box Get Closer?

The reward is based on IoU, which stands for Intersection over Union. IoU measures the overlap between the predicted box and the ground-truth box.

If an action increases IoU, the agent receives a positive reward. If an action decreases IoU, the agent receives a negative reward. If the agent triggers detection at the correct time, it receives a larger final reward.

In simple terms, the reward asks one question: did the box get closer to the object? This gives the agent a clear signal for learning useful box movements.

### Slide 10 - Why DQN? Because the State Is Visual

The paper uses DQN, or Deep Q-Network, because the state is visual.

The agent has to understand the image content inside the current box and choose the next action. To do this, the current region is resized, and a pre-trained CNN extracts visual features.

The method also uses action history. The current crop alone may not tell the agent how it arrived there, so recent movements provide useful memory.

The Q-network outputs Q-values for the possible actions. A Q-value is a guess about the long-term benefit of an action. The agent chooses the action with the highest value and continues the search.

### Slide 11 - Why MAP Alone Is Not Enough

The experiments cannot be judged only by a single detection score.

The slide shows three points. First, detection quality: the final boxes still need to be accurate enough.

Second, search cost: the method claims to avoid checking too many regions, so the number of processed regions matters.

Third, learned behavior: the box should move because of a learned policy, not just random or rule-based search.

So we should read the experiments through all three points.

### Slide 12 - Why These Baselines?

The baselines are meaningful because each one tests a different claim.

Sliding window tests search cost. If the proposed method uses far fewer regions, active search is more efficient than full-image scanning.

Region proposal methods test dependence on proposals. The question is whether the agent can move the box by itself instead of relying on a separate proposal step.

Random or rule-based search tests the value of learning. If the learned policy performs better, then the movement is not just box motion; it is useful decision-making.

Detection baselines test accuracy. They show whether the method keeps enough detection quality while reducing search.

### Slide 13 - Detection Result: Use the Actual Table

The main detection result is reported on Pascal VOC 2007. The proposed method, Ours AAR, reaches 46.1 MAP. Regionlets reaches 40.2, DetNet reaches 30.5, and R-CNN reaches 54.2.

The first thing to notice is that R-CNN has the highest MAP. So this paper does not beat every detection method.

This table matches the detection baselines and the region proposal comparison from Slide 12. Regionlets, DetNet, and R-CNN give us reference points for judging accuracy.

The gap with R-CNN shows a limitation of the proposed method. Detection based on region proposals is still stronger in raw accuracy. But the method is higher than Regionlets and DetNet, so active search does not simply give up detection quality.

The main point is balanced: not the best detector, but a working detector with a different search process.

### Slide 14 - Efficiency Result: How Many Actions?

The efficiency result shows why the method matters beyond the detection table.

The median number of actions is 11. This means that a typical successful detection checks about 11 regions. The mean is 25.6 actions across 5,147 detections.

The difference between the median and the mean suggests that some difficult images require longer searches.

This result connects to the sliding window comparison. A sliding window method checks many locations, while this method chooses regions step by step.

It also connects to random or rule-based search. The point is not only that the box moves, but that the policy learns where to move.

### Slide 15 - Visual Result: What the Agent Actually Sees

The visual results show what the agent actually does.

The search path changes depending on the image. The agent does not follow a fixed scanning pattern. It chooses movements based on what it sees in the current region.

This is a strength because we can see the search process, not only the final box.

For example, in some images the agent quickly moves toward the object, while in others it makes several adjustments before selecting the trigger action. This gives us information that a final detection score alone cannot show.

So this slide supports the claim that the model is making image-based decisions before the trigger.

### Slide 16 - Limitations and Interpretation: What Can We Claim?

Taken together, the results show that active localization can work. But they do not show that this method beats every standard detector.

First, the raw detection score is not the best. R-CNN has a higher MAP. So the contribution is not top accuracy. The contribution is showing that learned actions can produce useful localization with a smaller search process.

Second, the action space uses a fixed set of actions. This makes the behavior easy to explain, but it can limit precision for small objects or fine boundaries.

Third, the reward is mostly based on IoU change. IoU is clear and useful, but it does not fully cover hidden objects, context, class confusion, or hard real-world cases.

So this paper is not a direct replacement for modern object detectors. Its value is that it shows localization can be seen as step-by-step decision-making.

### Slide 17 - Final Takeaway

To conclude, this paper changes how we think about localization.

Previous methods mainly generated and evaluated candidate regions. This paper turns localization into box transformation guided by a policy.

The strengths are clear. It connects object localization with reinforcement learning. It shows that an agent can approach an object with a small number of actions. And it gives us a search process that is easy to explain.

The limitations are also clear. Its MAP is lower than R-CNN, the fixed action set can limit precision, and the IoU-based reward cannot cover every difficulty in real detection.

So the best conclusion is not that this paper proposes the strongest detector. It shows a different view of localization: not only scoring boxes, but moving the box and deciding when to stop.

Thank you for listening.

## 마지막 한글 전체 요약 발표문

지금부터는 PPT 내용을 처음부터 다시 한글로 요약해서 설명하겠습니다.

이 논문을 이해하려면 먼저 기존 위치 검출 방식이 왜 자연스러웠는지를 봐야 합니다. 기존 방식은 보통 이미지에서 후보 영역을 만들고, 각 영역의 특징을 추출한 뒤, 분류기나 점수화 함수를 통해 가장 좋은 박스를 고르는 파이프라인을 사용했습니다. 이 방식은 꽤 합리적입니다. 어려운 공간 탐색 문제를 후보 생성, 특징 추출, 분류, 후처리라는 여러 단계로 나누어 해결할 수 있기 때문입니다.

대표적인 방법들도 각각 장점이 있었습니다. 슬라이딩 윈도우는 이미지 전체를 체계적으로 훑을 수 있어서 포괄성이 좋습니다. 리전 프로포절은 객체가 있을 법한 영역에 집중해 후보 수를 줄입니다. 원샷 회귀는 박스 위치를 직접 예측하거나 보정하기 때문에 단순하고 빠른 방향을 제시합니다. 그래서 이 논문은 기존 방법들이 무의미하다고 말하는 것이 아니라, 기존 방법들이 다루지 못한 다른 관점을 제안합니다.

그 공통적인 한계는 탐색 행동 자체가 학습된 순차적 의사결정으로 표현되지 않는다는 점입니다. 기존 접근은 주로 "어떤 후보 박스가 맞는가?"를 묻지만, 이 논문은 "현재 박스에서 다음에 어떤 행동을 해야 하는가?"를 묻습니다. 그래서 검출기는 강화학습의 에이전트가 되고, 현재 시각 영역은 상태, 박스 변환은 행동, IoU 변화는 보상이 됩니다. 이것이 RL을 사용하는 핵심 이유입니다. 위치 검출을 박스 평가 문제가 아니라, 객체에 가까워지는 행동 선택 문제로 바꾸기 때문입니다.

제안 방법에서 박스는 최종 결과물이기 전에 조작 대상입니다. 에이전트는 넓은 영역에서 시작해 현재 박스를 관찰하고, 다음 행동을 선택하면서 탐색 범위를 점차 좁혀 갑니다. 행동은 이동, 크기 변경, 비율 조정, 그리고 최종 검출을 선언하는 trigger로 구성됩니다.

학습 신호는 IoU 변화에서 나옵니다. 박스가 정답에 더 가까워지면 좋은 행동으로 평가되고, 멀어지면 나쁜 행동으로 평가됩니다. DQN은 현재 시각 상태와 행동 이력을 바탕으로 어떤 행동이 장기적으로 더 유리한지 추정합니다.

실험에서는 이 방법이 적은 수의 영역만 보고도 객체를 찾을 수 있는지, 정확도는 충분한지, 학습된 정책이 무작위 탐색보다 의미 있는지를 확인합니다. Pascal VOC 2007 결과에서 제안 방법인 Ours AAR은 46.1 MAP을 기록했고, Regionlets는 40.2, DetNet은 30.5, R-CNN은 54.2였습니다. R-CNN이 절대 성능은 더 높지만, 이 논문의 핵심은 최고 정확도가 아니라 능동적 탐색으로 경쟁력 있는 검출을 수행했다는 점입니다.

효율성 측면에서는 중앙값 기준 11번의 행동, 평균 25.6번의 행동만 사용했습니다. 즉, 수많은 후보 박스를 전부 평가하지 않고도 에이전트가 방향성 있게 객체에 접근할 수 있음을 보여줍니다.

장점은 위치 검출을 강화학습 문제로 명확히 재정의했다는 점, 적은 수의 행동으로 탐색한다는 점, 그리고 박스 이동 과정을 해석할 수 있다는 점입니다. 한계는 행동이 이산적이라 작은 객체나 복잡한 장면에서 정밀도가 떨어질 수 있고, 보상이 IoU 중심으로 단순하며, 현대 검출기를 대체하기에는 제한적이라는 점입니다.

결론적으로 이 논문은 객체 검출을 "많은 후보 박스를 평가하는 방식"에서 "정책이 박스를 움직이며 찾는 방식"으로 바꿔 설명한 연구입니다. 정확도만 보면 최고는 아니지만, 능동적이고 효율적인 탐색이라는 관점에서 의미 있는 기여를 한 논문입니다.
