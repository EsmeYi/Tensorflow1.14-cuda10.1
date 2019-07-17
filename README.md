# Tensorflow r1.14 GPU
## Environment

~~~
docker pull nvidia/cuda-ppc64le:10.0-cudnn7-devel-ubuntu18.04
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

## OpenBLAS 0.3.5

~~~
git clone -b v0.3.5 https://github.com/xianyi/OpenBLAS.git OpenBLAS-0.3.5
cd OpenBLAS-0.3.5
make TARGET=power9
make TARGET=power9 PREFIX=$HOMEPATH/inst install
~~~

## Boost 1.66.0

~~~
wget -qc https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.gz
tar xzf boost_1_66_0.tar.gz
cd boost_1_66_0
./bootstrap.sh --with-toolset=gcc --prefix=$HOMEPATH/inst
./b2 dll-path="$HOMEPATH/inst/lib" install
~~~

## scipy, h5py, future, keras, mock, enum34, wheel

~~~

~~~

## bazel
download a pre-builted bazel bin from [here](https://oplab9.parqtec.unicamp.br/pub/ppc64el/bazel/readme.html)
~~~
mv bazel_bin_<version> /usr/local/bin/bazel 
chmod +x /usr/local/bin/bazel
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
bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" --action_env=PYTHON_BIN_PATH=$PYTHON_BIN_PATH --test_env=PYTHON_BIN_PATH=$PYTHON_BIN_PATH --distinct_host_configuration=false
bazel-bin/tensorflow/tools/pip_package/build_pip_package ~/tensorflow_package
pip install ~/tensorflow_package/tensorflow-<version>.whl
~~~

