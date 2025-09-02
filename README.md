<img width="904" height="590" alt="image" src="https://github.com/user-attachments/assets/4577ae9c-388f-46e6-a676-1a1f730cbcec" />

## Detailed Explanation

Personalized summarization requires more than generic text compression. A summary that is informative for one reader may omit crucial details for another.  
**Walk2Pers** addresses this by explicitly modeling **how user interests evolve over time** instead of treating them as static profiles.

###  1. Behavior Encoder
The **Behavior Encoder** processes the user’s interaction history (`Bhist`) — a sequence of events like **click**, **skip**, **generate summary**, and **summary generated**.  
- Each interaction is treated as an **operator** that transforms the hidden state:
  - **Clicks** reinforce memory-consistent features.  
  - **Skips** act as *controlled resistance* (not null events), bending the trajectory away from uninteresting content.  
  - **Generate summary** interactions represent deeper engagement, pulling the state toward headline-anchored semantics.  
  - **Summary generated** signals are integrated using a **soft-min pooling** operator, taking the conservative path when signals disagree.
- The evolution is **geometry-aware**: at each step the model predicts a **rotation angle (θ)** and a **step size (m)**, producing a smooth, bounded update.  
- A **dual-memory mechanism** keeps two pathways:  
  - reinforcement (clicks),  
  - suppression (skips).  
  These are fused adaptively to form the evolving hidden state.

The output is a contextualized embedding trajectory (`e′seq`), with auxiliary losses that supervise timestep prediction and next-step classification.

---

###  2. Inverse Decoder
The **Behavior Inverse Decoder** makes the latent state interpretable.  
- It applies a safe inverse of the encoder’s nonlinearity, then reconstructs:  
  - **c′pos** → pseudo-content embedding (what the predicted behavior “looks like” as document content).  
  - **ŝpos** → pseudo-summary embedding (user’s inferred summary preference).  
- This disentanglement allows the system to recover both **content-level** and **summary-level** personalization signals.

---

###  3. Personalized T5 Summarizer
Finally, these behavior signals condition a **frozen T5 model**:  
- A **gating layer** modulates the T5 encoder states using `e′last`.  
- A **single-key cross-attention adapter** injects the personalized summary signal `ŝpos`, letting the model selectively weight tokens.  
- The **T5 decoder** then generates a user-specific summary.  

During training:  
- The **encoder loss** supervises trajectory evolution.  
- The **generation loss** supervises summary quality.  
- Final training objective:  
  \[
  \text{Total Loss} = 0.6 \times \text{Encoder Loss} + 0.4 \times \text{Generation Loss}
  \]

---

### Why It Matters
Traditional methods either:  
- predict discrete actions (click/skip classifiers), or  
- rely on static personalization signals (profiles, prompts).  

Walk2Pers is different:  
- It **models preference drift explicitly** as a **rotation-driven random walk**, producing smooth, geometry-aware transitions.  
- It **separates roles**: encoder = interest evolution, inverse decoder = interpretability, T5 = powerful but frozen backbone.  
- It remains **efficient**: personalization is injected through lightweight adapters while keeping T5 parameters frozen.

---

###  Applications
- Personalized news feeds.  
- Adaptive educational content delivery.  
- Recommendation systems where engagement patterns matter.  

By combining explicit user modeling with the generative strength of LLMs, Walk2Pers shows a path toward more adaptive, interpretable, and user-centric summarization systems.
