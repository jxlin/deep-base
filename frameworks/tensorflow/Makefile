
GPU_SUPPORT ?= 0

.PHONY: dependencies
dependencies:
	apt-get -y install software-properties-common
	add-apt-repository -y ppa:webupd8team/java
	apt-get update
	echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections
	apt-get -y install oracle-java8-installer unzip
	wget "https://github.com/bazelbuild/bazel/releases/download/0.4.4/bazel-0.4.4-installer-linux-x86_64.sh"
	chmod +x bazel-0.4.4-installer-linux-x86_64.sh
	./bazel-0.4.4-installer-linux-x86_64.sh
	rm -f bazel-0.4.4-installer-linux-x86_64.sh
	bash /usr/local/lib/bazel/bin/bazel-complete.bash
	apt-get -y install swig
	apt-get -y install libcurl3-dev
	pip install protobuf==3.2.0
	pip install six --upgrade --target="/usr/lib/python2.7/dist-packages"

.PHONY: src
src:
	git clone --recurse-submodules https://github.com/tensorflow/tensorflow src
	cd src; git checkout v1.0.1

.PHONY: build
build:
ifeq ($(GPU_SUPPORT),1)
	cd src; echo "/usr/local/lib/python2.7/dist-packages" > configure_params.txt
	mkdir /usr/lib/x86_64-linux-gnu/include
	ln -s /usr/include/cudnn.h /usr/lib/x86_64-linux-gnu/include/cudnn.h
	cd src; TF_CUDA_COMPUTE_CAPABILITIES=3.0,3.5,5.2 TF_NEED_CUDA=1 LD_LIBRARY_PATH=/usr/local/cuda/extras/CUPTI/lib64:$(LD_LIBRARY_PATH) tensorflow/tools/ci_build/builds/configured GPU
	#cd src; TF_NEED_OPENCL=0 PYTHON_LIB_PATH=/usr/lib/python2.7/dist-packages TF_ENABLE_XLA=0 TF_NEED_HDFS=0 TF_NEED_JEMALLOC=1 CC_OPT_FLAGS="-march=native" GCC_HOST_COMPILER_PATH=/usr/bin/gcc PYTHON_BIN_PATH=/usr/bin/python TF_NEED_GCP=0 CUDA_TOOLKIT_PATH="/usr/local/cuda" CUDNN_INSTALL_PATH="/usr/local/cuda" TF_NEED_CUDA=1 TF_CUDA_COMPUTE_CAPABILITIES="3.0" TF_CUDNN_VERSION="5" TF_CUDA_VERSION="8.0" TF_CUDA_VERSION_TOOLKIT=8.0 TF_NEED_HDFS=0 ./configure < configure_params.txt
	cd src; bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
else
	cd src; TF_NEED_OPENCL=0 PYTHON_LIB_PATH=/usr/lib/python2.7/dist-packages TF_ENABLE_XLA=0 TF_NEED_HDFS=0 PYTHON_BIN_PATH=/usr/bin/python TF_NEED_JEMALLOC=1 CC_OPT_FLAGS="-march=native" TF_NEED_GCP=0 TF_NEED_CUDA=0 ./configure
	cd src; bazel build -c opt //tensorflow/tools/pip_package:build_pip_package
endif

.PHONY: install
install:
	cd src; bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
	pip install /tmp/tensorflow_pkg/*

.PHONY: load_test
load_test:
ifeq ($(GPU_SUPPORT),1)
	LD_LIBRARY_PATH=$(LD_LIBRARY_PATH):/usr/local/cuda/lib64 python test/load.py
else
	python test/load.py
endif
