---?image=assets/DESIGN_BELLEVUE2018.png&position=center&size=auto 100%&color=#4b285b

---?image=assets/mathew-gardner.jpg&position=center&size=auto 100%&color=#4b285b

Note:

- Elixir for ~2.5 years
- Joined TC 1.5 years
- Learning ML on side
- ML opportunity at TC

---

## Interfacing with Machine-Learned Models

<br>

Note:

- Inspired me to research ML & Elixir

+++

## What is Machine Learning?

<br>

@ul

- Computer science meets statistics
- We try to get programs to improve without being explicitly programmed to do so
- Use data as past experiences from which to _learn_

@ulend

Note:

- Supervised
- Unsupervised
- Deep Learning
- SciKit-Learn

+++

## What is a model?

<br>

@ul

- Represents what we've learned from our dataset
- May be a matrix whose values are learned weights to be applied during a matrix transformation
- May be other learned values such as the slope of a line

@ulend

Note:

- Build artifact from ML CI
- Black-box function

+++?image=assets/regression.png&position=center&size=auto 75%

Note:

- regression, line best fit

+++?image=assets/classification.svg&position=center&size=auto 60%

Note:

- classification, separate samples

+++

## Classic ML Example - The Iris Data Set

Note:

- Identify species of Iris
- 150 samples
- 4 features
- sepal length, sepal width, petal length, petal width

+++?image=assets/setosa.jpg&position=center&size=auto 100%

Note:

- setosa

+++?image=assets/versicolor.jpg&position=center&size=auto 100%

Note:

- versicolor

+++?image=assets/virginica.jpg&position=center&size=auto 100%

Note:

- virginica

+++

## Our Goal

@ul

- Identify a species of Iris from a set of sepal and petal measurements
- Train a statistical model to classify the Iris (Setosa, Versicolour, and Virginica)
- Use measurements that are already labeled the correct class

@ulend

Note:

- features
- supervised

+++

## [Looking at Our Data](http://scikit-learn.org/stable/auto_examples/datasets/plot_iris_dataset.html)

| IDX | Sepal Length (cm) | Sepal Width (cm) | Petal Length (cm) | Petal Width (cm) | Class |
| - | - | - | - | - | - |
| 1 | 6.5 | 3.0 | 5.2 | 2.0 | 2 |
| 2 | 6.6 | 3.0 | 4.4 | 1.4 | 1 |
| 3 | 4.9 | 3.1 | 1.5 | 0.1 | 0 |
| 4 | 5.8 | 2.8 | 5.1 | 2.4 | 2 |
| 5 | 5.6 | 2.7 | 4.2 | 1.3 | 1 |

Note:

- Dataset has 150 samples

+++?image=assets/sepals.png&position=center&size=auto 90%

Note:

- Obvious line between red and others

+++?image=assets/eigenvectors.png&position=center&size=auto 90%

Note:

- Can see how the samples are clustered

+++

## Show me some code already!

+++?gist=https://gist.github.com/mathewdgardner/10aa4e5751da61c5fdb17b0aa3b8d585&lang=python&title=Train LinearSVC model

@[1-4](Import libraries)
@[6-7](Load Iris data set)
@[8](Select features and label)
@[10-12](Train a model)
@[14-15](Pickle the model to load later)

Note:

- Python is OOP
- is mutable!

---

## So we've got a model, now what?

+++

## How we'll interact

@ul

- HTTP
- gRPC
- Ports
- NIFs
- Elixir!

@ulend

Note:

- Outside -> in approach

---

## HTTP Endpoint

Basic _near_ RESTful endpoint

+++?gist=https://gist.github.com/mathewdgardner/07d497694911266bd95e27a6e9407d80&lang=python&title=Simple Python HTTP Server

@[1-6](Import libraries)
@[34-38](Load our model)
@[8-9](Define our request handler)
@[10-13](Parse the body)
@[15-17](Make prediction)
@[19-27](Send response)
@[40-49](Start our server)

Note:

- Basic HTTP server in python

+++?gist=https://gist.github.com/mathewdgardner/f30ca808ff28ce3368865ce7d0f9ff13&lang=elixir&title=Simple HTTP client

@[2-6](Define predict function)
@[7-18](Create payload)
@[20-26](Make request and get prediction)

---

## gRPC

+++

## gRPC Service

@ul

- Uses HTTP2
- Long lived connection
- Header compression
- Connection is multiplexed

@ulend

Note:

- less overhead
- faster
- tensorflow serving uses gRPC

+++?gist=https://gist.github.com/mathewdgardner/3d87117bf9461ebe9e1824f2766a1af3&lang=protobuf&title=gRPC Proto File

@[10-12](Define our service)
@[21-23](Define our request)
@[25-27](Define our response)
@[14-19](Define our features)

+++

## Generate Python service code from proto file

```sh
python -m grpc_tools.protoc -I ../protos --python_out=. --grpc_python_out=. ../protos/prediction.proto
```

Note:

- both need the proto file

+++?gist=https://gist.github.com/mathewdgardner/2b7e6d0c12fb4748ec66cd5d854916f8&lang=python&title=Python gRPC Server

@[27-30](Load our model)
@[32-36](Create gRPC server)
@[34](Add predictions service with model)
@[10-13](Define our service)
@[15-23](Define our prediction function)

+++

## Generate Elixir client code from proto file

```sh
protoc --elixir_out=plugins=grpc:./lib -I ../protos ../protos/prediction.proto
```

Note:

- both need the proto file

+++?gist=https://gist.github.com/mathewdgardner/1a23a76ab0ee317bfa6f3fb7845c3933&lang=elixir&title=Elixir gRPC client

@[2](Using GenServer behaviour)
@[4-6](Using GenServer behaviour)
@[18-22](Connect to gRPC server)
@[8-14](Define predict function)
@[24-36](Make request to gRPC server)
@[27-32](Formulate request)
@[33](Formulate request)
@[34](Send request to make prediction)
@[35](Reply with prediction)

Note:

- Use genserver to hold channel state

---

## Ports

+++

## Using ErlPort

@ul

- Can call Python functions from Erlang
- Can call Erlang functions from Python
- Can send messages back and forth
- Uses STDIN and STDOUT under the hood

@ulend

+++?gist=https://gist.github.com/mathewdgardner/b9b461d0dff11b4e63f65ce768c0b48c&lang=python&title=Python prediction

@[1-8](Very little Python code)
@[3-5](Load our model)
@[7-8](Define predict function)

Note:

- Just load model
- just predict

+++?gist=https://gist.github.com/mathewdgardner/b5e0b96f04b856939c126f211ec58a7d&lang=elixir&title=Elixir with erlport

@[2](Using GenServer behaviour)
@[4-6](Using GenServer behaviour)
@[18-24](Establish port with erlport)
@[8-14](Define predict function)
@[26-31](Define predict function)
@[29](Call over to python land)
@[30](Reply with our prediction)
@[33-36](Gracefully shutdown)

---

## NIFs

+++

## DANGER ZONE!

@ul

- Crashes in your NIF will crash the BEAM
- NIFs that need lots of CPU time need to yield back to the VM
- Loaded NIFs are public functions
- With great power comes great responsibility

@ulend

Note:

- Don't do this unless you know C
- Implement time slices
- Andrew bennet last year "well behaved nifs"

+++?image=assets/sklearn-porter.png&position=center&size=auto 90%

Note:

- Can generate code for lots of models
- Chose C
- chose linear svc

+++?gist=https://gist.github.com/mathewdgardner/518da7c65ecbf8450777878bcda2ee91&lang=c&title=Generated C code

@[5-6](Learned values)
@[8-22](Just a nested for-loop)
@[25-38](Drop main function)

Note:

- for SVM classifier
- don't need main
- only want predict and numbers

+++?gist=https://gist.github.com/mathewdgardner/12f0d2baaa39b3824c487b166358ed28&lang=c&title=NIF C code

@[1](Use erl_nif library)
@[4](Define predict function wrapper)
@[6-12](Turn Erlang values into C values)
@[14-20](Make features array)
@[22-23](Call generated predict function)
@[25-26](Return Erlang integer from prediction)
@[29-36](Tell erl_nif which function to load into the BEAM)

Note:

- include erl_nif.h
- Make wrapper function for generated func
- convert values to c
- make features array
- call predict
- convert c int to erlang int
- export func

+++?gist=https://gist.github.com/mathewdgardner/87f49ef6eb6ebe0b12a0b011dbfc2c88&lang=Makefile&title=Build the NIF

@[1](Find erlang path to include erl_nif.h)
@[3](Compile flags)
@[9-10](Compile nif.so)

Note:

- shared object
- include erlang path

+++?gist=https://gist.github.com/mathewdgardner/1f6efeeef8fe4271add8e51a2e4e66d0&lang=elixir&title=NIFs to load in Elixir

@[2-6](Load nif.so on module load)
@[8-13](Raise if NIF isn't loaded when called)

+++?gist=https://gist.github.com/mathewdgardner/683269f2e6c31f88bc50430fc3834ecf&lang=elixir&title=Mix task to make

@[6-7](Call make command)

+++?gist=https://gist.github.com/mathewdgardner/97810f03a97a12e81254136b0e0d1039&lang=elixir&title=Add make to compilers list

@[11](Add make to compilers list)

---

## Elixir

+++

## Converted / Generated Elixir

@ul

- Fun experiment
- Fast, since compiled
- Probably high memory overhead, since functional and immutable
- Is eligible for hot code-reloading

@ulend

+++?gist=https://gist.github.com/mathewdgardner/d4941547c3cf8a7e97a35358677aa4df&lang=elixir&title=Generated Elixir code

@[3-4](Learned values)
@[8-10](The outer for-loop)
@[12-14](The inner for-loop)
@[16-19](The class condition)

---

## Room for Improvements

+++

## We want to...

@ul

- Utilize CPU cores
- Keep memory usage low
- Keep request latency low

@ulend

---

## We can...

@ul

- Spawn workers so we can do predictions simultaneously
- Keep a resource pool of ports
- Buffer prediction requests and send them as a batch

@ulend

Note:

- Utilizes concurrency and may spread over CPU cores
- Hold more Python processes open capable of being called
- Maybe one for each core, but this is tuneable
- The model's predict function can make many predictions in one call and will split the work over multiple cores
- A GenServer with a timeout mechanism to flush its buffer (or GenStage)

+++

## Work In Progress - YMMV

@ul

- We may need to tune in production
- What strategy you choose may depends on the model itself and your scale
- So far, ErlPort is looking nice!

@ulend

---

## References

+++

## Simple http server in python

- https://gist.github.com/bsingr/a5ef6834524e82270154a9a72950c842

+++

## gRPC in Python & Elixir

- https://grpc.io/docs/quickstart/python.html#before-you-begin
- https://medium.com/@yuu.ishikawa/machine-learning-as-a-microservice-in-python-16ba4b9ea4ee
- https://github.com/tony612/grpc-elixir
- https://github.com/tony612/protobuf-elixir

+++

## Python port in Elixir

- http://erlport.org/docs/python.html
- http://blog.techdominator.com/article/using-python-from-elixir.html
- https://github.com/mendrugory/piton

+++

## Python multiprocessing

- https://medium.com/@bfortuner/python-multithreading-vs-multiprocessing-73072ce5600b

+++

## NIFs

- https://spin.atomicobject.com/2015/03/16/elixir-native-interoperability-ports-vs-nifs/
- http://andrealeopardi.com/posts/using-c-from-elixir-with-nifs/
- https://www.theerlangelist.com/article/outside_elixir
- https://github.com/elixir-lang/elixir/wiki/Interoperability-with-C
