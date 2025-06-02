## Building workflows with ROS nodes

The examples in this directory showcase how to incorporate ROS primitives into a workflow.

We use rclrs for this demo, and currently the build pipeline to use ROS messages for rclrs takes some extra steps since Rust is not (yet) a core language for ROS.

To run this demo, follow these steps:

1. Install ROS Jazzy according to the [normal installation instructions](https://docs.ros.org/en/jazzy/Installation.html).

2. Install Rust according to the [normal installation instructions](https://www.rust-lang.org/tools/install).

3. Run these commands to set up the workspace with the message bindings:

```bash
sudo apt install -y git libclang-dev python3-pip python3-vcstool # libclang-dev is required by bindgen
```

```bash
pip install git+https://github.com/colcon/colcon-cargo.git --break-system-packages
```

```bash
pip install git+https://github.com/colcon/colcon-ros-cargo.git --break-system-packages
```

Create a workspace with the necessary repos:

```bash
mkdir -p workspace/src && cd workspace
```

```bash
git clone https://github.com/open-rmf/ros2-impulse-examples src/ros2-impulse-examples
```

```bash
vcs import src < src/ros2-impulse-examples/example-messages.repos
```

Source the ROS distro and build the workspace:

```bash
source /opt/ros/jazzy/setup.bash
```

```bash
colcon build
```

4. After `colcon build` has finished, you should see a `.cargo/config.toml` file inside your workspace, with `[patch.crates-io.___]` sections pointing to the generated message bindings. Now you should source the workspace using

```bash
source install/setup.bash
```

Then you can go into the `ros2-impulse-examples` directory and run the example with

```bash
cd src/ros2-impulse-examples
```

```bash
cargo run --bin nav-example
```

The `nav-example` workflow will wait until it receives some goals to process. You can send it some randomized goals by **opening a new terminal** in the same directory and running

```bash
source ../../install/setup.bash
```

```bash
cargo run --bin goal-requester
```

Now the `nav-example` workflow will wait until a planner service is available for it to send `GetPlan` service requests to. Using the last terminal, run

```bash
cargo run --bin fake-plan-generator
```

Now the workflow will print out all the plans that it has received from the `fake-plan-generator` and stored in its buffer. If the workflow were connected to a real navigation system, it could pull these plans from the buffer one at a time and execute them.

Our workflow also allows the paths to be cleared out of the buffer, i.e. "cancelled". To cancel the current set of goals, run:

```bash
cargo run --bin goal-requester -- --cancel
```

After that you should see a printout of

```
Paths currently waiting to run:
[]
```

which indicates that the buffer has been successfully cleared out.
