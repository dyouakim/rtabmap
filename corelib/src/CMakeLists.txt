
SET(SRC_FILES
	Rtabmap.cpp
	RtabmapThread.cpp
	
	Statistics.cpp
	
	Memory.cpp
	
	DBDriver.cpp
	DBDriverSqlite3.cpp
	DBReader.cpp
	
    Camera.cpp
    CameraThread.cpp
    CameraRGB.cpp
    CameraRGBD.cpp
    CameraStereo.cpp
    CameraModel.cpp
    
    EpipolarGeometry.cpp
	VisualWord.cpp
	VWDictionary.cpp
	BayesFilter.cpp
	Parameters.cpp
    Signature.cpp
	Features2d.cpp
	Transform.cpp
	
	util2d.cpp
	
	util3d.cpp
	util3d_filtering.cpp
	util3d_mapping.cpp
	util3d_transforms.cpp
	util3d_registration.cpp
	util3d_surface.cpp
	util3d_features.cpp
	util3d_correspondences.cpp
	util3d_motion_estimation.cpp
	
	SensorData.cpp
	Graph.cpp
	Compression.cpp
	Link.cpp
	
	Odometry.cpp
	OdometryThread.cpp
	OdometryBOW.cpp
	OdometryOpticalFlow.cpp
	OdometryMono.cpp
	OdometryICP.cpp
	
	toro3d/posegraph3.cpp
	toro3d/treeoptimizer3_iteration.cpp
	toro3d/treeoptimizer3.cpp
	
	toro3d/posegraph2.cpp
	toro3d/treeoptimizer2.cpp
	
	rtflann/ext/lz4.c
	rtflann/ext/lz4hc.c
	
	sqlite3/sqlite3.c
)

SET(INCLUDE_DIRS
    ${PROJECT_SOURCE_DIR}/utilite/include
	${CMAKE_CURRENT_SOURCE_DIR}/../include
	${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_BINARY_DIR}
    ${OpenCV_INCLUDE_DIRS}
	${PCL_INCLUDE_DIRS}
	${ZLIB_INCLUDE_DIRS}
)

SET(LIBRARIES
	${OpenCV_LIBS} 
	${PCL_LIBRARIES} 
	${ZLIB_LIBRARIES} 
)

IF(Freenect_FOUND)
	ADD_DEFINITIONS("-DWITH_FREENECT")
	IF(Freenect_DASH_INCLUDES)
	   ADD_DEFINITIONS("-DFREENECT_DASH_INCLUDES")
	ENDIF(Freenect_DASH_INCLUDES)
	SET(INCLUDE_DIRS
		${INCLUDE_DIRS}
		${Freenect_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${Freenect_LIBRARIES}
	)
ENDIF(Freenect_FOUND)

IF(OpenNI2_FOUND)
	ADD_DEFINITIONS("-DWITH_OPENNI2")
	SET(INCLUDE_DIRS
		${INCLUDE_DIRS}
		${OpenNI2_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${OpenNI2_LIBRARIES}
	)
ENDIF(OpenNI2_FOUND)

IF(freenect2_FOUND)
	ADD_DEFINITIONS("-DWITH_FREENECT2")
	SET(INCLUDE_DIRS
		${INCLUDE_DIRS}
		${freenect2_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${freenect2_LIBRARIES}
	)
ENDIF(freenect2_FOUND)

IF(DC1394_FOUND)
	ADD_DEFINITIONS("-DWITH_DC1394")
	SET(INCLUDE_DIRS
		${INCLUDE_DIRS}
		${DC1394_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${DC1394_LIBRARIES}
	)
ENDIF(DC1394_FOUND)

IF(FlyCapture2_FOUND)
	ADD_DEFINITIONS("-DWITH_FLYCAPTURE2")
	SET(INCLUDE_DIRS
		${INCLUDE_DIRS}
		${FlyCapture2_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${FlyCapture2_LIBRARIES}
	)
ENDIF(FlyCapture2_FOUND)

IF(G2O_FOUND)
	ADD_DEFINITIONS("-DWITH_G2O")
	SET(INCLUDE_DIRS 
		${INCLUDE_DIRS} 
		${G2O_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${G2O_LIBRARIES}
	)
	#Newest versions require std11
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
	
	SET(SRC_FILES
    	${SRC_FILES}
		vertigo/g2o/edge_se2MaxMixture.cpp
		vertigo/g2o/edge_se2Switchable.cpp
		vertigo/g2o/edge_se3Switchable.cpp
		vertigo/g2o/edge_switchPrior.cpp
		vertigo/g2o/types_g2o_robust.cpp
		vertigo/g2o/vertex_switchLinear.cpp
	)
ENDIF(G2O_FOUND)

IF(GTSAM_FOUND)
	ADD_DEFINITIONS("-DWITH_GTSAM")
	SET(INCLUDE_DIRS 
		${INCLUDE_DIRS} 
		${GTSAM_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		gtsam
	)
ENDIF(GTSAM_FOUND)

IF(cvsba_FOUND)
	ADD_DEFINITIONS("-DWITH_CVSBA")
	SET(INCLUDE_DIRS 
		${INCLUDE_DIRS} 
		${cvsba_INCLUDE_DIRS}
	)
	SET(LIBRARIES
		${LIBRARIES}
		${cvsba_LIBS}
	)
ENDIF(cvsba_FOUND)

####################################
# Generate resources files
####################################
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/resources/DatabaseSchema.sql.in ${CMAKE_CURRENT_SOURCE_DIR}/resources/DatabaseSchema.sql)

SET(R
	${CMAKE_CURRENT_SOURCE_DIR}/resources/DatabaseSchema.sql
)

#replace semicolons by spaces
foreach(arg ${R})
   set(RESOURCES "${RESOURCES}" "${arg}")
endforeach(arg ${R})

SET(RESOURCES_HEADERS 
	${CMAKE_CURRENT_BINARY_DIR}/DatabaseSchema_sql.h 
)

ADD_CUSTOM_COMMAND(
   OUTPUT ${RESOURCES_HEADERS}
   COMMAND ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/uresourcegenerator -n rtabmap -p ${CMAKE_CURRENT_BINARY_DIR} ${RESOURCES}
   COMMENT "[Creating resources]"
   DEPENDS ${R} uresourcegenerator
)

####################################
# Generate resources files END
####################################

add_definitions(${PCL_DEFINITIONS})


# Make sure the compiler can find include files from our library.
INCLUDE_DIRECTORIES(${INCLUDE_DIRS})

# Add binary that is built from the source file "main.cpp".
# The extension is automatically found.
ADD_LIBRARY(rtabmap_core ${SRC_FILES} ${RESOURCES_HEADERS})
TARGET_LINK_LIBRARIES(rtabmap_core rtabmap_utilite ${LIBRARIES})

SET_TARGET_PROPERTIES(
     rtabmap_core
   PROPERTIES
     VERSION ${RTABMAP_VERSION}
     SOVERSION ${RTABMAP_MAJOR_VERSION}.${RTABMAP_MINOR_VERSION}
)

INSTALL(TARGETS rtabmap_core
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT runtime
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT devel
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT devel)
		
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../include/ 
		DESTINATION "${INSTALL_INCLUDE_DIR}"
		COMPONENT devel 
		FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp" 
		PATTERN ".svn" EXCLUDE)

