#
#    Copyright (c) 2022-2024 Project CHIP Authors
#
#    Licensed under the Apache License, Version 2.0 (the "License");
#    you may not use this file except in compliance with the License.
#    You may obtain a copy of the License at
#
#        http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS,
#    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#    See the License for the specific language governing permissions and
#    limitations under the License.
#

cmake_minimum_required(VERSION 3.13.1)

get_filename_component(CHIP_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/third_party/connectedhomeip REALPATH)
get_filename_component(TELINK_COMMON ${CHIP_ROOT}/examples/platform/telink REALPATH)
get_filename_component(GEN_DIR ${CHIP_ROOT}/zzz_generated/ REALPATH)

include(${TELINK_COMMON}/common.cmake)
include(${CHIP_ROOT}/config/telink/app/enable-gnu-std.cmake)
include(${CHIP_ROOT}/src/app/chip_data_model.cmake)

project(chip-telink-lighting-example)

target_include_directories(app PRIVATE
                           include
                           ${GEN_DIR}/app-common
                           ${GEN_DIR}/lighting-app
                           ${TELINK_COMMON}/common/include
                           ${TELINK_COMMON}/util/include
                           ${TELINK_COMMON}/app/include
                           ${TELINK_COMMON}/zephyr_ext)

if(CONFIG_TFLM_FEATURE)
  target_include_directories(app PRIVATE
                             src
                             src/tflm/micro_speech
                             src/tflm/models
                             ${ZEPHYR_BASE}/../modules/lib/tflite-micro-telink
                             ${ZEPHYR_BASE}/../modules/lib/tflite-micro-telink/third_party/ruy
                             ${ZEPHYR_BASE}/../modules/lib/tflite-micro-telink/third_party/gemmlowp
                             ${ZEPHYR_BASE}/../modules/lib/tflite-micro-telink/third_party/flatbuffers/include
                             ${ZEPHYR_BASE}/../modules/lib/tflite-micro-telink/third_party/kissfft/tools
                             ${ZEPHYR_BASE}/../modules/lib/tflite-micro-telink/third_party/kissfft)
endif()

target_sources(app PRIVATE
               src/AppTask.cpp
               src/ZclCallbacks.cpp
               ${TELINK_COMMON}/common/src/mainCommon.cpp
               ${TELINK_COMMON}/common/src/AppTaskCommon.cpp
               ${TELINK_COMMON}/util/src/LEDManager.cpp
               ${TELINK_COMMON}/util/src/ButtonManager.cpp
               ${TELINK_COMMON}/util/src/ThreadUtil.cpp
               ${TELINK_COMMON}/util/src/PWMManager.cpp
               ${TELINK_COMMON}/util/src/ColorFormat.cpp
               ${TELINK_COMMON}/zephyr_ext/zephyr_key_matrix.c
               ${TELINK_COMMON}/zephyr_ext/zephyr_key_pool.c
               ${TELINK_COMMON}/zephyr_ext/zephyr_led_pool.c
               ${TELINK_COMMON}/zephyr_ext/zephyr_ws2812.c)

if(CONFIG_TFLM_FEATURE)
  target_sources(app PRIVATE
                 src/tflm/audio/app_audio.c
                 src/tflm/audio/app_codec.c
                 src/tflm/audio/audio_common.c
                 src/tflm/models/audio_preprocessor_int8_model_data.cpp
                 src/tflm/models/audio_preprocessor_int8_model_data.h
                 src/tflm/models/micro_speech_quantized_model_data.cpp
                 src/tflm/models/micro_speech_quantized_model_data.h
                 src/tflm/micro_speech/micro_model_settings.h
                 src/tflm/micro_speech/tflite_micro_micro_speech.cpp
                 src/tflm/micro_speech/tflite_micro_micro_speech.h
                 src/tflm/errno_stub.c)
endif()

chip_configure_data_model(app
    ZAP_FILE ${CMAKE_CURRENT_SOURCE_DIR}/../lighting-common/lighting-app.zap
)

if(CONFIG_PWM)
	target_sources(app PRIVATE 
                   ${TELINK_COMMON}/zephyr_ext/zephyr_pwm_pool.c)
endif()

if(CONFIG_BOOTLOADER_MCUBOOT)
  target_sources(app PRIVATE ${TELINK_COMMON}/util/src/OTAUtil.cpp)
endif()

if (CONFIG_CHIP_PW_RPC)
  include(${TELINK_COMMON}/Rpc.cmake)
endif(CONFIG_CHIP_PW_RPC)
