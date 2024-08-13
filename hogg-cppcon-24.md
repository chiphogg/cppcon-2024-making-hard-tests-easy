# (Title image placeholder)

Notes:

- Welcome!
- Two kinds of audience
  - People testing functions w/complicated inputs
    - use concrete example
  - Motion planning for self driving
    - Lucky!  Blueprint in design space
- Mention generic principles at the end

---

## Disclaimer

Notes:

- May not be exactly what you would find if you went to Aurora today
  - Interfaces evolve
  - Changed some terms
  - Some features are incomplete
- The point of the talk is to communicate design principles

---

# Tests

Notes:

- Quickly orient on kinds of tests
- Today's core problem: complicated inputs

---

## Testing Pyramid

Notes:

- Tests: on a spectrum
  - Unit: fine-grained, fast, isolated
  - Integration: bring components together; a little slower
  - End-to-end: entire system; slowest
    - Self driving e2e examples: sim, track, on-road
- Also: low means easy to fix, tight iteration cycles

---

## Arrange, Act, Assert

Notes:

- Unit/integration tests: 3 phases
  - Arrange: set things up
  - Act: the thing you're testing
  - Assert: did it do the right thing?
- Today's problem is in Arrange

---

## When inputs are complicated

Notes:

- Real world functions can have complicated inputs.  Examples:
  - Point cloud from one lidar sweep
  - Lane graph
  - Vehicle motion history
- Two bad choices
  - Realistic data (hard to set up / understand)
  - Simple fake data (doesn't test much)
- How to solve?
  - Tolstoy: "all simple function inputs are alike; each complicated input is complicated in its own way"
  - Pick _one_ complicated input, study, and generalize

---

# Motion Planning Crash Course

Notes:

- Learn just enough about Motion Planning to understand the example

---

## Autonomy stack

Notes:

- Black box: sensor data in, Gas/brake/steering commands out
- Repeat/re-plan multiple times per second
- Refine: well-defined components
  - Communicate via message passing, pub/sub
  - e.g., perception, control

---

## Motion Planner

Notes:

- Inputs:
  - Map (what's the world?)
  - Localization (where am I?)
  - Goal / Route (where am I going?)
  - Actors (who else is out there?)
- Output: trajectory
- Inner architecture
  - World builder
  - Plan generator
  - Plan chooser
  - Could have tests for any of these
- Cycle: the unit of integration test
  - Inputs: really complicated! (map, actors)
  - How to describe actor positions, in source code?
    - x/y/z/theta?  No, forces reader to make a grid
    - Relative to map?  No, map is for planner, not humans
  - In both cases: tons of boilerplate, obscures intent
  - Not even enough!  Need to describe history of motion

---

## Tactic: imagine source code

TODO(1): C++ source code
TODO(2): Incrementally reveal

Notes:

- First, make the test easy.  (Warning: this may be hard!)  Then, write the easy test.
  - _If the test were easy to write, what would it look like?_
  - Let your imagination run wild!
  - Stop, look at what you wrote, think about interfaces.  Could they work?  How?
- A motion planning test **describes a scene**
- First insight: map is foundational
  - Localization, goal, actors --- all relative to map
- Second insight: get all poses from _paths in scene_
  - physically meaningful
  - easy to write, easy to read

---

## Levels of solution

Notes:

- Level 1: paths, poses, motions
- Level 2: map abstraction
- Level 3: scene builder
- Level 0?  Any guesses?

---

## Level 0: units library!

Notes:

- Built for _these libraries_!
  - ...but brought in other stakeholders
- CppCon 2021: shared what we learned
  - No better library available
- CppCon 2023: we open sourced it!
  - Still the best pre-C++20 units library today!

---

# Level 1: Poses, Paths, and Motions

---

## `Pose3D`

Notes:

- "Pose" means "where": position plus orientation
- Need _readable related poses_
  - `.turn_left()`
  - `.move_forward()`
- You can drive this little thing all around!
- **Point:** easy to _write_, easy to _read_.

---

## Poses come from paths

Notes:

- Ironclad rule: get **poses** from **meaningful paths in scene**
  - No x/y/z/theta.  Ever!  I will change it when you're not looking
- Path object, `Ribbon`: map along-path position onto pose
  - "Road-optimized": more than just poses
  - Consider curved parallel paths

---

## `Ribbon`: road-optimized path

Notes:

- Same solution as real world: "mile markers"
  - Index path with "position": affine space type
    - Like `time_point` from `std::chrono`
    - Separate type helps distinguish points, from distances
    - "Mile marker math"
- `Ribbon` extra responsibilities:
  - Map between "position differences" and physical distances
    - `.advance()`
    - `.geodesic_displacement()`
  - Make laterally offset **aligned** paths

---

## Aside: interface benefits

Notes:

- Well chosen interface is force multiplier!
  - New kinds of `Ribbon`
    - `ConstantCurvatureRibbon`
    - `ConcatenatedRibbon`
    - `SplineRibbon`
  - New use cases: we can build all kinds of things!
    - Actor trajectory
    - Synthetic maps
    - Anything with a path
  - Implementation becomes extremely easy (`make_curved_path`, `make_straight_path` examples)
  - Every new implementation makes every use case richer

---

## Aside (2): lightweight value types

Notes:

- Can hide the polymorphism

---

## `Motion`: speed profiles

Notes:

- `Motion` maps time onto along-path displacement and its derivatives
  - `MotionSnapshot`: displacement, velocity, acceleration
- Implicitly constructible from speed
  - Covers 95% of use cases
  - Interface should take a motion; you can pass `20 * m / s`, `65 * MPH`, etc.
- Easy to make others for specific use cases
  - `braking_from(...)` example
- Caveat: "constant acceleration"
  - Easy to implement, but nonsense at far `dt` values
  - Much rather be reasonable when queried at any time

---

## `Ribbon` and `Motion` compose

Notes:

- (Explain mapping)
- A scene described in terms of `Ribbon` and `Motion` is not static!
  - Can query at near-past and near-future times!
  - History; predictions

---

# Level 2: Map Abstractions

Notes:

- A map is where you get paths that have meaning

---

## Two Basins in Design Space

Notes:

- Map Sketch: fully synthetic
- Backdrops: bundle nice paths on top of a real world map

---

## Map Sketch

Notes:

- Road sketcher: readable synthetic roads
  - Number of lanes is number of times you say "finish lane"
  - Add a shoulder
  - Define the road frame
  - Set lane change availability
- _Road sketch_ is **not a road**
  - It's a high level **description** of a road, for **human test authors**
  - For _full usage_, need something that can make a map that is _consistent with_ this description
  - Could still use this for medium-complexity needs

---

## Backdrops

Notes:

- Backdrop: not a scene, but the "backdrop" where a scene _takes place_
  - Contains a real map
  - Also contains well named paths (left boundary of right lane, etc.)
  - Has its own unit tests to make sure paths match up with map data

---

## Comparison

Notes:

- Using map
  - Easy for both
- Making map
  - Easy for sketch, hard for backdrop
- Fidelity
  - Great for backdrop, poor for sketch unless you have synthetic map capabilities
- Ease of getting up and running
  - Backdrop is easier

---

# Level 3: Scene Builder

Notes:

---

## Core scene data

Notes:

- Absolutely minimal information needed: map, goal, path
  - These are easy to set from the backdrop
  - Everything else: use setters
- What to _do_ with it?  run-cycle-through-X

---

## Ego vehicle data

Notes:

- "Ego vehicle": that's us!
- `.set_vehicle_type()`: variant of `Passcar`, `TractorTrailer`
  - Can configure further, e.g., `.trailer_loading_percent = percent(40)`
- `.set_motion()`: vehicle speed (or speed profile)
  - Arguably should be part of core scene

---

## Actors

Notes:

- Our actors in unit tests are simply boxes
  - Gives a lot of value
  - Set length, width, height
- Construct with their path (or a pose, for a straight path)
- `motion`: speed profile
- actor "category": vehicle, pedestrian, etc.

---

## Interlude: Visualization

Notes:

- Source code is as readable as possible... but it's not enough!
- Must make it very easy to see the actual, built scene
- "Fail and visualize to": send data to S3 that a web tool can read
  - Interactive 3D visualizations!
    - That's where these screenshots come from
  - Why "fail"?  So we don't land test that tries-and-fails to send data to S3

---

## Construction and blockages

Notes:

- How to _describe_ a construction blockage?
  - Physically: bunch of barrels
  - **Logically:** a **boundary**
    - Hey, we have paths!
  - Strategy: describe the boundary
    - Pick a "main path" from the scene
    - Add checkpoints of _along-path positions_
    - Can add _lateral offsets_
  - Implementation: drop barrels whose edges fall along this path
    - If you block the left side, the barrels are **on** the left side
- Example
  - We're in the left lane, two lane road, position zero
  - Block _left side_ along _right boundary_
  - Start 20 meters up, moved left by the lane width
  - Checkpoint at 50 meters, no lateral displacement
  - Ends at 150 meters
- What did you picture?  Was it something like this?
  - Note that tapering is very important

---

## Value of intent-first interfaces

Notes:

- We just saw a readable, flexible way to _place physical barrels_ in our scene, to describe construction.
- How does the planner _reason about construction?_
  - This is all about interfaces evolving over time!
- Consider a simple categorical implementation, maybe for a first pass
  - Divide into regions "unblocked", "partially blocked" (we can still fit), "fully blocked"
  - OK to get started
- Consider refined implementation:
  - Width as function of position
  - Much richer... but a big change

---

## Blockage test: categorical implementation

Notes:

- Here's source code for a test when the planner treats construction categorically

---

## Blockage test: refined implementation

Notes:

- Here's that test's source code after the planner gets refactored to treat it _quantitatively_
- Can you spot the differences?
  - Two: slide title, and slide number
  - When you write tests in **intent based** way, using **physical features**:
    - Aaaalll those tests don't need to change when you refactor!

---

# Scene builder implementation strategy

Notes:

- Everything so far is "what will users write"
- But... why does this work?

---

## Planner is based on messages

Notes:

- Recall: planner gets input messages (map, route, actors, so on)
- Scene builder builds a **high level description** of a scene
- Decomposes perfectly:
  - For each input type, ask: what messages would I expect to see if I were in this situation?
  - Remember!  Can query scene at near-past times!
    - So for actors, it's easy to get whole _history_ of messages

---

## `SceneMessages`: a container of inputs

Notes:

- Planner inputs represented by tag types
  - We have _canonical instances_ of these types, ready-made values
- For each input: a sequence of timestamped messages
- Can index into container with tag types
  - Takes care of awkwardness around saying, e.g., std-get-vector-timestamped-message-T
- Also more robust when actually running the planner
  - Can't "forget" to populate messages in a topic

---

## `InputSampler`: filling up `SceneMessages`

Notes:

- Two separate questions for each input type
  - What times should I see these messages?
  - What _contents_ should a message have, at a _given time_?
- This separation is key to testing extreme situations

---

## Input Tweaks

Notes:

- An "input tweak" combines a particular input, and some change
  - "drop all" means we never got any messages
  - "set latest age" shifts all timestamps, so we can simulate staleness
  - "mutate last" means do anything to the last message
- Key: **first** choose times, **then** generate message contents
  - e.g., "set latest age" for actors to 1 second (yikes!):
    - you get actor positions from _one second ago_, not current positions with a timestamp field tweaked
  - effortlessly consistent!
  - great for testing fault conditions

---

other stuffs

- problems, thinking ahead
  - ribbon: position -> pose.  Assumes self-consistency
    - Make generic "ribbon test harness", useful for unit tests of new ribbon implementations
  - actor history: assumes no collisions with each other or ego
    - Can imagine adding a collision check step
    - _Will not impact public interface!_
    - Don't need to add this in the MVP
  - dynamic feasibility
    - "path plus motion" is naive, just ask a controls engineer
    - Example: two lane road.  Can make a U-turn path --- but can a semi _drive it?_
    - Dynamic feasibility checks for _ego_ vehicle could explode the whole scheme!
    - Solution: trajectory fitter.  Path and motion are inputs... **and outputs**!
      - Throw exception if it changes too much, because test is badly written
      - **Does not change the public interface**
    - Takeaway:
      - **Not** that you need to do this right away
      - Just: make sure you're not coding yourself into a corner
