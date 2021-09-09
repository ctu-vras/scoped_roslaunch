# scoped_roslaunch

This package overrides the system-installed `/opt/ros/melodic/bin/roslaunch` to add a not-yet merged
feature to it.
The feature is https://github.com/ros/ros_comm/pull/2059 - fixing the `default="true"` argument of
`<machine>` tag to be really limited only to the scope in which the machine is defined. Wiki docs
say it behaves like this, but https://github.com/ros/ros_comm/issues/1884 proves that it does not.

To use the new feature, define machines with `default="scope"`.

This package also adds `default="parent"` option, which means that the machine is set as default in the
parent scope. This is useful e.g. when you want to have one file with definitions of all machines
and include this file in places where you want to use the machines.

## Example usage

### `default="scope"` usage

```XML
<launch>
  <node name="a" pkg="rostopic" type="rostopic" args="pub -r 1 /a std_msgs/Int32 'data: 1'" />
  <group>
      <machine name="b" address="robot" user="robot" password="robot" env-loader="/home/robot/env.sh" default="scope" />
      <node name="b" pkg="rostopic" type="rostopic" args="pub -r 1 /b std_msgs/Int32 'data: 1'" machine="b" />
      <node name="b2" pkg="rostopic" type="rostopic" args="pub -r 1 /b std_msgs/Int32 'data: 1'" />
  </group>
  <node name="c" pkg="rostopic" type="rostopic" args="pub -r 1 /a std_msgs/Int32 'data: 1'" />
</launch>
```

In this example, nodes `a` and `c` are launched on the local machine and nodes `b` and `b2` are launched on machine `b`.

If used with `default="true"`, `c` would also be launched on the remote machine (contrary to what wiki docs say).

### `default="parent"` usage

`top.launch`:

```XML
<launch>
  <node name="a" pkg="rostopic" type="rostopic" args="pub -r 1 /a std_msgs/Int32 'data: 1'" />
  <group>
      <include file="$(dirname)/machine.launch" />
      <node name="b" pkg="rostopic" type="rostopic" args="pub -r 1 /b std_msgs/Int32 'data: 1'" machine="b" />
      <node name="b2" pkg="rostopic" type="rostopic" args="pub -r 1 /b std_msgs/Int32 'data: 1'" />
  </group>
  <node name="c" pkg="rostopic" type="rostopic" args="pub -r 1 /a std_msgs/Int32 'data: 1'" />
</launch>
```

`machine.launch`:

```XML
<launch>
  <machine name="b" address="robot" user="robot" password="robot" env-loader="/home/robot/env.sh" default="parent" />
</launch>
```

This setting also launches `a` and `c` on the local machine and `b` and `b2` on machine `b`.

## Remote process stdout transmission and line-buffering

Although a bit unrelated, we also needed to see the output of nodes launched via machine tags.
Standard roslaunch throws stdout away: https://github.com/ros/ros_comm/issues/1106 .

Using this package, roslaunch is patched to display also stdout of the remotely launched nodes.
Moreover, it makes some changes to make buffering of the output more streamlined and nicer.