FROM nvidia/cuda:8.0-devel


RUN apt-get update

#INSTALL gfortran
RUN apt-get -y install gfortran gfortran-5 libgfortran-5-dev

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

#Download amgx
WORKDIR /opt
RUN git clone https://github.com/NVIDIA/AMGX.git
ENV AMGX_DIR=/opt/AMGX
WORKDIR AMGX
RUN mkdir build && cd build && cmake ../ && make -j16 all
#RUN mkdir lib/ && cp build/libamgxsh.so build/libamgx.a lib/
RUN cd build && make install

RUN apt install -y doxygen

#CLONE PETSc AND SET CORRECT REVISION
WORKDIR /opt
RUN git clone https://bitbucket.org/petsc/petsc
WORKDIR petsc
RUN git reset --hard  42a884f06af174f528873c26bb8874e3bcc00aa1 

ENV PETSC_DIR=/opt/petsc
ENV PETSC_ARCH=hypre-opts

#CONFIGURE PETSc
RUN ./configure PETSC_ARCH=hypre-opts --download-parmetis --download-metis --download-ptscotch --download-hypre --with-clanguage=c --with-cc=mpicc --with-cxx=mpicxx --with-fc=mpif90 --with-debugging=0 COPTFLAGS=”-O3 -march=native -mtune=native” CXXOPTFLAGS=”-O3 -march=native -mtune=native” FOPTFLAGS=”-O3 -march=native -mtune=native”
RUN make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all
RUN make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check


#DOWNLOAD AMGX-wrapper
WORKDIR /opt
RUN git clone https://github.com/barbagroup/AmgXWrapper.git
ENV AMGX_WRAPPER_DIR=/opt/AmgXWrapper

#DO THE FOLLOWING LATER WHEN WE NEED LIBRARY TO INVCLUDE IN PROJECT
# BUILD EXAMPLES FOR NOW 
#RUN mkdir /tmp/build
#WORKDIR /tmp/build
RUN mkdir $AMGX_WRAPPER_DIR/build 
WORKDIR $AMGX_WRAPPER_DIR/build
RUN cmake \
    -D CMAKE_INSTALL_PREFIX=$AMGX_WRAPPER_DIR \
    -D PETSC_DIR=${PETSC_DIR} \
    -D PETSC_ARCH=${PETSC_ARCH} \
    -D CUDA_DIR=/usr/local/cuda \
    -D AMGX_DIR=${AMGX_DIR} \
    -D BUILD_SHARED_LIBS=OFF \
    ../

RUN make && make install


RUN mkdir /opt/amgx-wrapper-examples
WORKDIR /opt/amgx-wrapper-examples 
RUN cmake -D PETSC_DIR=$PETSC_DIR -D PETSC_ARCH=$PETSC_ARCH -D CUDA_DIR=/usr/local/cuda -D AMGX_DIR=$AMGX_DIR $AMGX_WRAPPER_DIR/example/poisson/ && make

RUN mkdir /opt/wrapper-example-from-files
WORKDIR /opt/wrapper-example-from-files
RUN cmake -D PETSC_DIR=$PETSC_DIR -D PETSC_ARCH=$PETSC_ARCH -D CUDA_DIR=/usr/local/cuda -D AMGX_DIR=$AMGX_DIR $AMGX_WRAPPER_DIR/example/solveFromFiles/ && make

WORKDIR /opt/amgx-wrapper-examples 



CMD mpiexec --allow-run-as-root -n 4 bin/poisson -caseName test-petsc -mode PETSc -cfgFileName configs/PETSc_SolverOptions_GAMG.info -Nx 500 -Ny 500 && \
	mpiexec --allow-run-as-root -n 1 bin/poisson -caseName test -mode AmgX_GPU -cfgFileName  configs/AmgX_SolverOptions_Classical.info -Nx 500 -Ny 500





#CMD /gpu-lib/stable/faster_gpu_quadratic_fem.out -pc_type sacusp -ksp_type cg -print_timings -nz 28 -mesh_path "../meshes/2order/h0-01/" -log_view
# mpiexec --allow-run-as-root -n 4 bin/solveFromFiles -caseName test -mode PETSc -cfgFileName configs/PETSc_SolverOptions_Hypre.info -matrixFileName /data/h0-005/A_matrix_binary -rhsFileName /data/h0-005/f_vec_binary -exactFileName /data/h0-005/u_vec_binary

