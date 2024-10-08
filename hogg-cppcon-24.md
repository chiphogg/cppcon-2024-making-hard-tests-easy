<section data-background="./Making Hard Tests Easy-_TitleCard copy.png" data-background-size="contain">

Notes:

- Welcome!
- Two kinds of audience
  - People testing functions w/complicated inputs
    - deep dive on one concrete example
    - try to extract general principles
  - Motion planning for self driving
    - Lucky!  Clear, specific paths through design space

---

## Aurora and Me

<div class="r-stack">
<img src="./figures/aurora-truck-car.jpg">
<div class="fragment" style="position: absolute; width: 100%; text-align: center;">
<img src="./figures/sqb.png" style="width: 60%;">
</div>
</div>

Notes:

- Aurora
- deliver benefits of self driving, safely, quickly, broadly
  - launching first product _this year_: self-driving trucks, Dallas to Houston
  - Me: Chip Hogg
    - Almost 4 years at Aurora, on motion planning team, almost 9 in self driving
    - Main contribution: making good tests easy
  - Disclaimer: APIs not exact
    - Goal: whatever communicates **ideas** most clearly

---

# Tests

Notes:

- Let's talk about tests generally.

---

## Testing Pyramid

<div class="container">

<div class="r-stack">
<img src="./figures/testing_pyramid/pyramid.svg">
<img class="fragment fade-in" src="./figures/testing_pyramid/pyramid_focus.svg">
</div>
<div class="r-stack nolinenum">
<div class="fragment fade-in">

```cpp
TEST(Florpinate, ProducesGoodStuff) {





















}
```

</div>
<div class="fragment fade-in">

```cpp [2-8]
TEST(Florpinate, ProducesGoodStuff) {
  //
  // ARRANGE
  //
  // Create the inputs you'll need
  //
  const auto a = make_input_one();
  const auto b = make_input_two();














}
```

</div>
<div class="fragment fade-in">

```cpp [10-15]
TEST(Florpinate, ProducesGoodStuff) {
  //
  // ARRANGE
  //
  // Create the inputs you'll need
  //
  const auto a = make_input_one();
  const auto b = make_input_two();

  //
  // ACT
  //
  // Perform the action under test
  //
  const auto result = florpinate(a, b);







}
```

</div>
<div class="fragment fade-in">

```cpp [17-22]
TEST(Florpinate, ProducesGoodStuff) {
  //
  // ARRANGE
  //
  // Create the inputs you'll need
  //
  const auto a = make_input_one();
  const auto b = make_input_two();

  //
  // ACT
  //
  // Perform the action under test
  //
  const auto result = florpinate(a, b);

  //
  // ASSERT
  //
  // Check the properties of what you made
  //
  EXPECT_THAT(result.stuff(), IsGood());
}
```

</div>
<div class="fragment fade-in">

```cpp [2-8]
TEST(Florpinate, ProducesGoodStuff) {
  //
  // ARRANGE
  //
  // Create the inputs you'll need
  //
  const auto a = make_input_one();
  const auto b = make_input_two();

  //
  // ACT
  //
  // Perform the action under test
  //
  const auto result = florpinate(a, b);

  //
  // ASSERT
  //
  // Check the properties of what you made
  //
  EXPECT_THAT(result.stuff(), IsGood());
}
```

</div>
</div>
</div>

Notes:

- Tests: on a spectrum
  - ranging from
    - e2e tests, whole system
    - integration tests
    - unit tests
    - even compiler errors, line/sub-line level
  - top: bigger scope, but slower/more expensive
  - bottom: fine grained, fast, cheap, easy to understand
  - need **all** --- complementary; defense-in-depth
- We're here in the middle: from big unit tests to integration tests
  - Or: "things you write a C++ test case for"

- Unit/integration tests: 3 phases
- Arrange: set things up
- Act: the thing you're testing
- Assert: did it do the right thing?
- Today's problem is mostly in Arrange

---

## When inputs are complicated

```cpp
Trajectory predict_motion(const Trajectory &input, DurationD dt_prediction);
```

<div class="fragment">
<img src="./figures/lane-change-trajectory.png" style="width: 55%;">

Example: vehicle motion history

- Changing lanes
- Accelerating

</div>

Notes:

- Real world functions can have complicated inputs.
  - Example: predict vehicle motion from history
- Input: changing lanes, accelerating
  - Two bad choices
    - Realistic data (hard to set up / understand; fragile)
    - Simple fake data (doesn't test much)
  - How to solve?
    - According to Tolstoy:
      - "all simple function inputs are alike; each complicated input is complicated in its own way"
    - Pick _one_ complicated input, study, and generalize

---

# Motion Planning<br>Crash Course

Notes:

Target: **04:15**

- Example: motion planning component, self driving
  - don't know self driving?  No problem!
  - Learn just enough so anyone can understand the example

---

## Autonomy stack

<div class="r-stack">
<img src="./figures/autonomy/autonomy_stack.svg" style="width: 100%;">
<img src="./figures/autonomy/full_stack_0.svg" style="width: 100%;" class="fragment fade-in">
<img src="./figures/autonomy/full_stack_1.svg" style="width: 100%;" class="fragment fade-in">
<img src="./figures/autonomy/full_stack_2.svg" style="width: 100%;" class="fragment fade-in">
<img src="./figures/autonomy/full_stack_3.svg" style="width: 100%;" class="fragment fade-in">
<img src="./figures/autonomy/full_stack_4.svg" style="width: 100%;" class="fragment fade-in">
</div>

Notes:

- Start with black box: (...)
  - Let's refine
  - Well defined components, communicate by messages
- Sensors tell us...
- fetch map...
- plan route...
- these feed in to **motion planner**
  - given situation, produce trajectory (path and speed)
- controls knows how...
  - Repeat/re-plan multiple times per second
  - How to implement **motion planner**?

---

## Motion Planner

<div class="r-stack">
<img src="./figures/mp_parts/stages_0.svg" style="width: 100%;">
<img class="fragment fade-in" src="./figures/mp_parts/stages_1.svg" style="width: 100%;">
<img class="fragment fade-in" src="./figures/mp_parts/stages_2.svg" style="width: 100%;">
<img class="fragment fade-in" src="./figures/mp_parts/stages_3.svg" style="width: 100%;">
<img class="fragment fade-in" src="./figures/mp_parts/stages_4.svg" style="width: 100%;">
</div>

Notes:

- Three stage architecture: extremely flexible
- Assess the situation
  - Assemble inputs into a coherent whole
- Come up with a bunch of ideas
  - Hundreds, thousands
- Pick the best one
  - Robust, flexible!
    - e.g., reserve emergency maneuvers
    - will do when better than anything else
  - Assess, propose, pick, repeat:
    - drive truck from A to B
    - also recipe for winning
  - From inputs to trajectory: **one cycle**
    - What are we testing?
      - Whole thing
      - Any stage
      - Any sub-stage
- Notice: if we had these inputs, easy to make any downstream complicated inputs
  - Can just run (parts of) planner
  - but... these are complicated!

---

## First, make the (tests) easy

<img src="./figures/kent-beck-tweet.png">

<div class="fragment">
<p>How to "make the (test) easy"?</p>
<ol>
  <li>Write imagined source code</li>
  <li>Think through implementation: feasibility, usability, scope, ...</li>
</ol>
</div>

Notes:

Target: **08:54**

- Kent Beck: "Make the change easy (warning: this may be hard), then make the easy change"
  - Cultivate this habit: feels like a cheat code
  - how to move **fast** with **confidence**
- So: **how** to "make the change easy"?
  - 1. Figure out what "easy" would even look like
    - If you can't, no point in going further
    - Write source code, imagine you have everything you could want
  - 2. Look at what you wrote, judge the design
    - Feasibility: can you see your way to implementing?
    - Usability: easy to use correctly, hard to use incorrectly?
    - Scope: Doesn't have to do everything!  What are core use cases, peripheral, out-of-scope...

---

## What would "easy" look like?

<div class="container">

<div>

- Two lane highway

</div>
<div>

- 65 MPH in right lane

</div>
<div>

- Car in front doing 50 MPH

</div>
</div>

<div class="r-stack easy nolinenum">

<div>

```cpp
TEST_F(MotionPlanner, PlansLaneChangeAroundSlowVehicle) {






















}
```

</div>
<div class="fragment fade-in">

```cpp [2-3]
TEST_F(MotionPlanner, PlansLaneChangeAroundSlowVehicle) {
    const auto map = two_lane_straight_highway();  // Revisit this API...
    const auto lane = right_lane(only_road(map));




















}
```

</div>
<div class="fragment fade-in">

```cpp [10-20]
TEST_F(MotionPlanner, PlansLaneChangeAroundSlowVehicle) {
    const auto map = two_lane_straight_highway();  // Revisit this API...
    const auto lane = right_lane(only_road(map));






    const auto plan =
        SceneBuilder{
            {
                .map = map,
                .goal = final_pose(lane),
                .ego_path = {nominal_path(lane), 0_m_pos},
                .ego_motion = 65 * MPH,
            },
        }

        .run_cycle_through(RANKER);



}
```

</div>
<div class="fragment fade-in">

```cpp [5-8,19]
TEST_F(MotionPlanner, PlansLaneChangeAroundSlowVehicle) {
    const auto map = two_lane_straight_highway();  // Revisit this API...
    const auto lane = right_lane(only_road(map));

    const auto slow_car =
        car_sketcher({nominal_path(lane), 40_m_pos})
            .set_motion(50 * MPH)
            .sketch();

    const auto plan =
        SceneBuilder{
            {
                .map = map,
                .goal = final_pose(lane),
                .ego_path = {nominal_path(lane), 0_m_pos},
                .ego_motion = 65 * MPH,
            },
        }
        .add_actor(slow_car)
        .run_cycle_through(RANKER);



}
```

</div>
<div class="fragment fade-in">

```cpp [22-23]
TEST_F(MotionPlanner, PlansLaneChangeAroundSlowVehicle) {
    const auto map = two_lane_straight_highway();  // Revisit this API...
    const auto lane = right_lane(only_road(map));

    const auto slow_car =
        car_sketcher({nominal_path(lane), 40_m_pos})
            .set_motion(50 * MPH)
            .sketch();

    const auto plan =
        SceneBuilder{
            {
                .map = map,
                .goal = final_pose(lane),
                .ego_path = {nominal_path(lane), 0_m_pos},
                .ego_motion = 65 * MPH,
            },
        }
        .add_actor(slow_car)
        .run_cycle_through(RANKER);

    // Out of scope for this talk...
    EXPECT_THAT(plan, ChangesLaneTo(left_lane(only_road(map))));
}
```

</div>
<div class="fragment fade-in-then-out">
  <video style="width: 80%;">
    <source src="./figures/vis3d.webm" type="video/webm" style="width: 50%">
  </video>
</div>
<div class="fragment fade-in">

```cpp
TEST_F(MotionPlanner, PlansLaneChangeAroundSlowVehicle) {
    const auto map = two_lane_straight_highway();  // Revisit this API...
    const auto lane = right_lane(only_road(map));

    const auto slow_car =
        car_sketcher({nominal_path(lane), 40_m_pos})
            .set_motion(50 * MPH)
            .sketch();

    const auto plan =
        SceneBuilder{
            {
                .map = map,
                .goal = final_pose(lane),
                .ego_path = {nominal_path(lane), 0_m_pos},
                .ego_motion = 65 * MPH,
            },
        }
        .add_actor(slow_car)
        .run_cycle_through(RANKER);

    // Out of scope for this talk...
    EXPECT_THAT(plan, ChangesLaneTo(left_lane(only_road(map))));
}
```

</div>
</div>

Notes:

- What's an example motion planner test case?
  - 2 ln hwy, 65 in R ln, car doing 50 in front, nobody else
  - Want planner to propose a lane change
- Get a map from somewhere (TBD)
  - Get the right lane
- Build the scene!
  - (read code)
- Slow car
  - (read code)
- Expect a lane change (somehow)
  - Readable: easy to picture
  - Stretch goal: interactive 3D vis
- (show vis)

---

## Levels of solution

<div class="container_60_40">
<div>

### Level 3: Scene Builder

</div>

<div>
<img style="width: 70%;" src="./figures/levels/level_3.png">
</div>
</div>

<div class="fragment container_60_40">
<div>

### Level 2: Map Abstractions

</div>
<div>
<img style="width: 70%;" src="./figures/levels/level_2.png">
</div>
</div>

<div class="fragment container_60_40">
<div>

### Level 1: Poses, Paths, and Motions

</div>
<div>
<img style="width: 70%;" src="./figures/levels/level_1.png">
</div>
</div>

<div class="fragment container_60_40">
<div>

### Level 0: ...

</div>
<div>

## 🤔

</div>
</div>

Notes:

- To build: **levels** of solution
  - Level 3: scene builder
- Level 2: map abstraction
- Level 1: paths, poses, motions
- Level 0?  Any guesses?

---

## Level 0: units library!

<div class="r-stack">
<div>
<img src="./figures/au.png">
<br>
<div class="repo">aurora-opensource/au</div>
</div>
<img style="width: 90%;" class="fragment" src="./figures/units/units_0.png">
<img style="width: 90%;" class="fragment" src="./figures/units/units_1.png">
<img style="width: 90%;" class="fragment" src="./figures/units/units_2.png">
</div>

Notes:

- Built for _these libraries_!
  - Imagined APIs without it (😬)
  - ...but brought in other stakeholders across Aurora
- CppCon 2021: shared what to look for
- _At the time:_ no **public** library had these properties
  - So we open sourced it
- Presented it at CppCon 2023
  - So: that's level 0...

---

# Level 1:<br>Poses, Paths, and Motions

Notes:

Target: **15:03**

- Level 1: first _true_ foundational level
  - Poses, paths, motions
  - Build everything else on top of

---

## `Pose3D`

<div class="container">

<div class="r-stack">
<div class="fragment fade-in-then-out" data-fragment-index="1">

```cpp




Pose3D pose = get_starting_pose();




```

</div>

<div class="fragment fade-in-then-out" data-fragment-index="2">

```cpp




Pose3D pose = get_starting_pose()
    .move_forward(20 * m);



```

</div>

<div class="fragment fade-in-then-out" data-fragment-index="3">

```cpp




Pose3D pose = get_starting_pose()
    .move_forward(20 * m)
    .turn_right();


```

</div>

<div class="fragment fade-in-then-out" data-fragment-index="4">

```cpp
const auto curvature =
    BodyFrameCurvature{}
        .set_yaw_left_rate(0.05 * rad / m);

Pose3D pose = get_starting_pose()
    .move_forward(20 * m)
    .turn_right()
    .move_forward(20 * m, curvature);
```

</div>
</div>

<div class="pose">

<h4 class="fragment fade-in" data-fragment-index="1">Top:</h4>
<div class="r-stack">
<img class="fragment fade-in" src="./figures/pose/pose_top_01.png" data-fragment-index="1">
<img class="fragment fade-in" src="./figures/pose/pose_top_02.png" data-fragment-index="2">
<img class="fragment fade-in" src="./figures/pose/pose_top_03.png" data-fragment-index="3">
<img class="fragment fade-in" src="./figures/pose/pose_top_04.png" data-fragment-index="4">
</div>

<h4 class="fragment fade-in" data-fragment-index="1">Perspective:</h4>
<div class="r-stack">
<img class="fragment fade-in" src="./figures/pose/pose_persp_01.png" data-fragment-index="1">
<img class="fragment fade-in" src="./figures/pose/pose_persp_02.png" data-fragment-index="2">
<img class="fragment fade-in" src="./figures/pose/pose_persp_03.png" data-fragment-index="3">
<img class="fragment fade-in" src="./figures/pose/pose_persp_04.png" data-fragment-index="4">
</div>

</div>

</div>

Notes:

- "Pose" means "where": position plus orientation
- **Job**: given _starting_ pose, easy + readable to make _related_ pose
  - strategy: chainable APIs
- Move fwd
- Turn, defaults to 90 degrees
- More complicated motion: constant curvature
  - Easy to understand how starting pose changes
  - ...where does that "starting pose" come from though?

---

## Poses come from paths

<div class="r-stack">
<img src="./figures/pose-from-path/path_01.png">
<img class="fragment fade-in" src="./figures/pose-from-path/path_02.png" data-fragment-index="1">
<img class="fragment fade-in" src="./figures/pose-from-path/path_03.png" data-fragment-index="2">
<img class="fragment fade-in" src="./figures/pose-from-path/path_04.png" data-fragment-index="3">
</div>

Notes:

- **Strong** preference: get pose from **meaningful paths in scene**
  - Avoid x/y/z/theta: fragile, hard to read
  - Here's a path: right boundary of right lane of road
- Give it a position, get a pose, move it to where you want
- Move right
- Turn left
  - Easy to see: "facing the boundary"


---

## Danger: curves ahead

<div class="r-stack">
<img src="./figures/dangerous-curves/path_01.png">
<img class="fragment fade-in" src="./figures/dangerous-curves/path_02.png" data-fragment-index="1">
<img class="fragment fade-in" src="./figures/dangerous-curves/path_03.png" data-fragment-index="2">
<img class="fragment fade-in" src="./figures/dangerous-curves/path_04.png" data-fragment-index="3">
</div>

Notes:

- For right boundary
- Gonna want **left** boundary
  - line them up: dilemma!
- Natural approach: positions get out of alignment
  - inside track more efficient
- Can force-align: but then position differences aren't accurate distances!
  - WDYT?  First approach?  Second?  Any strong opinions?

  - Good news: real world has this problem too
    - Can use same solution: mile markers
    - Positions aren't **exact measurements:** just **labels** for points
    - Subtracting _positions_ gives a _displacement_
      - still _approximately_ correct despite curvature
    - similar semantics to chrono `time_point`
      - Can't "add" two positions
  - Now we can appreciate our path type: `Ribbon`

---

## `Ribbon`: road-optimized path

<div class="r-stack">
<img src="./figures/ribbon-is-road-optimized/ribbon_01.png">
<img class="fragment fade-in" src="./figures/ribbon-is-road-optimized/ribbon_02.png" data-fragment-index="1">
<img class="fragment fade-in" src="./figures/ribbon-is-road-optimized/ribbon_03.png" data-fragment-index="2">
</div>

Notes:

- Mathematically, "Ribbon" = 3D path + surface orientation
  - pose-at-each-position
  - points along path
  - `Ribbon` is "road-optimized".  What does that mean?
    - Translates between "position differences" and "real physical displacements"
  - Let's take a point `p`, at the 10 meter position.
- Path can Advance by 10 meters
  - get 21.8 meter position (not just 20)
- Measure along-path displacement, with "geodesic displacement", get just 8.5 meters
  - Paths act as basic "reference frames" for scene
  - For actor: pair **path** with **position**
    - pairing so useful, it deserves its own type

---

## `RelativePath`: interface type for actors

(Including us!)


<div class="container">
<div>
<div>

#### `RelativePath` interfaces

</div>
<div class="r-stack nolinenum">
<div>

```cpp
class RelativePath {
 public:






  Pose3D pose_at_ds(DisplacementD ds) const;
};
```

</div>
<div class="fragment" data-fragment-index="2">

```cpp [3-4]
class RelativePath {
 public:
  explicit(false)
  RelativePath(Ribbon path, PositionD current);




  Pose3D pose_at_ds(DisplacementD ds) const;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="3">

```cpp [6-7]
class RelativePath {
 public:
  explicit(false)
  RelativePath(Ribbon path, PositionD current);

  explicit(false)
  RelativePath(Pose3D pose);

  Pose3D pose_at_ds(DisplacementD ds) const;
};
```

</div>
</div>
</div>
<div class="fragment" data-fragment-index="1">

#### Example API:

```cpp
void set_actor_path(RelativePath path);
```

<div class="fragment" data-fragment-index="2">
<div>

#### Example callsites:

</div>
<div class="r-stack nolinenum">
<div class="fragment fade-in-then-out" data-fragment-index="2">

```cpp
// Pass Ribbon-and-Position as a pair.
set_actor_path({nominal_path(lane), 40_m_pos});




```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="3">

```cpp [4-9]
// Pass Ribbon-and-Position as a pair.
set_actor_path({nominal_path(lane), 40_m_pos});

// Pass a pose; get a straight path through the pose!
set_actor_path(pose_facing_lane_boundary)
```

</div>
</div>
</div>
</div>

Notes:

- `RelativePath`:
  - Given signed along-path distance from where you started, where are you now?
- Designed to be **interface type**
  - Here's an example API
- Pass braced pair of path and position
  - Easy to understand what is meant
- Pass a pose
  - Get straight path
  - e.g., ped facing road
    - no pre-existing path there
    - make the pose, imply path

---

## `Motion`: speed profiles

$dt \rightarrow \left(ds, \frac{ds}{dt}, \frac{d^2s}{dt^2}\right)$

<div class="container">
<div class="r-stack nolinenum">
<div>

```cpp
class Motion {
 public:


  MotionSnapshot snapshot_at(DurationD dt) const;
};
```

</div>
<div class="fragment fade-in" data-fragment-index="2">

```cpp [5]
class Motion {
 public:


  MotionSnapshot snapshot_at(DurationD dt) const;
};
```

</div>
<div class="fragment fade-in" data-fragment-index="3">

```cpp [3]
class Motion {
 public:
  explicit(false) Motion(VelocityD constant_speed);

  MotionSnapshot snapshot_at(DurationD dt) const;
};
```

</div>
</div>

<div class="r-stack nolinenum">
<div class="fragment fade-in-then-out" data-fragment-index="2">

```cpp
struct MotionSnapshot {
  DisplacementD displacement = ZERO;
  VelocityD velocity         = ZERO;
  AccelerationD acceleration = ZERO;
};


```

</div>
<div class="fragment fade-in" data-fragment-index="3">

```cpp [6]
struct MotionSnapshot {
  DisplacementD displacement = ZERO;
  VelocityD velocity         = ZERO;
  AccelerationD acceleration = ZERO;
};


```

</div>
</div>
</div>

Notes:

- `Motion` maps elapsed time onto along-path displacement and its derivatives
  - velocity, accel
  - What's "motion snapshot"?
- Struct of quantities
- Implicitly constructible from speed
  - Motion is an **interface type**
  - Tempted to write constant-speed-only interface?
    - Use motion instead
    - Still pass any speed: 20 m/s, 65 MPH, etc.
  - Other polymorphic constructors not shown here
  - Later on, more complicated motions like braking:
    - Fluent readable builder API
    - Add functionality with no extra cost

---

## `RelativePath` and `Motion` compose

`Motion`: $\ \ \ dt \rightarrow \left(ds, \frac{ds}{dt}, \frac{d^2s}{dt^2}\right)$

`RelativePath`: $\ \ \ ds \rightarrow \left(\text{pose}, \text{curvature}\right)$

<div class="fragment fade-in">

```cpp
const auto [ds, v, a] = motion.snapshot_at(dt);
const auto [pose, curvature] = path.point_at_ds(ds);
```

</div>

Notes:

- (Explain mapping)
- In code...
  - Note: we're using the fuller, curvature API here
  - A scene described in terms of `Ribbon` and `Motion` is not static!
    - Can query at near-past and near-future times!
    - History; predictions
    - Motion is **self-consistent**:
      - You get a pose and velocity
      - Pose a short time later is what velocity would predict

---

## Aside: hidden polymorphism

<div class="container">
<div>

```cpp
class RibbonImplementation {
 public:
  // Map: position -> pose
  virtual Pose3D pose_at(PositionD s) const = 0;

  // Advance by a physical distance.
  virtual PositionD advance(
    PositionD s, DisplacementD distance) const = 0;

  // Measure physical along-path distance.
  virtual DisplacementD geodesic_displacement(
    PositionD start, PositionD end) const = 0;
};
```

</div>
<div class="r-stack">
<div class="fragment fade-in" data-fragment-index="1">

```cpp
class Ribbon {















 private:
  std::shared_ptr<const RibbonImplementation> impl_;
};
```

</div>
<div class="fragment fade-in" data-fragment-index="2">

```cpp
class Ribbon {
 public:
    Pose3D pose_at(PositionD s) const {
      return impl_->pose_at(s);
    }

    PositionD advance(
        PositionD s, DisplacementD d) const {
      return impl_->advance(s, d);
    }

    DisplacementD geodesic_displacement(
        PositionD start, PositionD end) const {
      return impl_->geodesic_displacement(start, end);
    }

 private:
  std::shared_ptr<const RibbonImplementation> impl_;
};
```

</div>
</div>
</div>

<!--
<div class="fragment fade-in" data-fragment-index="3">

```cpp
Ribbon get_flat_ribbon_turning_left(CurvatureD k, Pose3D origin_pose) {
    // The origin pose is really arbitrary, as long as its orientation has no pitch or roll.
    return make_ribbon<ConstantLeftTurnRateRibbonFrom>(origin_pose, k);
}
```

</div>
-->

Notes:

- Ribbon, etc. use polymorphism
  - End users don't see it
  - Interface class w/ pure virtuals
- shared-ptr-to-const
  - shared-ptr-to-non-const would be hidden global variable :(
  - **value semantics**, size of shared-ptr for any complexity
  - **lightweight value types**.
  - Immutable
  - pass all around program, can't get it wrong
  - don't care about heap costs in tests
- Implement API functions: delegate
  - (Wouldn't you like reflection here?)

---

# Level 2: Map Abstractions

Notes:

Target: **25:32**

- A map is where you get paths that have meaning

---

## Two Basins in Design Space

<div class="container">
<div class="fragment fade-in" data-fragment-index="1">
<h3>Map Sketch</h3>
<div>
<img src="./figures/maps-basins/sketch.png">
</div>
</div>

<div class="fragment fade-in" data-fragment-index="2">
<h3>Backdrops</h3>
<div class="r-stack">
<img class="fragment fade-in" data-fragment-index="2" src="./figures/maps-basins/real_maps.png">
<img class="fragment fade-in" data-fragment-index="3" src="./figures/maps-basins/real_maps_lane_boundary.png">
</div>
</div>
</div>

Notes:

- Map Sketch: fully synthetic
- Backdrops: bundle nice paths on top of a real world map

---

## Map Sketch

<div class="container">

<div class="r-stack">
<div class="fragment fade-in-then-out" data-fragment-index="1">

```cpp
const auto road =
  RoadSketcher{}






    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="2">

```cpp
const auto road =
  RoadSketcher{}


    .finish_lane_and_start_another()



    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="3">

```cpp
const auto road =
  RoadSketcher{}


    .finish_lane_and_start_another()
    .finish_lane_and_start_another()


    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="4">

```cpp
const auto road =
  RoadSketcher{}
    .define_road_frame(flat_ribbon_turning_left)

    .finish_lane_and_start_another()
    .finish_lane_and_start_another()


    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="5">

```cpp
const auto road =
  RoadSketcher{}
    .define_road_frame(flat_ribbon_turning_left)
    .set_left_shoulder_width(4 * m)
    .finish_lane_and_start_another()
    .finish_lane_and_start_another()


    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="6">

```cpp
const auto road =
  RoadSketcher{}
    .define_road_frame(flat_ribbon_turning_left)
    .set_left_shoulder_width(4 * m)
    .finish_lane_and_start_another()
    .finish_lane_and_start_another(
      NO_LC_AVAILABILITY)

    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="7">

```cpp
const auto road =
  RoadSketcher{}
    .define_road_frame(flat_ribbon_turning_left)
    .set_left_shoulder_width(4 * m)
    .finish_lane_and_start_another()
    .finish_lane_and_start_another(
      begin_lc_availability(100_m_pos))

    .finish_lane_and_road();
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="8">

```cpp
const auto road =
  RoadSketcher{}
    .define_road_frame(flat_ribbon_turning_left)
    .set_left_shoulder_width(4 * m)
    .finish_lane_and_start_another()
    .finish_lane_and_start_another(
      begin_lc_availability(100_m_pos))
    .convert_to_onramp_at(50_m_pos)
    .finish_lane_and_road();
```

</div>
</div>

<div>
<div class="r-stack" style="width: 80%">
<img class="fragment fade-in" data-fragment-index="1" src="./figures/map-sketch/persp_01.png">
<img class="fragment fade-in" data-fragment-index="2" src="./figures/map-sketch/persp_02.png">
<img class="fragment fade-in" data-fragment-index="3" src="./figures/map-sketch/persp_03.png">
<img class="fragment fade-in" data-fragment-index="4" src="./figures/map-sketch/persp_04.png">
<img class="fragment fade-in" data-fragment-index="5" src="./figures/map-sketch/persp_05.png">
<img class="fragment fade-in" data-fragment-index="6" src="./figures/map-sketch/persp_06.png">
<img class="fragment fade-in" data-fragment-index="7" src="./figures/map-sketch/persp_07.png">
<img class="fragment fade-in" data-fragment-index="8" src="./figures/map-sketch/persp_08.png">
</div>
<div class="r-stack" style="width: 80%">
<img class="fragment fade-in" data-fragment-index="1" src="./figures/map-sketch/top_01.png">
<img class="fragment fade-in" data-fragment-index="2" src="./figures/map-sketch/top_02.png">
<img class="fragment fade-in" data-fragment-index="3" src="./figures/map-sketch/top_03.png">
<img class="fragment fade-in" data-fragment-index="4" src="./figures/map-sketch/top_04.png">
<img class="fragment fade-in" data-fragment-index="5" src="./figures/map-sketch/top_05.png">
<img class="fragment fade-in" data-fragment-index="6" src="./figures/map-sketch/top_06.png">
<img class="fragment fade-in" data-fragment-index="7" src="./figures/map-sketch/top_07.png">
<img class="fragment fade-in" data-fragment-index="8" src="./figures/map-sketch/top_08.png">
</div>
</div>

</div>

Notes:

- Road sketcher: readable synthetic roads
  - Number of lanes is number of times you say "finish lane"
- That's how you **make** it.  To **use** it...
  - **Two** ways, depending on what you're doing

---

## Two clients for sketches

<div class="r-stack two-clients">
<img src="./figures/map-sketch-deps/deps_0.svg">
<img class="fragment" src="./figures/map-sketch-deps/deps_1.svg">
<img class="fragment" src="./figures/map-sketch-deps/deps_2.svg">
</div>

Notes:

- One role: tests
  - "bucket of well named paths"
  - Place actors in the scene
  - "nominal path of left lane"
- Other role helps with "real map data"
  - Need to pass to functions
  - Map sketch: not a map!
    - _high level description_
  - What's the dependency relationship?
- No dependency!  Neither knows about the other
  - **Builder Library** is the other role
  - **iterates** over data in map sketch
  - Builds whatever kind of map data you're making
  - Some effort to build map data
    - **already have** map data?  Another option...
    - Just lacks human-usable paths

---

## Backdrops

<div class="r-stack">
<div class="container">
<div>
<img src="./figures/maps-basins/real_maps_lane_boundary.png">
</div>

<div class="r-stack nolinenum">
<div class="fragment fade-in-then-out" data-fragment-index="1">

```cpp
class Hwy3LanesWExit {
 public:
  Hwy3LanesWExit();











 private:
  AVMap map_;

  Lane left_;
  Lane center_;
  Lane right_;
  Lane exit;

  PositionD start_position_;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="2">

```cpp
class Hwy3LanesWExit {
 public:
  Hwy3LanesWExit();

  friend Lane left_lane(const Hwy3LanesWExit &h) {
    return h.left_;
  }

  friend Position split_point(const Hwy3LanesWExit&) {
    return 0_m_pos;
  }

  // ...

 private:
  AVMap map_;

  Lane left_;
  Lane center_;
  Lane right_;
  Lane exit;

  PositionD start_position_;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="3">

```cpp
class Hwy3LanesWExit {
 public:
  Hwy3LanesWExit();

  friend Lane left_lane(const Hwy3LanesWExit &h) {
    return h.left_;
  }

  friend Position split_point(const Hwy3LanesWExit&) {
    return 0_m_pos;
  }

  // ...

 private:
  AVMap map_;

  Lane left_;
  Lane center_;
  Lane right_;
  Lane exit;

  PositionD start_position_;
};
```

</div>
<div class="fragment fade-in" data-fragment-index="4">

```cpp
Hwy3LanesWExit::Hwy3LanesWExit() :
  map_{fetch_map("hwy_3_lanes_w_exit"), "f7a4fff3"},
  left_{
    .map=map_,
    .path={"a43b8847", "cd3eede8", "07d85409"},
    .left_bound={"45493fb2", "1e963a78", "dff6e747"},
    .right_bound={"e45657dd", "e196e064", "7efc19fc"},
  },
  center_{
    .map=map_,
    .path={"9ba58c68", "7e04b76a", "25877bfd"},
    .left_bound={"18389e4c", "95ad65dd", "8d544a9b"},
    .right_bound={"66e3b528", "6280f64d", "ec46cb72"},
    .align_to=left_,
  },
  right_{ /* ... */ },
  exit_{ /* ... */ },
  start_position_{
    path_length({"a43b8847", "cd3eede8"})}
  {}
```

</div>
<div class="fragment fade-in" data-fragment-index="5">

```cpp [3,14]
Hwy3LanesWExit::Hwy3LanesWExit() :
  map_{fetch_map("hwy_3_lanes_w_exit"), "f7a4fff3"},
  left_{
    .map=map_,
    .path={"a43b8847", "cd3eede8", "07d85409"},
    .left_bound={"45493fb2", "1e963a78", "dff6e747"},
    .right_bound={"e45657dd", "e196e064", "7efc19fc"},
  },
  center_{
    .map=map_,
    .path={"9ba58c68", "7e04b76a", "25877bfd"},
    .left_bound={"18389e4c", "95ad65dd", "8d544a9b"},
    .right_bound={"66e3b528", "6280f64d", "ec46cb72"},
    .align_to=left_,
  },
  right_{ /* ... */ },
  exit_{ /* ... */ },
  start_position_{
    path_length({"a43b8847", "cd3eede8"})}
  {}
```

</div>
</div>
</div>

<div>
<img class="fragment fade-in-then-out" data-fragment-index="3" src="./figures/backdrops/graph.png" style="width: 70%;">
</div>
</div>

Notes:

- So, add nice paths to real map: "backdrop"
  - What _is_ it?
- class with data members you'd expect
- hidden friend functions
  - aesthetic choice: "left lane of map"
- To construct, view raw map data...
- Add sequences of GUIDs for segments
- Make sure lanes are aligned
  - Unit tests!
  - Labor intensive, but gives full map + nice paths

---

## Comparison

<table>
  <tr>
    <th></th>
    <th>Map Sketch</th>
    <th>Backdrops</th>
  </tr>
  <tr class="fragment">
    <td>Using One</td>
    <td class="good">Easy</td>
    <td class="good">Easy</td>
  </tr>
  <tr class="fragment">
    <td>Making One</td>
    <td class="good">Easy</td>
    <td class="poor">Very labor intensive</td>
  </tr>
  <tr class="fragment">
    <td>Fidelity</td>
    <td class="fair">OK, but needs big investment</td>
    <td class="good">Perfect</td>
  </tr>
  <tr class="fragment">
    <td>Startup cost</td>
    <td class="poor">High</td>
    <td class="good">Low technical risk</td>
  </tr>
</table>


---

# Level 3: Scene Builder

Notes:

Target: **31:52**

- Now we have:
  - user-friendly poses, paths, and motions
  - _some_ solution for map data that gives meaningful paths
  - Time to enable writing real tests!
    - MP test cases: in terms of _scene_; so, **scene builder**

---

## Scene Builder: Interfaces (core)

<div class="r-stack nolinenum">
<div style="width: 100%;">

```cpp
TEST_F(MotionPlanner, CanRunInTest) {















}
```

</div>
<div class="fragment fade-in" data-fragment-index="1" style="width: 100%;">

```cpp [2]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};














}
```

</div>
<div class="fragment fade-in" data-fragment-index="2" style="width: 100%;">

```cpp [4]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;












}
```

</div>
<div class="fragment fade-in" data-fragment-index="3" style="width: 100%;">

```cpp [5-13]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;
  const auto cycle_result =
    SceneBuilder{
      {




      },
    }



}
```

</div>
<div class="fragment fade-in" data-fragment-index="4" style="width: 100%;">

```cpp [8]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,



      },
    }



}
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="5" style="width: 100%;">

```cpp [9]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),


      },
    }



}
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="6" style="width: 100%;">

```cpp [10]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),
        .ego_path = {nominal_path(right_lane(road)), ego_position},

      },
    }



}
```

</div>
<div class="fragment fade-in" data-fragment-index="7" style="width: 100%;">

```cpp [11]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),
        .ego_path = {nominal_path(right_lane(road)), ego_position},
        .ego_motion = 65 * MPH,
      },
    }



}
```

</div>
<div class="fragment fade-in" data-fragment-index="8" style="width: 100%;">

```cpp
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_start_position(road) - 65 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),
        .ego_path = {nominal_path(right_lane(road)), ego_position},
        .ego_motion = 65 * MPH,
      },
    }
      .run_cycle_through(RANKER);

  EXPECT_THAT(cycle_result, ProducesValidPlan());
}
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="9">
  <video style="width: 70%;">
    <source src="./figures/scene-builder-core/result.webm" type="video/webm">
  </video>
</div>

<div class="fragment fade-in" data-fragment-index="10"></div>

</div>

Notes:

- Basic example
- Backdrop: "highway merge, subsequent exit"
- We're 65 m behind merge start
- When describing scene, two kinds of inputs:
  - Constructor: things you need every single time
    - communicate requiredness to users
  - Setters: more optional things
- Core data: map
- Goal
- Ego path
- Ego motion
- Run full cycle, expect valid plan
  - Can we picture this scene?
  - In the right lane
  - Merge starts 65 m ahead
  - Exit after the merge
- Let's see
  - Distance rings show about 65 m back
  - Can see the merge, and the exit up ahead
- Readable test source code, says everything we care about, nothing we don't, backed up by vis
  - Now let's look at some setters

---

## Putting actors in the scene

<div class="container">
<div>

#### Actor _Sketch_

<div class="r-stack nolinenum">
<div>

```cpp
class ActorSketch {
 public:
  Pose3D ground_pose(DurationD dt = ZERO) const;
  DisplacementD length() const;
  // ...





};
```

</div>
<div class="fragment fade-in" data-fragment-index="1">

```cpp [9-10]
class ActorSketch {
 public:
  Pose3D ground_pose(DurationD dt = ZERO) const;
  DisplacementD length() const;
  // ...



 private:
  ActorSketchData data_;
};
```

</div>
<div class="fragment fade-in" data-fragment-index="4">

```cpp [7]
class ActorSketch {
 public:
  Pose3D ground_pose(DurationD dt = ZERO) const;
  DisplacementD length() const;
  // ...

  perception::Actor convert_to_actor();

 private:
  ActorSketchData data_;
};
```

</div>
<div class="fragment fade-in" data-fragment-index="5">

```cpp [6]
class ActorSketch {
 public:
  Pose3D ground_pose(DurationD dt = ZERO) const;
  DisplacementD length() const;
  // ...

  perception::Actor convert_to_actor();

 private:
  ActorSketchData data_;
};
```

</div>
</div>

<div class="r-stack nolinenum">
<div class="fragment fade-in" data-fragment-index="1">

```cpp
struct ActorSketchData {
  int64_t id;






};

```

</div>
<div class="fragment fade-in" data-fragment-index="2">

```cpp [4-5]
struct ActorSketchData {
  int64_t id;

  RelativePath path;
  Motion motion;



};

```

</div>
<div class="fragment fade-in" data-fragment-index="3">

```cpp [7-8]
struct ActorSketchData {
  int64_t id;

  RelativePath path;
  Motion motion;

  ActorCategory category;
  Eigen::Vector3d extents_m;
};

```

</div>
<div class="fragment fade-in" data-fragment-index="4">

```cpp [4-5]
struct ActorSketchData {
  int64_t id;

  RelativePath path;
  Motion motion;

  ActorCategory category;
  Eigen::Vector3d extents_m;
};

```

</div>
<div class="fragment fade-in" data-fragment-index="5">

```cpp [3]
struct ActorSketchData {
  int64_t id;

  RelativePath path;
  Motion motion;

  ActorCategory category;
  Eigen::Vector3d extents_m;
};

```

</div>
</div>
</div>
<div class="fragment fade-in" data-fragment-index="5">

#### Actor _Sketcher_

<div class="r-stack nolinenum">
<div>

```cpp
class ActorSketcher {
 public:
  explicit ActorSketcher(RelativePath path);

  ActorSketcher &set_motion(Motion motion);
  ActorSketcher &set_category(ActorCategory category);
  ActorSketcher &set_length(DisplacementD length);
  // ...

  ActorSketch sketch();
};
```

</div>
<div class="fragment fade-in" data-fragment-index="6">

```cpp [4]
class ActorSketcher {
 public:
  explicit ActorSketcher(RelativePath path);

  ActorSketcher &set_motion(Motion motion);
  ActorSketcher &set_category(ActorCategory category);
  ActorSketcher &set_length(DisplacementD length);
  // ...

  ActorSketch sketch();
};
```

</div>
</div>
<div class="r-stack nolinenum">
<div class="fragment fade-in" data-fragment-index="6">

```cpp
ActorSketcher car_sketcher(RelativePath path) {
  return ActorSketcher{path}
    .set_category(ActorCategory::VEHICLE)
    .set_length(4.5 * m);
    .set_width(2.0 * m);
    .set_height(1.5 * m);
}




```

</div>
<div class="fragment fade-in" data-fragment-index="7">

```cpp [9-10]
ActorSketcher car_sketcher(RelativePath path) {
  return ActorSketcher{path}
    .set_category(ActorCategory::VEHICLE)
    .set_length(4.5 * m);
    .set_width(2.0 * m);
    .set_height(1.5 * m);
}

ActorSketcher truck_sketcher(RelativePath path);
ActorSketcher pedestrian_sketcher(RelativePath path);
```

</div>
</div>
</div>
</div>

Notes:

- We use _sketches_: high level descriptions of actors.
  - Can query properties...
- Data includes an ID,
- path-and-motion,
- actor type, and 3D extents
- Can convert to a "real" actor type
  - _Including a kinematically self-consistent sequence of historical observations_
    - Perception actor has this, _you get this for free_
- We make this with an **actor sketcher**.  Here's the generic version
  - (describe...)
  - OK, but verbose
- Pre-configured actor sketchers show intent:
  - Car: set category, make car-sized
- truck sketcher, pedestrian sketcher, etc.
  - Prefer these

---

## Scene Builder: Adding actors

<div class="container_75_25">
<div class="r-stack nolinenum">
<div>

```cpp












const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), ego_position},
      .ego_motion = 65 * MPH,
    },
  }


    .run_cycle_through(RANKER);
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="2">

```cpp [1-4,22]
const auto car = car_sketcher({nominal_path(onramp(road)), ego_position})
  .set_motion(65 * MPH)

  .sketch()








const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), ego_position},
      .ego_motion = 65 * MPH,
    },
  }
    .add_actor(car)

    .run_cycle_through(RANKER);
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="3">

```cpp [2-3]
const auto car = car_sketcher({nominal_path(onramp(road)), ego_position})
  .set_motion(
    accelerating_from(35 * MPH).to(75 * MPH).at(2 * m/s/s).currently(65 * MPH))
  .sketch()








const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), ego_position},
      .ego_motion = 65 * MPH,
    },
  }
    .add_actor(car)

    .run_cycle_through(RANKER);
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="4">

```cpp [6-10,23]
const auto car = car_sketcher({nominal_path(onramp(road)), ego_position})
  .set_motion(
    accelerating_from(35 * MPH).to(75 * MPH).at(2 * m/s/s).currently(65 * MPH))
  .sketch()

const auto ped = pedestrian_sketcher({right_boundary(onramp(road))
                      .pose_at(ego_position + 40 * m)
                      .turn_left()
                      .move_backward(3 * m)})
            .sketch()


const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), ego_position},
      .ego_motion = 65 * MPH,
    },
  }
    .add_actor(car)
    .add_actor(ped)
    .run_cycle_through(RANKER);
```

</div>
<div class="fragment fade-in" data-fragment-index="5">

```cpp
const auto car = car_sketcher({nominal_path(onramp(road)), ego_position})
  .set_motion(
    accelerating_from(35 * MPH).to(75 * MPH).at(2 * m/s/s).currently(65 * MPH))
  .sketch()

const auto ped = pedestrian_sketcher({right_boundary(onramp(road))
                      .pose_at(ego_position + 40 * m)
                      .turn_left()
                      .move_backward(3 * m)})
            .sketch()


const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), ego_position},
      .ego_motion = 65 * MPH,
    },
  }
    .add_actor(car)
    .add_actor(ped)
    .run_cycle_through(RANKER);
```

</div>
</div>
<div class="r-stack">
<img src="./figures/scene-builder-actors/scene_01.png">
<img class="fragment fade-in" data-fragment-index="7" src="./figures/scene-builder-actors/scene_02.png">
</div>
</div>

Notes:

- Now let's add some actors!
  - Recall: here's our old scene
- Add a car on the on-ramp at our same position (same "mile marker"), doing 65
- Could use a more realistic scenario, getting up to speed
  - `accelerating_from`: a motion builder
    - Could you write it?  (Yeah!)
    - Didn't change our car sketcher, but when we wrote this motion builder, it got more powerful!
    - why to take **`Motion`** in your APIs
- Now add pedestrian facing the on ramp, 3 m away, stopped.
- _Can you picture this scene?_
- Yes, there they are!
  - Can't vis acceleration; sorry

---

## Interlude: Visualization

<div class="container">
<div class="r-stack nolinenum">
<div class="fragment fade-out" data-fragment-index="2">

```cpp [15]
const auto road = HighwayMergeWithSubsequentExit{};

const auto p =
  merge_completion_position(road) - 300 * m;

const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), p},
      .ego_motion = 65 * MPH,
    },
  }
    .run_cycle_through(RANKER);




```

</div>
<div class="fragment fade-in" data-fragment-index="2">

```cpp [15-18]
const auto road = HighwayMergeWithSubsequentExit{};

const auto p =
  merge_completion_position(road) - 300 * m;

const auto cycle_result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(right_lane(road)),
      .ego_path = {nominal_path(right_lane(road)), p},
      .ego_motion = 65 * MPH,
    },
  }
    .run_cycle_through(
      RANKER,
      {.fail_and_visualize_to = "my_test"}
    );
```

</div>
</div>
<div class="r-stack nolinenum">
<div class="fragment fade-in" data-fragment-index="1">

```cpp
struct VisualizeTest {
    std::string fail_and_visualize_to = "";
};

```

</div>
</div>
</div>


Notes:

- Here's where we're getting vis
  - Here's an example scene we built
- There's a second argument of _this_ type
- Use designated initializer: "fail and visualize to" "my test"
  - If nonempty, send data to S3 that a web tool can read
  - Interactive 3D visualizations!
    - That's where these screenshots come from
  - Why "fail"?  So we don't land test that tries-and-fails to send data to S3

---

# Intent-first APIs<br>and Evolution

Notes:

Target: **40:15**

- All our APIs are very high level and intent based
- Obviously great for writing and reading
- Consider implications for **maintainability** as code evolves

---

## Construction and blockages

<div>
  <img src="./figures/construction.jpg">
  <figcaption>

  By <a href="https://www.flickr.com/photos/oregondot/">OregonDOT</a> (Flickr), CC BY-SA 2.0 DEED, <https://www.flickr.com/photos/oregondot/35245496740/in/photostream/>

  </figcaption>
</div>

Notes:

- How to _describe_ a construction blockage?
  - Physically: bunch of barrels
  - **Logically:** a **boundary**
    - Hey, we have paths!
  - Strategy: describe the boundary
    - Pick a "main path" from the scene
    - Add checkpoints of _along-path positions_
    - Can add _lateral offsets_
    - Piecewise linear result
  - Implementation: drop barrels whose edges touch this path
    - If you block the left side, the barrels are **on** the left side

---

## Blockage example

<div class="container">
<div class="r-stack nolinenum">
<div>

```cpp
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;









const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }

    .run_cycle_through(RANKER);
```

</div>
<div class="fragment" data-fragment-index="1">

```cpp [5,26]
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;

const auto construction =







const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);
```

</div>
<div class="fragment" data-fragment-index="2">

```cpp [5,6-7,26]
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;

const auto construction =
  block_left_side_along(
      right_boundary(left_lane(road)))





const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);
```

</div>
<div class="fragment" data-fragment-index="3">

```cpp [5,8-9,26]
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;

const auto construction =
  block_left_side_along(
      right_boundary(left_lane(road)))
    .from(ego_position + 50 * m,
      MoveLeft{width(left_lane(road))})



const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);
```

</div>
<div class="fragment" data-fragment-index="4">

```cpp [5,10,26]
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;

const auto construction =
  block_left_side_along(
      right_boundary(left_lane(road)))
    .from(ego_position + 50 * m,
      MoveLeft{width(left_lane(road))})
    .through(ego_position + 100 * m)


const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);
```

</div>
<div class="fragment" data-fragment-index="5">

```cpp [5,11,26]
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;

const auto construction =
  block_left_side_along(
      right_boundary(left_lane(road)))
    .from(ego_position + 50 * m,
      MoveLeft{width(left_lane(road))})
    .through(ego_position + 100 * m)
    .to(ego_position + 250 * m);

const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);
```

</div>
<div class="fragment" data-fragment-index="6">

```cpp [5-11,26]
const auto road = TwoLaneStraightRoad{};

const auto ego_position = 0_m_pos;

const auto construction =
  block_left_side_along(
      right_boundary(left_lane(road)))
    .from(ego_position + 50 * m,
      MoveLeft{width(left_lane(road))})
    .through(ego_position + 100 * m)
    .to(ego_position + 250 * m);

const auto result =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(left_lane(road)),
      .ego_path =
        {
          nominal_path(left_lane(road)),
          ego_position,
        },
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);
```

</div>
</div>

<div>
<div class="r-stack">
<img src="./figures/construction-tests/persp_01.png">
<img class="fragment fade-in" data-fragment-index="7" src="./figures/construction-tests/persp_02.png">
</div>
<div class="r-stack">
<img src="./figures/construction-tests/top_01.png">
<img class="fragment fade-in" data-fragment-index="7" src="./figures/construction-tests/top_02.png">
</div>
</div>
</div>

Notes:

- Example
- Build `construction`, pass to "dot-add-blockage"
- Block left side along right boundary of our lane
  - Dashed line is main path
  - Barrels to the left of it
- "dot-from": 50 m ahead, left by lane width
- "dot-through" checkpt later, no lateral displacement (back on main path)
- "dot-to" 250 m ahead, also on main path
- Can we picture barrels?
- Was it something like this?
  - Note tapering of first pt: important!

---

## Value of intent-first interfaces

<div class="fragment fade-in" data-fragment-index="1">

#### Categorical

</div>

<div class="r-stack">
<img src="./figures/categorical_vs_continuous/bare_barrels.png" style="width: 80%">
<img class="fragment fade-in" data-fragment-index="1" src="./figures/categorical_vs_continuous/categorical.png" style="width: 80%">
</div>

<div class="fragment fade-in" data-fragment-index="2">

#### Continuous

<img src="./figures/categorical_vs_continuous/continuous.png" style="width: 80%">

</div>

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

## Test contents are identical

<div class="container_60_40">
<div class="identical_contents">

```cpp
const auto road = TwoLaneStraightRoad{};

const auto ego_lane = left_lane(road);
const auto ego_position = 0_m_pos;

const auto construction =
  block_left_side_along(right_boundary(ego_lane))
    .from(ego_position + 50 * m, MoveLeft{width(ego_lane)})
    .through(ego_position + 100 * m)
    .to(ego_position + 250 * m);

const auto selected_plan =
  SceneBuilder{
    {
      .map = road,
      .goal = final_pose(ego_lane),
      .ego_path = {nominal_path(ego_lane), ego_position},
      .ego_motion = 65 * MPH,
    },
  }
    .add_blockage(construction)
    .run_cycle_through(RANKER);

EXPECT_THAT(selected_plan, PlansLaneChange(Direction::RIGHT));
```

</div>
<div>

#### Categorical

<div class="r-stack">
<img src="./figures/categorical_vs_continuous/bare_barrels.png">
<img src="./figures/categorical_vs_continuous/categorical.png">
</div>

#### Continuous

<div class="r-stack">
<img src="./figures/categorical_vs_continuous/continuous.png">
</div>

</div>
</div>

Notes:

- Nothing in source code needs to change
  - due to **intent based** interfaces
  - and using **physical features**

---

# Scene builder implementation strategy

Notes:

Target: **44:32**

- Everything so far is "what will users write"
- But... how does this work?

---

## Building up `SceneDescription`

<div class="container">
<div>

<div class="r-stack nolinenum">
<div>

```cpp
struct SceneDescription {
  MapData map;
  Goal goal;
  EgoState ego;
  std::vector<ActorSketch> actors;
  std::vector<BlockageSketch> blockages;

  PlannerConfig config;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="1">

```cpp [4]
struct SceneDescription {
  MapData map;
  Goal goal;
  EgoState ego;
  std::vector<ActorSketch> actors;
  std::vector<BlockageSketch> blockages;

  PlannerConfig config;
};
```

</div>
</div>
<div class="fragment" data-fragment-index="1">

```cpp
struct EgoState {
  RelativePath path;
  Motion motion;
  TurnSignalHistory turn_signal;
};
```

</div>

</div>

<div class="fragment">

```cpp
class SceneBuilder {
 public:
  // ...

  SceneBuilder &add_actor(ActorSketch sketch);

  // ...

 private:
  SceneDescription scene_;
};
```

<div class="fragment">

```cpp
SceneBuilder &SceneBuilder::add_actor(
    ActorSketch sketch) {
  scene_.actors.push_back(sketch);
  return *this;
}
```

</div>

</div>
</div>

Notes:

- SceneBuilder builds up "scene description"
  - map, goal, ego, so on
- Nested
  - ego:
    - path, motion
    - turn signal
- SB holds SD
- implementations for fluent APIs: obvious
  - Not super interesting.
  - Here's how it helps

---

## Planner is based on messages

<div class="r-stack">
<div class="r-stack planner_messages">
<img src="./figures/planner_messages/planner_messages_0.svg">
<img class="fragment fade-in" data-fragment-index="1" src="./figures/planner_messages/planner_messages_1.svg">
<img class="fragment fade-in" data-fragment-index="2" src="./figures/planner_messages/planner_messages_2.svg">
<img class="fragment fade-in" data-fragment-index="3" src="./figures/planner_messages/planner_messages_3.svg">
<img class="fragment fade-in" data-fragment-index="5" src="./figures/planner_messages/planner_messages_4.svg">
<img class="fragment fade-in" data-fragment-index="6" src="./figures/planner_messages/planner_messages_5.svg">
<img class="fragment fade-in" data-fragment-index="7" src="./figures/planner_messages/planner_messages_6.svg">
</div>

<div class="container planner_msgs_code">
<div class="r-stack nolinenum">
<div class="fragment fade-in-then-out" data-fragment-index="4">

```cpp
struct SceneDescription {
  MapData map;
  Goal goal;
  EgoState ego;
  std::vector<ActorSketch> actors;
  std::vector<BlockageSketch> blockages;

  PlannerConfig config;
};
```

```cpp
struct EgoState {
  RelativePath path;
  Motion motion;
  TurnSignalHistory turn_signal;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="5">

```cpp [1,4,9]
struct SceneDescription {
  MapData map;
  Goal goal;
  EgoState ego;
  std::vector<ActorSketch> actors;
  std::vector<BlockageSketch> blockages;

  PlannerConfig config;
};
```

```cpp [1,2,3,5]
struct EgoState {
  RelativePath path;
  Motion motion;
  TurnSignalHistory turn_signal;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="6">

```cpp [1,2,9]
struct SceneDescription {
  MapData map;
  Goal goal;
  EgoState ego;
  std::vector<ActorSketch> actors;
  std::vector<BlockageSketch> blockages;

  PlannerConfig config;
};
```

```cpp [1,5]
struct EgoState {
  RelativePath path;
  Motion motion;
  TurnSignalHistory turn_signal;
};
```

</div>
<div class="fragment fade-in-then-out" data-fragment-index="7">

```cpp [1,5,6,9]
struct SceneDescription {
  MapData map;
  Goal goal;
  EgoState ego;
  std::vector<ActorSketch> actors;
  std::vector<BlockageSketch> blockages;

  PlannerConfig config;
};
```

```cpp [1,5]
struct EgoState {
  RelativePath path;
  Motion motion;
  TurnSignalHistory turn_signal;
};
```

</div>

</div>

<div><!-- Empty second column --></div>
</div>
</div>

Notes:

- Modules **generate** msgs
- **Publish** to pub/sub
- Planner **subscribes**
- **Cycle start**:
  - each buffer has _msgs_
  - rcvd at some _timestamp_
- _have_ scene desc, _need_ to fill buffers
  - **Take each one separately,** independently
  - Ask: what would I see, in this spot?
- ego pose: easy: path and motion
  - **again:** includes **history** of poses!
- map is map
- actors: read actors, get barrels from blockages
  - we know what data to create
  - need to **store**

---

## Storing planner input messages

<div class="r-stack storing">
<div>
<div class="container nolinenum">
<div>

#### Planner input "tag types"

<div class="r-stack">
<div>

```cpp
struct EgoPoseTopic {

};


struct MapTopic {

};


struct ActorsTopic {

};


```

</div>
<div class="fragment">

```cpp
struct EgoPoseTopic {
  using MsgType = localization::proto::Pose;
};


struct MapTopic {
  using MsgType = mapping::proto::Map;
};


struct ActorsTopic {
  using MsgType = perception::proto::Actors;
};


```

</div>
<div class="fragment">

```cpp [4,9,14]
struct EgoPoseTopic {
  using MsgType = localization::proto::Pose;
};
constexpr auto EGO_POSE = EgoPoseTopic{};

struct MapTopic {
  using MsgType = mapping::proto::Map;
};
constexpr auto MAP = MapTopic{};

struct ActorsTopic {
  using MsgType = perception::proto::Actors;
};
constexpr auto ACTORS = ActorsTopic{};
```

</div>
</div>
<div class="fragment">

#### One topic, one message

```cpp
template <typename Topic>
struct TimedMessage {
  Timestamp               publish_time;
  typename Topic::MsgType message;
};
```

</div>
</div>
<div>
<div class="fragment">

#### "All planner inputs"

```cpp
using PlannerInputs = std::tuple<
  EgoPoseTopic, MapTopic, ActorsTopic, // ...
>;
```

</div>

<div class="fragment">

#### "All messages for one cycle"

```cpp
TimedMessagesForEachOf<PlannerInputs>;

// Same as:
// std::tuple<
//  std::vector<TimedMessage<EgoPoseTopic>>,
//  std::vector<TimedMessage<MapTopic>>,
//  std::vector<TimedMessage<ActorsTopic>>,
//  ...
// >;
```

</div>
</div>
</div>
</div>
<div class="fragment fade-in-then-out" style="background-color: yellow;">

```cpp
template <typename T>
struct TimedMessagesForEachOfImpl;
template <typename T>
using TimedMessagesForEachOf = typename TimedMessagesForEachOfImpl<T>::type;

template <typename... Topics>
struct TimedMessagesForEachOfImpl<std::tuple<Topics...>> {
  using type = std::tuple<std::vector<TimedMessage<Topics>>...>;
};
```

</div>
<div class="fragment"></div>
</div>

Notes:

- Planner inputs: tag types
- Say which msg
- _Canonical instances_
  - Pass to APIs
- Timed msg for topic: "has ..."
- Also: tuple of "all planner inputs"
  - reason, programatically, at compile time, about "all inputs"
- To get **all** msgs, **all** topics, for a cycle
  - "timed messages for each of" planner inputs
  - boils down to tuple of vectors
- Usual metaprogramming techniques
  - don't worry about details ("pause on youtube" slide)
- has right shape.  Is it usable?

---

## `SceneMessages`: better ergonomics

<div class="r-stack nolinenum">
<div style="width: 60%; margin: auto;" class="fragment">

```cpp
class SceneMessages {
 public:







 private:
  TimedMessagesForEachOf<PlannerInputs> data_{};
};
```

</div>
<div style="width: 60%; margin: auto;" class="fragment">

```cpp [3-8]
class SceneMessages {
 public:
  template <typename Topic>
  std::vector<TimedMessage<Topic>> &operator[](Topic) {
    return std::get<std::vector<TimedMessage<Tag>>>(data_);
  }

  // Also a `const` version, naturally...

 private:
  TimedMessagesForEachOf<PlannerInputs> data_{};
};
```

</div>
</div>

<div class="container nolinenum">

<div>

#### Using raw `TimedMessagesForEachOf`

```cpp
TimedMessagesForEachOf<PlannerInputs> messages;
std::get<std::vector<TimedMessage<EgoPoseTopic>>>(
  messages);
```

</div>

<div class="fragment">

#### Using `SceneMessages`

```cpp
SceneMessages messages;
messages[EGO_POSE];
```

</div>

</div>

Notes:

- Can make tuple (wordy), and access one vector (clumsy)
  - End users want: "give me the msgs for this topic"
- Wrap in a class
- Provide square bracket operator, accept tag by **value**
- Interfaces way more ergonomic!
  - "Feels like" python style associative container
  - All dispatching happens when program is **built**
  - Nice container...
  - This "scene msgs" is what scene description fills up
    - before we see **how**... _one more_ use case

---

# Testing Fault Conditions

Notes:

- Must make sure planner **detects** faults
  - and **responds appropriately**

---

## Fault tests: desired syntax

<div class="r-stack nolinenum">
<div class="fragment fade-out" data-fragment-index="1">

```cpp
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_completion_position(road) - 300 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),
        .ego_path = {nominal_path(right_lane(road)), ego_position},
        .ego_motion = 65 * MPH,
      },
    }

      .run_cycle_through(RANKER);

  EXPECT_THAT(cycle_result.issues, IsEmpty());
}
```

</div>
<div class="fragment fade-in" data-fragment-index="1">

```cpp [14,17]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_completion_position(road) - 300 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),
        .ego_path = {nominal_path(right_lane(road)), ego_position},
        .ego_motion = 65 * MPH,
      },
    }
      .add_input_tweak(EGO_POSE, SetLatestAge{STALE_EGO_POSE_THRESHOLD + 1 * ms});
      .run_cycle_through(RANKER);

  EXPECT_THAT(cycle_result.issues, Contains(Issues::EGO_POSE_LOST));
}
```

</div>
<div class="fragment fade-in" data-fragment-index="2">

```cpp [14,17]
TEST_F(MotionPlanner, CanRunInTest) {
  const auto road = HighwayMergeWithSubsequentExit{};

  const auto ego_position = merge_completion_position(road) - 300 * m;
  const auto cycle_result =
    SceneBuilder{
      {
        .map = road,
        .goal = final_pose(right_lane(road)),
        .ego_path = {nominal_path(right_lane(road)), ego_position},
        .ego_motion = 65 * MPH,
      },
    }
      .add_input_tweak(EGO_POSE, InvalidateLast{});
      .run_cycle_through(RANKER);

  EXPECT_THAT(cycle_result.issues, Contains(Issues::EGO_POSE_INVALID));
}
```

</div>
</div>

Notes:

- Here's a simple planner test from before
  - Let's imagine we stopped getting ego pose messages
- Add input tweak
  - ego pose
  - set latest age 1 ms past threshold
- Or, make last msg invalid
  - expect invalid msg issue
  - **Point is**: easy to test that planner detects fault _condition_ appropriately
    - Need _other_ tests for fault _response_

---

## Filling up `SceneMessages`

### Two-phase approach:

<div class="container nolinenum">

<div class="fragment">

#### What _times_ would we get this topic?

```cpp
std::vector<Timestamp> default_message_times(
  EgoPoseTag,
  const SceneDescription &scene);
```

</div>

<div class="fragment">

#### What _contents_ would it have, at a _given time_?

```cpp
EgoPoseTag::MsgType build_message(
  EgoPoseTag,
  const SceneDescription &scene,
  Timestamp t);
```

</div>

</div>

Notes:

- Two separate questions for each input type: "_in this situation..._"
- What _times_?
- What _contents_ at a _given time_?
  - Get a vector of times from first
  - Call the second once for each time
  - **Key:** tweaks can apply to set-of-**times**,
    - **before** they reach the second function
  - Msg **contents** always accurate for the time
    - e.g., show where the actor **was** when the stale message came

---

## Input Tweaks

<div class="container nolinenum">
<div>

```cpp
struct InputTweak {
  /* std::variant<
    EgoPoseTopic, MapTopic, ActorsTopic, ...> */
  VariantFromTuple<PlannerInputs>
    topic;

  Variant<DropAll, SetLatestAge, InvalidateLast, ...>
    tweak;
};
```

</div>

<div class="fragment fade-in tweaks">

#### Untweaked:

<img src="./figures/tweak/results_0.svg">

<div class="fragment fade-in">

#### `.tweak = SetLatestAge{600 * ms}`:

<img src="./figures/tweak/results_1.svg">

</div>
<div class="fragment fade-in">

#### `.tweak = DropAll{}`:

<img src="./figures/tweak/results_2.svg">

</div>
<div class="fragment fade-in">

#### `.tweak = InvalidateLast{}`:

<img src="./figures/tweak/results_3.svg">

</div>
</div>

Notes:

- An "input tweak" combines
  - Particular **topic** (first member)
  - Some **change** to msgs (second member)
- Examples
  - untweaked
- `SetLatestAge` shifts timestamps (simulate staleness)
- `DropAll`: no messages (simulate never-arrived)
- `InvalidateLast` (simulate invalid input)
  - **Point:** Easy to write readable tests for fault handling

---

# Wrapping Up

Notes:

Target: **52:08**

- 2 conclusions, one for each audience

---

## Summary: MP Tests

<div class="container">
<div class="fragment fade-in">

#### Motion Planning

<img src="./figures/mp_parts/planner_summary.svg">

</div>

<div class="fragment levels_of_solution">

#### Levels of solution

<div class="fragment">
<span>

### 3. Scene

</span>
<span><img src="./figures/levels/level_3.png"></span>
</div>
<div class="fragment">
<span>

### 2. Maps

</span>
<span><img src="./figures/levels/level_2.png"></span>
</div>
<div class="fragment">
<span>

### 1. Primitives

</span>
<span><img src="./figures/levels/level_1.png"></span>
</div>
<div class="fragment">
<span>

### 0. Units

</span>
<span><img src="./figures/au.png"></span>
</div>
<div class="repo fragment" style="font-size: 80%;">aurora-opensource/au</div>
</div>

<div class="fragment">

#### Intent based APIs

<img src="./figures/construction-tests/persp_03.png" style="width: 80%;">

<div class="fragment fault">

#### Fault handling

<div class="r-stack"><img src="./figures/tweak/results_0.svg"></div>
<div class="r-stack">
<img class="fragment fade-in-then-out" src="./figures/tweak/results_1.svg">
<img class="fragment fade-in-then-out" src="./figures/tweak/results_2.svg">
<img class="fragment fade-in" src="./figures/tweak/results_3.svg">
</div>

</div>
</div>
</div>

Notes:

- Planning problem:
  - take msgs for pose, map, actors, so on
  - make plan for one cycle
- Solution based on levels
  - Level 3: describe a situation the planner finds itself in
    - Fluent APIs
  - Level 2: map abstraction, one of two basins in design space
    - Sketchers, or backdrops
  - Level 1: foundational types: chainable poses, aligned paths, speed profiles
  - Level 0: Au, our open source units library
    - Interesting: as you go down, each level gets more broadly useful
    - L3: **Aurora** MP only
    - L1: Many teams at Aurora
    - L0: people far beyond Aurora, including you!
- Intent-based APIs help as code evolves
- Can even handle fault conditions in callsite readable way!
  - That's one concrete problem with complex inputs.  General case?

---

## Summary: Complex Inputs Playbook

- Know your domain
- Figure out: what would "easy" even look like?
- Judge ideas: feasibility, usability, scope

Notes:

- Know your domain!
  - e.g., mile marker solution for aligned paths
  - Had been thinking about MP test APIs for years before this
- Brainstorm: in a perfect outcome, what source code would you be writing?
  - Give imagination free reign
  - Then, cast a critical eye:
- Think about feasibility, usability, and scope
  - Can you picture the implementation?
  - What will users try to do with this?
  - Easy to use correctly?  Hard to use incorrectly?
  - Scope: what are your core use cases?  "Stretch" ones?  And definitely out of scope?

---

<section data-background="./figures/aurora-truck-car.jpg" data-background-size="contain">

<h1 style="color: white; position: absolute; width: 100%; margin-top: -3em;">Questions?</h1>

Notes:

- How well will this _actually_ generalize?
  - Would love to hear from you!
  - I'll take any questions at this time
