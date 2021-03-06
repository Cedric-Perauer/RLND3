## Algorithm 

### General 

The method uses a DDPG algorithm. DDPG is an off-policy actor critic method for learning continious actions. In order to encourage
exploration, Ornstein-Uhlenbeck noise is added to the action. 
The actor is implemented as a target network, due to target networks decorrelating the process and generally improving performance. Soft updates are used in DDPG, which means 
that at every update step, the local networks are updated by a certain amount of the weights of the target networks according to a hyperparameter TAU (see below). 

Since this is a multi agents learning problem, every single agent has its own replay buffer that only the agent can read from and write to. 

### Loss  

The critic loss is analogous to a DQN loss and is simply the mean squared error between the targets of the current state (calculated by the target network) and the 
output of the local network : 

```python 
critic_loss = F.mse_loss(Q_expected, Q_targets)
```


The actor is updated by backpropagating the error that is evaluated by calculating the average of q-values for each state/action pair as output by the critic network. 

```python 
actor_loss = -self.critic_local(states, actions_pred).mean()
```

### Networks 

Both the actor and critic networks are very simple feed forward networks with 2 hidden layers with ReLu activations of size 400 and 300 respectively. Batch Norm is used after the first layer. 
Since the critic outputs a Q-value, it does not have an activation function in the final layer. Compared to that, the actor uses a tanh function at the output to output a continious action in the range of [-1,1] for each of the actions. 

## Hyperparameters 

```python 
BUFFER_SIZE = int(1e6) # replay buffer size
BATCH_SIZE = 256       # minibatch size
EPSILON = 1.0         # for epsilon in the noise process (act step)
EPSILON_DECAY = 1e-6

GAMMA = 0.99           # discount factor
TAU = 2e-3             # for soft update of target parameters
LR_ACTOR = 1e-3        # learning rate of the actor
LR_CRITIC = 1e-3       # learning rate of the critic
WEIGHT_DECAY = 0       # L2 weight decay
LEARN_EVERY = 1       # learning timestep interval
LEARN_NUM = 10            # number of learning passes
GRAD_CLIPPING = 1.0         # gradient clipping 
```
Ornstein-Uhlenbeck parameters : 

```python 
mu = 0.0
OU_SIGMA = 0.1
OU_THETA = 0.15
```


## Results

The Algorithm achieves the required average reward of at least 0.5 over the last 100 episodes in episode 261.
A plot of the reward over the episodes is provided below. 

<img src="graph.png" alt="drawing" width="600"/>

## Possible Improvements 

- Hyperaparameter Tuning 
- Prioritized Experience Replay
- [GAE](https://arxiv.org/abs/1506.02438) : combats the bias/variance tradeoff of TD estimates by introducing a hyperparameter lambda that allow to combine monte-carlo and TD estimates
- [Q-PROP](https://arxiv.org/abs/1611.02247): uses a Taylor Expansion to control the bias/variance tradeoff in off-policy methods 
- The batch algorithms TNPG and TRPO (TRPO enables faster learning through larger batch sizes) as suggested by the paper [Benchmarking Deep Reinforcement Learning for Continuous Control](https://arxiv.org/abs/1604.06778)
- Recurrent versions of the used method as suggested by [Benchmarking Deep Reinforcement Learning for Continuous Control](https://arxiv.org/abs/1604.06778), but that may make it more difficult to train 
