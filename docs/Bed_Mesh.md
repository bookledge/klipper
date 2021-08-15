# 베드 메쉬

베드 메쉬 모듈은 베드 표면의 불균일함을 보상하기 위해 사용됩니다. 이 기능을 사용하면 전체 베드면에서의 첫레이어 안착을 더 좋게 할 수 있습니다. 소프트웨어 기반의 수정은 완벽한 결과를 얻을 수 없습니다. 오직 대략적인 베드 형태만을 취할 수 있을 뿐입니다. 또한 베드 메쉬는 기계적이거나 전기적인 문제에 대한 보정을 할 수 없습니다. 만일 축이 틀어져 있거나 레벨링센서의 정확도가 떨어진다면 베드 메쉬 모듈은 레벨링 측정만으로는 정확한 결과를 얻지 못할 것입니다. 

메쉬 캘리브레이션에 앞서 필수적으로 당신의 레벨링 센서의 Z-offset 값을 캘리브레이션 해야 합니다. 
만일 Z 호밍에 Endstop 스위치가 사용된다면 이것역시 캘리브레이션이 필요합니다.   
보다 자세한 정보는 다음 2개의 문서를 참고하기 바랍니다. 

[Probe_Calibrate](Probe_Calibrate.md)

[Manual_Level](Manual_Level.md) 



## 기본 설정값

### 사각베드
이 예제는 250mm x 220mm 직사각형 베드를 가진 프린터를 가정하에 진행합니다. 
그리고 레벨링센서(프로브)는 X오프셋 값 24mm, Y오프셋은 5mm 라고 가정합니다.


```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35,6
mesh_max: 240, 198
probe_count: 5,3
```


- `speed: 120`\
  _Default 값: 50_\
  포인트 사이를 이동하는 속도

- `horizontal_move_z: 5`\
  _Default 값: 5_\
  포인트 사이를 이동하기 전에 Z 축 방향으로 올라가는 높이

- `mesh_min: 35,6`\
  _필수_\
  원점으로 부터 가장 가까운 첫번째 측정위치. 이 위치는 레벨링센서(프로브)위치에 따라 상대적이다.

- `mesh_max: 240,198`\
  _필수_\
  원점에서 가장 멀리 떨어진 측정위치. 이 지점은 가장 마지막 측정이 아닐 수도 있습니다. 왜냐하면 레벨 측정은 지그재그 경로로 되기 때문입니다. `mesh_min` 과 같이 레벨링센서위치에 따라 상대적이다. 

- `probe_count: 5,3`\
  _Default 값: 3,3_\
  x, y 각 축별로 측정되는 지점의 수. 정수값. 
  이 예제에서는 X 축방향으로 5개 점, Y 축방향으로 3개점을 측정합니다. 따라서 전체적으로 15개 측정위치가 됩니다. 만약 3x3 과 같이 정방형으로 측정하기 원한다면 정수 하나만으로 설정이 가능합니다. 
  즉, `probe_count: 3` 이렇게 입력하면 됩니다. 
  메쉬는 각 축별로 최소 3개의 측정포인트가 필요함을 기억하십시오. 


아래 그림은 `mesh_min`, `mesh_max`, and `probe_count` 옵션이 어떻게 측정위치를 생성하시지를 나타낸 것이다. 화살표는 `mesh_min` 으로 부터 시작한 레벨측정 순서를 나타낸다. 

참고로, 프로브가 `mesh_min` 위치일 때 노즐은 (11,1) 위치에 있게 되고,
프로브가 `mesh_max` 위치일 때 노즐은 (206,193)에 위치한다. 

![bedmesh_rect_basic](img/bedmesh_rect_basic.svg)


### 원형 베드 

이 예제는 베드의 반지름이 100mm 인 원형 베드를 가진 프린터를 가정하에 진행합니다. 
그리고 레벨링센서(프로브)는 직사각형 때의 예제와 마찬가지로 X오프셋 값 24mm, Y오프셋은 5mm 라고 가정합니다.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_radius: 75
mesh_origin: 0,0
round_probe_count: 5
```

- `mesh_radius: 75`\
  _필수_\
  `mesh_origin`에 상대적인 값으로 측정메쉬의 반지름을 mm 단위로 표시한다.
  프로브 오프셋이 메쉬 반지름의 크기를 제한합니다. 이 예제에서는 76보다 큰 반지름은 프린터의 가용반경을 넘어 동작하게 될 것입니다. 

- `mesh_origin: 0,0`\
  _Default 값: 0,0_\
  메쉬의 중심점. 이 좌표는 프로브 위치에 따라 상대적이다. 
  기본값이 0,0 이지만 베드의 더 넓은 부분을 측정할 수 있도록 원점을 잡는게 유용할 수 있다. 아래 나오는 그림을 참고하기 바란다.

- `round_probe_count: 5`\
  _Default 값: 5_\
  이 값은 X, Y 축에 따라 레벨링 측정을 할 최대 포인트수를 정수값으로 나타낸다. 
  "최대"값이라 함은 메쉬 원점을 따라 측정하는 포인트수를 뜻한다. 메쉬의 중심을 측정할 수 있도록 하기 위해 이 값은 홀수여야 한다. 


아래 도안은 얼마나 많은 측정포인트가 만들어졌는지를 나타내고 있다. 보다시피 `mesh_origin` 을 (-10, 0) 로 설정하게 되면 측정하는 영역의 반지름이 85mm 이 된다. 


![bedmesh_round_basic](img/bedmesh_round_basic.svg)



## 고급 설정

보다 고급설정을 위한 자세한 옵션에 대한 설명을 아래에 이어가도록 한다. 
각 예제는 위에 언급했던 직사각형 베드를 가진 프린터를 기준으로 하겠다. 
각 고급옵션들은 원형베드에서도 동일한 방법으로 적용될 수 있다. 

### 메쉬 인터폴레이션

측정된 포인트사이 지점의 Z값을 결정하는데 있어 간단한 bilinear 보간법을 이용해 측정매트릭스를 샘플링하는 것도 가능하지만, 메쉬밀도를 높이기 위해 보다 고급 보간알고리즘을 사용하는게 유용할 수 있다. 
이 알고리즘은 메쉬에 굴곡을 추가한다. 이것은 베드의 물질특성치에 대한 시뮬레이션을 시도한다. 
이것이 가능케 하기 위해 베드 메쉬는 라그랑지에와 바이큐빅 인터폴레이션을 제공한다. 


```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35,6
mesh_max: 240, 198
probe_count: 5,3
mesh_pps: 2,3
algorithm: bicubic
bicubic_tension: 0.2
```

- `mesh_pps: 2,3`\
  _Default 값: 2,2_\
  `mesh_pps` 옵션은 Mesh Points Per Segment(세그먼트 당 메쉬 포인트)의 줄임말이다. 
  이 옵션은 x, y 축을 따라 얼마나 많은 분할 포인트로 인터폴레이션을 할지 정해준다. 
  각 레벨링 측정포인트사이를 '분할'하여 공간을 나누도록 한다. 
  `probe_count` 처럼 `mesh_pps` 는 x,y 의 정수쌍으로 입력된다. 
  또한 정수 하나만 입력시 x,y 양쪽 축에 대한 값으로 인정된다. 
  예제에서는 X축을 따라 4개(5-1)의 분할, y 축을 따라 2개(3-1)의 분할이 존재한다. 
  이는 곧 x 축으로 9개의 인터폴레이션 포인트와, y 축으로 6개의 인터폴레이션 포인트가 만들어진다.
  이것은 총 13x8 의 메쉬를 형성하는 결과를 낳게된다. 
  mesh_pps 를 0 으로 하면 메쉬 인터폴레이션은 꺼지고, 측정 매트릭스가 직접적인 샘플링 위치값이 될 것이다. 

- `algorithm: lagrange`\
  _Default 값: lagrange_\
  메쉬를 인터폴레이션 하는데 사용되는 알고리즘. `lagrange` 와 `bicubic` 를 선택해줄 수 있다.
  라그랑지 인터폴레이션은 6개의 측정 포인트로 제한됩니다. 왜냐하면 진동이 더 큰수의 샘플로 발생하기 때문입니다. Bicubic 인터폴레이션은 각축을 따라 최소 4개의 측정 포인트가 필요합니다. 만약 4개 이하의 값이 주어지면 라그랑지 샘플링이 강제로 적용됩니다. 만약 `mesh_pps` 를 0 으로 설정하면 이 값은 무시되며 어떤 메쉬 인터폴레이션도 실행되지 않습니다. 

- `bicubic_tension: 0.2`\
  _Default 값: 0.2_\
  만약 `algorithm` 옵션이 bicubic 이면 tension 값 설정이 가능합니다. 
  더 높은 텐션값을 적용할 수록 더 많은 경사가 인터폴레이션될 것입니다. 
  이 값을 설정할 때 주의하십시오. 값이 커질 수록 또한 더 많은 오버슛이 발생합니다. 이 오버슛은 당신이 측정한 포인트보다 더 높거나 낮은 인터폴레이션 값을 내놓을 것입니다. 

아래 그림은 위에 사용된 옵션들이 어떻게 인터폴레이션 메쉬를 구성하는지를 나타낸다. 

![bedmesh_interpolated](img/bedmesh_interpolated.svg)

### 이동 Move Splitting

Bed Mesh works by intercepting gcode move commands and applying a transform
to their Z coordinate. Long moves must be and split into smaller moves
to correctly follow the shape of the bed. The options below control the
splitting behavior.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35,6
mesh_max: 240, 198
probe_count: 5,3
move_check_distance: 5
split_delta_z: .025
```

- `move_check_distance: 5`\
  _Default Value: 5_\
  The minimum distance to check for the desired change in Z before performing
  a split.  In this example, a move longer than 5mm will be traversed by the
  algorithm.  Each 5mm a mesh Z lookup will occur, comparing it with the Z
  value of the previous move.  If the delta meets the threshold set by
  `split_delta_z`, the move will be split and traversal will continue.  This
  process repeats until the end of the move is reached, where a final
  adjustment will be applied.  Moves shorter than the `move_check_distance`
  have the correct Z adjustment applied directly to the move without
  traversal or splitting.

- `split_delta_z: .025`\
  _Default Value: .025_\
  As mentioned above, this is the minimum deviation required to trigger a
  move split.  In this example, any Z value with a deviation +/- .025mm
  will trigger a split.

Generally the default values for these options are sufficient, in fact the
default value of 5mm for the `move_check_distance` may be overkill. However an
advanced user may wish to experiment with these options in an effort to squeeze
out the optimial first layer.

### Mesh Fade

When "fade" is enabled Z adjustment is phased out over a distance defined
by the configuration.  This is accomplished by applying small adjustments
to the layer height, either increasing or decreasing depending on the shape
of the bed. When fade has completed, Z adjustment is no longer applied,
allowing the top of the print to be flat rather than mirror the shape of the
bed.  Fade also may have some undesirable traits, if you fade too quickly it
can result in visible artifacts on the print.  Also, if your bed is
significantly warped, fade can shrink or stretch the Z height of the print.
As such, fade is disabled by default.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35,6
mesh_max: 240, 198
probe_count: 5,3
fade_start: 1
fade_end: 10
fade_target: 0
```

- `fade_start: 1`\
  _Default Value: 1_\
  The Z height in which to start phasing out adjustment.  It is a good idea
  to get a few layers down before starting the fade process.

- `fade_end: 10`\
  _Default Value: 0_\
  The Z height in which fade should complete.  If this value is lower than
  `fade_start` then fade is disabled.  This value may be adjusted depending
  on how warped the print surface is.  A significantly warped surface should
  fade out over a longer distance.  A near flat surface may be able to reduce
  this value to phase out more quickly.  10mm is a sane value to begin with if
  using the default value of 1 for `fade_start`.

- `fade_target: 0`\
  _Default Value:  The average Z value of the mesh_\
  The `fade_target` can be thought of as an additional Z offset applied to the
  entire bed after fade completes.  Generally speaking we would like this value
  to be 0, however there are circumstances where it should not be.  For
  example,  lets assume your homing position on the bed is an outlier, its
  .2 mm lower than the average probed height of the bed.  If the `fade_target`
  is 0, fade will shrink the print by an average of .2 mm across the bed.  By
  setting the `fade_target` to .2, the homed area will expand by .2 mm, however
  the rest of the bed will have an accurately sized.  Generally its a good idea
  to leave `fade_target` out of the configuration so the average height of the
  mesh is used, however it may be desirable to manually adjust the fade target
  if one wants to print on a specific portion of the bed.

### The Relative Reference Index

Most probes are suceptible to drift, ie: inaccuracies in probing introduced by
heat or interference.  This can make calculating the probe's z-offset
challenging, particuarly at different bed temperatures.  As such, some printers
use an endstop for homing the Z axis, and a probe for calibrating the mesh.
These printers can benefit from configuring the relative reference index.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35,6
mesh_max: 240, 198
probe_count: 5,3
relative_reference_index: 7
```

- `relative_reference_index: 7`\
  _Default Value: None (disabled)_\
  When the probed points are generated they are each assigned an index.  You
  can look up this index in klippy.log or by using BED_MESH_OUTPUT (see the
  section on Bed Mesh GCodes below for more information).  If you assign an
  index to the `relative_reference_index` option, the value probed at this
  coordinate will replace the probe's z_offset.  This effectively makes
  this coordinate the "zero" reference for the mesh.

When using the relative reference index, you should choose the index nearest
to the spot on the bed where Z endstop calibration was done.  Note that
when looking up the index using the log or BED_MESH_OUTPUT, you should use
the coordinates listed under the "Probe" header to find the correct index.

### Faulty Regions

It is possible for some areas of a bed to report inaccurate results when
probing due to a "fault" at specific locations.  The best example of this
are beds with series of integrated magnets used to retain removable steel
sheets.  The magnetic field at and around these magnets may cause an inductive
probe to trigger at a distance higher or lower than it would otherwise,
resulting in a mesh that does not accurately represent the surface at these
locations.  **Note: This should not be confused with probe location bias, which
produces inaccurate results across the entire bed.**

The `faulty_region` options may be configured to compensate for this affect.
If a generated point lies within a faulty region bed mesh will attempt to
probe up to 4 points at the boundaries of this region.  These probed values
will be averaged and inserted in the mesh as the Z value at the generated
(X, Y) coordinate.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35,6
mesh_max: 240, 198
probe_count: 5,3
faulty_region_1_min: 130.0, 0.0
faulty_region_1_max: 145.0, 40.0
faulty_region_2_min: 225.0, 0.0
faulty_region_2_max: 250.0, 25.0
faulty_region_3_min: 165.0, 95.0
faulty_region_3_max: 205.0, 110.0
faulty_region_4_min: 30.0, 170.0
faulty_region_4_max: 45.0, 210.0
```

- `faulty_region_{1...99}_min`\
  `faulty_region_{1..99}_max`\
  _Default Value: None (disabled)_\
  Faulty Regions are defined in a way similar to that of mesh itself, where
  minimum and maximum (X, Y) coordinates must be specified for each region.
  A faulty region may extend outside of a mesh, however the alternate points
  generated will always be within the mesh boundary.  No two regions may
  overlap.

The image below illustrates how replacement points are generated when
a generated point lies within a faulty region.  The regions shown match those
in the sample config above.  The replacement points and their coordinates
are identified in green.

![bedmesh_interpolated](img/bedmesh_faulty_regions.svg)

## Bed Mesh Gcodes

### Calibration

`BED_MESH_CALIBRATE METHOD=[manual | automatic] [<probe_parameter>=<value>]
 [<mesh_parameter>=<value>]`\
_Default Method:  automatic if a probe is detected, otherwise manual_

Initiates the probing procedure for Bed Mesh Calibration.  If `METHOD=manual`
is selected then manual probing will occur.  When switching between automatic
and manual probing the generated mesh points will automatically be adjusted.

It is possible to specify mesh parameters to modify the probed area.  The
following parameters are available:
- Rectangular beds (cartesian):
  - `MESH_MIN`
  - `MESH_MAX`
  - `PROBE_COUNT`
- Round beds (delta):
  - `MESH_RADIUS`
  - `MESH_ORIGIN`
  - `ROUND_PROBE_COUNT`
- All beds:
  - `RELATIVE_REFERNCE_INDEX`
  - `ALGORITHM`
See the configuration documentation above for details on how each parameter
applies to the mesh.

### Profiles

`BED_MESH_PROFILE SAVE=name LOAD=name REMOVE=name`

After a BED_MESH_CALIBRATE has been performed, it is possible to save the
current mesh state into a named profile.  This makes it possible to load
a mesh without re-probing the bed.  After a profile has been saved using
`BED_MESH_PROFILE SAVE=name` the `SAVE_CONFIG` gcode may be executed
to write the profile to printer.cfg.

Profiles can be loaded by executing `BED_MESH_PROFILE LOAD=name`.

It should be noted that each time a BED_MESH_CALIBRATE occurs, the current
state is automatically saved to the _default_ profile.  If this profile
exists it is automatically loaded when Klipper starts.  If this behavior
is not desirable the _default_ profile can be removed as follows:

`BED_MESH_PROFILE REMOVE=default`

Any other saved profile can be removed in the same fashion, replacing
_default_ with the named profile you wish to remove.

### Output

`BED_MESH_OUTPUT PGP=[0 | 1]`

Outputs the current mesh state to the terminal.  Note that the mesh itself
is output

The PGP parameter is shorthand for "Print Generated Points".  If `PGP=1` is
set, the generated probed points will be output to the terminal:

```
// bed_mesh: generated points
// Index | Tool Adjusted | Probe
// 0 | (11.0, 1.0) | (35.0, 6.0)
// 1 | (62.2, 1.0) | (86.2, 6.0)
// 2 | (113.5, 1.0) | (137.5, 6.0)
// 3 | (164.8, 1.0) | (188.8, 6.0)
// 4 | (216.0, 1.0) | (240.0, 6.0)
// 5 | (216.0, 97.0) | (240.0, 102.0)
// 6 | (164.8, 97.0) | (188.8, 102.0)
// 7 | (113.5, 97.0) | (137.5, 102.0)
// 8 | (62.2, 97.0) | (86.2, 102.0)
// 9 | (11.0, 97.0) | (35.0, 102.0)
// 10 | (11.0, 193.0) | (35.0, 198.0)
// 11 | (62.2, 193.0) | (86.2, 198.0)
// 12 | (113.5, 193.0) | (137.5, 198.0)
// 13 | (164.8, 193.0) | (188.8, 198.0)
// 14 | (216.0, 193.0) | (240.0, 198.0)
```

The "Tool Adjusted" points refer to the nozzle location for each point, and
the "Probe" points refer to the probe location.  Note that when manually
probing the "Probe" points will refer to both the tool and nozzle locations.

### Clear Mesh State

`BED_MESH_CLEAR`

This gcode may be used to clear the internal mesh state.

### Apply X/Y offsets

`BED_MESH_OFFSET [X=<value>] [Y=<value>]`

This is useful for printers with multiple independent extruders, as an offset
is necessary to produce correct Z adjustment after a tool change.  Offsets
should be specified relative to the primary extruder.  That is, a positive
X offset should be specified if the secondary extruder is mounted to the
right of the primary extruder, and a positive Y offset should be specified
if the secondary extruder is mounted "behind" the primary extruder.
