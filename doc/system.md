<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [System Stack overview](#system-stack-overview)
- [Deep Learning Programming Style](#deep-learning-programming-style)
  - [Symbolic vs. Imperative Programs](#symbolic-vs-imperative-programs)
    - [Imperative Programs Tend to be More Flexible](#imperative-programs-tend-to-be-more-flexible)
    - [Symbolic Programs Tend to be More Efficient](#symbolic-programs-tend-to-be-more-efficient)
  - [Case Study: Backprop and AutoDiff](#case-study-backprop-and-autodiff)
- [GPU in DeepLearning](#gpu-in-deeplearning)
  - [GPU architecture](#gpu-architecture)
    - [GPU vs CPU](#gpu-vs-cpu)
    - [Streaming Multiprocessor (SM)](#streaming-multiprocessor-sm)
  - [GPU Memory architecture](#gpu-memory-architecture)
    - [GPU memory model](#gpu-memory-model)
    - [GPU memory latency](#gpu-memory-latency)
    - [GPU memory bandwidth](#gpu-memory-bandwidth)
  - [GPU programming model](#gpu-programming-model)
    - [SIMD](#simd)
    - [Kernel Execution](#kernel-execution)
    - [Control Flow](#control-flow)
    - [Thread Hierarchy & Memory Hierarchy](#thread-hierarchy--memory-hierarchy)
  - [GPU Example](#gpu-example)
    - [Vector Add](#vector-add)
    - [Slide window sum (Use of thread block)](#slide-window-sum-use-of-thread-block)
    - [Matrix Operation](#matrix-operation)
    - [Tips for High Performance](#tips-for-high-performance)
- [Machine Learning HW](#machine-learning-hw)
  - [Hardware Specialization in Deep Learning](#hardware-specialization-in-deep-learning)
    - [Roofline model](#roofline-model)
    - [TPU](#tpu)
  - [HW/SW Codesign](#hwsw-codesign)
    - [Tensorization](#tensorization)
    - [Memory Architecting](#memory-architecting)
    - [Data Type](#data-type)
  - [Latency Hiding](#latency-hiding)
- [Memory Optimization](#memory-optimization)
  - [Build an Executor for a Given Graph](#build-an-executor-for-a-given-graph)
    - [Dynamic Memory Allocation:](#dynamic-memory-allocation)
    - [Static memory allocation](#static-memory-allocation)
  - [Common Patterns of Memory Planning](#common-patterns-of-memory-planning)
  - [Memory Allocation and Scheduling](#memory-allocation-and-scheduling)
  - [Memory Plan with Gradient Calculation](#memory-plan-with-gradient-calculation)
- [Parallel schedule](#parallel-schedule)
  - [Common patterns of parallelization](#common-patterns-of-parallelization)
  - [Data parallelism](#data-parallelism)
    - [DAG based scheduler](#dag-based-scheduler)
    - [mutation aware scheduler](#mutation-aware-scheduler)
    - [Queue based Implementation of scheduler](#queue-based-implementation-of-scheduler)
  - [Model parallelization](#model-parallelization)
  - [Summary](#summary)
- [Distributed Machine Learning](#distributed-machine-learning)
  - [Deep learning computation model](#deep-learning-computation-model)
  - [model parallelism vs data parallelism](#model-parallelism-vs-data-parallelism)
  - [Data Parallelism](#data-parallelism)
    - [Parameter Averaging](#parameter-averaging)
    - [Update based method](#update-based-method)
    - [Asychronous Stochastic Gradient Descent](#asychronous-stochastic-gradient-descent)
  - [Distributed Asychronous Stochastic Gradient Descent](#distributed-asychronous-stochastic-gradient-descent)
- [Parameter Server](#parameter-server)
  - [Design principal](#design-principal)
  - [Key consideration](#key-consideration)
  - [System view](#system-view)
    - [Challenge](#challenge)
    - [Large data](#large-data)
    - [Synchronization](#synchronization)
    - [Fault tolerance](#fault-tolerance)
  - [PS in GPU cluster](#ps-in-gpu-cluster)
    - [GPU challenge](#gpu-challenge)
      - [GPU bandwidth example](#gpu-bandwidth-example)
    - [solutions](#solutions)
      - [model parallelism](#model-parallelism)
      - [Asychronous: Overlap between computation and communication](#asychronous-overlap-between-computation-and-communication)
      - [Asychronous: Sufficient Factor Broadcasting](#asychronous-sufficient-factor-broadcasting)
- [All reduce Approach](#all-reduce-approach)
  - [communication primitive](#communication-primitive)
  - [Ring-based Collective communication](#ring-based-collective-communication)
    - [MPI Reduce and Allreduce](#mpi-reduce-and-allreduce)
    - [limitation of reduce via map](#limitation-of-reduce-via-map)
  - [Allreduce in practice](#allreduce-in-practice)
  - [Tree based Allreduce vs Ring based Allreduce](#tree-based-allreduce-vs-ring-based-allreduce)
    - [limitations of Tree Allreduce](#limitations-of-tree-allreduce)
    - [Ring Allreduce](#ring-allreduce)
    - [Scatter-Reduce](#scatter-reduce)
    - [AllGather](#allgather)
    - [Overhead Analysis](#overhead-analysis)
    - [RABIT: A Reliable Allreduce and Broadcast Interface](#rabit-a-reliable-allreduce-and-broadcast-interface)
      - [Fault-tolerant](#fault-tolerant)
      - [Recovery](#recovery)
      - [Consensus Protocol](#consensus-protocol)
      - [Routing](#routing)
  - [Allreduce vs Paramter server](#allreduce-vs-paramter-server)
- [Nvidia Multi-GPU Lib: NCCL](#nvidia-multi-gpu-lib-nccl)
    - [Ring based collective](#ring-based-collective)
  - [NCCL Implementation](#nccl-implementation)
    - [Performance](#performance)
- [FPGA on Machine Leaning/Deep Learning](#fpga-on-machine-leaningdeep-learning)
  - [FPGA vs CPU/GPU](#fpga-vs-cpugpu)
  - [FPGA advantage on computation](#fpga-advantage-on-computation)
    - [Training and inference stage difference](#training-and-inference-stage-difference)
    - [FPGA vs GPU on parallelism](#fpga-vs-gpu-on-parallelism)
    - [FPGA vs ASIC](#fpga-vs-asic)
  - [FPGA advantage on communication](#fpga-advantage-on-communication)
  - [Microsoft Experience on FPGA](#microsoft-experience-on-fpga)
    - [History](#history)
    - [Bing Search via FPGA](#bing-search-via-fpga)
    - [FPGA in networking](#fpga-in-networking)
    - [FPGA in data center](#fpga-in-data-center)
  - [FPGA with CPU in future](#fpga-with-cpu-in-future)
  - [reference](#reference)
- [Cloud ML platform](#cloud-ml-platform)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# System Stack overview

![DL_sys](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/DL_sys.png)

# Deep Learning Programming Style

## Symbolic vs. Imperative Programs

* Whether to build networks with bigger (more abstract) or more atomic operations.
* Whether to build networks with bigger (more abstract) or more atomic operations.

If you are a Python or C++ programmer, then you’re already familiar with imperative programs. Imperative-style programs perform computation as you run them. Most code you write in Python is imperative, as is the following NumPy snippet.
```
import numpy as np
a = np.ones(10)
b = np.ones(10) * 2
c = b * a
d = c + 1
```


When the program executes c = b * a, it runs the actual numerical computation.

Symbolic programs are a bit different. With symbolic-style programs, we first define a (potentially complex) function abstractly. When defining the function, no actual numerical computation takes place. We define the abstract function in terms of placeholder values. Then we can compile the function, and evaluate it given real inputs. In the following example, we rewrite the imperative program from above as a symbolic-style program:
```
A = Variable('A')
B = Variable('B')
C = B * A
D = C + Constant(1)
# compiles the function
f = compile(D)
d = f(A=np.ones(10), B=np.ones(10)*2)
```

As you can see, in the symbolic version, when C = B * A is executed, no computation occurs. Instead, this operation generates a computation graph (also called a symbolic graph) that represents the computation. The following figure shows a computation graph to compute D.

Most symbolic-style programs contain, either explicitly or implicitly, a compile step. This converts the computation graph into a function that we can later call. In the above example, numerical computation only occurs in the last line of code. The defining characteristic of symbolic programs is their clear separation between building the computation graph and executing it. For neural networks, we typically define the entire model as a single compute graph.

Among other popular deep learning libraries, Torch, Chainer, and Minerva embrace the imperative style. Examples of symbolic-style deep learning libraries include Theano, CGT, and TensorFlow. We might also view libraries like CXXNet and Caffe, which rely on configuration files, as symbolic-style libraries. In this interpretation, we’d consider the content of the configuration file as defining the computation graph.

Now that you understand the difference between these two programming models, let’s compare the advantages of each.

### Imperative Programs Tend to be More Flexible

### Symbolic Programs Tend to be More Efficient

```
import numpy as np
a = np.ones(10)
b = np.ones(10) * 2
c = b * a
d = c + 1
...
```

Assume that each cell in the array occupies 8 bytes of memory. How much memory do you need to execute this program in the Python console?

As an imperative program we need to allocate memory at each line. That leaves us allocating 4 arrays of size 10. So we’ll need 4 * 10 * 8 = 320 bytes. On the other hand, if we built a computation graph, and knew in advance that we only needed d, we could reuse the memory originally allocated for intermediate values. For example, by performing computations in-place, we might recycle the bits allocated for b to store c. And we might recycle the bits allocated for c to store d. In the end we could cut our memory requirement in half, requiring just 2 * 10 * 8 = 160 bytes.

Symbolic programs are more restricted. When we call compile on D, we tell the system that only the value of d is needed. The intermediate values of the computation, in this case c, is then invisible to us.

We benefit because the symbolic programs can then safely reuse the memory for in-place computation. But on the other hand, if we later decide that we need to access c, we’re out of luck. So imperative programs are better prepared to encounter all possible demands. If we ran the imperative version of the code in a Python console, we could inspect any of the intermediate variables in the future.

Symbolic programs can also perform another kind of optimization, called operation folding. Returning to our toy example, the multiplication and addition operations can be folded into one operation, as shown in the following graph. If the computation runs on a GPU processor, one GPU kernel will be executed, instead of two. In fact, this is one way we hand-craft operations in optimized libraries, such as CXXNet and Caffe. Operation folding improves computation efficiency

## Case Study: Backprop and AutoDiff

```
A = Variable('A')
B = Variable('B')
C = B * A
D = C + Constant(1)
# get gradient node.
gA, gB = D.grad(wrt=[A, B])
# compiles the gradient function.
f = compile([gA, gB])
grad_a, grad_b = f(A=np.ones(10), B=np.ones(10)*2)
```

The grad function of **D** generates a backward computation graph, and returns a gradient node, **gA**, **gB**, which correspond to the red nodes in the following figure.

![comp_graph_backward](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/comp_graph_backward.png)

The imperative program actually does the same thing as the symbolic program. It implicitly saves a backward computation graph in the grad closure. When you invoked d.grad, you start from d(D), backtrack through the graph to compute the gradient, and collect the results.

The gradient calculations in both symbolic and imperative programming follow the same pattern. What’s the difference then? Recall the be prepared to encounter all possible demands requirement of imperative programs. If you are creating an array library that supports automatic differentiation, you have to keep the grad closure along with the computation. This means that none of the history variables can be garbage-collected because they are referenced by variable d by way of function closure.

What if you want to compute only the value of d, and don’t want the gradient value? In symbolic programming, you declare this with f=compiled([D]). This also declares the boundary of computation, telling the system that you want to compute only the forward pass. As a result, the system can free the memory of previous results, and share the memory between inputs and outputs.

Imagine running a deep neural network with n layers. If you are running only the forward pass, not the backward(gradient) pass, you need to allocate only two copies of temporal space to store the values of the intermediate layers, instead of n copies of them. However, because imperative programs need to be prepared to encounter all possible demands of getting the gradient, they have to store the intermediate values, which requires n copies of temporal space.

As you can see, the level of optimization depends on the restrictions on what you can do. Symbolic programs ask you to clearly specify these restrictions when you compile the graph. One the other hand, imperative programs must be prepared for a wider range of demands. Symbolic programs have a natural advantage because they know more about what you do and don’t want.

https://mxnet.incubator.apache.org/architecture/program_model.html

# GPU in DeepLearning

## GPU architecture

### GPU vs CPU

The overhead on Too much overhead in compute resources and energy efficiency in Fetch/decode/Write Back, so CPU has more overhead compared with GPU

![CPU_GPU](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/CPU_GPU.png)

### Streaming Multiprocessor (SM)

The streaming multiprocessors (SMs) are the part of the GPU that runs our CUDA kernels. Each SM contains the following.

* Thousands of registers that can be partitioned among threads of execution
* Several caches:
– Shared memory for fast data interchange between threads
– Constant cache for fast broadcast of reads from constant memory
– Texture cache to aggregate bandwidth from texture memory
– L1 cache to reduce latency to local or global memory
* Warp schedulers that can quickly switch contexts between threads and issue instructions to warps that are ready to execute
Execution cores for integer and floating-point operations:
– Integer and single-precision floating point operations
– Double-precision floating point
– Special Function Units (SFUs) for single-precision floating-point transcendental functions

![GPU_SM](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_SM.png)

The reason there are many registers and the reason the hardware can context switch between threads so efficiently are to maximize the throughput of the hardware. The GPU is designed to have enough state to cover both execution latency and the memory latency of hundreds of clock cycles that it may take for data from device memory to arrive after a read instruction is executed.

The SMs are general-purpose processors, but they are designed very differently than the execution cores in CPUs: They target much lower clock rates; they support instruction-level parallelism, but not branch prediction or speculative execution; and they have less cache, if they have any cache at all. For suitable workloads, the sheer computing horsepower in a GPU more than makes up for these disadvantages.

## GPU Memory architecture

### GPU memory model

![GPU_memory](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_memory.png)

* GPU has more register than L1 cache, so computation is more powerful but limited memory to save state/Data
* L1 cache is controlled by programmer  

### GPU memory latency

![GPU_memory_latency](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_memory_latency.png)

### GPU memory bandwidth

## GPU programming model

### SIMD

* SIMT: Single Instruction, Multiple Threads
* Programmer writes code for a single thread in a simple C program. All threads executes the same code, but can takedifferent paths.
* Threads are grouped into a block. Threads within the same block can synchronize execution.
* Blocks are grouped into a grid. Blocks are independently scheduled on the GPU, can be executed in any order.
* A kernel is executed as a grid of blocks of threads.

![GPU_SIMT](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_SIMT.png)

### Kernel Execution

* Each block is executed by one SM and does not migrate.
* Several concurrent blocks can reside on one SM depending on block’s memory requirement and the SM’s memory resources.

![GPU_kernel](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_kernel.png)

* A warp consists of 32 threads. A warp is the basic schedule unit in kernel execution.
* A thread block consists of 32-thread warps.
* Each cycle, a warp scheduler selects one ready warps and dispatches the warps to CUDA cores to execute.

![GPU_kernel_2](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_kernel_2.png)

In conclusion

* In CUDA, Every kernel will be executed on SM ( Streaming Multiprocessors ).There will be many of them on the GPU.
* When you invoke a kernel, you will specify how many threads you want to launch in a block and how many blocks you want to launch.
These Blocks gets ultimately mapped to one of the SM's to execute.
* All the threads in a block can share the memory on the SM as they are on the same SM. Now, we have blocks which execute on SM.
* But SM wont directly give the threads the Execution resources.Instead it will try to divide the threads in the block again into Warps(32 threads).
* The Warps in each of the block exhibit SIMD execution ( Single instruction Multiple data ). If there is a memory access to any thread in a warp, SM switches to next warp. This way SM always have some work to do

### Control Flow

###  Thread Hierarchy & Memory Hierarchy

![thread_memory](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/thread_memory.png)


## GPU Example

### Vector Add

![GPU_vector_add](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_vector_add.png)

```
#define THREADS_PER_BLOCK   512
void vecAdd(const float* A, const float* B, float* C, int n) {
    float *d_A, *d_B, *d_C;
    int size = n * sizeof(float);
    cudaMalloc((void **) &d_A, size);
    cudaMemcpy(d_A, A, size, cudaMemcpyHostToDevice);
    cudaMalloc((void **) &d_B, size);
    cudaMemcpy(d_B, B, size, cudaMemcpyHostToDevice);
    cudaMalloc((void **) &d_C, size);
    int nblocks = (n + THREADS_PER_BLOCK - 1) / THREADS_PER_BLOCK;
    vecAddKernel<<<nblocks, THREADS_PER_BLOCK>>>(d_A, d_B, d_C, n);  //Launch the GPU kernel asynchronously
     cudaMemcpy(C, d_C, size, cudaMemcpyDeviceToHost);
    cudaFree(d_A); cudaFree(d_B); cudaFree(d_C);
}
```

### Slide window sum (Use of thread block)

* Consider computing the sum of a sliding window over a vector:
  * Each output element is the sum of input elements within a radius
  * Example: image blur kernel
* If radius is 3, each output element is sum of 7 input elements
* Overhead:
  * Each input element is read 7 times!
  * Neighboring threads read most of the same elements
* solutions: A thread block first cooperatively loads the needed input data into the shared memory.

![GPU_slide_win](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_slide_win.png)

```
__global__ void windowSumKernel(const float* A, float* B, int n) {
    __shared__ float temp[THREADS_PER_BLOCK + 2 * RADIUS];
    int out_index = blockDim.x * blockIdx.x + threadIdx.x;
    int in_index = out_index + RADIUS;
    int local_index = threadIdx.x + RADIUS;
    if (out_index < n) {
        temp[local_index] = A[in_index];
        if (threadIdx.x < RADIUS) {
            temp[local_index - RADIUS] = A[in_index - RADIUS];
            temp[local_index + THREADS_PER_BLOCK] = A[in_index+THREADS_PER_BLOCK];
        }
        __syncthreads();
        float sum = 0.;
                for (int i = -RADIUS; i <= RADIUS; ++i) {
                    sum += temp[local_index + i];
                }
                B[out_index] = sum;
            }
        }
}
```

### Matrix Operation

### Tips for High Performance
* Use existing libraries, which are highly optimized, e.g. cublas, cudnn.
* Use nvprof or nvvp (visual profiler) to debug the performance.
* Use high level language to write GPU kernels.


# Machine Learning HW

## Hardware Specialization in Deep Learning

> Idea: tailor your chip architecture to the characteristics of a stable workload

* Does deep learning constitute a stable workload to justify ASIC-based hardware acceleration?
* What benchmarks would benefit from improvements on clock frequency?
* What benchmarks would benefit from higher memory bandwidth?



For example in CNN:

### Roofline model
* Given:
  * Processor BW
  * Processor Flop/s
* You can find the Speed of Light: Approach the roofline
* Such simple bounds are a powerful tool

The most basic Roofline model can be visualized by plotting floating-point performance as a function of machine peak performance, machine peak bandwidth, and arithmetic intensity. The resultant curve is effectively a performance bound under which kernel or application performance exists, and includes two platform-specific performance ceilings[clarification needed]: a ceiling derived from the memory bandwidth and one derived from the processor's peak performance (see figure on the right)


![roofline_math](http://mathurl.com/yb7qdzu2.png)

where __P__ is the attainable performance, __π__  is the peak performance, __β__ is the peak bandwidth and __I__ is the arithmetic intensity. The point at which the performance saturates at the peak performance level that is where the diagonal and horizontal roof meet, is defined as ridge point.

* the kernel or application is said to be memory-bound if __I≤π/β__ .
* the computation is said to be compute-bound if __I≥π/β__ .

![roofline](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/roofline.png)

### TPU

![TPU_overview](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/TPU_overview.png)

![TPU_compare](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/TPU_comp.png)

## HW/SW Codesign

### Tensorization

![HWSW_codesign_tensorization](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/HWSW_codesign_tensorization.png)


### Memory Architecting

![HWSW_codesign_memory](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/HWSW_codesign_memory.png)


### Data Type
* Reducing type width can result in a quadratic increase of compute resources, and linear increase of storage/bandwidth
* But it also affects classification accuracy!

![HWSW_codesign_INT](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/HWSW_codesign_INT.png)

## Latency Hiding

* Without latency hiding, we are wasting compute/memory resources. By exploiting pipeline parallelism, we can hide memory latency
* Pipeline Parallelism requires
  * Concurrent tasks need to access non-overlapping regions of memory
  * Data dependences need to be explicit!
* enforce read-after-write (RAW) and write-after-read (WAR) dependences
* Takeaway: work partitioning and explicit dependence graph execution (EDGE) unlocks pipeline parallelism to hide the latency of memory accesses

![Latency_hiding](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/Latency_hiding.png)

# Memory Optimization

The maximum size of the model we can try is bounded by total RAM available of a Titan X card (12G)

## Build an Executor for a Given Graph

* Allocate temp memory for intermediate computation
* Traverse and execute the graph by topology order.

### Dynamic Memory Allocation:

* Allocate when needed
* Recycle when a memory is not needed.
* Useful for both declarative and imperative executions

### Static memory allocation

* Plan for reuse ahead of time
* Analog: register allocation algorithm in compiler

##  Common Patterns of Memory Planning

* **Inplace** store the result in the input

  * We can only do inplace if result op is the only consumer of the current value


![memory_planing](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/memory_planing.png)

> Concurrency vs Memory Optimization

![concurrent_memory](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/concurrent_memory.png)

* **Normal Sharing** reuse memory that are no longer needed.

## Memory Allocation and Scheduling

* Explicitly add the control flow dependencies ○ Needed in TensorFlow
* Enable mutation in the scheduler, no extra job needed
  * Both operation “mutate” the same memory
  * Supported in MXNet

##  Memory Plan with Gradient Calculation

Why do we need automatic differentiation that extends the graph instead of backprop in graph?



# Parallel schedule

## Common patterns of parallelization

* Map parts of workload to different devices
* Require special dependency patterns (wave style), e.g. LSTM


## Data parallelism

* Train replicated version of model in each machine
* Synchronize the gradient

![ML_parallel](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/ML_parallel.png)

> Design goal

* Write Serial Program
* Possibly dynamically (not declare graph beforehand)
* Run in Parallel
* Respect serial execution order

### DAG based scheduler

```
engine.push(lambda op, deps=[])
```

* Explicit push operation and its dependencies
* Can reuse the computation graph structure
* Useful when all results are immutable
* Used in typical frameworks (e.g. TensorFlow)

> What are the drawbacks?

### mutation aware scheduler

The user then calls **push** to tell the engine about the function to execute. The user also needs to specify the dependencies of the operation, using **read_vars** and **write_vars**:

* **read_vars** are variable tags for objects that the operation will read from, without changing their internal state.
* **mutate_vars** are variable tags for objects whose internal states the operation will mutate.

![mutation_scheduler](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/mutation_scheduler.png)

The preceding figure shows how to push operation __B = A + 1__ to the dependency engine. B.data and A.data are the allocated space. Note that the engine is only aware of variable tags. Any execution function can be processed. This interface is generic for the operations and resources we want to schedule.

###  Queue based Implementation of scheduler

* Like scheduling problem in OS
* Maintain a pending operation queue
* Schedule new operations with event update

> Examples

The engine maintains a queue for each variable. Green blocks represents a read action, while red blocks represent mutations.

![dep_parallel](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/dep_parallel.png)


Upon building this queue, the engine sees that the first two green blocks at the beginning of A‘s queue could actually be run in parallel because they are both read actions and won’t conflict with each other. The following graph illustrates this point.

## Model parallelization

## Summary

* Automatic scheduling makes parallelization easier
* Mutation aware interface to handle resource contention
* Queue based scheduling algorithm


# Distributed Machine Learning

http://dlsys.cs.washington.edu/schedule

## Computation vs Communication 

Some model has high computation requirement, while the parameter size is not so large, so computation overhead is more important.

* Resnet or most CNN based solution, the parameters size of CNN is moderate, but computation overhead is high. 
* Full connected, parameter size is large: 


## Deep learning computation model

Lots of machine learning and deep learning, like NN, graphical model, Matrix Factorization, can be abstracted as

![ML_equation](http://mathurl.com/yat9ozoy.png)


* each iteration ![theta](http://mathurl.com/y9g6t9ew.png) is the model parameters, and  ![delta](http://mathurl.com/y7vtqbnk.png) function is the update function,
* each time  ![delta](http://mathurl.com/y7vtqbnk.png) will get the current model's parameter ![theta_t](http://mathurl.com/y8aaxmyx.png) and the training  data ![D_t](http://mathurl.com/y88cxtk6.png) as input, calculate  and directly added into current paramter set  ![theta_t](http://mathurl.com/y8aaxmyx.png), to update paramter into ![theta_t+1](http://mathurl.com/ya2kbabj.png)
* Iterate this until some conditions meet, like Iterate  __T__ times, or other stop conditions

In NN training, like using SGD, ![theta](http://mathurl.com/y9g6t9ew.png) is the NN parameter, __L__ is the optimization function. ![delta_L](http://mathurl.com/y9v7pmh4.png) is the gradient decent. So in each batch, using the original data set ![D_t](http://mathurl.com/y88cxtk6.png) and calculate the target optimization function __L__ decent to ![theta](http://mathurl.com/y9g6t9ew.png), mutiply learning rate and add momentum to get updated parameters


## model parallelism vs data parallelism

In model parallelism, different machines in the distributed system are responsible for the computations in different parts of a single network - for example, each layer in the neural network may be assigned to a different machine.

In data parallelism, different machines have a complete copy of the model; each machine simply gets a different portion of the data, and results from each are somehow combined.

Of course, these approaches are not mutually exclusive. Consider a cluster of multi-GPU systems. We could use model parallelism (model split across GPUs) for each machine, and data parallelism between machines.

## Data Parallelism

Data parallel approaches to distributed training keep a copy of the entire model on each worker machine, processing different subsets of the training data set on each. Data parallel training approaches all require some method of combining results and synchronizing the model parameters between each worker. A number of different approaches have been discussed in the literature, and the primary differences between approaches are

* Parameter averaging vs. update (gradient)-based approaches
* Synchronous vs. asynchronous methods
* Centralized vs. distributed synchronization




### Parameter Averaging
Parameter averaging is the conceptually simplest approach to data parallelism. With parameter averaging, training proceeds as follows:

1. Initialize the network parameters randomly based on the model configuration
2. Distribute a copy of the current parameters to each worker
3. Train each worker on a subset of the data
4. Set the global parameters to the average the parameters from each worker
5. While there is more data to process, go to step 2

![parameter_average](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/parameter_average.png)

Steps 2 through 4 are demonstrated in the image below. In this diagram, W represents the parameters (weights, biases) in the neural network. Subscripts are used to index the version of the parameters over time, and where necessary for each worker machine.

we instead perform learning on m examples in each of the n workers (where worker 1 gets examples 1, ..., m, worker 2 gets examples m + 1, ..., 2m and so on), we have:

![parameter_average_math](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/parameter_average_math.png)


Now, parameter averaging is conceptually simple, but there are a few complications that we’ve glossed over.

First, how should we implement averaging? The naive approach is to simply average the parameters after each iteration. While this can work, we are likely to find that the overhead of doing so to be impractically high; network communication and synchronization costs may overwhelm the benefit obtained from the extra machines. Consequently, parameter averaging is generally implemented with an averaging period (in terms of number of minibatches per worker) greater than 1. However, if we average too infrequently, the local parameters in each worker may diverge too much, resulting in a poor model after averaging. The intuition here is that the average of N different local minima are not guaranteed to be a local minima:

### Update based method

A conceptually similar approach to parameter averaging is what we might call ‘update based’ data parallelism. The primary difference between the two is that instead of transferring parameters from the workers to the parameter server, we will transfer the updates (i.e., gradients post learning rate and momentum, etc.) instead. This gives an update of the form:

![Stochastic_average_math](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/stochastic_average_math.png)

The worker machine diagram looks like:

![Stochastic_average](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/stochastic_average.png)

If we again define our loss function as L, then parameter vector W at iteration i + 1 for simple SGD training with learning rate α is obtained by:

![Stochastic_average_math_2](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/Stochastic_average_math_2.png)

![Stochastic_average_math_3](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/Stochastic_average_math_3.png)

Consequently, there is an equivalence between parameter averaging and update-based data parallelism, when parameters are updated synchronously (this last part is key). This equivalence also holds for multiple averaging steps and other updaters (not just simple SGD).

### Asychronous Stochastic Gradient Descent

Update-based data parallelism becomes more interesting (and arguably more useful) when we relax the synchronous update requirement. That is, by allowing the updates ∆Wi,j to be applied to the parameter vector as soon as they are computed (instead of waiting for N ≥ 1 iterations by all workers), we obtain asynchronous stochastic gradient descent algorithm. Async SGD has two main benefits:

* First, we can potentially gain higher throughput in our distributed system: workers can spend more time performing useful computations, instead of waiting around for the parameter averaging step to be completed.

* Second, workers can potentially incorporate information (parameter updates) from other workers sooner than when using synchronous (every N steps) updating.

By introducing asynchronous updates to the parameter vector, we introduce a new problem, known as the stale gradient problem. The stale gradient problem is quite simple: the calculation of gradients (updates) takes time. By the time a worker has finished these calculations and applies the results to the global parameter vector, the parameters may have been updated a number of times. This problem is illustrated in the figure below.

![stale_gradient](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/stale_gradient.png)

Most variants of asynchronous stochastic gradient descent maintain the same basic approach, but apply a variety of strategies to minimize the impact of the stale gradients, whilst attempting to maintaining high cluster utilization. It should be noted that parameter averaging is not subject to the stale gradient problem due to the synchronous nature of the algorithm.

* Scaling the value λ separately for each update ∆Wi,j based on the staleness of the gradients

* Implementing ‘soft’ synchronization protocols ([9])

* Use synchronization to bound staleness. For example, the system of [4] delays faster workers when necessary, to ensure that the maximum staleness is below some threshold

## Distributed Asychronous Stochastic Gradient Descent

* No centralized parameter server is present in the system (instead, peer to peer communication is used to transmit model updates between workers).
* Updates are heavily compressed, resulting in the size of network communications being reduced by some 3 orders of magnitude.

# Parameter Server

## Design principal

> Design the equation into distributed way


 ![distribute_ML](http://mathurl.com/y79pumk2.png)

Compared wih single machine, only to distribute calculation into different machines.


* The training will be mainly on ![delta](http://mathurl.com/y7vtqbnk.png) function, since the core distribute function to be executed in different machine is ![delta](http://mathurl.com/y7vtqbnk.png) function

* In each iteration, each machine will calculate the ![delta](http://mathurl.com/y7vtqbnk.png) function independently, there is __NO dependency__ between different machines

* Before each iteration, the model parameters ![theta](http://mathurl.com/y9g6t9ew.png)  will be shared by all Nodes.

> So the architecture will looks like

![PS](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/PS.jpg)

Here the parameter sever is logic concept, it dose not need to be a single centralized server. but could be servers in data center.

## Key consideration

* How to store all parameter
* Push and Pull API Design
* Synchronous or Asynchronous
* Communication bandwidth
* Fault tolerance(if some servers is down, how to recover)
* Node computation model, CPU to GPU


## System view
* Initialize model with small random values Paid Once
  * fairly trivial to parallelize
* Try to train the right answer for your input set Iterate through the input set many many times
* Adjust the model: Send a small update to the model parameters



### Challenge

* Three main challenges of implementing a parameter server:
  * Accessing parameters requires lots of network bandwidth
  * Training is sequential and synchronization is hard to scale
  * Fault tolerance at scale (~25% failure rate for 10k machine-hour jobs)

* General purpose machine-learning frameworks
  * Many have synchronization points -> difficult to scale
  * Key observation: cache state between iterations

### Large data

What are parameters of a ML model? Usually an element of a vector, matrix, etc. Need to do lots of linear algebra operations.

* Introduce new constraint: ordered keys
* Typically some index into a linear algebra structure
* High model complexity leads to overfitting: Updates don’t touch many parameters
  * Range push-and-pull: Can update a range of rows instead of single key
  * When sending ranges, use compression

### Synchronization

* ML models try to find a good local min/min
* Need updates to be generally in the right direction
* Not important to have strong consistency guarantees all the time
* Parameter server introduces Bounded Delay

### Fault tolerance
* Server stores all state, workers are stateless
  * However, workers cache state across iterations
* Keys are replicated for fault tolerance
* Jobs are rerun if a worker fails

## PS in GPU cluster

Two challenges for GPU clusters

* GPU has its own memory, so we need to have data inside CPU memory. Since all data will need to be pulled from remote node, and most of networking protocol only support RAM to RAM (like RDMA). Nvidia has own GPU direct technology, but it requires dedicated HW support

* Need to have memcpy from RAM into GPU memory

So if we have Push and Pull API and two memcpy, we can have PS ported to GPU. here are two simple implementation: https://github.com/sailing-pmls/bosen and https://github.com/sailing-pmls/pmls-caffe

### GPU challenge

In most times, GPU is too fast so we can easily have problem when deploy the real model into systems

* Memory Copy: Memory copy has overhead and compared with GPU fast computation the overhead is higher
* Communication bandwidth/latency: different node and networking condition could lead to different latency, may need to spend quite long time just to wait a node to finish
* GPU memory: limited memory(Nvidia has largest 12G) compared with RAM.

#### GPU bandwidth example
> Example: Training Alexnet in Nvidia Gefore Titan X (GM200)

61.5M paramerers for Alexnet, Batch size 256, a GPU can finish computation in 0.25s for each iteration.  assume we have 8 GPUs and all numbers are float.

* In PS  client, in order to pass all parameters in synchronous way, in each 0.25s, each GPU machine need to push and pull 61.5M. so the network bandwidth needed is 61.5M × 2 / 0.25s = 492M/s, converting to networking Tput is 492M/s * 32 bit = 15744 Mbps = 15.7 Gbps
* In server side, in each 0.25s, server need to Push and Pull 61.5M*8 = 492M gradient paramter values. Network bandwidth needed is 492M * 2 / 0.25s = 3936 M/s, converting networking Tput is 3936M/s * 32 bit = 125.9 Gbps

Even the highest AWS GPU can only provide 20Gbps, and it only has 8 GPU(not even the best Volta)

### solutions

#### model parallelism
Since PS server is a logic concept, we can distribute all parameters evenly in 8 GPUs, so each machine is both client worker and server. each node only need to compute 1/8 parameter size. so each node will need to push and pull as worker, and push and pull as server. 4 × 61.5M / 0.25s × 7/8 = 840 M/s = 26Gbps, but it is only assumption in optimized situation. communication need for distribute GPUs are very high

like using GPU to train Resnet, Resnet has deeper network so computation time is higher compared with communication time, also it is CNN based so parameter size is smaller

http://www.pdl.cmu.edu/PDL-FTP/CloudComputing/GeePS-cui-eurosys16.pdf

#### Asychronous: Overlap between computation and communication

The basic $ {\large \texttt{pull} \rightarrow \Delta_{\mathcal{L}} \rightarrow \texttt{push}} $ gives us hint that we could overlap computation with communication to achieve better results.


The idea is in NN training, both backward propagation and parameter pass are based on layer. __i th__ layer parameter push/Pull are independent with __i-1 th__ layer. Also since parameter update are independent, we do not need to wait all parameter to be ready and pull all of them. instead, we can pull after one layer has finished training.  

Also most model(CNN) has high computation need in initial CNN stage with less parameter and communication bandwidth, but less computation with more parameter/communication bandwidth in later stage like full connected layer

> Wait-free Backpropagation（WFBP）

WFBP can be illustrated as

![WFBP](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/WFBP.jpg)

test results can be found in https://www.usenix.org/system/files/conference/atc17/atc17-zhang.pdf

but, still, as model becomes larger, GPU becomes faster, networking bandwidth will be a big challenge. compared with PCIe in single machine, multiple nodes's communication will be a bottleneck

#### Asychronous: Sufficient Factor Broadcasting
http://auai.org/uai2016/proceedings/papers/10.pdf

Assume a __NxM__ full connected layer network with Batch size B, so the gradient parameters can be divided as num of B product of vectors with length N and M, so in reality, we can only send B times vector N and M instead of a large matrix.

Similar system is Microsoft Project Adam : https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-chilimbi.pdf

# All reduce Approach

## communication primitive

* Reduce:

Multiple senders send information and combine in one node

![reduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/reduce.png)

* All reduce

Multiple senders send information and combine and distribute to all nodes

![all_reduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/all_reduce.png)

Traditional Collective communication assumes a tree style topology, but it dose not assume a flat tree, instead it use ring-based Collective communication.

## Ring-based Collective communication

Ring based assumes all nodes are in a circle.

* Assume there are K nodes, N data, and bandwidth is B

total communication time is:

```
(K-1)*N/B
```

* Split the data into S, then we split N/S data

total communication time is:

```
S*(N/S/B) + (k-2)* (N/S/B) = N(S+K-2)/(SB) --> N/B

if S>>K, which is normally true
```

![collective_communication](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/collective_communication.png)

http://mpitutorial.com

### MPI Reduce and Allreduce
http://mpitutorial.com/tutorials/mpi-reduce-and-allreduce/

![MPI_reduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/mpi_reduce.png)

![MPI_allreduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/mpi_allreduce.png)

### limitation of reduce via map

* persistent resources requirements

machine learning programs usually consume more resources, they often demand allocation of temporal results and caching. Since a machine learning process is generally iterative, it is desirable to make it __persistent__ across iterations.

## Allreduce in practice

An abstraction that overcomes such limitations is Allreduce. Allreduce avoids the unnecessary map phases, real- location of memory and disk reads-writes between iterations.

* A single map phase takes place, followed by one or more Allreduce stages

* The program persists across iterations, and it re-uses the resources allocated previously. It is therefore a more convenient abstraction for solving many large scale machine learning problems.

![reduce_allreduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/reduce_allreduce.png)





Allreduce is an abstraction commonly used for solving machine learning problems. It is an operation where every node starts with a local value and ends up with an aggre- gate global result.



* OpenMPI is non fault-tolerant



In Allreduce settings, nodes are organized in a tree structure. Each node holds a portion of the data and computes some values on it. Those values are passed up the tree and aggregated, until a global aggregate value is calculated in the root node (reduce). The global value is then passed down to all other nodes (broadcast).

![tree_allreduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/tree_allreduce.png)

Several machine learning problems fit into this abstrac- tion, where every node works on some portion of the data, computes some local statistics (e.g. local gradients) and then invokes the Allreduce primitive to get a global ag- gregated result (e.g. global gradient) in order to further proceed with the computations.

## Tree based Allreduce vs Ring based Allreduce

### limitations of Tree Allreduce
![allreduce_issue](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/allreduce_issue.png)

If we have 300M parameters and each is 4 Bytes. it will have 1.2G data, and assuming bandwidth is 1GB/s. if we use 2 GPUs, delayed 1.2s, 10 GPUs, delayed 10.8s for each epoch

### Ring Allreduce
GPUs have been placed in logical ring. GPU get data from left GPU and sends to right GPU
![ring_reduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/ring_reduce.png)

### Scatter-Reduce
assuming we have N GPUs in system and every GPU has same length array. GPU will chunk the data into N small pieces.

GPU will execute N-1 times iterations. In every iteration, sender and receiver are different. n GPU will send n block and receive n-1  

```
GPU #, Send, receive
0 Chunk 0 Chunk 4
1 Chunk 1 Chunk 0
2 Chunk 2 Chunk 1
3 Chunk 3 Chunk 2
4 Chunk 4 Chunk 3
```

The figure shows the data flow for one iteration
![ringreduce_data](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/ringreduce_data.png)

This figure shows middle results after one iteration
![ringreduce_middle](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/ringreduce_middle.png)

At the end, every GPU block will have all results from same blocks along other GPUs
![ringreduce_end](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/ringreduce_end.png)

### AllGather
After scatter reduce, every GPU has some values from all GPUs, they need to exchange these values so all GPUs have all values

allgather is same as scatter reduce, except that it is not sum, just simple copy, the process will look like
![allgather](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/allgather.png)

And after N-1 iteration, final results will be
![allgather_end](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/allgather_end.png)

### Overhead Analysis
* Communication cost

N GPUs, in Scatter-Reduce stage, each one will have N-1 data receiving, and send k data(K is array length in single GPU), in allgather stage, another N-1 receving and N sending

For Each GPU,
```
2(N−1)⋅K
```

### RABIT: A Reliable Allreduce and Broadcast Interface
http://homes.cs.washington.edu/~icano/projects/rabit.pdf

* Flexibility in programming: Programs can call Ra- bit functions in any order, as opposed to frameworks where callbacks are offered and called at specific points in time.
* Programs persistence: Programs continue running over all iterations, instead of being restarted on each one of them.
* Fault tolerance: Rabit programs can recover their state (e.g. machine learning model) using synchronous function calls.
* MPIcompatible: Code that uses Rabit API also compiles with existing MPI compilers, i.e. users can use MPI Allreduce with no code modification.

#### Fault-tolerant

1. Pause every node until the failed node is fully recov- ered.
2. Detect the model version we need to recover by ob- taining the minimum operation number. This is done using the Consensus Protocol.
3. Transfer the model to the failed node using the Routing Protocol.
4. Resume the execution of the failed node using the received model.
5. Resume the execution of the other nodes as soon as the failed node catches up

![allreduce_recovery](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/allreduce_recovery.png)

#### Recovery
Message Passing: nodes send messages to their neighbors to find out information about them. It can be used in many algorithms such as Shortest Path

#### Consensus Protocol
The consensus protocol agrees on the model version to recover using an Allreduce min operation. In the example shown in Figure,
![min_recovery](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/min_recovery.png)
the value of each node is the model version they store. The green node on the left subtree is recovering so it wants to find which model it needs in order to make progress. It does so with the Allreduce min operation. After the value 0 is broadcast-ed to every node, everyone knows that model 0 is the version to be used
for recovery.

#### Routing
Routing: Once the model version to recover is agreed among the nodes, the routing protocol executes. It finds the shortest path a failed node needs to use in order to retrieve the model. It uses two rounds of message passing.
* The first round computes the distance from a failed node to the nearest node that has the model.
* The second round sends requests through the shortest path until it reaches the node that owns the model, and retrieves it.



## Allreduce vs Paramter server
http://hunch.net/?p=151364

# Nvidia Multi-GPU Lib: NCCL

NCCL (pronounced "Nickel") is a stand-alone library of standard collective communication routines, such as all-gather, reduce, broadcast, etc., that have been optimized to achieve high bandwidth over PCIe, Nvlink、InfiniBand

### Ring based collective

* Implement in GPU environment

![GPU_allreduce](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/GPU_allreduce.png)

## NCCL Implementation

NCCL Implements CUDA C++ kernels. including 3 primitives: Copy，Reduce，ReduceAndCopy. NCCL can support PCIe、NVlink、GPU Direct P2P. NCCL 2.0 can support many machines using Sockets (Ethernet), InfiniBand with GPU Direct RDMA

![NCCL_mode](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/NCCL_mode.png)


### Performance

* Single node multiple GPU Cards

![allreduce_perf_1](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/allreduce_perf_1.png)

DGX_1 can achieve 60G bps, the former 3 is single machine multiple GPU cards. first is 2 GPU connects via 2 CPUs with QPI, second is 2 GPUs connect with switch, and finally with CPU. third one is all four GPU cards via PCIe.

* Multiple nodes and Multiple GPUs

![allreduce_perf_2](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/allreduce_perf_2.png)

PCIe within machine and InfiniBand among nodes


# FPGA on Machine Leaning/Deep Learning

## FPGA vs CPU/GPU

Both CPU and GPU are based on Von Neumann architecture:

* need to compile code to instructions

In Von Neumann architecture, since CPU/GPU need to execute any instruction. it needs to have parts consisting of a processing unit containing an arithmetic logic unit and processor registers; a control unit containing an instruction register and program counter; a memory to store both data and instructions.

> So GPU/CPU use SIMD(Single Instruction Multiple Data) to process to let multiple process unit to process different data with same logic to achieve parallelism

* have shared memory

Memory is needed to save states and share memory for communication

On the other hand, FPGA dose not need to have

* No instruction set

For FPGA, all logic has been fixed once programming the new logic.

* No shared memory

FPGA's register and BRAM belong to its own logic, no need for cache and sharing state among different logic. and all link between current logic and other logic is decided when programming, so no need for communication



## FPGA advantage on computation

> The core advantage of FPGA in machine learning is delay

### Training and inference stage difference

In Training stage, throughput/bandwidth is more important than latency/delay. In inference stage, delay/latency is more important. like in search/recommendation, we want to have the results to be returned back in hundred ms

### FPGA vs GPU on parallelism

* FPGA has both pipeline parallel and data parallelism

* GPU only has data parallelism

for example, if a data process needs 10 steps, FPGA can set up a 10 stage pipeline, each step process one logic. For GPU, it will set  10 computation units with same instruction sets, and all computation unit will do same thing (SIMD), so it needs to wait all 10 data unit to be ready(input) and process them then output in parallelism.

> For streaming computation, FPGA has delay advantage

### FPGA vs ASIC

ASIC still have delay and computation best performance, however,
* it dose have large investment both capital and time
* if the algorithm changes, need to re tap out
* and for data center, managing different ASIC in hetergeneous environment is not easy

## FPGA advantage on communication

* FPGA can direct connect to 40G, 100G ethernet and process data packet in line rate

* CPU need to get data from ethernet adaptor, and for 64 bytes small packet, most of ethernet adaptor can not handle, so need to have multiple network card to support.

* The latency to get data from network card to CPU and then send back is normally 4-5 us even we use like DPDK. and the OS in CPU usually add uncertain time

* Similar as CPU. GPU need to get data from network card and send back, which adds more latency

## Microsoft Experience on FPGA

### History

Microsoft has evolved largely in 3 different stages

1. FPGA clusters

2. FPGA in each machine connected via dedicated network

3. FPGA in each machine between data center network card and switch machine

### Bing Search via FPGA

![BingFPGA](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/BingFPGA.png)

### FPGA in networking

* The need for FPGA in networking process

CPU used to be able to handle the networking packet easily in 1G networking and hard disk time, however, when the networking speed has increased to 40Gbps and SSD speed is 1Gbps, CPU load becomes high. for example, a Hyper-V virtual network switch can only process 25Gbps and can not achieve 40Gbps line speed. and performance decrease when packet size is small, for AES/SHA, CPU can only process like 100Mbps, which is even less than SSD 1Gbps. Following graph shows performance

![networkingCPU](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/networkingCPU.png)

* Azure networking via FPGA

In order to achieve network virtualization. Azure deploy the FPGA between network card and switch: FPGA has 4 GB DDR3-1333 DRAM, and connect to CPU socket via PCIe Gen3 x8

![azurenetworkingFPGA](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/azurenetworkingFPGA.png)

![FPGAvirtualnetwork](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/FPGAvirtualnetwork.png)

### FPGA in data center

The data center can use the high throughput and low latency FPGA to create a Accelerator layer between network switch layer and server layer, as shown in below

![FPGADC](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/FPGADC.png)

Also as more FPGAs are used, the performance boost could be Accelerated. for example, a single FPGA may not be able to load all NN parameters, so still need to have overhead to process DRAM parameter. but with lots of FPGA, we can load NN model into many different FPGAs

## FPGA with CPU in future

FPGA is more suitable for repeatable, streaming style process with low latency. CPU can be used for complex task(where FPGA need to have dedicated logic resource to process different task, so if many different tasks need to be processed, FPGA has large resource waste).

## reference

https://www.zhihu.com/question/24174597

* [Serving DNNs in Real Time at Datacenter Scale with Project Brainwave]https://www.microsoft.com/en-us/research/uploads/prod/2018/03/mi0218_Chung-2018Mar25.pdf

* Large-Scale Reconfigurable Computing in a Microsoft Datacenter https://www.microsoft.com/en-us/research/wp-content/uploads/2014/06/HC26.12.520-Recon-Fabric-Pulnam-Microsoft-Catapult.pdf
* A Reconfigurable Fabric for Accelerating Large-Scale Datacenter Services, ISCA'14 https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/Catapult_ISCA_2014.pdf

* Microsoft Has a Whole New Kind of Computer Chip—and It’ll Change Everything: A Cloud-Scale Acceleration Architecture, MICRO'16 https://www.microsoft.com/en-us/research/wp-content/uploads/2016/10/Cloud-Scale-Acceleration-Architecture.pdf

* ClickNP: Highly Flexible and High-performance Network Processing with Reconfigurable Hardware - Microsoft Research[6] Daniel Firestone, SmartNIC: Accelerating Azure's Network with. FPGAs on OCS servers.

* How to Use FPGAs for Deep Learning Inference to Perform Land Cover Mapping on Terabytes of Aerial Images: https://blogs.technet.microsoft.com/machinelearning/2018/05/29/how-to-use-fpgas-for-deep-learning-inference-to-perform-land-cover-mapping-on-terabytes-of-aerial-images/

# Cloud ML platform

https://docs.microsoft.com/en-us/azure/machine-learning/data-science-virtual-machine/dsvm-tools-overview

https://github.com/Azure/kubeflow-labs

https://docs.microsoft.com/en-us/azure/aks/?WT.mc_id=blog-medium-abornst
