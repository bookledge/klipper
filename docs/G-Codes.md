# G-Codes

이 문서는 Klipper가 지원하는 명령에 대해 설명합니다. OctoPrint Terminal 탭에 입력할 수 있는 명령입니다.

## G-Code 명령어

Klipper는 다음과 같은 표준 G-Code 명령을 지원합니다:
- 이동 (G0 또는 G1): `G1 [X<위치>] [Y<위치>] [Z<위치>] [E<위치>] [F<속도>]`
- 지연(멈춤): `G4 P<밀리세컨드>`
- 원점으로 복귀: `G28 [X] [Y] [Z]`
- 모터 끄기: `M18` 또는 `M84`
- 현재 동작이 완료될 때까지 대기: `M400`
- 익스트루더 사출에 절대/상대 거리 사용: `M82`, `M83`
- 절대/상대 좌표 사용: `G90`, `G91`
- 위치 설정: `G92 [X<위치>] [Y<위치>] [Z<위치>] [E<위치>]`
- Set speed factor override percentage: `M220 S<백분율>`
- Set extrude factor override percentage: `M221 S<백분율>`
- 가속도 설정: `M204 S<값>` 또는 `M204 P<값> T<값>`
 - 참고: S를 지정하지 않고 P와 T를 모두 지정하면 가속도는 P와 T의 최소값으로 설정됩니다. P 또는 T 중 하나만 지정하면 명령이 적용되지 않습니다.
- 익스트루더(베드) 온도 가져오기: `M105`
- 익스트루더 온도 설정: `M104 [T<인덱스>] [S<온도>]`
- 익스트루더 온도 설정될때 까지 기다리기: `M109 [T<인덱스>] S<온도>`
  - Note: 참고: M109는 항상 온도가 요청된 값으로 안정될 때까지 기다립니다.
- 베드 온도 설정: `M140 [S<온도>]`
- 베드 온도 설정될때 까지 기다리기: `M190 S<온도>`
  - 참고: M190는 항상 온도가 요청된 값으로 안정될 때까지 기다립니다.
- 팬 속도 설정: `M106 S<값>`
- 팬 끄기: `M107`
- 비상 정지: `M112`
- 현재 위치 가져오기: `M114`
- 펌웨어 버전 가져오기: `M115`

위 명령어에 대한 자세한 내용은 
[RepRap G-Code documentation](http://reprap.org/wiki/G-code) 를 참조하십시오.

Klipper 의 목표는 표준 구성에서 일반적인 타사 소프트웨어 
(예: OctoPrint, Printrun, Slic3r, Cura 등)에서 생성된 G-Code 명령을 지원하는 것입니다. 
가능한 모든 G-Code 명령을 지원하는 것이 목표는 아닙니다. 대신 Klipper 는 사람이 읽을 수 있는 
["extended G-Code commands"](#extended-g-code-commands)을 선호합니다.

덜 일반적인 G-Code 명령이 필요한 경우 사용자 지정 
[gcode_macro config section](Config_Reference.md#gcode_macro)으로 이를 구현할 수 
있습니다. 예를 들어, `G12`, `G29`, `G30`, `G31`, `M42`, `M80`, `M81`, `T1` 등을 
구현하는 데 사용할 수 있습니다.

### G-Code SD 카드 명령

Klipper는 [virtual_sdcard config section](Config_Reference.md#virtual_sdcard) 이 
활성화된 경우 다음 표준 G-Code 명령도 지원합니다:
- SD 카드 목록 보기: `M20`
- SD 카드 초기화: `M21`
- SD 파일 선택: `M23 <filename>`
- SD 프린트 시작/재시작: `M24`
- SD 프린트 잠시멈춤: `M25`
- SD 위치 설정: `M26 S<offset>`
- SD 프린트 상태 리포트: `M27`

또한 "virtual_sdcard" 구성 섹션이 활성화된 경우 다음 확장 명령을 사용할 수 있습니다.
- SD 파일 로드 및 프린트: `SDCARD_PRINT_FILE FILENAME=<filename>`
- 파일 언로드 및 SD 상태 지우기: `SDCARD_RESET_FILE`

### arcs

[gcode_arcs config section](Config_Reference.md#gcode_arcs)이 활성화된 경우 
다음 표준 G-Code 명령을 사용할 수 있습니다.
- Controlled Arc Move (G2 or G3): `G2 [X<pos>] [Y<pos>] [Z<pos>]
  [E<pos>] [F<speed>] I<value> J<value>`

### 펌웨어 리트랙션

[firmware_retraction config section](Config_Reference.md#firmware_retraction) 이 
활성화된 경우 다음 표준 G-Code 명령을 사용할 수 있습니다.
- Retract: `G10`
- Unretract: `G11`

### 디스플레이 명령

[display config section](Config_Reference.md#display)이 활성화된 경우 다음 표준 
G-코드 명령을 사용할 수 있습니다.
- Display Message: `M117 <message>`
- Set build percentage: `M73 P<percent>`

### 기타 사용 가능한 G 코드 명령

다음 표준 G-Code 명령을 현재 사용할 수 있지만 사용하지 않는 것이 좋습니다.
- Get Endstop Status: `M119` (대신 QUERY_ENDSTOPS 사용하세요)

## 확장된 G 코드 명령

Klipper는 일반 구성 및 상태에 대해 "확장된" G-Code 명령을 사용합니다. 이러한 확장 
명령은 모두 유사한 형식을 따릅니다. 명령 이름으로 시작하고 하나 이상의 매개변수가 뒤따를 수 있습니다.
예제: `SET_SERVO SERVO=myservo ANGLE=5.3`. 
이 문서에서 명령과 매개변수는 대문자로 표시되지만 대소문자를 구분하지 않습니다. 
(따라서 "SET_SERVO"와 "set_servo"는 모두 동일한 명령을 실행합니다.)

다음 표준 명령이 지원됩니다:
- `QUERY_ENDSTOPS`: 엔드스톱을 조사하고 "triggered" 되었는지 또는 "open" 상태인지 보고합니다.
  이 명령은 일반적으로 엔드스톱이 올바르게 작동하는지 확인하는 데 사용됩니다.
- `QUERY_ADC [NAME=<config_name>] [PULLUP=<value>]`: 구성된 아날로그 핀에 대해 수신된 
  마지막 아날로그 값을 보고합니다. NAME 이 제공되지 않으면 사용 가능한 adc 이름 목록이 보고됩니다. 
  PULLUP 이 제공되면 (Ohms 단위의 값으로) 해당 풀업이 제공된 등가 저항과 함께 원시 아날로그 값이 
  보고됩니다.
- `GET_POSITION`: 툴헤드의 현재 위치에 대한 정보를 반환합니다.
- `SET_GCODE_OFFSET [X=<pos>|X_ADJUST=<adjust>] 
  [Y=<pos>|Y_ADJUST=<adjust>] [Z=<pos>|Z_ADJUST=<adjust>] 
  [MOVE=1 [MOVE_SPEED=<speed>]]`: 향후 G-Code 명령에 적용할 위치 오프셋을 설정합니다.
  것은 일반적으로 Z 베드 오프셋을 가상으로 변경하거나 압출기를 전환할 때 노즐 XY 오프셋을 
  설정하는 데 사용됩니다. 예를 들어 "SET_GCODE_OFFSET Z=0.2"가 전송되면 향후 G-Code 
  이동은 Z 높이에 0.2mm가 추가됩니다.X_ADJUST 스타일 매개변수를 사용하는 경우 조정이 기존 
  오프셋에 추가됩니다. (예: "SET_GCODE_OFFSET Z = -0.2" 다음에 
  "SET_GCODE_OFFSET Z_ADJUST = 0.3"을 입력하면 총 Z 오프셋이 0.1이 됩니다.) 
  "MOVE=1"이 지정되면 지정된 오프셋을 적용하기 위해 툴 헤드 이동이 실행됩니다. 
  (그렇지 않으면 오프셋은 주어진 축을 지정하는 다음 절대 G 코드 이동에 적용됩니다.)
  "MOVE_SPEED"가 지정되면 지정된 속도(mm/s)로 툴 헤드 이동이 수행됩니다; 그렇지 않으면 
  툴 헤드 이동은 마지막으로 지정된 G 코드 속도를 사용합니다.
- `SAVE_GCODE_STATE [NAME=<state_name>]`: 현재 g-code 좌표 구문 분석 상태를 저장합니다. 
  g-code 상태를 저장하고 복원하는 것은 스크립트와 매크로에서 유용합니다. 
  이 명령은 현재 g-code 절대 좌표 모드(G90/G91), 절대 압출 모드(M82/M83), 원점(G92), 
  오프셋(SET_GCODE_OFFSET), 속도 오버라이드(M220), 압출기 오버라이드(M221), 현재 XYZ 위치 
  및 상대 압출기 "E" 위치 그리고 이동 속도를 저장합니다,   
  NAME 이 제공되면 저장된 상태의 이름을 주어진 문자열로 지정할 수 있습니다.
  NAME 이 제공되지 않으면 기본값은 "default" 입니다.
- `RESTORE_GCODE_STATE [NAME=<state_name>]
  [MOVE=1 [MOVE_SPEED=<speed>]]`: SAVE_GCODE_STATE 를 통해 이전에 저장한 상태를 복원합니다.
  "MOVE=1"이 지정되면 이전 XYZ 위치로 다시 이동하기 위해 툴 헤드 이동이 실행됩니다.
  "MOVE_SPEED"가 지정되면 지정된 속도(mm/s)로 툴 헤드 이동이 수행됩니다;
  그렇지 않으면 툴헤드 이동은 복원된 g-코드 속도를 사용합니다.
- `PID_CALIBRATE HEATER=<config_name> TARGET=<temperature>
  [WRITE_FILE=1]`: PID 캘리브레이션 테스트를 수행합니다.
  지정된 히터는 지정된 목표 온도에 도달할 때까지 활성화되며, 그런 다음 히터가 여러 사이클 동안
  꺼졌다 켜집니다. WRITE_FILE 매개변수가 활성화된 경우, /tmp/heattest.txt 파일이 테스트 중에 
  가져온 모든 온도 샘플의 로그와 함께 생성됩니다.
- `TURN_OFF_HEATERS`: 모든 히터를 끕니다.
- `TEMPERATURE_WAIT SENSOR=<config_name> [MINIMUM=<target>] [MAXIMUM=<target>]`:
  주어진 온도 센서가 제공된 최소값 이상 및/또는 제공된 최대값 이하일 때까지 기다립니다.
- `SET_VELOCITY_LIMIT [VELOCITY=<value>] [ACCEL=<value>]
  [ACCEL_TO_DECEL=<value>] [SQUARE_CORNER_VELOCITY=<value>]`: 프린터의 속도 제한을 수정합니다.
- `SET_HEATER_TEMPERATURE HEATER=<heater_name> [TARGET=<target_temperature>]`:
  히터의 목표 온도를 설정합니다. 목표 온도가 제공되지 않으면 목표는 0입니다.
- `ACTIVATE_EXTRUDER EXTRUDER=<config_name>`: 여러 압출기가 있는 프린터에서 
  이 명령은 활성 압출기를 변경하는 데 사용됩니다.
- `SET_PRESSURE_ADVANCE [EXTRUDER=<config_name>] [ADVANCE=<pressure_advance>]
  [SMOOTH_TIME=<pressure_advance_smooth_time>]`: 압력 보정 매개변수를 설정합니다.
  EXTRUDER를 지정하지 않으면 기본적으로 현재 사용중인 익스트루더가 설정됩니다.
- `SET_EXTRUDER_STEP_DISTANCE [EXTRUDER=<config_name>]
  [DISTANCE=<distance>]`: 제공된 익스트루더의  "step distance "에 대한 새 값을 설정합니다. 
  "step distance" 는 `rotation_distance/(full_steps_per_rotation*microsteps)` 입니다.
  Klipper 재설정 시 값이 유지되지 않습니다. 주의해서 사용하십시오. 
  작은 변화로도 압출기와 핫 엔드 사이에 과도한 압력이 발생할 수 있습니다. 
  사용하기 전에 필라멘트로 적절한 보정 단계를 수행하십시오. 
  'DISTANCE' 값이 포함되지 않은 경우 명령은 현재 단계 DISTANCE 를 반환합니다.
- `SET_STEPPER_ENABLE STEPPER=<config_name> ENABLE=[0|1]`: 지정된 스테퍼만 활성화 
  또는 비활성화합니다. 이것은 진단 및 디버깅 도구이며 주의해서 사용해야 합니다. 축 모터를 비활성화해도 
  원점 복귀 정보는 재설정되지 않습니다. 비활성화된 스테퍼를 수동으로 이동하면 기계가 안전 한계를 벗어나 
  모터를 작동할 수 있습니다. 이로 인해 축 구성 요소, 핫 엔드 및 인쇄 표면이 손상될 수 있습니다.
- `STEPPER_BUZZ STEPPER=<config_name>`: 주어진 스테퍼를 앞으로 1mm 이동한 다음 뒤로 1mm 
  이동하고 10회 반복합니다. 이것은 스테퍼 연결을 확인하는 데 도움이 되는 진단 도구입니다.
- `MANUAL_PROBE [SPEED=<speed>]`: 주어진 위치에서 노즐 높이를 측정하는 데 유용한 도우미 
  스크립트를 실행합니다. SPEED가 지정되면 TESTZ 명령의 속도를 설정합니다(기본값은 5mm/s).
  수동 프로브 중에 다음과 같은 추가 명령을 사용할 수 있습니다:
  - `ACCEPT`: 이 명령은 현재 Z 위치를 받아들이고 수동 프로빙을 종료합니다.
  - `ABORT`: 이 명령은 수동 프로빙을 종료합니다.
  - `TESTZ Z=<value>`: 이 명령은 "value" 에 지정된 양만큼 노즐을 위 또는 아래로 이동합니다. 
    예를 들어 `TESTZ Z=-.1`은 노즐을 0.1mm 아래로 이동하고 `TESTZ Z=.1`은 노즐을 0.1mm 
    위로 이동합니다. 값은 또한 '+', '-', '++' 또는 '--'로 노즐을 이전 시도에 비해 위 또는 
    아래로 움직일 수 있습니다.
- `Z_ENDSTOP_CALIBRATE [SPEED=<speed>]`: Z position_endstop 구성 설정을 보정하는 데
   유용한 도우미 스크립트를 실행합니다. 툴이 활성화되어 있는 동안 사용할 수 있는 매개변수 및 추가 
   명령에 대한 자세한 내용은 MANUAL_PROBE 명령을 참조하십시오.
- `Z_OFFSET_APPLY_ENDSTOP`: 현재 Z Gcode 오프셋(일명 babystepping)을 가져 와서 
  stepper_z endstop_position에서 뺍니다.이것은 자주 사용하는 베이비 스테핑 값을 가져 와서 
  "영구화"하는 역할을 합니다. 적용하려면 'SAVE_CONFIG'가 필요합니다.
- `TUNING_TOWER COMMAND=<command> PARAMETER=<name> 
  START=<value> FACTOR=<value> [BAND=<value>]`: 인쇄하는 동안 각 Z 높이에서 매개변수를 
  조정하기 위한 도구입니다. 이것은 `value = start + factor * z_height` 공식을 사용하여 값에 
  지정된 PARAMETER로 지정된 COMMAND를 실행합니다. BAND가 제공되면 z 높이의 BAND 밀리미터마다 
  조정이 이루어집니다. 이 경우 사용되는 공식은 
  `value = start + factor * ((floor(z_height / band) + .5) * band)`입니다.
- `SET_DISPLAY_GROUP [DISPLAY=<display>] GROUP=<group>`: LCD 디스플레이의 활성 
  디스플레이 그룹을 설정합니다. 이를 통해 구성에서 여러 디스플레이 데이터 그룹을 정의할 수 있습니다, 
  예를 들어 `[display_data <group> <elementname>]`을 선택하고 이 확장된 gcode 명령을 
  사용하여 둘 사이를 전환합니다. DISPLAY가 지정되지 않은 경우 기본값은 "display" 
  (기본 디스플레이)입니다.
- `SET_IDLE_TIMEOUT [TIMEOUT=<timeout>]`:  사용자가 유휴 시간 초과(초)를 설정할 수 있습니다.
- `RESTART`: 호스트 소프트웨어가 구성을 다시 로드하고 내부 재설정을 수행합니다. 
  이 명령은 마이크로 컨트롤러에서 오류 상태를 지우지 않습니다. (FIRMWARE_RESTART 참조) 
  새 소프트웨어를 로드하지도 않습니다([FAQ](FAQ.md#how-do-i-upgrade-to-the-latest-software 참조)).
- `FIRMWARE_RESTART`: 이는 RESTART 명령과 유사하지만 마이크로 컨트롤러의 모든 오류 상태도 지웁니다
- `SAVE_CONFIG`: 이 명령은 기본 프린터 config 파일을 덮어쓰고 호스트 소프트웨어를 다시 시작합니다. 
  이 명령은 캘리브레이션 테스트 결과를 저장하기 위해 다른 캘리브레이션 명령과 함께 사용됩니다.
- `STATUS`: Klipper 호스트 소프트웨어 상태를 보고합니다.
- `HELP`: 사용 가능한 확장 G 코드 명령 목록을 보고합니다.

### 매크로 명령

The following command is available when a [gcode_macro config section](Config_Reference.md#gcode_macro) is enabled (also see the [command templates guide](Command_Templates.md)):
- `SET_GCODE_VARIABLE MACRO=<macro_name> VARIABLE=<name> VALUE=<value>`: This command allows one to change the value of a gcode_macro variable at run-time. The provided VALUE is parsed as a Python literal.

### 사용자 정의 핀 명령

The following command is available when an [output_pin config section](Config_Reference.md#output_pin) is enabled:
- `SET_PIN PIN=config_name VALUE=<value> CYCLE_TIME=<cycle_time>`

Note: Hardware PWM does not currently support the CYCLE_TIME parameter and will use the cycle time defined in the config.

### 수동으로 제어되는 팬 명령

The following command is available when a [fan_generic config section](Config_Reference.md#fan_generic) is enabled:
- `SET_FAN_SPEED FAN=config_name SPEED=<speed>` This command sets the speed of a fan. <speed> must be between 0.0 and 1.0.

### Neopixel 및 Dotstar 명령

The following command is available when a [neopixel config section](Config_Reference.md#neopixel) or [dotstar config section](Config_Reference.md#dotstar) is enabled:
- `SET_LED LED=<config_name> RED=<value> GREEN=<value> BLUE=<value> WHITE=<value> [INDEX=<index>] [TRANSMIT=0] [SYNC=1]`: This sets the LED output. 
  Each color `<value>` must be between 0.0 and 1.0. The WHITE option is only valid on RGBW LEDs. If multiple LED chips are daisy-chained then one may specify INDEX to alter the color of just the given chip (1 for the first chip, 2 for the second, etc.). 
  If INDEX is not provided then all LEDs in the daisy-chain will be set to the provided color. 
  If TRANSMIT=0 is specified then the color change will only be made on the next SET_LED command that does not specify TRANSMIT=0; this may be useful in combination with the INDEX parameter to batch multiple updates in a daisy-chain. 
  By default, the SET_LED command will sync it's changes with other ongoing gcode commands. This can lead to undesirable behavior if LEDs are being set while the printer is not printing as it will reset the idle timeout. 
  If careful timing is not needed, the optional SYNC=0 parameter can be specified to apply the changes instantly and not reset the idle timeout.

### 서보 명령어

The following commands are available when a [servo config section](Config_Reference.md#servo) is enabled:
- `SET_SERVO SERVO=config_name [ANGLE=<degrees> | WIDTH=<seconds>]`: Set the servo position to the given angle (in degrees) or pulse width (in seconds). Use `WIDTH=0` to disable the servo output.

### 수동 스테퍼 명령

The following command is available when a [manual_stepper config section](Config_Reference.md#manual_stepper) is enabled:
- `MANUAL_STEPPER STEPPER=config_name [ENABLE=[0|1]] [SET_POSITION=<pos>] [SPEED=<speed>] [ACCEL=<accel>] [MOVE=<pos> [STOP_ON_ENDSTOP=[1|2|-1|-2]] [SYNC=0]]`: This command will alter the state of the stepper. Use the ENABLE parameter to enable/disable the stepper. 
  Use the SET_POSITION parameter to force the stepper to think it is at the given position. Use the MOVE parameter to request a movement to the given position. 
  If SPEED and/or ACCEL is specified then the given values will be used instead of the defaults specified in the config file. 
  If an ACCEL of zero is specified then no acceleration will be performed. 
  If STOP_ON_ENDSTOP=1 is specified then the move will end early should the endstop report as triggered (use STOP_ON_ENDSTOP=2 to complete the move without error even if the endstop does not trigger, use -1 or -2 to stop when the endstop reports not triggered). 
  Normally future G-Code commands will be scheduled to run after the stepper move completes, however if a manual stepper move uses SYNC=0 then future G-Code movement commands may run in parallel with the stepper movement.

### 익스트루더 스테퍼 명령

The following command is available when an [extruder_stepper config section](Config_Reference.md#extruder_stepper) is enabled:
- `SYNC_STEPPER_TO_EXTRUDER STEPPER=<extruder_stepper config_name> [EXTRUDER=<extruder config_name>]`: This command will cause the given STEPPER to become synchronized to the given EXTRUDER, overriding the extruder defined in the "extruder_stepper" config section.

### 프로브

The following commands are available when a [probe config section](Config_Reference.md#probe) is enabled (also see the [probe calibrate guide](Probe_Calibrate.md)):
- `PROBE [PROBE_SPEED=<mm/s>] [LIFT_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>] [SAMPLES_TOLERANCE=<mm>] [SAMPLES_TOLERANCE_RETRIES=<count>] [SAMPLES_RESULT=median|average]`: Move the nozzle downwards until the probe triggers. 
  If any of the optional parameters are provided they override their equivalent setting in the [probe config section](Config_Reference.md#probe).
- `QUERY_PROBE`: Report the current status of the probe ("triggered" or "open").
- `PROBE_ACCURACY [PROBE_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>]`: Calculate the maximum, minimum, average, median, and standard deviation of multiple probe samples. 
  By default, 10 SAMPLES are taken. Otherwise the optional parameters default to their equivalent setting in the probe config section.
- `PROBE_CALIBRATE [SPEED=<speed>] [<probe_parameter>=<value>]`: Run a helper script useful for calibrating the probe's z_offset. 
  See the PROBE command for details on the optional probe parameters. See the MANUAL_PROBE command for details on the SPEED parameter and the additional commands available while the tool is active. 
  Please note, the PROBE_CALIBRATE command uses the speed variable to move in XY direction as well as Z.
- `Z_OFFSET_APPLY_PROBE`: Take the current Z Gcode offset (aka, babystepping), and subtract if from the probe's z_offset.
  This acts to take a frequently used babystepping value, and "make it permanent".  
  Requires a `SAVE_CONFIG` to take effect.

### BLTouch

The following command is available when a [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [BL-Touch guide](BLTouch.md)):
- `BLTOUCH_DEBUG COMMAND=<command>`: This sends a command to the BLTouch. It may be useful for debugging. Available commands are: `pin_down`, `touch_mode`, `pin_up`, `self_test`, `reset`, (*1): `set_5V_output_mode`, `set_OD_output_mode`, `output_mode_store`

  *** Note that the commands marked by (*1) are solely supported by a BL-Touch V3.0 or V3.1(+)

- `BLTOUCH_STORE MODE=<output_mode>`: This stores an output mode in the EEPROM of a BLTouch V3.1 Available output_modes are: `5V`, `OD`

### 델타 캘리브레이션

The following commands are available when the [delta_calibrate config section](Config_Reference.md#linear-delta-kinematics) is enabled (also see the [delta calibrate guide](Delta_Calibrate.md)):
- `DELTA_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>]`: This command will probe seven points on the bed and recommend updated endstop positions, tower angles, and radius. 
  See the PROBE command for details on the optional probe parameters. 
  If METHOD=manual is specified then the manual probing tool is activated - see the MANUAL_PROBE command above for details on the additional commands available while this tool is active.
- `DELTA_ANALYZE`: This command is used during enhanced delta calibration. See [Delta Calibrate](Delta_Calibrate.md) for details.

### BED Tilt

The following commands are available when the [bed_tilt config section](Config_Reference.md#bed_tilt) is enabled:
- `BED_TILT_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>]`: This command will probe the points specified in the config and then recommend updated x and y tilt adjustments. 
  See the PROBE command for details on the optional probe parameters. 
  If METHOD=manual is specified then the manual probing tool is activated - see the MANUAL_PROBE command above for details on the additional commands available while this tool is active.

### Mesh Bed 레벨링

The following commands are available when the [bed_mesh config section](Config_Reference.md#bed_mesh) is enabled (also see the [bed mesh guide](Bed_Mesh.md)):
- `BED_MESH_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>] [<mesh_parameter>=<value>]`: This command probes the bed using generated points specified by the parameters in the config. 
  After probing, a mesh is generated and z-movement is adjusted according to the mesh. See the PROBE command for details on the optional probe parameters. 
  If METHOD=manual is specified then the manual probing tool is activated - see the MANUAL_PROBE command above for details on the additional commands available while this tool is active.
- `BED_MESH_OUTPUT PGP=[<0:1>]`: This command outputs the current probed z values and current mesh values to the terminal.  
  If PGP=1 is specified the x,y coordinates generated by bed_mesh, along with their associated indices, will be output to the terminal.
- `BED_MESH_MAP`: Like to BED_MESH_OUTPUT, this command prints the current state of the mesh to the terminal.  
  Instead of printing the values in a human readable format, the state is serialized in json format. 
  This allows octoprint plugins to easily capture the data and generate height maps approximating the bed's surface.
- `BED_MESH_CLEAR`: This command clears the mesh and removes all z adjustment.  
  It is recommended to put this in your end-gcode.
- `BED_MESH_PROFILE LOAD=<name> SAVE=<name> REMOVE=<name>`: This command provides profile management for mesh state.  
  LOAD will restore the mesh state from the profile matching the supplied name. 
  SAVE will save the current mesh state to a profile matching the supplied name.  
  Remove will delete the profile matching the supplied name from persistent memory.  
  Note that after SAVE or REMOVE operations have been run the SAVE_CONFIG gcode must be run to make the changes to peristent memory permanent.
- `BED_MESH_OFFSET [X=<value>] [Y=<value>]`:  Applies X and/or Y offsets to the mesh lookup.  
  This is useful for printers with independent extruders, as an offset is necessary to produce correct Z adjustment after a tool change.

### Bed Screws Helper

The following commands are available when the [bed_screws config section](Config_Reference.md#bed_screws) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws)):
- `BED_SCREWS_ADJUST`: This command will invoke the bed screws adjustment tool. 
  It will command the nozzle to different locations (as defined in the config file) and allow one to make adjustments to the bed screws so that the bed is a constant distance from the nozzle.

### Bed Screws Tilt Adjust Helper

The following commands are available when the [screws_tilt_adjust config section](Config_Reference.md#screws_tilt_adjust) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws-using-the-bed-probe)):
- `SCREWS_TILT_CALCULATE [DIRECTION=CW|CCW] [<probe_parameter>=<value>]`: This command will invoke the bed screws adjustment tool. 
  It will command the nozzle to different locations (as defined in the config file) probing the z height and calculate the number of knob turns to adjust the bed level. 
  If DIRECTION is specified, the knob turns will all be in the same direction, clockwise (CW) or counterclockwise (CCW). 
  See the PROBE command for details on the optional probe parameters.
  IMPORTANT: You MUST always do a G28 before using this command.

### Z Tilt

The following commands are available when the [z_tilt config section](Config_Reference.md#z_tilt) is enabled:
- `Z_TILT_ADJUST [<probe_parameter>=<value>]`: This command will probe the points specified in the config and then make independent adjustments to each Z stepper to compensate for tilt. 
  See the PROBE command for details on the optional probe parameters.

### Dual Carriages

The following command is available when the zdual_carriage config section](Config_Reference.md#dual_carriage) is enabled:
- `SET_DUAL_CARRIAGE CARRIAGE=[0|1]`: This command will set the active carriage. It is typically invoked from the activate_gcode and deactivate_gcode fields in a multiple extruder configuration.

### TMC stepper drivers

The following commands are available when any of the [tmcXXXX config sections](Config_Reference.md#tmc-stepper-driver-configuration) are enabled:
- `DUMP_TMC STEPPER=<name>`: This command will read the TMC driver registers and report their values.
- `INIT_TMC STEPPER=<name>`: This command will intitialize the TMC registers. Needed to re-enable the driver if power to the chip is turned off then back on.
- `SET_TMC_CURRENT STEPPER=<name> CURRENT=<amps> HOLDCURRENT=<amps>`: This will adjust the run and hold currents of the TMC driver.
  (HOLDCURRENT is not applicable to tmc2660 drivers.)
- `SET_TMC_FIELD STEPPER=<name> FIELD=<field> VALUE=<value>`: This will alter the value of the specified register field of the TMC driver. 
  This command is intended for low-level diagnostics and debugging only because changing the fields during run-time can lead to undesired and potentially dangerous behavior of your printer.
  Permanent changes should be made using the printer configuration file instead. 
  No sanity checks are performed for the given values.

### Endstop adjustments by stepper phase

The following commands are available when an [endstop_phase config section](Config_Reference.md#endstop_phase) is enabled (also see the [endstop phase guide](Endstop_Phase.md)):
- `ENDSTOP_PHASE_CALIBRATE [STEPPER=<config_name>]`: If no STEPPER parameter is provided then this command will reports statistics on endstop stepper phases during past homing operations. 
  When a STEPPER parameter is provided it arranges for the given endstop phase setting to be written to the config file (in conjunction with the SAVE_CONFIG command).

### Force movement

The following commands are available when the [force_move config section](Config_Reference.md#force_move) is enabled:
- `FORCE_MOVE STEPPER=<config_name> DISTANCE=<value> VELOCITY=<value> [ACCEL=<value>]`: This command will forcibly move the given stepper the given distance (in mm) at the given constant velocity (in mm/s). 
  If ACCEL is specified and is greater than zero, then the  given acceleration (in mm/s^2) will be used; otherwise no acceleration is performed. No boundary checks are performed; 
  no kinematic updates are made; 
  other parallel steppers on an axis will not be moved. 
  Use caution as an incorrect command could cause damage! 
  Using this command will almost certainly place the low-level kinematics in an incorrect state; 
  issue a G28 afterwards to reset the kinematics. 
  This command is intended for low-level diagnostics and debugging.
- `SET_KINEMATIC_POSITION [X=<value>] [Y=<value>] [Z=<value>]`: Force the low-level kinematic code to believe the toolhead is at the given cartesian position. 
  This is a diagnostic and debugging command; 
  use SET_GCODE_OFFSET and/or G92 for regular axis transformations. 
  If an axis is not specified then it will default to the position that the head was last commanded to. 
  Setting an incorrect or invalid position may lead to internal software errors. 
  This command may invalidate future boundary checks; issue a G28 afterwards to reset the kinematics.

### SDcard loop

When the [sdcard_loop config section](Config_Reference.md#sdcard_loop) is enabled, the following extended commands are available:
- `SDCARD_LOOP_BEGIN COUNT=<count>`: Begin a looped section in the SD print. A count of 0 indicates that the section should be looped indefinately.
- `SDCARD_LOOP_END`: End a looped section in the SD print.
- `SDCARD_LOOP_DESIST`: Complete existing loops without further iterations.

### Send message (respond) to host

The following commands are availabe when the
[respond config section](Config_Reference.md#respond) is enabled.
- `M118 <message>`: echo the message prepended with the configured
  default prefix (or `echo: ` if no prefix is configured).
- `RESPOND MSG="<message>"`: echo the message prepended with the
  configured default prefix (or `echo: ` if no prefix is configured).
- `RESPOND TYPE=echo MSG="<message>"`: echo the message prepended with
  `echo: `.
- `RESPOND TYPE=command MSG="<message>"`: echo the message prepended
  with `// `.  Octopint can be configured to respond to these messages
  (e.g.  `RESPOND TYPE=command MSG=action:pause`).
- `RESPOND TYPE=error MSG="<message>"`: echo the message prepended
  with `!! `.
- `RESPOND PREFIX=<prefix> MSG="<message>"`: echo the message
  prepended with `<prefix>`. (The `PREFIX` parameter will take
  priority over the `TYPE` parameter)

### Pause Resume

The following commands are available when the
[pause_resume config section](Config_Reference.md#pause_resume) is
enabled:
- `PAUSE`: Pauses the current print. The current position is captured
  for restoration upon resume.
- `RESUME [VELOCITY=<value>]`: Resumes the print from a pause, first
  restoring the previously captured position.  The VELOCITY parameter
  determines the speed at which the tool should return to the original
  captured position.
- `CLEAR_PAUSE`: Clears the current paused state without resuming the
  print. This is useful if one decides to cancel a print after a
  PAUSE. It is recommended to add this to your start gcode to make
  sure the paused state is fresh for each print.
- `CANCEL_PRINT`: Cancels the current print.

### Filament Sensor

The following command is available when the
[filament_switch_sensor or filament_motion_sensor config section](Config_Reference.md#filament_switch_sensor)
is enabled.
- `QUERY_FILAMENT_SENSOR SENSOR=<sensor_name>`: Queries the current
  status of the filament sensor. The data displayed on the terminal
  will depend on the sensor type defined in the confguration.
- `SET_FILAMENT_SENSOR SENSOR=<sensor_name> ENABLE=[0|1]`: Sets the
  filament sensor on/off. If ENABLE is set to 0, the filament sensor
  will be disabled, if set to 1 it is enabled.

### Firmware Retraction

The following commands are available when the
[firmware_retraction config section](Config_Reference.md#firmware_retraction)
is enabled. These commands allow you to utilise the firmware
retraction feature available in many slicers, to reduce stringing
during non-extrusion moves from one part of the print to another.
Appropriately configuring pressure advance reduces the length of
retraction required.
- `SET_RETRACTION [RETRACT_LENGTH=<mm>] [RETRACT_SPEED=<mm/s>]
  [UNRETRACT_EXTRA_LENGTH=<mm>] [UNRETRACT_SPEED=<mm/s>]`: Adjust the
  parameters used by firmware retraction. RETRACT_LENGTH determines
  the length of filament to retract and unretract. The speed of
  retraction is adjusted via RETRACT_SPEED, and is typically set
  relatively high. The speed of unretraction is adjusted via
  UNRETRACT_SPEED, and is not particularly critical, although often
  lower than RETRACT_SPEED. In some cases it is useful to add a small
  amount of additional length on unretraction, and this is set via
  UNRETRACT_EXTRA_LENGTH. SET_RETRACTION is commonly set as part of
  slicer per-filament configuration, as different filaments require
  different parameter settings.
- `GET_RETRACTION`: Queries the current parameters used by firmware
  retraction and displays them on the terminal.
- `G10`: Retracts the extruder using the currently configured
  parameters.
- `G11`: Unretracts the extruder using the currently configured
  parameters.

### Skew Correction

The following commands are available when the
[skew_correction config section](Config_Reference.md#skew_correction)
is enabled (also see the [skew correction guide](skew_correction.md)):
- `SET_SKEW [XY=<ac_length,bd_length,ad_length>] [XZ=<ac,bd,ad>]
  [YZ=<ac,bd,ad>] [CLEAR=<0|1>]`: Configures the [skew_correction]
  module with measurements (in mm) taken from a calibration print.
  One may enter measurements for any combination of planes, planes not
  entered will retain their current value. If `CLEAR=1` is entered
  then all skew correction will be disabled.
- `GET_CURRENT_SKEW`: Reports the current printer skew for each plane
  in both radians and degrees. The skew is calculated based on
  parameters provided via the `SET_SKEW` gcode.
- `CALC_MEASURED_SKEW [AC=<ac_length>] [BD=<bd_length>]
  [AD=<ad_length>]`: Calculates and reports the skew (in radians and
  degrees) based on a measured print. This can be useful for
  determining the printer's current skew after correction has been
  applied. It may also be useful before correction is applied to
  determine if skew correction is necessary. See skew_correction.md
  for details on skew calibration objects and measurements.
- `SKEW_PROFILE [LOAD=<name>] [SAVE=<name>] [REMOVE=<name>]`: Profile
  management for skew_correction. LOAD will restore skew state from
  the profile matching the supplied name. SAVE will save the current
  skew state to a profile matching the supplied name. Remove will
  delete the profile matching the supplied name from persistent
  memory. Note that after SAVE or REMOVE operations have been run the
  SAVE_CONFIG gcode must be run to make the changes to peristent
  memory permanent.

### Delayed GCode

The following command is enabled if a
[delayed_gcode config section](Config_Reference.md#delayed_gcode) has
been enabled (also see the
[template guide](Command_Templates.md#delayed-gcodes)):
- `UPDATE_DELAYED_GCODE [ID=<name>] [DURATION=<seconds>]`:  Updates the
  delay duration for the identified [delayed_gcode] and starts the timer
  for gcode execution.  A value of 0 will cancel a pending delayed gcode
  from executing.

### Save Variables

The following command is enabled if a
[save_variables config section](Config_Reference.md#save_variables)
has been enabled:
- `SAVE_VARIABLE VARIABLE=<name> VALUE=<value>`: Saves the variable to
  disk so that it can be used across restarts. All stored variables
  are loaded into the `printer.save_variables.variables` dict at
  startup and can be used in gcode macros. The provided VALUE is
  parsed as a Python literal.

### Resonance compensation

The following command is enabled if an
[input_shaper config section](Config_Reference.md#input_shaper) has
been enabled (also see the
[resonance compensation guide](Resonance_Compensation.md)):
- `SET_INPUT_SHAPER [SHAPER_FREQ_X=<shaper_freq_x>]
  [SHAPER_FREQ_Y=<shaper_freq_y>]
  [DAMPING_RATIO_X=<damping_ratio_x>]
  [DAMPING_RATIO_Y=<damping_ratio_y>] [SHAPER_TYPE=<shaper>]
  [SHAPER_TYPE_X=<shaper_type_x>] [SHAPER_TYPE_Y=<shaper_type_y>]`:
  Modify input shaper parameters. Note that SHAPER_TYPE parameter
  resets input shaper for both X and Y axes even if different shaper
  types have been configured in [input_shaper] section. SHAPER_TYPE
  cannot be used together with either of SHAPER_TYPE_X and
  SHAPER_TYPE_Y parameters. See
  [config reference](Config_Reference.md#input_shaper) for more
  details on each of these parameters.

### Temperature Fan Commands

The following command is available when a
[temperature_fan config section](Config_Reference.md#temperature_fan)
is enabled:
- `SET_TEMPERATURE_FAN_TARGET temperature_fan=<temperature_fan_name>
  [target=<target_temperature>] [min_speed=<min_speed>]  [max_speed=<max_speed>]`: Sets the target temperature for a
  temperature_fan. If a target is not supplied, it is set to the
  specified temperature in the config file. If speeds are not supplied, no change is applied.

### Adxl345 Accelerometer Commands

The following commands are available when an
[adxl345 config section](Config_Reference.md#adxl345) is enabled:
- `ACCELEROMETER_MEASURE [CHIP=<config_name>] [RATE=<value>]
  [NAME=<value>]`: Starts accelerometer measurements at the requested
  number of samples per second. If CHIP is not specified it defaults
  to "default". Valid rates are 25, 50, 100, 200, 400, 800, 1600,
  and 3200. The command works in a start-stop mode: when executed for
  the first time, it starts the measurements, next execution stops
  them. If RATE is not specified, then the default value is used
  (either from `printer.cfg` or `3200` default value). The results of
  measurements are written to a file named
  `/tmp/adxl345-<chip>-<name>.csv` where `<chip>` is the name of the
  accelerometer chip (`my_chip_name` from `[adxl345 my_chip_name]`) and
  `<name>` is the optional NAME parameter. If NAME is not specified it
  defaults to the current time in "YYYYMMDD_HHMMSS" format. If the
  accelerometer does not have a name in its config section (simply
  `[adxl345]`) <chip> part of the name is not generated.
- `ACCELEROMETER_QUERY [CHIP=<config_name>] [RATE=<value>]`: queries
  accelerometer for the current value. If CHIP is not specified it
  defaults to "default". If RATE is not specified, the default value
  is used. This command is useful to test the connection to the
  ADXL345 accelerometer: one of the returned values should be a
  free-fall acceleration (+/- some noise of the chip).
- `ADXL345_DEBUG_READ [CHIP=<config_name>] REG=<register>`: queries
  ADXL345 register <register> (e.g. 44 or 0x2C). Can be useful for
  debugging purposes.
- `ADXL345_DEBUG_WRITE [CHIP=<config_name>] REG=<reg> VAL=<value>`:
  writes raw <value> into a register <register>. Both <value> and
  <register> can be a decimal or a hexadecimal integer. Use with care,
  and refer to ADXL345 data sheet for the reference.

### Resonance Testing Commands

The following commands are available when a
[resonance_tester config section](Config_Reference.md#resonance_tester)
is enabled (also see the
[measuring resonances guide](Measuring_Resonances.md)):
- `MEASURE_AXES_NOISE`: Measures and outputs the noise for all axes of
  all enabled accelerometer chips.
- `TEST_RESONANCES AXIS=<axis> OUTPUT=<resonances,raw_data>
  [NAME=<name>] [FREQ_START=<min_freq>] [FREQ_END=<max_freq>]
  [HZ_PER_SEC=<hz_per_sec>] [INPUT_SHAPING=[<0:1>]]`: Runs the resonance
  test in all configured probe points for the requested <axis>
  and measures the acceleration using the accelerometer chips configured
  for the respective axis. <axis> can either be X or Y, or specify an
  arbitrary direction as `AXIS=dx,dy`, where dx and dy are floating point
  numbers defining a direction vector (e.g. `AXIS=X`, `AXIS=Y`, or
  `AXIS=1,-1` to define a diagonal direction). Note that `AXIS=dx,dy` and
  `AXIS=-dx,-dy` is equivalent. If `INPUT_SHAPING=0` or not set (default),
  disables input shaping for the resonance testing, because it is not valid
  to run the resonance testing with the input shaper enabled.
  `OUTPUT` parameter is a comma-separated list of which outputs will be
  written. If `raw_data` is requested, then the raw accelerometer data
  is written into a file or a series of files
  `/tmp/raw_data_<axis>_[<point>_]<name>.csv` with (`<point>_` part of
  the name generated only if more than 1 probe point is configured).
  If `resonances` is specified, the frequency response is calculated
  (across all probe points) and written into
  `/tmp/resonances_<axis>_<name>.csv` file. If unset, OUTPUT defaults
  to `resonances`, and NAME defaults to the current time in
  "YYYYMMDD_HHMMSS" format.
- `SHAPER_CALIBRATE [AXIS=<axis>] [NAME=<name>]
  [FREQ_START=<min_freq>] [FREQ_END=<max_freq>]
  [HZ_PER_SEC=<hz_per_sec>] [MAX_SMOOTHING=<max_smoothing>]`:
  Similarly to `TEST_RESONANCES`, runs the resonance test as configured,
  and tries to find the optimal parameters for the input shaper for the
  requested axis (or both X and Y axes if `AXIS` parameter is unset).
  If `MAX_SMOOTHING` is unset, its value is taken from `[resonance_tester]`
  section, with the default being unset. See the
  [Max smoothing](Measuring_Resonances.md#max-smoothing) of the measuring
  resonances guide for more information on the use of this feature.
  The results of the tuning are printed to the console, and the frequency
  responses and the different input shapers values are written to a CSV
  file(s) `/tmp/calibration_data_<axis>_<name>.csv`. Unless specified, NAME
  defaults to the current time in "YYYYMMDD_HHMMSS" format. Note that
  the suggested input shaper parameters can be persisted in the config
  by issuing `SAVE_CONFIG` command.

### Palette 2 Commands

The following command is available when the
[palette2 config section](Config_Reference.md#palette2)
is enabled:
- `PALETTE_CONNECT`: This command initializes the connection with
  the Palette 2.
- `PALETTE_DISCONNECT`: This command disconnects from the Palette 2.
- `PALETTE_CLEAR`: This command instructs the Palette 2 to clear all of the
  input and output paths of filament.
- `PALETTE_CUT`: This command instructs the Palette 2 to cut the filament
  currently loaded in the splice core.
- `PALETTE_SMART_LOAD`: This command start the smart load sequence on the
  Palette 2. Filament is loaded automatically by extruding it the distance
  calibrated on the device for the printer, and instructs the Palette 2
  once the loading has been completed. This command is the same as pressing
  **Smart Load** directly on the Palette 2 screen after the filament load
  is complete.

Palette prints work by embedding special OCodes (Omega Codes)
in the GCode file:
- `O1`...`O32`: These codes are read from the GCode stream and processed
  by this module and passed to the Palette 2 device.
