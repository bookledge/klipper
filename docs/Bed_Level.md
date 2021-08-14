# 베드 레벨링

베드레벨링(때로는 베드 트램밍tramming으로 불려지기도 한다)은 고품질 출력을 얻는데 결정적이다. 
만약 베드레벨이 제대로 맞춰져 있지 않으면 베드 안착에 문제가 생길 수 있다. 그래서 모서리 들뜸이나 프린트 전반에 걸친 자잘하나 문제들이 일어날 수 있다. 
이 문서는 클리퍼에서 베드레벨을 시행하는데 가이드를 제공한다. 

베드레벨링의 목표를 이해하는 것이 중요하다. 만약 프린터가 `X0 Y0 Z10` 위치로 가라고 명령을 받았다면
노즐은 프린터의 베드로 부터 정확히 10mm 위에 있어야 한다. 
연이어서 `X50 Z10`으로 이동하라고 명령을 하면 노즐은 베드와의 간격을 정확히 10mm 유지한 상태에서 수평으로 이동해야 한다. 

고품질 출력물을 얻기 위해 프린터는 약 0.025mm (25마이크로미터) 내의 정확도로 Z 높이를 캘리브레이션 해야 한다. 
이정도 거리는 보통의 사람 머리카락두께보다 훨씬 더 얇은 값이다. 
이 스케일은 눈으로는 측정될 수 없는 값이다. 열팽창와 같은 아주 미세한 것도 이정도 스케일에는 영향을 미칠 수 있다. 
높은 정확도의 레벨링을 얻을 수 있는 비밀은 반복에 있다. 그리고 프린터 자체의 모션시스템의 정확성을 이용하는데 있다. 

## 적절한 캘리브레이션 메카니즘을 선택하라

프린터들마다 각자 저마다의 베드레벨링 방식을 채택하고 있다. 
그것들 대부분은 결국에는 아래 기술한 "종이테스트"에 의존한다. 
특별한 타입의 프린터를 위한 실제 과정은 다른 문서에서 소개하겠다.

이 캘리브레이션 도구들을 이용하기전에 [config check document](Config_checks.md) 에 기재된 내용으로 꼭 체크해보기 바란다. 
베드레벨링을 하기 앞서 프린터의 기본동작상태를 점검하는 것은 필수적이다. 

오토레벨링센서가 장착된 프린터의 경우는 [Probe Calibrate](Probe_Calibrate.md) 문서에 있는 지시내용을 따라 반드시 캘리브레이션 하도록 하라. 
델타 프린터라면 [Delta Calibrate](Delta_Calibrate.md) 를 참고하기 바란다.
그리고, 베드나사와 전통적인 Z endstop 을 가진 프린터는 [Manual Level](Manual_Level.md) 문서를 참고하라.


During calibration it may be necessary to set the printer's Z
`position_min` to a negative number (eg, `position_min = -2`). The
printer enforces boundary checks even during calibration
routines. Setting a negative number allows the printer to move below
the nominal position of the bed, which may help when trying to
determine the actual bed position.

## The "paper test"

The primary bed calibration mechanism is the "paper test". It involves
placing a regular piece of "copy machine paper" between the printer's
bed and nozzle, and then commanding the nozzle to different Z heights
until one feels a small amount of friction when pushing the paper back
and forth.

It is important to understand the "paper test" even if one has an
"automatic Z probe". The probe itself often needs to be calibrated to
get good results. That probe calibration is done using this "paper
test".

In order to perform the paper test, cut a small rectangular piece of
paper using a pair of scissors (eg, 5x3 cm). The paper generally has a
width of around 100 microns (0.100mm). (The exact width of the paper
isn't crucial.)

The first step of the paper test is to inspect the printer's nozzle
and bed. Make sure there is no plastic (or other debris) on the nozzle
or bed.

**Inspect the nozzle and bed to ensure no plastic is present!**

If one always prints on a particular tape or printing surface then one
may perform the paper test with that tape/surface in place. However,
note that tape itself has a width and different tapes (or any other
printing surface) will impact Z measurements. Be sure to rerun the
paper test to measure each type of surface that is in use.

If there is plastic on the nozzle then heat up the extruder and use a
metal tweezers to remove that plastic. Wait for the extruder to fully
cool to room temperature before continuing with the paper test. While
the nozzle is cooling, use the metal tweezers to remove any plastic
that may ooze out.

**Always perform the paper test when both nozzle and bed are at room
temperature!**

When the nozzle is heated, its position (relative to the bed) changes
due to thermal expansion. This thermal expansion is typically around a
100 microns, which is about the same width as a typical piece of
printer paper. The exact amount of thermal expansion isn't crucial,
just as the exact width of the paper isn't crucial. Start with the
assumption that the two are equal (see below for a method of
determining the difference between the two widths).

It may seem odd to calibrate the distance at room temperature when the
goal is to have a consistent distance when heated. However, if one
calibrates when the nozzle is heated, it tends to impart small amounts
of molten plastic on to the paper, which changes the amount of
friction felt. That makes it harder to get a good calibration.
Calibrating while the bed/nozzle is hot also greatly increases the
risk of burning oneself. The amount of thermal expansion is stable, so
it is easily accounted for later in the calibration process.

**Use an automated tool to determine precise Z heights!**

Klipper has several helper scripts available (eg, MANUAL_PROBE,
Z_ENDSTOP_CALIBRATE, PROBE_CALIBRATE, DELTA_CALIBRATE). See the
documents
[described above](#choose-the-appropriate-calibration-mechanism) to
choose one of them.

Run the appropriate command in the OctoPrint terminal window. The
script will prompt for user interaction in the OctoPrint terminal
output. It will look something like:
```
Recv: // Starting manual Z probe. Use TESTZ to adjust position.
Recv: // Finish with ACCEPT or ABORT command.
Recv: // Z position: ?????? --> 5.000 <-- ??????
```

The current height of the nozzle (as the printer currently understands
it) is shown between the "--> <--". The number to the right is the
height of the last probe attempt just greater than the current height,
and to the left is the last probe attempt less than the current height
(or ?????? if no attempt has been made).

Place the paper between the nozzle and bed. It can be useful to fold a
corner of the paper so that it is easier to grab. (Try not to push
down on the bed when moving the paper back and forth.)

![paper-test](img/paper-test.jpg)

Use the TESTZ command to request the nozzle to move closer to the
paper. For example:
```
TESTZ Z=-.1
```

The TESTZ command will move the nozzle a relative distance from the
nozzle's current position. (So, `Z=-.1` requests the nozzle to move
closer to the bed by .1mm.) After the nozzle stops moving, push the
paper back and forth to check if the nozzle is in contact with the
paper and to feel the amount of friction. Continue issuing TESTZ
commands until one feels a small amount of friction when testing with
the paper.

If too much friction is found then one can use a positive Z value to
move the nozzle up. It is also possible to use `TESTZ Z=+` or `TESTZ
Z=-` to "bisect" the last position - that is to move to a position
half way between two positions. For example, if one received the
following prompt from a TESTZ command:
```
Recv: // Z position: 0.130 --> 0.230 <-- 0.280
```
Then a `TESTZ Z=-` would move the nozzle to a Z position of 0.180
(half way between 0.130 and 0.230). One can use this feature to help
rapidly narrow down to a consistent friction. It is also possible to
use `Z=++` and `Z=--` to return directly to a past measurement - for
example, after the above prompt a `TESTZ Z=--` command would move the
nozzle to a Z position of 0.130.

After finding a small amount of friction run the ACCEPT command:
```
ACCEPT
```
This will accept the given Z height and proceed with the given
calibration tool.

The exact amount of friction felt isn't crucial, just as the amount of
thermal expansion and exact width of the paper isn't crucial. Just try
to obtain the same amount of friction each time one runs the test.

If something goes wrong during the test, one can use the `ABORT`
command to exit the calibration tool.

## Determining Thermal Expansion

After successfully performing bed leveling, one may go on to calculate
a more precise value for the combined impact of "thermal expansion",
"width of the paper", and "amount of friction felt during the paper
test".

This type of calculation is generally not needed as most users find
the simple "paper test" provides good results.

The easiest way to make this calculation is to print a test object
that has straight walls on all sides. The large hollow square found in
[docs/prints/square.stl](prints/square.stl) can be used for this.
When slicing the object, make sure the slicer uses the same layer
height and extrusion widths for the first level that it does for all
subsequent layers. Use a coarse layer height (the layer height should
be around 75% of the nozzle diameter) and do not use a brim or raft.

Print the test object, wait for it to cool, and remove it from the
bed. Inspect the lowest layer of the object. (It may also be useful to
run a finger or nail along the bottom edge.) If one finds the bottom
layer bulges out slightly along all sides of the object then it
indicates the nozzle was slightly closer to the bed then it should
be. One can issue a `SET_GCODE_OFFSET Z=+.010` command to increase the
height. In subsequent prints one can inspect for this behavior and
make further adjustment as needed. Adjustments of this type are
typically in 10s of microns (.010mm).

If the bottom layer consistently appears narrower than subsequent
layers then one can use the SET_GCODE_OFFSET command to make a
negative Z adjustment. If one is unsure, then one can decrease the Z
adjustment until the bottom layer of prints exhibit a small bulge, and
then back-off until it disappears.

The easiest way to apply the desired Z adjustment is to create a
START_PRINT g-code macro, arrange for the slicer to call that macro
during the start of each print, and add a SET_GCODE_OFFSET command to
that macro. See the [slicers](Slicers.md) document for further
details.
