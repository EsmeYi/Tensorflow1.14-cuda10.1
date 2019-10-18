# Tensorflow r1.14 cuda-10.1
## Environment

~~~
docker pull nvidia/cuda-ppc64le:10.1-cudnn7-devel-ubuntu18.04
docker run -it <docker image ID>
export CPU=power9
~~~

## Java 8

~~~
apt update
apt install openjdk-8-jre-headless
apt install openjdk-8-jdk
update-alternatives --config java
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-ppc64el/"
~~~

## Python and pip

~~~
apt update
apt upgrade
apt install python2.7 python-pip
export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
export PYTHONPATH=/usr/lib/python2.7/site-packages
export PYTHON_BIN_PATH=/usr/bin/python
export PYTHON_LIB_PATH=/usr/lib/python2.7/site-packages
~~~

## protobuf-3.7.1
~~~
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.7.1/protobuf-all-3.7.1.tar.gz
tar -zxvf protobuf-all-3.7.1.tar.gz
cd protobuf-3.7.1

make uninstall
make distclean

export CPU=power9
CFLAGS="-mcpu=$CPU -mtune=$CPU -O3" CXXFLAGS="-mcpu=$CPU -mtune=$CPU -O3" ./configure --prefix=/usr --disable-shared --enable-static

make clean
make -j20
make -j20 check
make install
ldconfig

which protoc
protoc --version
~~~

## protobuf-3.7.1 (--cpp_implementation) Python from source
it would take extrem long time for protobuf.ParseFromString if whithout `--cpp_implementation`
~~~
cd protobuf-3.7.1/python/
python setup.py build --cpp_implementation
python setup.py test --cpp_implementation
python setup.py install --cpp_implementation
# install from wheel
python setup.py bdist_wheel --cpp_implementation
pip install dist/protobuf-3.7.1-cp27-cp27mu-linux_ppc64le.whl --force-reinstall
# verify
python -c "import google.protobuf"
~~~

## bazel-0.25.0
download a pre-builted bazel bin from [here](https://oplab9.parqtec.unicamp.br/pub/ppc64el/bazel/readme.html)
~~~
mv bazel_bin_<version> /usr/local/bin/bazel 
chmod +x /usr/local/bin/bazel
~~~

build bazel from source
~~~
apt-get install zip unzip rsync
wget https://github.com/bazelbuild/bazel/releases/download/0.25.0/bazel-0.25.0-dist.zip
uzip bazel-0.25.0-dist.zip
env EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh
# Build successful! Binary is here: /home/bazel-0.25.0/output/bazel
rsync -avP output/bazel /usr/bin
~~~

## Advance Toolchain 12.0 (only for Power user)
[Doc site](https://developer.ibm.com/linuxonpower/advance-toolchain/advtool-installation/)
~~~
wget ftp://ftp.unicamp.br/pub/linuxpatch/toolchain/at/ubuntu/dists/trusty/6976a827.gpg.key
sudo apt-key add 6976a827.gpg.key
vi /etc/apt/sources.list
(Configure the Advance Toolchain repositories by adding the following line to /etc/apt/sources.list:) 
    deb ftp://ftp.unicamp.br/pub/linuxpatch/toolchain/at/ubuntu xenial at12.0
apt-get update
apt-get install advance-toolchain-at12.0-runtime \
                advance-toolchain-at12.0-devel \
                advance-toolchain-at12.0-perf \
                advance-toolchain-at12.0-mcore-libs
export PATH=/opt/at12.0/bin:/opt/at12.0/sbin:$PATH
# check gcc before build, should be AT11.0
which gcc
~~~

## OpenBLAS 0.3.5

~~~
git clone -b v0.3.5 https://github.com/xianyi/OpenBLAS.git OpenBLAS-0.3.5
cd OpenBLAS-0.3.5
make TARGET=power9
make TARGET=power9 PREFIX=/usr install
~~~

## Boost 1.66.0

~~~
wget -qc https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.gz
tar xzf boost_1_66_0.tar.gz
cd boost_1_66_0
./bootstrap.sh --with-toolset=gcc --prefix=/usr
./b2 dll-path="/usr/lib" install
~~~


## scipy, h5py, future, keras, mock, enum34, wheel

~~~
apt-get install python-scipy python-h5py
pip install future keras_applications keras_preprocessing wheel autograd enum34 mock
...
~~~

## (optional)TensorRT

~~~
download <TensorRT-5.1.3.6-version>.tar and untar
cd TensorRT-5.1.3.6
cd python && install 
cd uff && install
export LD_LIBRARY_PATH=/path/TensorRT-5.1.3.6/lib:$LD_LIBRARY_PATH
cp TensorRT-5.1.3.6/lib/* /usr/lib/
cp TensorRT-5.1.3.6/include/* /usr/include/
~~~

## Tensorflow

~~~
export PYTHON_BIN_PATH="/usr/bin/python"
export TF_CUDA_PATHS=/usr,/usr/local/cuda
# check gcc before build, should be AT11.0
which gcc

# fix build error
-------------------------------------
vim /opt/at12.0/include/bits/floatn.h
-------------------------------------
/* Defined to 1 if the current compiler invocation provides a
   floating-point type with the IEEE 754 binary128 format, and this glibc
   includes corresponding *f128 interfaces for it.  */
#if defined _ARCH_PWR8 && defined __LITTLE_ENDIAN__ && (_CALL_ELF == 2) \
&& defined __FLOAT128__
# define __HAVE_FLOAT128 1
#else
# define __HAVE_FLOAT128 0
#endif

/* add the following block of fix tensorflow build error */
#if CUDART_VERSION
#undef __HAVE_FLOAT128
#define __HAVE_FLOAT128 0
#endif

/* Defined to 1 if __HAVE_FLOAT128 is 1 and the type is ABI-distinct
   from the default float, double and long double types in this glibc.  */
#if __HAVE_FLOAT128
-------------------------------------
vi ./tensorflow/compiler/tf2tensorrt/convert/convert_nodes.cc
-------------------------------------
//line 4195
-  constexpr auto get_matrix_op =
+  const/*expr*/ auto get_matrix_op =
-------------------------------------

git clone -b r1.14 https://github.com/tensorflow/tensorflow.git tensorflow-1.14
cd tensorflow-1.14
./configure
bazel clean
bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" --action_env=PYTHON_BIN_PATH=$PYTHON_BIN_PATH --test_env=PYTHON_BIN_PATH=$PYTHON_BIN_PATH --distinct_host_configuration=false
bazel-bin/tensorflow/tools/pip_package/build_pip_package ../tensorflow_package
pip install ../tensorflow_package/tensorflow-<version>.whl
~~~

## Opencv-python
~~~
git clone https://github.com/skvark/opencv-python.git
cd opencv-python/
git fetch --all --tags --prune
git checkout ‘’
git submodule update --init --recursive

pip install pyparsing
apt-get install qt4-qmake
apt-get install libqt4-dev

python setup.py bdist_wheel
pip install dist/opencv_python-version.whl
~~~
