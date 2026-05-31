# Active Object Localization with Deep Reinforcement Learning Expected Q&A

This document is an English version of the expected Q&A for the presentation.  
The question numbers correspond to the Korean Q&A document.

1. Questions about topics mentioned in the slides or script but not explained in depth
2. Questions about limitations or additional issues that may come up outside the presentation

---

## Part 1. Questions About Topics Mentioned But Not Explained in Depth

### Q1. What exactly does active localization mean in this paper?

Active localization means treating object localization as a sequential search process rather than a one-shot bounding box prediction problem.

The system does not only evaluate candidate boxes passively. It observes the current region and chooses the next action. In this paper, those actions include moving the box, changing its scale, adjusting its aspect ratio, and selecting the trigger action.

The key idea is that the detector becomes an agent that chooses actions, not only a scoring function.

---

### Q2. In this paper, what are the state, action, and reward?

The state consists of visual information from the current box and recent action history. More specifically, the current region is represented using CNN fc6 features, and the action history is added to provide information about how the agent reached the current region.

The actions are transformations applied to the current box. The paper uses eight box transformation actions and one trigger action. The agent can move the box horizontally or vertically, make it larger or smaller, adjust its aspect ratio, or stop the search.

The reward is based on whether the action improves IoU. If IoU increases, the agent receives a positive reward. If IoU decreases, it receives a negative reward. If the agent triggers detection at the right time, it receives a larger terminal reward.

---

### Q3. Why can this problem be formulated as an MDP?

The problem has a sequential structure. The agent observes the current state, chooses an action, and that action creates the next state.

In object localization, the current box determines what the agent sees next. A good action can make the next observation more useful, while a bad action can move the box away from the object.

So localization is not only an independent classification problem. It can be modeled as a sequence of decisions with states, actions, rewards, and next states. That is why the MDP formulation fits this paper.

Strictly speaking, the current crop may not contain all information about the full image. The paper partially addresses this by including action history in the state.

---

### Q4. Why does the paper use a discrete action space?

A discrete action space makes the learning problem simpler. Instead of predicting arbitrary coordinates, the agent only has to choose one action from a fixed set.

It also improves interpretability. Each step can be described as a clear box movement, such as move left, move right, scale up, or trigger.

The downside is precision. Discrete actions may be too coarse for small objects, cluttered scenes, or cases that require fine boundary adjustment.

---

### Q5. What is the role of the trigger action?

The trigger action means that the agent decides the current box should be used as the final detection result.

Other actions keep transforming the box. The trigger action stops the search. This is important because localization is not only about where to move, but also about when to stop.

If the agent triggers too early, the box may be inaccurate. If it triggers too late, the search becomes inefficient.

---

### Q6. Why is an IoU-based reward appropriate?

The goal of localization is to make the predicted box overlap well with the ground-truth box. IoU directly measures that overlap.

If IoU increases, the box has moved closer to the target object. If IoU decreases, the box has moved away from the target. This gives the agent a clear learning signal.

However, IoU only measures geometric overlap. It does not directly capture context, occlusion, class ambiguity, or semantic difficulty.

---

### Q7. Why does the paper use DQN?

DQN is suitable because the state is visual and the action space is discrete.

The paper has nine possible actions, so the Q-network can output one Q-value for each action. The agent then chooses the action with the highest value.

The visual state is represented with CNN features. The paper does not train the entire CNN from scratch. Instead, it uses fc6 features from a pre-trained CNN and feeds them into the Q-network.

In short, the CNN describes the visual state, and Q-learning selects the next spatial action.

---

### Q8. Why is action history included in the state?

The current crop alone may not tell the agent how it arrived there. The same visible region can have different meanings depending on the previous movements.

Action history gives the agent partial memory of the search trajectory. It can help reduce repeated or unnecessary movements and make the current decision more informed.

It is not a perfect memory mechanism, but it provides useful context for sequential localization.

---

### Q9. What does class-specific mean in this method?

Class-specific means that the agent is trained for a target object category, rather than being a fully category-independent object proposal method.

For example, there may be one policy for cows and another policy for cars. This allows the agent to learn search behavior that is useful for a particular category.

The advantage is category-specific reasoning. The drawback is that the method is not a general objectness proposal method that finds all possible object categories.

---

### Q10. How does the method handle multiple objects in one image?

One search trajectory localizes one object instance. To find multiple objects, the process must be repeated.

After a detection is triggered, the method uses an inhibition of return mechanism. It marks the detected region so the agent is discouraged from returning to the same object.

This is not perfect. If the mark does not fully cover the object, duplicate detections can occur. Small or difficult objects may also be missed.

---

### Q11. How should we interpret 11 actions and 25.6 actions?

The median of 11 actions means that a typical successful detection processes about 11 regions.

The mean of 25.6 actions is higher because some difficult cases require much longer search trajectories. In other words, the distribution has a long tail.

These numbers support the efficiency claim. Instead of evaluating many candidate regions, the agent often reaches a good box through a short sequence of decisions.

---

### Q12. If the MAP is lower than R-CNN, why is the paper still meaningful?

The main contribution is not achieving the highest detection score.

The contribution is reformulating object localization as sequential decision-making and showing that a learned agent can move a box toward an object using relatively few evaluated regions.

R-CNN is a strong detector that combines object proposals and CNN classification. This paper does not claim to beat R-CNN in absolute accuracy.

Its value is in the formulation, efficiency, and interpretability of the search process.

---

### Q13. Why compare with sliding window, region proposal, and random or heuristic search?

Each baseline answers a different question.

Sliding window is a reference for exhaustive spatial scanning. It checks whether active search can reduce search cost.

Region proposal methods use an external mechanism to generate candidate regions. This comparison checks whether the agent can reduce dependence on a separate proposal stage.

Random or heuristic search checks whether sequential box movement alone is enough, or whether a learned policy is actually useful.

Detection baselines check whether the method loses too much accuracy while gaining efficiency.

---

### Q14. Why are qualitative results important?

Qualitative results show the search process behind the final detection.

If we only look at the final box, we cannot see how the agent reached that box. The qualitative examples show the sequence of attended regions, selected actions, and trigger decisions.

This makes the method more interpretable. It also helps us see whether the agent follows a fixed scanning pattern or makes image-dependent decisions.

---

## Part 2. Questions About Limitations or Additional Issues

### Q15. Is this method an object proposal method?

Not exactly. The paper explicitly treats the method differently from a category-independent object proposal algorithm.

Object proposal methods usually generate many category-independent candidate regions. This method is class-specific and uses a learned agent to move a box toward a target category.

However, the regions attended by the agent can be interpreted as proposal candidates in some evaluations. The paper uses this idea when analyzing recall.

---

### Q16. Why is overall recall a problem?

The method is strong in early recall, meaning it can find useful regions with a small number of candidates.

However, its overall recall can be lower than strong proposal methods when many candidates are allowed. This happens partly because the method is class-specific. If the feature representation does not recognize an object well, the agent may never localize it.

So the method is efficient for early search, but it may miss some object instances. Improving overall recall is one of its main challenges.

---

### Q17. Why is the method weak for small objects?

Small objects are difficult for several reasons.

First, discrete actions may be too coarse for fine boundary adjustment.

Second, when the crop is large, a small object may be visually weak compared to the background. The CNN features may not provide a strong signal for the correct action.

Third, small objects often appear in cluttered contexts, which makes IoU improvement harder.

---

### Q18. Can this method replace modern object detectors?

No, not directly.

This paper is important mainly as a conceptual contribution from 2015. It shows that object localization can be formulated as a reinforcement learning problem.

Modern detectors such as Faster R-CNN, YOLO-family models, and DETR-family models are much stronger in practical detection accuracy and scalability.

So this paper should be understood as an important example of active search and sequential decision-making in computer vision, not as a direct replacement for modern detectors.

---

### Q19. During testing, does the agent know IoU?

No. IoU requires the ground-truth box, so it is available only during training and evaluation.

During testing, the agent does not receive rewards and does not update the model. It simply follows the learned policy and selects the action with the highest predicted Q-value.

---

### Q20. Is a high Q-value the same as a high detection confidence score?

No. A Q-value estimates the expected long-term value of choosing an action in a state. It is not the same as a standard detector confidence score.

The paper can use Q-values to rank attended regions in certain evaluations, especially with special handling for trigger regions. But conceptually, Q-values are for action selection, not direct class confidence.

So the safe answer is: Q-value and detector confidence are related only indirectly.

---

### Q21. Why did the paper not train the CNN end-to-end?

The paper uses a pre-trained CNN and trains the Q-network on top of it.

This was a practical choice. Reinforcement learning can be unstable and sample-inefficient, especially when visual representations must also be learned from scratch.

Using a pre-trained CNN allows the paper to focus on learning the sequential box transformation policy.

End-to-end training might improve task-specific features, but it would also make training harder.

---

### Q22. Why use Q-learning or DQN instead of policy gradient?

The action space is small and discrete, so Q-learning is a natural choice.

The Q-network can output values for all nine actions, and the agent can choose the action with the highest value.

Policy gradient methods could also be used, especially for continuous control or more complex policies. But at the time, DQN was a strong and natural method for visual states with discrete actions.

---

### Q23. Can this method get stuck in a local optimum?

Yes. If the agent moves toward the wrong region, it can focus on background or an incorrect object and miss the target.

This risk is higher when the initial IoU is low, when the scene is cluttered, or when visual cues are ambiguous.

The paper uses action history, trigger decisions, repeated search, and inhibition of return to reduce some of these issues, but they are not fully solved.

---

### Q24. What happens in crowded scenes or scenes with overlapping objects?

Crowded scenes are difficult.

The agent may converge to only part of an object, confuse nearby instances, or trigger on a box that overlaps multiple objects.

Inhibition of return helps reduce duplicate detections, but it is not perfect. If the inhibited region does not fully cover the detected object, the system may still produce duplicates.

This means the method is intuitive for active single-object localization, but dense multi-object detection requires stronger memory and post-processing.

---

### Q25. Does fewer actions always mean faster real computation?

Not necessarily.

Fewer actions means fewer regions are processed, which suggests lower computational cost. But actual runtime depends on how CNN feature extraction is implemented, whether regions are processed on GPU, and whether computations are batched.

So it is safer to say that the method reduces the number of evaluated regions, rather than claiming that it always guarantees faster wall-clock runtime.

---

### Q26. If you had to name the biggest limitation, what would it be?

The biggest limitation is the trade-off between efficient search and robust recall or precise localization.

The agent can often reach an object with few actions, but it may miss objects that are small, cluttered, visually ambiguous, or poorly represented by the CNN features.

So the paper answers the question "Can localization be done as efficient active search?" quite well, but it does not fully solve the question "Can every object instance be found reliably?"

---

### Q27. How would you summarize the paper in one sentence?

This paper reformulates object localization from evaluating many candidate boxes into a sequential decision-making problem where an agent moves a bounding box and decides when to stop.

A shorter version is: this paper treats the detector not only as a scoring function, but as an agent that acts.

---

## Quick Answer Sentences

- The main contribution is not the highest MAP, but a new formulation of localization.
- Reinforcement learning is used to formulate and learn the state-action-reward process, not merely to describe the idea.
- Lower MAP than R-CNN is a limitation, but efficient active search is a separate contribution.
- Discrete actions make learning and interpretation easier, but they limit precision for small or cluttered objects.
- IoU reward directly matches localization geometry, but it does not capture semantic difficulty or context.
- Qualitative results matter because they show the search trajectory, not only the final box.

---

## Reference Basis

- Caicedo and Lazebnik, "Active Object Localization with Deep Reinforcement Learning", ICCV 2015.
- Paper abstract: class-specific active detection, simple transformation actions, Pascal VOC 2007 evaluation, and localization with roughly 11 to 25 regions.
- Paper method: MDP formulation, eight transformation actions plus one trigger action, CNN fc6 features, action history, and DQN-based Q-value prediction.
- Paper experiments: Pascal VOC 2007, MAP comparison, proposal/recall analysis, median 11 actions, mean 25.6 actions, recall and error mode analysis.
