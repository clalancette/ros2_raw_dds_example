# ROS 2 Raw DDS example

This repository contains an example of having a "raw" DDS node communicate with a ROS 2 network.
In this case "raw" means that it is a program that does not use any of the ROS 2 libraries, but instead uses a DDS implementation directly.
This repository has examples using [Fast-DDS](https://github.com/eProsima/Fast-DDS/) and [CycloneDDS](https://github.com/eclipse-cyclonedds/cyclonedds).

## Building from source

### Install ROS 2 Rolling

Follow the instructions at https://docs.ros.org/en/rolling/Installation.html.

### Build the prerequisites

1. `$ git clone https://github.com/eProsima/Fast-CDR -b v1.0.19`
1. `$ mkdir -p Fast-CDR/build`
1. `$ pushd Fast-CDR/build`
1. `$ cmake -DCMAKE_INSTALL_PREFIX=../../install ..`
1. `$ make -j10 install`
1. `$ popd`
1. `$ git clone https://github.com/eProsima/foonathan_memory_vendor`
1. `$ mkdir -p foonathan_memory_vendor/build`
1. `$ pushd foonathan_memory_vendor/build`
1. `$ cmake -DCMAKE_INSTALL_PREFIX=../../install ..`
1. `$ make -j10 install`
1. `$ popd`
1. `$ git clone https://github.com/eProsima/Fast-DDS -b 2.1.x`
1. `$ mkdir -p Fast-DDS/build`
1. `$ pushd Fast-DDS/build`
1. `$ cmake -DCOMPILE_TOOLS=OFF -DCMAKE_INSTALL_PREFIX=../../install ..`
1. `$ make -j10 install`
1. `$ popd`
1. `$ git clone https://github.com/eclipse-cyclonedds/cyclonedds`
1. `$ mkdir -p cyclonedds/build`
1. `$ pushd cyclonedds/build`
1. `$ cmake -DCMAKE_INSTALL_PREFIX=../../install ..`
1. `$ make -j10 install`
1. `$ popd`

### Build the example code

1. `$ mkdir -p build`
1. `$ pushd build`
1. `$ cmake .. -DCMAKE_INSTALL_PREFIX=../install ..`
1. `$ make -j10`
1. `$ popd`

## Running

This repository builds 4 binaries: `ros2_fastdds_send`, `ros2_fastdds_recv`, `ros2_cyclonedds_send`, and `ros2_cyclonedds_recv`.

`ros2_fastdds_send` uses Fast-DDS to publish a ROS 2 [UUID message](https://github.com/ros2/unique_identifier_msgs/blob/80c21658a1b17f0c4e24002552f2426db236985f/msg/UUID.msg) on the `/uuid_topic` once a second, incrementing the first number of the UUID every publication.

`ros2_cyclonedds_send` uses CycloneDDS to do the same.

`ros2_fastdds_recv` uses Fast-DDS to listen to the `/uuid_topic`, and print every time it gets a new UUID message.

`ros2_cyclonedds_recv` uses CycloneDDS to do the same.

### Demo 1 - Publish raw DDS using Fast-DDS into ROS 2 network

This demo requires 3 terminals.

#### Terminal 1

```
$ ./ros2_fastdds_send
```

#### Terminal 2

```
$ ./ros2_fastdds_recv
```

#### Terminal 3

```
$ . /opt/ros/rolling/setup.bash
$ RMW_IMPLEMENTATION=rmw_fastrtps_cpp ros2 topic list -t
$ RMW_IMPLEMENTATION=rmw_fastrtps_cpp ros2 topic echo /uuid_topic unique_identifier_msgs/msg/UUID
```

You should see the data being published from Terminal 1 into both Terminal 2 and Terminal 3.

### Demo 2 - Publish from ROS 2 network into raw DDS using Fast-DDS

This demo requires 2 terminals.

#### Terminal 1

```
$ . /opt/ros/rolling/setup.bash
$ RMW_IMPLEMENTATION=rmw_fastrtps_cpp ros2 topic pub /uuid_topic unique_identifier_msgs/msg/UUID "uuid:
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0"
```

#### Terminal 2

```
$ ./ros2_fastdds_recv
```

You should see the data being published from Terminal 1 into Terminal 2.

### Demo 3 - Publish raw DDS using CycloneDDS into ROS 2 network

This demo requires 3 terminals.

#### Terminal 1

```
$ ./ros2_cyclonedds_send
```

#### Terminal 2

```
$ ./ros2_cyclonedds_recv
```

#### Terminal 3

```
$ . /opt/ros/rolling/setup.bash
$ RMW_IMPLEMENTATION=rmw_cyclonedds_cpp ros2 topic list -t
$ RMW_IMPLEMENTATION=rmw_cyclonedds_cpp ros2 topic echo /uuid_topic unique_identifier_msgs/msg/UUID
```

You should see the data being published from Terminal 1 into both Terminal 2 and Terminal 3.

### Demo 4 - Publish from ROS 2 network into raw DDS using CycloneDDS

This demo requires 2 terminals.

#### Terminal 1

```
$ . /opt/ros/rolling/setup.bash
$ RMW_IMPLEMENTATION=rmw_cyclonedds_cpp ros2 topic pub /uuid_topic unique_identifier_msgs/msg/UUID "uuid:
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0
- 0"
```

#### Terminal 2

```
$ ./ros2_cyclonedds_recv
```

You should see the data being published from Terminal 1 into Terminal 2.

## How was this example generated?

This particular example uses the `unique_identifer_msgs/msg/UUID`, but the idea can be extended to any ROS 2 message.
To understand how, we need to briefly discuss how ROS 2 generates code from `.msg` files.

### How ROS 2 generates code from `.msg` files

At a very high-level, ROS 2 takes the data from `.msg` files and produces `.idl` files from those messages.
It then takes the `.idl` files and generates code specific to that particular message for that particular DDS implementation.
When ROS 2 is installed, it installs both the `.msg` and the generated `.idl` files into the installation directory.

### Generating code from `.idl` files

With the knowledge from the previous section we can look at generating Fast-DDS code for any ROS 2 message.
Because we are trying to not use any ROS 2 in this example directly, we need one more tool that will generate the code for us.
That tool is called [Fast-DDS-Gen](https://github.com/eProsima/Fast-DDS-Gen), and will convert our `.idl` files into code that we can use.

#### Building Fast-DDS-Gen

1. `$ git clone https://github.com/eProsima/Fast-DDS-Gen`
1. `$ pushd Fast-DDS-Gen`
1. `$ ./gradlew assemble`
1. `$ popd`

#### Generating code from `.idl` files

Now that we have Fast-DDS-Gen installed, we can use it to recreate the UUID code we have checked in to this repository.

We can run the following:

```
$ mkdir output ; ./Fast-DDS-Gen/scripts/fastddsgen -example CMake -d output -typeros2 /opt/ros/rolling/share/unique_identifier_msgs/msg/UUID.idl
```

Fast-DDS-Gen generated a bunch of files in the `output` subdirectory.
We'll go through each one, describe what it is used for, and describe the changes needed to make it ROS 2 compliant.

* `CMakeLists.txt` - Not used, ignore this file.
* `UUID.cxx` - The class implementation of the message in the IDL file.  No changes needed to make it ROS 2 compliant.
* `UUID.h` - The class definition of the message in the IDL file.  No changes needed to make it ROS 2 compliant.
* `UUIDPublisher.cxx` - The class implementation of a publisher for the message in the IDL file.  To make it ROS 2 compliant, the topic name for the `create_topic` call must be prefixed with the string `rt/` (for ROS Topic).
* `UUIDPublisher.h` - The class definition of a publisher for the message in the IDL file.  No changes needed to make it ROS 2 compliant.
* `UUIDPubSubMain.cxx` - Not used, ignore this file.
* `UUIDPubSubTypes.cxx` - The class implementation of the serialization/deserialization code for the message in the IDL file.  To make it ROS 2 compliant, the type name as set by `setName` must be set to `<namespace>::msg::dds_::<Type>_`.  For instance, for the UUID IDL file, the typename must be set to `unique_identifier_msgs::msg::dds_::UUID_`.
* `UUIDPubSubTypes.h` - The class definition of the serialization/deserialization code for the message in the IDL file.  No changes needed to make it ROS 2 compliant.
* `UUIDSubscriber.cxx` - The class implementation of a subscriber for the message in the IDL file.  To make it ROS 2 compliant, the topic name for the `create_topic` call must be prefixed with the string `rt/` (for ROS Topic).
* `UUIDSubscriber.h` - The class definition of a subscriber for the mssage in the IDL file.  No changes need to make it ROS 2 compliant.

## Making Fast-DDS interoperate with ROS 2

With all of the above explanation in mind, the following are what is needed to allow a raw Fast-DDS participant to communicate with a ROS 2 network:

1. The QoS of the publisher and subscriber have to match.  This is true for *all* participants in the network that wish to communicate.
1. The topic that the raw DDS participant uses must begin with `rt/`.
1. The type that the raw DDS participant uses must be of the form `<namespace>::msg::dds_::<Type>_`.
