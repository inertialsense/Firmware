############################################################################
#
#   Copyright (c) 2015 PX4 Development Team. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in
#    the documentation and/or other materials provided with the
#    distribution.
# 3. Neither the name PX4 nor the names of its contributors may be
#    used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
# INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
# BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS
# OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
# AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
# ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#
############################################################################

include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(geometry_files
	quad_x.toml
	quad_h.toml
	quad_plus.toml
	quad_wide.toml
	quad_deadcat.toml
	quad_vtail.toml
	quad_s250aq.toml
	quad_y.toml
	quad_x_pusher.toml
	hex_x.toml
	hex_plus.toml
	hex_cox.toml
	hex_t.toml
	octa_x.toml
	octa_plus.toml
	octa_cox.toml
	octa_cox_wide.toml
	twin_engine.toml
	tri_y.toml
	dodeca_top_cox.toml
	dodeca_bottom_cox.toml
)

set(geometries_list)
foreach(geom_file ${geometry_files})
	list(APPEND geometries_list ${CMAKE_CURRENT_SOURCE_DIR}/geometries/${geom_file})
endforeach()

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor.generated.h
	COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/geometries/tools/px_generate_mixers.py -f ${geometries_list} -o ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor.generated.h
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/geometries/tools/px_generate_mixers.py ${geometries_list}
	)
add_custom_target(mixer_gen
	DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor.generated.h
	DEPENDS ${geometries_list})

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_normalized.generated.h
	COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/geometries/tools/px_generate_mixers.py --normalize -f ${geometries_list} -o ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_normalized.generated.h
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/geometries/tools/px_generate_mixers.py ${geometries_list}
	)
add_custom_target(mixer_gen_norm
	DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_normalized.generated.h
	DEPENDS ${geometries_list})

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_6dof.generated.h
	COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/geometries/tools/px_generate_mixers.py --sixdof -f ${geometries_list} -o ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_6dof.generated.h
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/geometries/tools/px_generate_mixers.py ${geometries_list}
	)
add_custom_target(mixer_gen_6dof
	DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_6dof.generated.h
	DEPENDS ${geometries_list})

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_legacy.generated.h
	COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/multi_tables.py > ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_legacy.generated.h
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/multi_tables.py
	)
add_custom_target(mixer_gen_legacy
	DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/mixer_multirotor_legacy.generated.h)

px4_add_module(
	MODULE lib__mixer
	SRCS
		mixer.cpp
		mixer_group.cpp
		mixer_helicopter.cpp
		mixer_load.c
		mixer_multirotor.cpp
		mixer_simple.cpp
	DEPENDS
		platforms__common
		mixer_gen
		mixer_gen_norm
		mixer_gen_6dof
		mixer_gen_legacy
	)
# vim: set noet ft=cmake fenc=utf-8 ff=unix :
