FROM nvidia/cuda-ppc64le:9.0-devel


RUN apt-get update

#INSTALL gfortran
RUN apt-get -y install gfortran-powerpc-linux-gnu gfortran-5-powerpc-linux-gnu libgfortran-5-dev libgfortran-5-dev-ppc64el-cross

#INSTALL openmpi
RUN apt-get -y install libopenmpi-dev libopenmpi1.10 openmpi-bin openmpi-common

#INSTALL ATLAS
RUN apt-get -y install libatlas-base-dev libatlas-dev libatlas3-base

#INSTALL git
RUN apt-get -y install git cmake

#INSTALL python
RUN apt-get -y install python

#INSTALL DIV
RUN apt-get -y install bison flex 

#CLONE PETSc AND SET CORRECT REVISION
WORKDIR /opt
RUN git clone https://bitbucket.org/petsc/petsc
WORKDIR petsc
#RUN git reset --hard v3.7.6-5079-g20d13fa #ONLY NEED THIS FOR COMPATIBILITY WITH TDT4195 Project

ENV PETSC_DIR=/opt/petsc
ENV PETSC_ARCH=opts

#CONFIGURE PETSc
#RUN ./configure PETSC_ARCH=hypre-opts --download-parmetis --download-metis --download-ptscotch --download-hypre --with-clanguage=c --with-cc=mpicc --with-cxx=mpicxx --with-fc=mpif90 --with-debugging=0 COPTFLAGS='-O3 -march=native -mtune=native' CXXOPTFLAGS='-O3 -march=native -mtune=native' FOPTFLAGS='-O3 -march=native -mtune=native'
RUN ./configure PETSC_ARCH=opts --with-clanguage=c --with-cc=mpicc --with-cxx=mpicxx --with-fc=mpif90 --with-debugging=0 COPTFLAGS='-O3 -march=native -mtune=native' CXXOPTFLAGS='-O3 -march=native -mtune=native' FOPTFLAGS='-O3 -march=native -mtune=native'
RUN make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all
RUN make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check


#THE FOLLOWING IS ONLY COMPATIBLE WITH PETSC CONFIGURED WITH CUSP
#SKIP FOR NOW
#COPY FILES
#RUN mkdir /gpu-lib/
#RUN mkdir /gpu-lib/stable
#ADD stable /gpu-lib/stable
#RUN mkdir /gpu-lib/meshes
#ADD meshes /gpu-lib/meshes
#RUN ls /gpu-lib
#RUN ls /gpu-lib/stable

#COMPILE EXECUTABLE
#WORKDIR /gpu-lib/stable
#RUN make faster-gpu

#Download amgx
WORKDIR /opt
RUN git clone https://github.com/NVIDIA/AMGX.git
ENV AMGX_DIR=/opt/AMGX
WORKDIR AMGX
RUN mkdir build && cd build && cmake ../ && make -j16 all
RUN mkdir lib/ && cp build/libamgxsh.so build/libamgx.a lib/


#DOWNLOAD AMGX-wrapper
WORKDIR /opt
RUN git clone https://github.com/barbagroup/AmgXWrapper.git
ENV AMGX_WRAPPER_DIR=/opt/AmgXWrapper

#DO THE FOLLOWING LATER WHEN WE NEED LIBRARY TO INVCLUDE IN PROJECT
# BUILD EXAMPLES FOR NOW 
#RUN mkdir /tmp/build
#WORKDIR /tmp/build

#RUN cmake \
#    -D CMAKE_INSTALL_PREFIX=/opt/amgx-wrapper \
#    -D PETSC_DIR=${PETSC_DIR} \
#    -D PETSC_ARCH=${PETSC_ARCH} \
#    -D CUDA_DIR=/usr/local/cuda \
#    -D AMGX_DIR=/opt/amgx \
#    /AmgxWrapper


RUN mkdir amgx-wrapper-examples
WORKDIR amgx-wrapper-examples 
RUN cmake -D PETSC_DIR=$PETSC_DIR -D PETSC_ARCH=$PETSC_ARCH -D CUDA_DIR=/usr/local/cuda -D AMGX_DIR=$AMGX_DIR $AMGX_WRAPPER_DIR/example/poisson/ && make


CMD mpiexec --allow-run-as-root -n 4 bin/poisson -caseName test-petsc -mode PETSc -cfgFileName configs/PETSc_SolverOptions_GAMG.info -Nx 500 -Ny 500 && \
	mpiexec --allow-run-as-root -n 4 bin/poisson -caseName test -mode AmgX_GPU -cfgFileName  configs/AmgX_SolverOptions_AGG.info -Nx 500 -Ny 500


#CMD /gpu-lib/stable/faster_gpu_quadratic_fem.out -pc_type sacusp -ksp_type cg -print_timings -nz 28 -mesh_path "../meshes/2order/h0-01/" -log_view
