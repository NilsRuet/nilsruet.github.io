---
title: "Highway Pursuit RL"
summary: "An RL agent that plays the game _Highway Pursuit_"
tags: ["Reinforcement Learning", "PyTorch", "PPO", "Stable-Baselines3", "C++", "Reverse engineering"]
draft: false
---
## Introduction

[GitHub](https://github.com/NilsRuet/highway-pursuit-Gym)

TL;DR: I turned a 2000s arcade-style racing game into a Gym environment by injecting custom hooks via a DLL and sharing pixel buffers with Python using shared memory. I then trained reinforcement learning agents from pixels using PPO and other algorithms to survive and destroy enemies. This project combined reverse engineering, C++/Python interprocess communication, and deep reinforcement learning.

### What is Hyghway Pursuit?
[Highway pursuit by Adam Dawes](https://adamdawes.com/games/highway-pursuit.html) is an arcade-like game where the player drives a car along a highway from a top-down perspective and gains score as the traveled distance increases. The road is filled with civilian and ennemy vehicles, both of which may collide and damage the player. Destroying civilian vehicles temporarily prevents score gains, while destroying ennemies rewards the player with a flat score increase.

<!-- Maybe one or two images of the game? And a gif of the agent playing ? -->

### Motivations
My main motivation for this project was to explore behaviours learned through Reinforcement Learning on an existing, human-playable problem, rather than on made-for-RL tasks.
Games offer ideal settings for this experimentation: they're fun, challenging, and provide intuitive ways to interpret and evaluate what an agent has learned.
I was also technically interested in adapting an existing, non-RL-compliant application into a real-time Gymnasium-compatible environment.

### Why _Highway Pursuit_ ?
One of the key motivation was nostalgia, _Highway Pursuit_ being a game I fondly remember and wanted to visit from a technical perspective. In addition, this game has several properties that make it well-suited for RL:
- The player's score provide a dense and straightforward reward signal
- The game's dynamics are relatively simple
- Designed for early-2000s hardware, it computationally lightweight and can be significantly accelerated on modern machines.

## Adapting the game to Reinforcement Learning
In this section, I outline the technical challenges and choices involved in turning the _Highway Pursuit_ into a Gym-compatible environment.

### Specificities of the environment
Creating a custom Gym environment requires defining the observation space, action space, and reward function.
- For technical simplicity (and to avoid cherry-picking internal game variables) I decided to use raw pixels as observations. While this simplifies the implementation, it also introduces challenges: meaningful features are harder to learn, and the environment becomes partially-observable as not everything happening in the game is visible on screen. 
- The action space is a simple multi-binary space that that directly maps the keyboard controls.
- The in-game score serves as the reward function. It doesn't require shaping, since it's already dense: the score increases as the player drives forward or destroys ennemies, both of which are fairly common occurences (even using random actions).

### Architecture
The project consists of two main components:
- A Python package that provides a Gym-compliant environment and forwards instructions such as `reset` or `step` to the game
- A game-side server, injected into the game process, which synchronizes the game and executes the received instructions

The top priority guiding technical decisions is data collection speed, i.e., making the environment as efficient as possible.
To achieve this, the server is injected into the game process using a DLL and controls the game via function hooks.
Communication between the Python package and the server also needs to support high data throughput (particularly for transmitting raw pixels) so shared memory is used for interprocess communication, enabling fast and efficient data transfer. 

<!-- Todo: figure, can also mention semaphores -->

### Technical Details

One of the most important modifications made to the game is enabling accelerated gameplay. The simplest and most effective solution implemented here is to hook time-providing functions and replace their return values with fully controlled custom variables. The game uses the Windows Performance Counter to track elapsed time, which means that hooking `QueryPerformanceFrequency` and `QueryPerformanceCounter` is sufficient to implement custom time control.  
For example, the performance counter is incremented by 1/60th of a second each time the environment’s `step` function is called, effectively advancing the game by one frame per step.

Another core modification involves synchronizing the game’s main loop with the server. This is achieved using a pair of semaphores, which ensures that the game progresses by exactly one frame for each instruction sent by the server.

Efficiently accessing raw pixel data is also a significant technical challenge. The game runs on _Direct3D 8_ (D3D8), and the most efficient method to retrieve the rendering buffer is to call D3D8 functions and copy the buffer into shared memory.  
Accessing D3D8 functions also enables the use of custom settings, which is particularly useful for lowering the rendering resolution. This improves game performance and speeds up data transmission.  
It also allows the game to run in windowed mode and in the background, enabling multiple environments to operate in parallel on a single machine. Optionally, rendering can be disabled entirely, although this results in only a modest performance gain.

Finally, several internal game functions are hooked to support both data collection (e.g., accessing the reward signal) and utility purposes (e.g., skipping the intro, starting a new game, or respawning the player).


### Main challenges
Turning the game into a reliable RL environment involved overcoming several non-trivial challenges:

- Locating and reverse-engineering internal game functions, which required understanding how the game handles input, logic, and scoring.
- Interfacing with Direct3D 8 to efficiently retrieve raw pixel data.
- _Direct3D 8_ is an outdated framework that only runs on Windows 10 and 11 via compatibility wrappers:
  - This makes D3D8 functions non-static, so I had to dynamically resolve vtables to locate function pointers without depending on a specific wrapper implementation.
  - It also introduces instability when using system-level hooks like `QueryPerformanceCounter`, as some wrapper versions use the Performance Counter internally during D3D initialization.
- Debugging very low-level code like game function hooks is non-trivial. It requires a clear understanding of existing assembly code, how custom code is injected, and what might cause crashes (which often occur without any indication of what went wrong).

### Takeaways
Beyond the technical implementation, this project was a great opportunity to gain hands-on experience in multiple low-level systems areas.
- Gained hands-on experience with reverse engineering, inter-process communication, and synchronization mechanisms.
- Despite its age, working with D3D8 offered valuable insight into the inner workings of rendering engines.
- Gained practical experience in building a custom Gym environment from scratch, including observation, action, and reward design, as well as performance optimization.


## Training an agent
### Preprocessing
- frame stacking 
- lower resolution. 240x320 RGB images, chose to downsample each layer to keep some color information (important for vehicle color), knowing the "lateral" information is not entirely lost since ever relevant visual element is more than 3px wide.

### Algorithms
- Primarly focused on PPO, custom implementation by weight & biases mostly tweaked for debugging performance and profiling
- Experimented with other algorithms from SB3 like DQN, A2C, PPO+RNN. The learned behaviours and performance did not substantially change, either due to limited training times, pre-processing/architectural bottlenecks outweighing the algorithm choice, or the environment being deceptively difficult.

### Results & agent behaviour
- Agent can learn to survive and destroy ennemies
- Learns that comboing kills increases rewards multiplicatively
- With some setting sometimes learn to gain health back by slowly driving at the side of the road, where it is unlikely to get rear-ended by other entites
- Fails to explore and generally drives slowly, never getting past the first biome/are the game has to offer
- Struggles with rear-ending in general
- TODO: show some gifs of the trained agent

### Training takeaways
- Also learned about how PPO is implemented
- Hands-on experience with the deep RL training workflow ; preprocessing, network architecture, hyperparameter tuning, performance monitoring and model saving.
- Pre-processing and overall "environment" decisions are more important to optimize first than hyperparameters or the choice of an algorithm. Notably:
  - Size of the observations and normalizing them
  - Normalizing the rewards
  - Stacking enough frames with appropriate spacing
  - Some hyperparameters like the discount factor and learning-rate annealing can still be important
- Even with a few hundred steps per second, this env is quite slow (especially since training on pixels), faster environments are required to realistically compare algorithms, hyperparameters etc. on lower scale hardware / distributed computing and several training nodes are requires for complexe environments using today's techniques.

### Challenges
- Slow training makes it difficult to compare configurations and hyperparameters with modest hardware
- Exploration problem, trouble with making the agent develop behaviours other than the basic "drive slowly and shoot ennemis on sight".

## Future work and areas of improvement
- After basic profiling, most of the time is spent in the game. Future idea is to parallelize the training loop and data collection loop? Using offline algorithms?
- Since the env is partially observable, and current setup doesn't capture long-term dynamics very well, worth looking into recurrent architectures?

