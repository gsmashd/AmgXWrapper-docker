FROM nvidia/cuda-ppc64le:9.0-devel

#RUN echo "Hello"
RUN apt-get update

#INSTALL gfortran
#RUN apt-get -y install gfortran-powerpc-linux-gnu gfortran-5-powerpc-linux-gnu libgfortran-5-dev libgfortran-5-dev-ppc64el-cross
#RUN apt-get -y install gfortran gfortran-5 libgfortran-5-dev

#RUN apt-get install -y software-properties-common && \
#	add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
#	apt update && \
#	apt-get install -y gfortran-8 libgfortran-8-dev && \
#	ln -s /usr/bin/gfortran-8 /usr/bin/gfortran 

#RUN apt install -y wget
#WORKDIR /tmp
#RUN  wget https://download.open-mpi.org/release/open-mpi/v3.1/openmpi-3.1.1.tar.gz
#RUN gunzip -c openmpi-3.1.1.tar.gz | tar xf -
#WORKDIR openmpi-3.1.1
#RUN mkdir /opt/openmpi 
#RUN ./configure --enable-mpi-fortran=yes --prefix=/opt/openmpi
#RUN make -j32 all 
#RUN make install
#WORKDIR /

RUN apt-get -y install gfortran gfortran-5 libgfortran-5-dev gcc g++

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



#INSTALL openmpi
#RUN apt-get -y install libopenmpi-dev libopenmpi1.10 openmpi-bin openmpi-common

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



#RUN git clone https://bitbucket.org/petsc/petsc
#WORKDIR petsc
#RUN git reset --hard v3.7.6-5079-g20d13fa #ONLY NEED THIS FOR COMPATIBILITY WITH TDT4195 Project

#ENV PETSC_DIR=/opt/petsc
#ENV PETSC_ARCH=hypre
#RUN ls /opt/openmpi/include
#ENV PATH=/opt/openmpi/include:$PATH
#CONFIGURE PETSc
#RUN ./configure PETSC_ARCH=hypre-opts --download-parmetis --download-metis --download-ptscotch --download-hypre --with-debugging=0 --with-mpi-dir=/opt/openmpi COPTFLAGS='-O3 -march=native -mtune=native' CXXOPTFLAGS='-O3 -march=native -mtune=native'
#RUN ./configure PETSC_ARCH=opts --with-clanguage=c --with-cc=mpicc --with-cxx=mpicxx --with-fc=mpif90 --with-debugging=0 COPTFLAGS='-O3 -march=native -mtune=native' CXXOPTFLAGS='-O3 -march=native -mtune=native' FOPTFLAGS='-O3 -march=native -mtune=native'
#RUN ./configure PETSC_ARCH=hypre --download-parmetis --download-metis --download-ptscotch --download-hypre --with-debugging=0 --with-cc=mpicc --with-cxx=mpicxx --with-fc=0 --download-f2cblaslapack

#RUN make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH -j 16 all
#RUN make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check

ENV PETSCVERSION=3.9.4
RUN wget http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-$PETSCVERSION.tar.gz && tar zxvf petsc-$PETSCVERSION.tar.gz && mv petsc-$PETSCVERSION petsc && rm petsc-$PETSCVERSION.tar.gz

ENV OPTIMFLAGS="-DNDEBUG -Wfatal-errors -fPIC -O3 -funroll-loops -finline-functions -Wall -march=native -mavx2 -ftree-vectorize -fno-stack-protector -mtune=native -pthread"

ENV PETSC_DIR=/opt/petsc
ENV PETSC_ARCH=hypre-opts
ENV FC=/usr/local/openmpi/bin/mpif90
WORKDIR /opt/petsc

#CONFIGURE PETSc
RUN ./configure PETSC_ARCH=hypre-opts --with-mpidir=/usr/local/openmpi --download-parmetis --download-metis --download-ptscotch --download-hypre --with-clanguage=c --with-cc=$CC --with-cxx=$CXX --with-fc=$FC --with-debugging=0 COPTFLAGS=$OPTIMFLAGS CXXOPTFLAGS=$OPTIMFLAGS FOPTFLAGS=$OPTIMFLAGS
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
#WORKDIR /opt
#RUN git clone https://github.com/NVIDIA/AMGX.git
#ENV AMGX_DIR=/opt/AMGX
#WORKDIR AMGX
#RUN mkdir build && cd build && cmake ../ && make -j16 all
#RUN mkdir lib/ && cp build/libamgxsh.so build/libamgx.a lib/

#Download amgx
WORKDIR /opt
RUN git clone https://github.com/NVIDIA/AMGX.git
ENV AMGX_DIR=/opt/AMGX
WORKDIR AMGX
RUN mkdir build && cd build && cmake -DCMAKE_C_COMPILER=$CC -DCMAKE_CXX_COMPILER=$CXX ../ && make -j16 all
RUN cd build && make install

RUN apt install -y doxygen

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




#DOWNLOAD AMGX-wrapper
#WORKDIR /opt
#RUN git clone https://github.com/barbagroup/AmgXWrapper.git
#ENV AMGX_WRAPPER_DIR=/opt/AmgXWrapper

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


#RUN mkdir amgx-wrapper-examples
#WORKDIR amgx-wrapper-examples 
#RUN cmake -D PETSC_DIR=$PETSC_DIR -D PETSC_ARCH=$PETSC_ARCH -D CUDA_DIR=/usr/local/cuda -D AMGX_DIR=$AMGX_DIR $AMGX_WRAPPER_DIR/example/poisson/ && make


#CMD mpiexec --allow-run-as-root -n 4 bin/poisson -caseName test-petsc -mode PETSc -cfgFileName configs/PETSc_SolverOptions_GAMG.info -Nx 500 -Ny 500 && \
	#mpiexec --allow-run-as-root -n 4 bin/poisson -caseName test -mode AmgX_GPU -cfgFileName  configs/AmgX_SolverOptions_AGG.info -Nx 500 -Ny 500



#CMD /gpu-lib/stable/faster_gpu_quadratic_fem.out -pc_type sacusp -ksp_type cg -print_timings -nz 28 -mesh_path "../meshes/2order/h0-01/" -log_view
