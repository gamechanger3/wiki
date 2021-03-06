# ð­ æ¼å-è¯è®ºåç®æ³ Actor-Critic

---

â­ ä»å¤©æä»¬æ¥è¯´è¯´å¼ºåå­¦ä¹ ä¸­çä¸ç§ç»åä½ Actor Critic (æ¼åè¯å¤å®¶), **å®åå¹¶äº ä»¥å¼ä¸ºåºç¡ (æ¯å¦ Q learning) å ä»¥å¨ä½æ¦çä¸ºåºç¡ (æ¯å¦ Policy Gradients) ä¸¤ç±»å¼ºåå­¦ä¹ ç®æ³**.

## 1. Actor-Critic æ¹æ³ï¼ACï¼

<img src="https://gitee.com/veal98/images/raw/master/img/20201113160456.png" style="zoom:50%;" />

- ð **ç­ç¥ç½ç»** - Actor-Critic ä¸­ç **ãActorã** ï¼åèº«æ¯ Policy Gradients, è¿è½è®©å®æ¯«ä¸è´¹åå°å¨è¿ç»­å¨ä½ä¸­éååéçå¨ä½.

  å¯¹äº Actor ç½ç» $ð_ð$ï¼ç®æ æ¯æå¤§ååæ¥ææï¼éè¿ $ðð½(ð) / ðð$ åå¯¼æ°æ¥æ´æ°ç­ç¥ç½ç»çåæ° ðï¼

  <img src="https://gitee.com/veal98/images/raw/master/img/20201113160646.png" style="zoom: 50%;" />

- ð **ä»·å¼ç½ç»** - Actor Critic ä¸­ç **ãCriticã** ï¼åèº«æ¯ Q-learning æèå¶ä»ç Value-Based çå­¦ä¹ æ³ , ç¨äºè¯ä¼°å½åç¶æçå¥½åãè½è¿è¡åæ­¥æ´æ°, èä¼ ç»ç Policy Gradients åæ¯ååæ´æ°, è¿éä½äºå­¦ä¹ æç.

  å¯¹äº Critic ç½ç» $ð_ð^ð$ï¼ç®æ æ¯å¨éè¿ MC æ¹æ³æè TD æ¹æ³è·å¾åç¡®ç $ð_ð^ð(ð ð¡)$ å¼å½æ°ä¼°è®¡ï¼

  <img src="https://gitee.com/veal98/images/raw/master/img/20201113160851.png" style="zoom:50%;" />

ð¸ **ä¸å¥è¯æ¦æ¬ Actor Critic æ¹æ³**:

ç»åäº Policy Gradient (Actor) å å¼å½æ°è¿ä¼¼ Function Approximation (Critic) çæ¹æ³. â­ **`Actor` åºäºæ¦çéè¡ä¸º, `Critic` åºäº `Actor` çè¡ä¸ºè¯å¤è¡ä¸ºçå¾å, `Actor` æ ¹æ® `Critic` çè¯åä¿®æ¹éè¡ä¸ºçæ¦ç**.

<img src="https://gitee.com/veal98/images/raw/master/img/20201119110417.png" style="zoom: 55%;" />

> ð¡ **`Actor` ä¿®æ¹è¡ä¸ºæ¶å°±åèçç¼çä¸ç´ååå¼è½¦, `Critic` å°±æ¯é£ä¸ªæ¶æ¹åçæ¹å `Actor` å¼è½¦æ¹åç.**
>
> æèè¯´è¯¦ç»ç¹, å°±æ¯ `Actor` å¨è¿ç¨ Policy Gradient çæ¹æ³è¿è¡ Gradient ascent çæ¶å, ç± `Critic` æ¥åè¯ä», è¿æ¬¡ç Gradient ascent æ¯ä¸æ¯ä¸æ¬¡æ­£ç¡®ç ascent, å¦æè¿æ¬¡çå¾åä¸å¥½, é£ä¹å°±ä¸è¦ ascent é£ä¹å¤.

ð¸ **Actor Critic æ¹æ³çä¼å¿**: å¯ä»¥è¿è¡åæ­¥æ´æ°, æ¯ä¼ ç»ç Policy Gradient è¦å¿«.

ð¸ **Actor Critic æ¹æ³çå£å¿**: åå³äº Critic çä»·å¼å¤æ­, ä½æ¯ Critic é¾æ¶æ, åå ä¸ Actor çæ´æ°, å°±æ´é¾æ¶æ. ä¸ºäºè§£å³æ¶æé®é¢, Google Deepmind æåºäº `Actor Critic` åçº§ç `Deep Deterministic Policy Gradient`. åèèåäº DQN çä¼å¿, è§£å³äºæ¶æé¾çé®é¢. æä»¬ä¹åä¹ä¼è¦è®²å° Deep Deterministic Policy Gradient. 

## 2. Advantage AC ç®æ³ï¼A2Cï¼

ä¸é¢ä»ç»çéè¿è®¡ç®ä¼å¿å¼å½æ° $ð´^ð(ð , ð)$ ç Actor Critic ç®æ³ç§°ä¸º `Advantage Actor-Critic ç®æ³`ï¼å®æ¯ç®åä½¿ç¨ Actor Critic ææ³çä¸»æµç®æ³ä¹ä¸

<img src="https://gitee.com/veal98/images/raw/master/img/20201113161620.png" style="zoom:50%;" />

> ð å¶å® Actor Critic ç³»åç®æ³ä¸ä¸å®è¦ä½¿ç¨ä¼å¿å¼å½æ° $ð´^ð(ð , ð)$ï¼è¿å¯ä»¥æå¶å®åç§

## 3. Asynchronous Advantage AC ç®æ³ï¼A3Cï¼

Reinforcement learning æä¸ä¸ªé®é¢å°±æ¯å®å¾æ¢ãé£æä¹å¢å è®­ç»çéåº¦å¢ï¼å°±æ¯ `Asynchronous(å¼æ­¥ç) Advantage Actor-Critic` 

A3C æ¯ DeepMind åºäº Advantage Actor-Criticï¼A2C ç®æ³æåºæ¥çå¼æ­¥çæ¬ï¼**å° Actor-Critic ç½ç»é¨ç½²å¨å¤ä¸ªçº¿ç¨ä¸­åæ¶è¿è¡è®­ç»ï¼å¹¶éè¿å¨å±ç½ç»æ¥åæ­¥åæ°ãè¿ç§å¼æ­¥è®­ç»çæ¨¡å¼å¤§å¤§æåäºè®­ç»æçï¼è®­ç»éåº¦æ´å¿«ï¼å¹¶ä¸ç®æ³æ§è½ä¹æ´å¥½**ã

<img src="https://gitee.com/veal98/images/raw/master/img/20201113162317.png" style="zoom: 62%;" />

æ¹æ³æ­¥éª¤ï¼

- **æ¯ä¸ª worker é½ä¼ copy å¨å±åæ°**
- æ¯ä¸ª worker é½ä¸ç¯å¢è¿è¡äºå¨ï¼å¹¶å¾å° sample data
- è®¡ç®æ¢¯åº¦
- æ´æ°å¨å±åæ°





## ð References

- [Bilibili - æå®æ¯ãæ·±åº¦å¼ºåå­¦ä¹ ã](https://www.bilibili.com/video/BV1MW411w79n)
- [Github - LeeDeepRL - Notes](https://datawhalechina.github.io/leedeeprl-notes/)
- [CSDN - æå®æ¯æ·±åº¦å¼ºåå­¦ä¹ ç¬è®° - jessie](https://blog.csdn.net/cindy_1102/article/details/87904928)
- ð [Github - Deep-Learning-with-TensorFlow-book](https://github.com/dragen1860/Deep-Learning-with-TensorFlow-book)
- [Github - DeepRL-TensorFlow2](https://github.com/marload/DeepRL-TensorFlow2) - ð Simple implementations of various popular Deep Reinforcement Learning algorithms using TensorFlow2