cmake_minimum_required(VERSION 3.5)

# Проект для ESP32-C6
set(PROJECT_NAME "esp32-c6-zigbee-window")
set(PROJECT_VER "1.0.0")

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(${PROJECT_NAME}) 