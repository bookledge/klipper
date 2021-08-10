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

The request dictionary may contain a "params" parameter which must be
of a dictionary type. The "params" provide additional parameter
information to the Klipper "endpoint" handling the request. Its
content is specific to the "endpoint".

The request dictionary may contain an "id" parameter which may be of
any JSON type. If "id" is present then Klipper will respond to the
request with a response message containing that "id". If "id" is
omitted (or set to a JSON "null" value) then Klipper will not provide
any response to the request. A response message is a JSON dictionary
containing "id" and "result". The "result" is always a dictionary -
its contents are specific to the "endpoint" handling the request.

If the processing of a request results in an error, then the response
message will contain an "error" field instead of a "result" field. For
example, the request:
`{"id": 123, "method": "gcode/script", "params": {"script": "G1
X200"}}`
might result in an error response such as:
`{"id": 123, "error": {"message": "Must home axis
first: 200.000 0.000 0.000 [0.000]", "error": "WebRequestError"}}`

Klipper will always start processing requests in the order that they
are received. However, some request may not complete immediately,
which could cause the associated response to be sent out of order with
respect to responses from other requests. A JSON request will never
pause the processing of future JSON requests.

## Subscriptions

Some Klipper "endpoint" requests allow one to "subscribe" to future
asynchronous update messages.

For example:

`{"id": 123, "method": "gcode/subscribe_output", "params":
{"response_template":{"key": 345}}}`

may initially respond with:

`{"id": 123, "result": {}}`

and cause Klipper to send future messages similar to:

`{"params": {"response": "ok B:22.8 /0.0 T0:22.4 /0.0"}, "key": 345}`

A subscription request accepts a "response_template" dictionary in the
"params" field of the request. That "response_template" dictionary is
used as a template for future asynchronous messages - it may contain
arbitrary key/value pairs. When sending these future asynchronous
messages, Klipper will add a "params" field containing a dictionary
with "endpoint" specific contents to the response template and then
send that template. If a "response_template" field is not provided
then it defaults to an empty dictionary (`{}`).

## Available "endpoints"

By convention, Klipper "endpoints" are of the form
`<module_name>/<some_name>`. When making a request to an "endpoint",
the full name must be set in the "method" parameter of the request
dictionary (eg, `{"method"="gcode/restart"}`).

### info

The "info" endpoint is used to obtain system and version information
from Klipper. It is also used to provide the client's version
information to Klipper. For example:
`{"id": 123, "method": "info", "params": { "client_info": { "version":
"v1"}}}`

If present, the "client_info" parameter must be a dictionary, but that
dictionary may have arbitrary contents. Clients are encouraged to
provide the name of the client and its software version when first
connecting to the Klipper API server.

### emergency_stop

The "emergency_stop" endpoint is used to instruct Klipper to
transition to a "shutdown" state. It behaves similarly to the G-Code
`M112` command. For example:
`{"id": 123, "method": "emergency_stop"}`

### register_remote_method

This endpoint allows clients to register methods that can be called
from klipper.  It will return an empty object upon success.

For example:
`{"id": 123, "method": "register_remote_method",
"params": {"response_template": {"action": "run_paneldue_beep"},
"remote_method": "paneldue_beep"}}`
will return:
`{"id": 123, "result": {}}`

The remote method `paneldue_beep` may now be called from Klipper. Note
that if the method takes parameters they should be provided as keyword
arguments. Below is an example of how it may called from a gcode_macro:
```
[gcode_macro PANELDUE_BEEP]
gcode:
  {action_call_remote_method("paneldue_beep", frequency=300, duration=1.0)}
```

When the PANELDUE_BEEP gcode macro is executed, Klipper would send something
like the following over the socket:
`{"action": "run_paneldue_beep",
"params": {"frequency": 300, "duration": 1.0}}`

### objects/list

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
