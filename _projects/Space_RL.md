---
layout: page
title: Space_RL
description: Reinforcement Learning Game playing
img: 
importance: 5
category: NCKU
---

> [GitHub](http://github.com/@KendellHsu/space_rl)
> [Demo Vedio](https://youtu.be/0Me6T6cgl3g)

## A. Problem Overview
This project aims to train a reinforcement learning agent to play a "Space Survival" game implemented in Pygame. The agent learns to control a spaceship by moving left/right and shooting, avoiding or destroying asteroids while collecting shield and weapon power-ups to extend survival and accumulate higher scores. The agent's state is represented by a 51-dimensional feature vector consisting of the player's own information, the eight nearest asteroids, and the two nearest power-ups. The Double DQN algorithm is used for policy learning. 

## B. Environment Setup
Hardware: One RTX 3080 and one RTX 4060 used as training machines
Software: torch(2.7.1), cuda(12.8)
[Detailed training environment setup](https://hackmd.io/@kendellhsu/RL_train_env)

## C. State Design
The entire state is output as a 1D numpy.array (length 51) and fed into the MLP model. It is divided into three categories: player (5 dimensions), asteroids (8×5 dimensions), and power-ups (2×3 dimensions).
### 1. Player Information (5 dimensions)

| Dimension         | Range/Normalization | Description                             |
| ---------- | ------ | ------------------------------ |
| `px_norm`  | $0,1$  | Player x coordinate / screen width (`WIDTH`)          |
| `hp_norm`  | $0,1$  | Player health / max health 100                 |
| `gun_oh_1` | {0,1}  | Gun level one-hot: level 1                |
| `gun_oh_2` | {0,1}  | Gun level one-hot: level ≥2               |
| `cd_norm`  | $0,1$  | Remaining shooting cooldown / total cooldown delay (`p.bullet_delay`) |

```python!
p = self.game.player.sprite
px_norm   = p.rect.centerx / WIDTH
hp_norm   = p.health / 100
gun_oh  = [1 if p.gun == 1 else 0, 1 if p.gun >= 2 else 0]
cd_norm   = len(p.bullet_timer) / p.bullet_delay  # 0~1****
```

### 2. Asteroids (8 asteroids × 5 dimensions, 40 dimensions total)
- Only the 8 nearest asteroids to the player are included; if fewer than 8 exist, pad with zeros.
- Each asteroid feature: relative position `(dx, dy)`, velocity `(vx, vy)`, radius `rr`.
- Velocity normalization upper bounds are `MAX_SPD_X=3` and `MAX_SPD_Y=10` respectively.

| Dimension   | Range     | Description                     |
| ---- | ------ | ---------------------- |
| `dx` | $-1,1$ | (Asteroid x – Player x) / `WIDTH`  |
| `dy` | $-1,1$ | (Asteroid y – Player y) / `HEIGHT` |
| `vx` | $-1,1$ | Asteroid horizontal velocity / `MAX_SPD_X`     |
| `vy` | $0,1$  | Asteroid vertical velocity / `MAX_SPD_Y`     |
| `rr` | Real     | Asteroid radius (pixels)            |

#### Implementation snippet
```python!
rocks = sorted(self.game.rocks,
               key=lambda r: (r.rect.y-p.rect.y)**2 + (r.rect.x-p.rect.x)**2)[:MAX_ROCK]
rock_feats = []
for r in rocks:
    dx = (r.rect.centerx - p.rect.centerx) / WIDTH
    dy = (r.rect.centery - p.rect.centery) / HEIGHT
    vx = r.speedx / MAX_SPD_X
    vy = r.speedy / MAX_SPD_Y
    rr = r.radius
    rock_feats += [dx, dy, vx, vy, rr]
rock_feats += [0.0] * (MAX_ROCK*5 - len(rock_feats)
```

### 3. Power-up Information (2 power-ups × 3 dimensions, 6 dimensions total)
- Only the 2 nearest power-ups to the player are included; if fewer than 2 exist, pad with zeros.
- Each power-up feature: relative position `(dx, dy)`, type `tp` (shield `+1` / gun `-1`).

| Dimension   | Range       | Description                       |
| ---- | -------- | ------------------------ |
| `dx` | $-1,1$   | (Power-up x – Player x) / `WIDTH`    |
| `dy` | $-1,1$   | (Power-up y – Player y) / `HEIGHT`   |
| `tp` | {+1, -1} | `shield` → +1；`gun` → -1 |


```python!
powers = sorted(self.game.powers,
                key=lambda pw: abs(pw.rect.y-p.rect.y))[:MAX_POWER]
power_feats = []
for pw in powers:
    dx = (pw.rect.centerx - p.rect.centerx) / WIDTH
    dy = (pw.rect.centery - p.rect.centery) / HEIGHT
    tp = 1 if pw.type=='shield' else -1
    power_feats += [dx, dy, tp]
power_feats += [0]*(MAX_POWER*3 - len(power_feats)
```

### 4. Concatenate into Final State Vector

```python!
state_vec = np.array(
    [px_norm, hp_norm] + gun_oh + [cd_norm] +
    rock_feats + power_feats,
    dtype=np.float32
)
return state_vec  # shape=(51,)
```
Total features: 5 + 40 + 6 = 51 dimensions.

## D. Reward Design
Each step (frame) starts with a fixed time penalty and is dynamically adjusted based on game events. Main parameters are defined at the top of Env.py:
```python!
# ---- reward parameters ----
ALPHA_HIT      = 0.8       # Asteroid destruction reward coefficient
PSI_MIN        = 0.4       # ψ(hp) = PSI_MIN + (1-PSI_MIN)*ϕ
BETA_COLL      = 1.2       # Collision penalty coefficient
GAMMA_SHIELD   = 1.4       # Health recovery reward coefficient
R_GUN          = 16        # Gun upgrade reward
MISS_SHOT_PEN  = -1.0      # Penalty for shooting during cooldown
COOLDOWN_BONUS = +0.5      # Cooldown completion bonus
```

### 0. Fixed Time Step Penalty
#### Encourage the agent to achieve high scores quickly

```python!
reward = -0.25
```

### 1. Asteroid Destruction (Hit Bonus)
#### Condition: `delta_score = self.game.score - score_before`, score change indicates newly destroyed asteroids
First calculate the current health ratio:
    $$ \phi = \frac{hp_{\mathrm{after}}}{100}$$
    $$\psi = \mathrm{\psi_MIN} + (1 - \mathrm{\psi_MIN}) \times \phi$$
Reward formula:
    $$reward += \alpha_{HIT}*Δscore*ψ$$

The higher the health, the higher ψ, encouraging the agent to actively shoot when in good health.

```python!
delta_score = self.game.score - score_before
if delta_score:
    ϕ = hp_after / 100
    ψ = PSI_MIN + (1-PSI_MIN)*ϕ
    hit_bonus = ALPHA_HIT * delta_score * ψ
    reward += hit_bonus
```


### 2. Asteroid Collision (Collision Penalty)
#### Condition: `self.game.is_collided == True`
- Calculate damage: r = hp_before - hp_after
- Weight factor: factor = 2 - ϕ (heavier penalty when at low health)

#### Penalty formula:
    $$ penalty= \beta_{COLL}×r×(2−ϕ)$$
    $$reward−=penalty$$

```python!
if self.game.is_collided:
    r = hp_before - hp_after
    factor = 2 - (hp_after/100)
    penalty = BETA_COLL * r * factor
    reward -= penalty
```

### 3. Power-up Collection (Power-up Bonus)
#### Condition: `self.game.is_power == True`
- Shield: If health gain hp_gain>0, then

Higher value when at low health, encouraging timely health recovery.
    $$\mathrm{reward} \mathrel{+}= hp\_gain \times \mathrm{\gamma_{SHIELD}} \times \bigl(1 - \phi\bigr)$$
- Gun upgrade: Fixed reward of 16

```python!
if self.game.is_power:
    hp_gain = hp_after - hp_before
    if hp_gain>0:
        reward += hp_gain * GAMMA_SHIELD * (1-ϕ)
    else:
        reward += R_GUN
```

### 4. Cooldown Shaping
- Detect shooting action:
    - Pressing shoot (`action==1` and `ready_before==True`) → `fired_now=True`, mark as entering cooldown.
    - Random button press penalty: If pressed but no bullet fired, and not yet penalized, give one `MISS_SHOT_PEN=-1.0`.
    - Cooldown completion reward: When cooldown ends (`in_cooldown==True` and `ready_after==True`), give `+0.5` and reset state.

```python!
# Detect fired_now and in_cooldown
if was_shooting and ready_before:
    self.in_cooldown = True
    self.cooldown_penalized = False

if was_shooting and not ready_before and not self.cooldown_penalized:
    reward += MISS_SHOT_PEN
    self.cooldown_penalized = True

if self.in_cooldown and ready_after:
    reward += COOLDOWN_BONUS
    self.in_cooldown = False
```
### 5. Wall-hit Penalty
#### Condition: When the agent attempts to move outside the wall boundaries.
If `player.rect.left == 0 and action==LEFT` or `player.rect.right==WIDTH and action==RIGHT`, penalize with `-0.5`.

```python!
hit_left  = (player.rect.left==0  and action==LEFT)
hit_right = (player.rect.right==WIDTH and action==RIGHT)
if hit_left or hit_right:
    reward -= 0.5
```


## E. Model Architecture
> The original `template_v2` provided a CNN-based neural network. Since `state` has been changed to pass a 51-dimensional vector, a smaller redesigned model is sufficient.
- Algorithm: Double DQN
- Network structure: A three-layer fully connected MLP (Multi-Layer Perceptron),
    - Input dimension `input_dim` → 128 → 128 → Output dimension `num_actions`
    - All hidden layers use ReLU activation
    - Final layer outputs Q-values for each action
```python!
class MLPDDQN(nn.Module):
    def __init__(self, input_dim, num_actions):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, num_actions)
        )
    def forward(self, x):
        return self.net(x)
```
- Output: Returns a tensor of shape `(128, num_actions)`, representing Q-values for each state-action pair, used by ε-greedy strategy to select actions.

## F. Training Process
### 0. Hyperparameters
| Hyperparameter                   | Value    |
| -------------------- | ----- |
| `num_episodes`       | 4000  |
| `batch_size`         | 64    |
| `gamma` (discount factor)       | 0.99  |
| `lr` (learning rate)           | 1e-4  |
| `epsilon_start`      | 1.0   |
| `epsilon_end`        | 0.05  |
| `epsilon_decay`      | 0.999 |
| `memory_capacity`    | 60000 |
| `target_update_freq` | 2000  |

### 1. Initialization
```python!
env          = SpaceShipEnv()
state_dim    = env._extract_state().shape[0]
num_actions  = len(env.action_space)
policy_net   = MLPDDQN(state_dim, num_actions).to(device)
target_net   = MLPDDQN(state_dim, num_actions).to(device)
target_net.load_state_dict(policy_net.state_dict())
target_net.eval()
optimizer    = optim.Adam(policy_net.parameters(), lr=lr)
memory       = ReplayMemory(memory_capacity)
epsilon      = epsilon_start
total_steps  = 0
```

### 2. Episode Loop (`for episode in range(start_episode, 4000`)
Based on template_v2, the main modification is changing from DQN to DDQN model
```diff
--- Original: DQN 
+++ Modified: DDQN 
- # Original DQN: directly use target_net's max Q value
- # target_q = rewards + γ * target_net(next_states).max(1)[0] * (1 - dones)
+ # DDQN: use policy_net to select action, then use target_net to evaluate that action's Q value
+ next_actions   = policy_net(next_states).argmax(dim=1).unsqueeze(1)
+ next_q_values  = target_net(next_states).gather(1, next_actions).squeeze(1)
+ target_q       = rewards + γ * next_q_values * (1 - dones)
```
- DQN: target_q = r + γ·maxₐ Qₜₐᵣ₍ₙₑₜ₎(s′, a)
- DDQN:
    1. Select action a* = arg maxₐ Qₚₒₗᵢcᵧ( s′, a )
    2. Evaluate action value Qₜₐᵣ₍ₙₑₜ₎( s′, a* )
    3. target_q = r + γ·Qₜₐᵣ₍ₙₑₜ₎( s′, a* )

## G. Result Presentation
> [Demo vedio](https://youtu.be/0Me6T6cgl3g)

Using `v2.4.5` as the main version for analysis

### Score Distribution Charts
![image](https://hackmd.io/_uploads/B1CvANnNeg.png)
![image](https://hackmd.io/_uploads/rJT_RN2Vxg.png)

### Reward Distribution Charts
![image](https://hackmd.io/_uploads/HymoCE3Vex.png)
![image](https://hackmd.io/_uploads/HkUaRNnNxe.png)

Using `checkpoint_ep1800` from `v2.4.5` for analysis

```
Episode 1: total_reward=-147.77, score=724
Episode 2: total_reward=279.31, score=2522
Episode 3: total_reward=-11.90, score=2614
Episode 4: total_reward=3328.23, score=10002
Episode 5: total_reward=19.11, score=1526
Episode 6: total_reward=-204.31, score=516
Episode 7: total_reward=3232.96, score=10054
Episode 8: total_reward=-56.24, score=800
Episode 9: total_reward=-85.92, score=3346
Episode 10: total_reward=317.93, score=1950
```
Agent running results:
```
Best score: 10054, Best reward: 3328.228000000003
Score — Mean: 3405.40, Std: 3422.08
Reward — Mean: 667.14, Std: 1316.51
```

### Analysis and Discussion:
- Convergence speed: The model converges to the highest score around episode 1800, then starts to decline
- Failure cases: When asteroids appear quickly and block the original escape path, or when asteroids are too large with no escape route


## H. Development Notes

> The following records core modifications and training results for each version

### v2.1

- Modification: Used default image (raw pixel) as `state`, model architecture same as teaching assistant's example.
- Performance: Training failed to converge, scores remained at random levels.

### v2.2
- Modification:
    - Introduced basic game strategy, making reward highly correlated with actual scores
    - Added asteroid collision penalty
    - Reward parameters: `α_HIT=1.4`、`β_coll=1.2`、`time_penalty=−0.5`
- Performance: Total score significantly improved, reaching around 1400 points but unable to improve further.

### v2.3
- Modification: To address slow CNN training and slow convergence, switched to MLP and directly input `np.array`
Call `_extract_state()` in `step()` to extract environment information
- Performance: Training speed significantly increased, but highest score did not improve

### v2.4
- Modification:
    -    Readjusted reward weights: `α_HIT 1.4→1.8`、`β_coll 1.2→1.4`、`time_penalty −0.3→0.4`
    -    Changed asteroid radius to discrete input
    -    Optimized Replay Buffer performance
-    Performance: Training curve smoother, but highest score still did not break existing upper limit.
### v2.4.3
Modification:
Integrated TensorBoard for real-time monitoring of loss and reward curves
Learning rate adjustment: 1e-4→5e-5；ε decay: 0.999→0.997
All rewards multiplied by 0.6
Performance: Average score broke 2000 points, occasionally reaching 10000 points in single runs; but power-up collection strategy remained unstable.
### v2.4.4
- Modification:
    - Found that excessive reward differences affected learning, switched to segmented time_penalty:
    ```json
    score<1000：−0.25
    1000≤score<3000：−0.3
    3000≤score<6000：−0.35
    ```
    - Readjusted α_HIT 1.8→0.8、β_coll 1.4→1.2
    - Conducted A/B testing for health recovery power-ups: γ_shield=1.4 vs. 1；result: γ_shield=1.4 performed best

- Performance: Score growth trend became unstable after 3000 points, highest reached close to 4000 points, still unable to consistently break through

### v2.4.5 (Best Version)
- Modification:
    -    Reduced asteroid feature dimensions, no longer using one-hot for each asteroid, only input radius rr
    -    Added "wall-hit penalty" for incorrect dodging at screen edges
    -    Fixed time_penalty = −0.25

-    Performance: Significantly increased probability of single-run breakthrough of 10000 points, but low-score situations still occur occasionally, suspected to be inherent game difficulty limitation
### v2.4.6
- Modification: Expanded action space to 6 actions (allowing simultaneous movement and shooting)
- Performance: Converged to 2000–3000 points but unable to improve further
### v2.4.7
- Modification: Replaced original Replay Buffer with Prioritized Replay Buffer
- Performance: Score initially rose above 1000 points, then declined and fluctuated in the 1000–2000 point range

### Future Optimization Directions:
#### I. Environment and Performance Optimization

1. Remove FPS Limitation
Change `clock.tick()` to frame counter calculating accumulated frame count, allowing training to no longer be constrained by display frame rate, significantly improving data collection speed.

2. Parallel Environments
Adopt vectorized environment or multi-process execution to parallelly collect multiple training trajectories, reducing training time.

#### II. Hyperparameters and Exploration Strategy
3. Dynamic ε-greedy Strategy
Currently ε converges in the [0.2,0.1] range, after which score performance declines. Could try linear or adaptive dynamic decay of ε

4. Systematic Hyperparameter Search
Introduce Grid Search, Bayesian Optimization, Hyperband, etc., to automatically tune learning rate, batch_size, γ, gradient clipping and other parameters

#### III. Reward Mechanism Adjustment

5. Enhanced Dodging Ability
Current agent still encounters issues of getting stuck at edges or being blocked by high-speed asteroids. Could try:
- Change edge penalty to center reward, making agent prefer staying in the center

## I. Reflection
I found this final project both interesting and challenging. From designing what information to include in the "state", to figuring out how to design rewards reasonably, each time I modified hyperparameters I had to understand the underlying meaning, then look at graphs and game videos to guess which parts didn't meet expectations, and then design the next version—just like conducting experiments. However, too many parameters and excessively long single training runs made it difficult to control all variables simultaneously within limited time, and I couldn't quickly verify hypotheses in a short time.

Learned several useful tools during the process:
1. Git Branch Management: Since I needed to constantly sync versions across different computers and create new branches, I chose Git as the version control tool. Due to the need to continuously update versions, I became more familiar with gitflow, but by the later versions things got a bit messy, and I spent considerable effort organizing. Next time I'll need to more rigorously plan branch workflows and PR reviews.

2. Matplotlib / TensorBoard: 
Initially used Matplotlib to plot training curves, and had to rerun the plots multiple times each update to compare trends. Later switched to TensorBoard, which allows viewing loss and reward curves in real-time during training, and can switch between version records with one click, saving a lot of comparison time—a very convenient package.

## K. Reference:
1. [DQN Game Playing - Basic Concepts](https://ithelp.ithome.com.tw/m/articles/10305461)
2. [Playing Atari with Deep Reinforcement Learning](https://arxiv.org/pdf/1312.5602)
3. [Paper Reading 1 - Playing Atari with Deep Reinforcement Learning](https://blog.csdn.net/songrotek/article/details/50581011)