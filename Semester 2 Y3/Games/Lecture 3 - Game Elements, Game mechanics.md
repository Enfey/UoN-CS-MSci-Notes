## Schell's Elemental Tetrad
- A **game** is a **system** in which **players** engage in **artificial conflict**, defined by **rules**, that result in **quantifiable outcome**
- Four basic elements:
	- Mechanics, aesthetics, story, technology
- Mechanics describe the goal of the game; how players can and cannot try to achieve it, and what happens when they try
	- Supported by technology
	- Aesthetics that emphasise them clearly
	- A story that makes sense of them 
## Mechanics
- Core of what a game is interactively
	- **What remains** when all the aesthetics, technology, and story is stripped away
- **Types of mechanics**
	- **Physics**
		- Motion and force
	- **Internal economy**
		- Elements that are collected, consumed, or traded
	- **Progression mechanisms**
		- How a player moves through the game world
	- **Tactical maneuvering**
		- Placement in the world to gain advantage
	- **Social interaction**
		- Between players

### Mechanics - Space and time
- **Space**
	- **Discrete** - limited number of spaces one can occupy
	- **Continuous** - allowed for unlimited movement within a limited space
	- Have a number of dimensions
		- Mechanically 2D or 3D
	- Having bounded areas that may or may not be connected
- **Time**
	- Continuous or discrete - turn based
	- Clocks - competing something in certain time
	- Races - being faster.

### Mechanics - Objects, Attributes, and State
- Game spaces are filled with **objects**
	- What can be seen or manipulated
- Objects have attributes
	- E.g., pos, colour, speed, maintaining the state of the object, determine what can/cannot do in game
- Some attributes are secret
	- Known by the game, known by all players, known only by one player

### Mechanics - Actions, Rules, Consequences
- **Actions** - what can players do?
	- Basic actions
		- Move forward, jump, shoot, collect bounty
	- Strategic actions or *emergent* gameplay
		- Protect a bounty by defending it
		- Force an opponent into making an unwanted move by flanking
		- We ask; how many ways can players achieve their goals?
- **Rules** - constrain actions, leading to goals
	- May be intrinsic to the definition of the action
	- Define the consequences of the action a player takes
	- Deals with rewards and punishments


### Mechanics - Skill
- Require the player to exercise physical, mental, and social skills
	- **Physical skills**
		- Strength, dexterity, coordination, physical endurance
		- Controller manipulation(often tied to dexterity+coordination)
	- **Mental skill**
		- Memory, observation, puzzle solving
	- **Social skill**
		- Reading an opponent, fooling an opponent, coordinating with teammates
	- **Real vs virtual skill**
		- Sword fighting a virtual skill that the player is pretending to have, but gives a feeling of power.
- **Chance** leads to uncertainty and surprise
	- What game elements are stochastic
	- The player can therefore predict, aim to control, guesses, anticipate risk etc.

### Core Mechanics = Gameplay Gestalt
- Identify the **core mechanic** of a game
	- The essential play activity players perform again and again in a game, and why
	- The purposeful interaction of actions, objects, that occur the most frequently
	- Used to describe the **experience** of a game when placed in context
		- "It's what you do in a game"
			- Difficult to distinguish between core and secondary mechanics
			- We ask essentially, what the gameplay loop is, and what mechanic drives that
- A single action
	- A driving game = driving
	- Trivial pursuit = answering questions
- A compound activity
	- Quake = set of interrelated mechanics such as moving, aiming, firing, managing resources such as health, ammo, armour
	- Starcraft = resource management, wargame strategy, rapid M&K skill

### Core Mechanics
- What the player does within the system as permitted by the rules, repeated over and over to manifest as experience for players
- They are the mechanism by which players make choices and permit a meaningful experience 
	- "Free movement within a more rigid structure" - Play
- Arguably a few pure mechanics, many variations on theme/context
	- Slight variations of core mechanics provide a "new" game
		- Add time limits, multiple levels, invert the mouse, change slide distance, reduce velocity when jumping down hill if diff size character, etc
		- Same interactivity, but alters the experience for players.

### Systematically analysing mechanics
- Reverse-engineering mechanics from observable behaviour
	- Inferred without player commentary or design intent
- Segment observed gameplay into loops
	- One extraction run
	- A combat encounter with a boss
- Watch this twice
	- One without pausing
	- Once again, list only observable facts
		- Player approaches slowly
		- Enemy suddenly attacks
		- Player rolls repeatedly
- Identify **primitives**
	- List all distinct player actions observed e.g., move, attack, dodge, heal, retreat
	- Identify objects, attributes, states, e.g., enemies, weapons, health, stamina, alive, hostil, idle
- Apply **Schell's Taxonomy**
	- Write only what is supported by observation
	- Identify/infer rules
	- Space and time mechanics
		- Open or closed? Safe or hostile? Does distance matter?
		- "Actions take time and expose player during execution"
	- Action mechanics
		- Which actions are frequent? Which seem risk?
		- "Healing is intentional"
	- Look for cross-category consistency
		- Do multiple mechanics reinforce the same behaviour?
-  ![](Pasted%20image%2020260127191537.png)

### Design philosophies - Ludology
- Focus on gameplay, rules
	- What the player can and cannot do, and the behavioural consequences of their actions, focusing on player agency
	- Goal-directed activity conducted within a framework of agreed rules
	- Design challenges:
		- Lack of player motivation
		- Hard to distinguish game from others with the same mechanics
- Action games: shoot while being hit, strafe to hiding spot, heal, repeat
- Learning a gameplay gestalt, or making progress by learning how to interact with the game
	- Playing the game = performing the gestalt
		- A pattern of repetitive perceptual, cognitive, and motor operations
	- Not necessarily designed into the system of a game
		- Emergent from the rules and the design.
### Design philosophies - Simulation
- Representation of the function, operation, of features of one process/system through another
	- Often involve the player configuring the simulation as a gameplay device
- Design challenges
	- No obvious end-state
	- Continue playing after all enemies are defeated
	- Driven forward by repetitive actions without an obvious goal
### Design philosophies - Narratology
- Games are a story medium
	- Focus on traditional narrative structure with emotionally compelling, strong artistic vision
- Design challenges
	- Author voice over player voice
		- Agency vs empathy
	- Poorly defined mechanics
		- Ludonarrative dissonance - the presented character is at odds with the player's actions
	- Gameplay may end up having little bearing on the story being told. 