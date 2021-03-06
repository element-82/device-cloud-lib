#
# Copyright (C) 2017-2018 Wind River Systems, Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software  distributed
# under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES
# OR CONDITIONS OF ANY KIND, either express or implied.
#

set( TARGET "api" )
set( TESTS
	"iot_action"
	"iot_alarm"
	"iot_attribute"
	"iot_base"
	"iot_base64"
	"iot_common"
	"iot_json_decode"
	"iot_json_encode"
	"iot_location"
	"iot_telemetry"
)

if( JSON_DEFINES )
	foreach( JSON_DEFINE ${JSON_DEFINES} )
		set( JSON_DEFINES_ "-D${JSON_DEFINE}=1" )
	endforeach( JSON_DEFINE )
	add_definitions( ${JSON_DEFINES_} )
endif( JSON_DEFINES )

include( "mock_api" )
include( "mock_osal" )
include( "mock_utilities" )

# iot_action.c
set( MOCK_API_PART ${MOCK_API_FUNC} )
list( REMOVE_ITEM MOCK_API_PART
	"iot_action_free"
	"iot_action_process"
)
set( TEST_IOT_ACTION_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_ACTION_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_action_test.c" )
set( TEST_IOT_ACTION_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_ACTION_UNIT "iot_action.c" "iot_base64.c" "iot_common.c" )

# iot_alarm.c
set( MOCK_API_PART ${MOCK_API_FUNC} )
list( REMOVE_ITEM MOCK_API_PART
	"iot_alarm_deregister"
)
set( TEST_IOT_ALARM_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_ALARM_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_alarm_test.c" )
set( TEST_IOT_ALARM_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_ALARM_UNIT "iot_alarm.c" )

# iot_attribute.c
set( TEST_IOT_ATTRIBUTE_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_ATTRIBUTE_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_attribute_test.c" )
set( TEST_IOT_ATTRIBUTE_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_ATTRIBUTE_UNIT "iot_attribute.c" )

# iot_base.c
set( MOCK_API_PART ${MOCK_API_FUNC} )
list( REMOVE_ITEM MOCK_API_PART
	"iot_error"
	"iot_log"
)
set( TEST_IOT_BASE_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_BASE_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_base_test.c" )
set( TEST_IOT_BASE_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_BASE_UNIT "iot_base.c" "iot_base64.c" "iot_common.c" "iot_option.c" )

# iot_base64.c
set( TEST_IOT_BASE64_MOCK )
set( TEST_IOT_BASE64_SRCS ${MOCK_API_SRCS} "iot_base64_test.c" )
set( TEST_IOT_BASE64_LIBS ${MOCK_API_LIBS} )
set( TEST_IOT_BASE64_UNIT "iot_base64.c" )

# iot_common.c
set( TEST_IOT_COMMON_MOCK ${MOCK_API_FUNC} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_COMMON_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_common_test.c" )
set( TEST_IOT_COMMON_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_COMMON_UNIT "iot_common.c" "iot_base64.c" )

# json/iot_json_decode.c
set( MOCK_API_PART ${MOCK_API_FUNC} )
list( REMOVE_ITEM MOCK_API_PART
	"iot_json_decode_array_at"
	"iot_json_decode_array_iterator"
	"iot_json_decode_array_iterator_next"
	"iot_json_decode_array_iterator_value"
	"iot_json_decode_array_size"
	"iot_json_decode_bool"
	"iot_json_decode_initialize"
	"iot_json_decode_integer"
	"iot_json_decode_number"
	"iot_json_decode_object_find"
	"iot_json_decode_object_iterator"
	"iot_json_decode_object_iterator_key"
	"iot_json_decode_object_iterator_next"
	"iot_json_decode_object_iterator_value"
	"iot_json_decode_object_size"
	"iot_json_decode_parse"
	"iot_json_decode_real"
	"iot_json_decode_string"
	"iot_json_decode_terminate"
	"iot_json_decode_type"
)
set( TEST_IOT_JSON_DECODE_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} ${MOCK_UTILITIES_FUNC} )
set( TEST_IOT_JSON_DECODE_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} ${MOCK_UTILITIES_SRCS} "iot_json_decode_test.c" )
set( TEST_IOT_JSON_DECODE_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} ${MOCK_UTILITIES_LIBS} iotutils )
set( TEST_IOT_JSON_DECODE_UNIT "json/iot_json_decode.c" )

# json/iot_json_encode.c
set( MOCK_API_PART ${MOCK_API_FUNC} )
list( REMOVE_ITEM MOCK_API_PART
	"iot_json_encode_array_end"
	"iot_json_encode_array_start"
	"iot_json_encode_bool"
	"iot_json_encode_dump"
	"iot_json_encode_initialize"
	"iot_json_encode_integer"
	"iot_json_encode_object_end"
	"iot_json_encode_object_start"
	"iot_json_encode_real"
	"iot_json_encode_string"
	"iot_json_encode_terminate"
)
set( TEST_IOT_JSON_ENCODE_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} ${MOCK_UTILITIES_FUNC} )
set( TEST_IOT_JSON_ENCODE_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} ${MOCK_UTILITIES_SRCS} "iot_json_encode_test.c" )
set( TEST_IOT_JSON_ENCODE_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} ${MOCK_UTILITIES_LIBS} iotutils )
set( TEST_IOT_JSON_ENCODE_UNIT "json/iot_json_encode.c" )

# iot_location.c
set( TEST_IOT_LOCATION_MOCK ${MOCK_API_FUNC} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_LOCATION_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_location_test.c" )
set( TEST_IOT_LOCATION_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_LOCATION_UNIT "iot_location.c" )

# iot_telemetry.c
set( MOCK_API_PART ${MOCK_API_FUNC} )
list( REMOVE_ITEM MOCK_API_PART
	"iot_telemetry_free"
)
set( TEST_IOT_TELEMETRY_MOCK ${MOCK_API_PART} ${MOCK_OSAL_FUNC} )
set( TEST_IOT_TELEMETRY_SRCS ${MOCK_API_SRCS} ${MOCK_OSAL_SRCS} "iot_telemetry_test.c" )
set( TEST_IOT_TELEMETRY_LIBS ${MOCK_API_LIBS} ${MOCK_OSAL_LIBS} )
set( TEST_IOT_TELEMETRY_UNIT "iot_telemetry.c" "iot_base64.c" "iot_common.c" )

include( TestSupport )
add_tests( ${TARGET} ${TESTS} )

