FROM nvidia/cuda:9.0-devel


RUN apt-get update

#INSTALL gfortran
RUN apt-get -y install gfortran gfortran-5 libgfortran-5-dev

RUN apt install -y wget 

#INSTALL openmpi
#RUN apt-get -y install libopenmpi-dev libopenmpi1.10 openmpi-bin openmpi-common
ENV OMPIVERSION=1.8.8
WORKDIR /tmp
RUN wget https://download.open-mpi.org/release/open-mpi/v1.8/openmpi-${OMPIVERSION}.tar.gz && tar zxvf openmpi-${OMPIVERSION}.tar.gz 
RUN ls
WORKDIR /tmp/openmpi-${OMPIVERSION}
RUN ./configure --prefix=/usr/local/openmpi | tee configure.log
RUN make -j 4 | tee make.log
RUN make install -j 4 | tee install.log
ENV CC=/usr/local/openmpi/bin/mpicc
ENV CXX=/usr/local/openmpi/bin/mpicxx
ENV MPIEXEC=/usr/local/openmpi/bin/mpiexec

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
RUN mkdir build && cd build && cmake -DCMAKE_C_COMPILER=$CC -DCMAKE_CXX_COMPILER=$CXX ../ && make -j16 all
#RUN mkdir lib/ && cp build/libamgxsh.so build/libamgx.a lib/
RUN cd build && make install

RUN apt install -y doxygen

#CLONE PETSc AND SET CORRECT REVISION
WORKDIR /opt


#RUN git clone https://bitbucket.org/petsc/petsc
#WORKDIR petsc
#RUN git reset --hard  42a884f06af174f528873c26bb8874e3bcc00aa1 
ENV PETSCVERSION=3.9.4
RUN wget http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-$PETSCVERSION.tar.gz && tar zxvf petsc-$PETSCVERSION.tar.gz && mv petsc-$PETSCVERSION petsc && rm petsc-$PETSCVERSION.tar.gz

ENV OPTIMFLAGS="-DNDEBUG -Wfatal-errors -fPIC -O3 -funroll-loops -finline-functions -Wall -march=native -mavx2 -ftree-vectorize -fno-stack-protector -mtune=native -pthread"

ENV PETSC_DIR=/opt/petsc
ENV PETSC_ARCH=hypre-opts
ENV FC=/usr/local/openmpi/bin/mpif90
WORKDIR /opt/petsc

#CONFIGURE PETSc
RUN ./configure PETSC_ARCH=hypre-opts --with-mpidir=/usr/local/openmpi --download-parmetis --download-metis --download-ptscotch --download-hypre --with-clanguage=c --with-cc=$CC --with-cxx=$CXX --with-fc=$FC --with-debugging=0 COPTFLAGS=$OPTIMFLAGS CXXOPTFLAGS=$OPTIMFLAGS FOPTFLAGS=$OPTIMFLAGS
#RUN ./configure PETSC_ARCH=hypre-opts --with-mpidir=/usr/local/openmpi --download-parmetis --download-metis --download-ptscotch --download-hypre --with-clanguage=c --with-debugging=0 COPTFLAGS=$OPTIMFLAGS CXXOPTFLAGS=$OPTIMFLAGS FOPTFLAGS=$OPTIMFLAGS
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
    -D CMAKE_C_COMPILER=$CC \
    -D CMAKE_CXX_COMPILER=$CXX \
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
RUN cmake -D CMAKE_C_COMPILER=$CC -D CMAKE_CXX_COMPILER=$CXX -D PETSC_DIR=$PETSC_DIR -D PETSC_ARCH=$PETSC_ARCH -D CUDA_DIR=/usr/local/cuda -D AMGX_DIR=$AMGX_DIR $AMGX_WRAPPER_DIR/example/poisson/ && make

RUN mkdir /opt/wrapper-example-from-files
WORKDIR /opt/wrapper-example-from-files
RUN cmake -DCMAKE_C_COMPILER=$CC -DCMAKE_CXX_COMPILER=$CXX -D PETSC_DIR=$PETSC_DIR -D PETSC_ARCH=$PETSC_ARCH -D CUDA_DIR=/usr/local/cuda -D AMGX_DIR=$AMGX_DIR $AMGX_WRAPPER_DIR/example/solveFromFiles/ && make

WORKDIR /opt/amgx-wrapper-examples 



CMD $MPIEXEC --allow-run-as-root -n 4 bin/poisson -caseName test-petsc -mode PETSc -cfgFileName configs/PETSc_SolverOptions_GAMG.info -Nx 500 -Ny 500 && \
	$MPIEXEC --allow-run-as-root -n 1 bin/poisson -caseName test -mode AmgX_GPU -cfgFileName  configs/AmgX_SolverOptions_Classical.info -Nx 500 -Ny 500





#CMD /gpu-lib/stable/faster_gpu_quadratic_fem.out -pc_type sacusp -ksp_type cg -print_timings -nz 28 -mesh_path "../meshes/2order/h0-01/" -log_view
# mpiexec --allow-run-as-root -n 4 bin/solveFromFiles -caseName test -mode PETSc -cfgFileName configs/PETSc_SolverOptions_Hypre.info -matrixFileName /data/h0-005/A_matrix_binary -rhsFileName /data/h0-005/f_vec_binary -exactFileName /data/h0-005/u_vec_binary

