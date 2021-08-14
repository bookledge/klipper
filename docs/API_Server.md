# API 서버

이 문서는 클리퍼 API 에 대해 기술하고 있습니다. 
이 인터페이스는 외부 어플리케이션을 쿼리할 수 있게 해주고
클리퍼 호스트 소프트웨어를 조정할 수 있게 해줍니다.

## API Socket 활성화

API 서버를 사용하기 위해서, klippy.py 호스트 소프트웨어가 '-a' 파라메터와 함께 시작되어야 합니다. 
예를 들어
```
~/klippy-env/bin/python ~/klipper/klippy/klippy.py ~/printer.cfg -a /tmp/klippy_uds -l /tmp/klippy.log
```

이는 호스트 소프트웨어가 Unix Domain Socket 을 생성하도록 하며
그리고 나선 클라이언트가 그 Socket의 연결을 오픈하고, 클리퍼에서 명령을 보낼 수 있습니다. 

## Request format

소켓에 보내지고 소신된 메시지들은 JSON 인코딩된 문자열입니다. 
이 문자열들은 ASCII 0x03 문자로 종료됩니다.
```
<json_object_1><0x03><json_object_2><0x03>...
```

Klipper 는 `scripts/whconsole.py` 도구를 포함하고 있습니다.
이 도구는 상위 메시지 프레이밍을 수행할 수 있습니다. 
예를 들어 : 
```
~/klipper/scripts/whconsole.py /tmp/klippy_uds
```

이 도구는 stdin 으로부터 연속된 JSON 명령어들을 읽을 수 있습니다. 
또한 그 읽은 명령어들을 Klipper 로 보내고 결과를 보고할 수 있습니다. 
그 도구는 각 JSON 명령을 한줄로 인식하며  
요청을 보낼때 자동으로 0x03 종료자를 추가할 것입니다. 
(Klipper API 서버는 newline 요청을 가지고 있지 않습니다.)

## API 프로토콜

통신소켓에 사용된 명령 프로토콜은 [json-rpc](https://www.jsonrpc.org/) 에 영감을 받았습니다.

요청은 다음과 같을 수 있습니다 :

`{"id": 123, "method": "info", "params": {}}`

그리고 응답은 아래와 같을 수 있습니다 : 

`{"id": 123, "result": {"state_message": "Printer is ready",
"klipper_path": "/home/pi/klipper", "config_file":
"/home/pi/printer.cfg", "software_version": "v0.8.0-823-g883b1cb6",
"hostname": "octopi", "cpu_info": "4 core ARMv7 Processor rev 4
(v7l)", "state": "ready", "python_path":
"/home/pi/klippy-env/bin/python", "log_file": "/tmp/klippy.log"}}`

각 요청은 JSON 딕셔너리여야 합니다. 
(이 문서는 "JSON object" - key/value 쌍은 `{}`를 포함하여 맵핑 - 
를 표현하기 위해 파이썬 용어 사전을 사용합니다.)

요청 딕셔너리는 "method" 파라메터를 반드시 포함해야 합니다. 
이 파라메터는 사용가능한 Klipper "endpoint"의 문자열 이름입니다. 

요청 딕셔너리는 "params" 파라메터를 포함할 수 있습니다. 이 파라메터는 반드시 딕셔너리 타입이어야 합니다.
"params" 는 클리퍼 "endpoint"가 요청을 다룰 수 있도록 추가적인 파라메터를 제공합니다. 
그 내용은 "endpoint"에 특정됩니다. 

요청 딕셔너리는 "id" 파라메터를 포함할 수 있습니다. 
이 파라메터는 JSON 타입이 될 수 있습니다. 
만일 "id" 가 존재한다면 클리퍼는 "id"를 포함한 응답 메시지와 함께 요청에 반응할 것입니다.
민일 "id" 가 생략되어 있다면 (혹은 JSON "null"값이 셋팅되어 있다면) 
클리퍼는 요청에 대해 어떤 응답도 제공하지 않을 것입니다.  
응답 메시지는 "id" 와 "result" 를 포함한 JSON 딕셔너리입니다. 
"result" 는 항상 딕셔너리 입니다. 
그것의 내용은 요청을 다루는 "endpoint"에 특정됩니다. 

만일 요청한 프로세싱이 에러가 나면,
응답메시지는 "result" 필드 대신에 "error" 필드르 포함할 것입니다. 
예를 들어, 요청이 : 
`{"id": 123, "method": "gcode/script", "params": {"script": "G1
X200"}}`
이렇게 들어가면 다음과 같은 결과를 냅니다 :
`{"id": 123, "error": {"message": "Must home axis
first: 200.000 0.000 0.000 [0.000]", "error": "WebRequestError"}}`

클리퍼는 항상 받은 순서대로 요청 프로세싱을 시작합니다. 하지만
몇몇 요청들은 즉각 완료되지 않을 수 있습니다. 
이로인해 관련된 응답이 다른 요청에 대한 응답에 대응하는 순서르 벗어나 보내어질 수 있습니다. 
JSON 요청은 절대로 미래의 JSON 요청의 프로세싱들을 중단시키지 않을 것입니다. 

## 구독

몇몇 클리퍼의 "endpoint" 요청들은 미래의 비동기적인 업데이트 메시지 구독을 허락합니다. 

예를 들면:

`{"id": 123, "method": "gcode/subscribe_output", "params":
{"response_template":{"key": 345}}}`

이것은 기초적으로 다음과 같이 응답합니다.

`{"id": 123, "result": {}}`

그리고, 클리퍼로 하여금 다음과 같은 미래 메시지들을 보내게 합니다. 

`{"params": {"response": "ok B:22.8 /0.0 T0:22.4 /0.0"}, "key": 345}`

구독 요청은 요청에 대한 "params" 필드에 있는 "response_template" 딕셔너리를 받아들입니다.
"response_template" 딕셔너리는 미래 미동기 메시지를 위한 템플릿으로 사용됩니다. 
그것은 임의의 키/값 쌍을 포함할 수 있습니다. 
이러한 미래 비동기 메시지들을 보내려 할 때 
클리퍼는 응답 템플릿에 "endpoint" 특정 딕셔너리 내용을 포함하고 있는 "params" 필드를 더할 것입니다. 
그리고, 그 템플릿을 보내게 됩니다. 
만약 "response_template" 필드가 제공되지 않는다면 빈 딕셔너리 (`{}`) 를 디폴트로 하게 됩니다.

## 사용가능한 "endpoints"

관행적으로 클리퍼 "endpoints"는 `<module_name>/<some_name>` 형식으로 이뤄집니다. 
"endpoint" 에 요청을 하고자 할 때,
full name 은 요청 딕셔너리의 "method" 파라메터에 셋팅되어 있어야만 합니다. 
(예, `{"method"="gcode/restart"}`).

### 정보

 "info" endpoint 클리퍼로 부터 시스템과 버전 정보를 가져오 수 있습니다. 
또한 클리퍼에 클라이언트 버전 정보르 제공하는데도 사용될 수 있습니다. 
예시 :
`{"id": 123, "method": "info", "params": { "client_info": { "version":
"v1"}}}`

만이 존재한다면, "client_info" 파라메터는 딕셔너리여야만 합니다. 
그러나 그 딕셔너리는 임의의 내용들을 가질 수 있습니다. 
클라이언트는 처음 클리퍼 API 서버와 연결될 때 클라이언트의 이름이나 소프트웨어 버전을 제공할 수 있습니다. 

### 비상정지(emergency_stop)

"emergency_stop" endpoint 는 클리퍼가 "shutdown" 상태로 가도록 지시하는데 사용합니다. 
그것은 `M112` gcode 와 비슷하게 행동합니다. 
예시 :
`{"id": 123, "method": "emergency_stop"}`

### 레지스터 원격 메쏘드(register_remote_method)

이 endpoint 는 클라이언트가 클리퍼로부터 불려지 수 있도록 메쏘드를 등록하게 해줍니다. 
이는 성공시 비어있는 객체를 리턴합니다. 

예시 :
`{"id": 123, "method": "register_remote_method",
"params": {"response_template": {"action": "run_paneldue_beep"},
"remote_method": "paneldue_beep"}}`
반환값 :
`{"id": 123, "result": {}}`

원격 메쏘드  `paneldue_beep` 는 클리퍼로 부터 불려질 수 있습니다. 
만일 메쏘드가 파라메터들을 가지게 되면 그것들은 키워드 아규먼트들로써 제공될 수 있음을 기억하십시오.
아래 어떻게 gcode 메크로에서 불러올 수 있는지에 대한 예시가 있습니다. :


```
[gcode_macro PANELDUE_BEEP]
gcode:
  {action_call_remote_method("paneldue_beep", frequency=300, duration=1.0)}
```

PANELDUE_BEEP gcode 매크로가 실행되면 
클리퍼는 소켓위에 아래와 같은 것을 내보낼 것이다.:

`{"action": "run_paneldue_beep",
"params": {"frequency": 300, "duration": 1.0}}`

### 객체/리스트 (objects/list)


This endpoint queries the list of available printer "objects" that one
may query (via the "objects/query" endpoint). For example:
`{"id": 123, "method": "objects/list"}`
might return:
`{"id": 123, "result": {"objects":
["webhooks", "configfile", "heaters", "gcode_move", "query_endstops",
"idle_timeout", "toolhead", "extruder"]}}`

### objects/query

This endpoint allows one to query information from printer objects.
For example:
`{"id": 123, "method": "objects/query", "params": {"objects":
{"toolhead": ["position"], "webhooks": null}}}`
might return:
`{"id": 123, "result": {"status": {"webhooks": {"state": "ready",
"state_message": "Printer is ready"}, "toolhead": {"position":
[0.0, 0.0, 0.0, 0.0]}}, "eventtime": 3051555.377933684}}`

The "objects" parameter in the request must be a dictionary containing
the printer objects that are to be queried - the key contains the
printer object name and the value is either "null" (to query all
fields) or a list of field names.

The response message will contain a "status" field containing a
dictionary with the queried information - the key contains the printer
object name and the value is a dictionary containing its fields. The
response message will also contain an "eventtime" field containing the
timestamp from when the query was taken.

Available fields are documented in the
[Status Reference](Status_Reference.md) document.

### objects/subscribe

This endpoint allows one to query and then subscribe to information
from printer objects. The endpoint request and response is identical
to the "objects/query" endpoint. For example:
`{"id": 123, "method": "objects/subscribe", "params":
{"objects":{"toolhead": ["position"], "webhooks": ["state"]},
"response_template":{}}}`
might return:
`{"id": 123, "result": {"status": {"webhooks": {"state": "ready"},
"toolhead": {"position": [0.0, 0.0, 0.0, 0.0]}},
"eventtime": 3052153.382083195}}`
and result in subsequent asynchronous messages such as:
`{"params": {"status": {"webhooks": {"state": "shutdown"}},
"eventtime": 3052165.418815847}}`

### gcode/help

This endpoint allows one to query available G-Code commands that have
a help string defined. For example:
`{"id": 123, "method": "gcode/help"}`
might return:
`{"id": 123, "result": {"RESTORE_GCODE_STATE": "Restore a previously
saved G-Code state", "PID_CALIBRATE": "Run PID calibration test",
"QUERY_ADC": "Report the last value of an analog pin", ...}}`

### gcode/script

This endpoint allows one to run a series of G-Code commands. For example:
`{"id": 123, "method": "gcode/script", "params": {"script": "G90"}}`

If the provided G-Code script raises an error, then an error response
is generated. However, if the G-Code command produces terminal output,
that terminal output is not provided in the response. (Use the
"gcode/subscribe_output" endpoint to obtain G-Code terminal output.)

If there is a G-Code command being processed when this request is
received, then the provided script will be queued. This delay could be
significant (eg, if a G-Code wait for temperature command is running).
The JSON response message is sent when the processing of the script
fully completes.

### gcode/restart

This endpoint allows one to request a restart - it is similar to
running the G-Code "RESTART" command. For example:
`{"id": 123, "method": "gcode/restart"}`

As with the "gcode/script" endpoint, this endpoint only completes
after any pending G-Code commands complete.

### gcode/firmware_restart

This is similar to the "gcode/restart" endpoint - it implements the
G-Code "FIRMWARE_RESTART" command. For example:
`{"id": 123, "method": "gcode/firmware_restart"}`

As with the "gcode/script" endpoint, this endpoint only completes
after any pending G-Code commands complete.

### gcode/subscribe_output

This endpoint is used to subscribe to G-Code terminal messages that
are generated by Klipper. For example:
`{"id": 123, "method": "gcode/subscribe_output", "params":
{"response_template":{}}}`
might later produce asynchronous messages such as:
`{"params": {"response": "// Klipper state: Shutdown"}}`

This endpoint is intended to support human interaction via a "terminal
window" interface. Parsing content from the G-Code terminal output is
discouraged. Use the "objects/subscribe" endpoint to obtain updates on
Klipper's state.

### pause_resume/cancel

This endpoint is similar to running the "PRINT_CANCEL" G-Code command.
For example:
`{"id": 123, "method": "pause_resume/cancel"}`

As with the "gcode/script" endpoint, this endpoint only completes
after any pending G-Code commands complete.

### pause_resume/pause

This endpoint is similar to running the "PAUSE" G-Code command. For
example:
`{"id": 123, "method": "pause_resume/pause"}`

As with the "gcode/script" endpoint, this endpoint only completes
after any pending G-Code commands complete.

### pause_resume/resume

This endpoint is similar to running the "RESUME" G-Code command. For
example:
`{"id": 123, "method": "pause_resume/resume"}`

As with the "gcode/script" endpoint, this endpoint only completes
after any pending G-Code commands complete.

### query_endstops/status

This endpoint will query the active endpoints and return their status.
For example:
`{"id": 123, "method": "query_endstops/status"}`
might return:
`{"id": 123, "result": {"y": "open", "x": "open", "z": "TRIGGERED"}}`

As with the "gcode/script" endpoint, this endpoint only completes
after any pending G-Code commands complete.
