::

  REP: Ixxxx
  Title: Message Structures of the ROS-Industrial Simple Message Protocol
  Author: G.A. vd. Hoorn <g.a.vanderhoorn@tudelft.nl>
  Status: Draft
  Type: Process
  Content-Type: text/x-rst
  Created: 15-Apr-2016
  Post-History: 


Abstract
========

This REP documents the Simple Message message structures that are part
of the *standard set* as defined in REP-I0004, and supported by the
generic clients in the ``industrial_robot_client`` package. Both
syntax and semantics of message structures and fields are described.

Driver authors may treat this document as the normative reference for
message type structures in the standard set within the ROS-Industrial
Simple Message protocol.


Outline
=======

#. Abstract_
#. Motivation_
#. Definitions_
#. Assumptions_
#. Overview_

   #. `Model of Operation`_
   #. Endianness_
   #. `Bytestream Layout`_
   #. `Shared Types`_

#. `Packet Layout`_

   #. Prefix_
   #. Header_
   #. Body_

#. `Message Definitions`_

   #. PING_
   #. GET_VERSION_
   #. JOINT_POSITION_
   #. JOINT_TRAJ_PT_
   #. JOINT_TRAJ_
   #. STATUS_
   #. JOINT_TRAJ_PT_FULL_
   #. JOINT_FEEDBACK_

#. `Defined Constants`_

   #. `Reply Codes`_
   #. `Communication Types`_
   #. `Special Sequence Numbers`_
   #. Tri-states_
   #. `Valid fields`_

#. `Application Procedure`_
#. References_
#. `Appendix A - Bytestream Examples`_

   #. `Example: JOINT_POSITION`_
   #. `Example: JOINT_TRAJ_PT`_
   #. `Example: STATUS`_

#. `Revision History`_
#. Copyright_


Motivation
==========

Like other network protocols, the ROS-Industrial Simple Message
protocol [#simple_message]_ defines a set of message structures to
allow senders and receivers to exchange information in a structured
and consistent way. In order to be able to provide generic
implementations of the Simple Message protocol (de)serialisation
libraries, to avoid potential incompatibility between clients and
servers and to assist developers in implementing new drivers, a
central registry of defined message identifiers, their structures and
their semantics is essential.

Identifiers are documented in *REP-I0004 - Assigned Message
Identifiers for the Simple Message Protocol* [#REP-I0004]_. Message
structures and their semantics are described in this document.

This document provides the normative reference for all messages that
are part of the *standard set*, and are thus supported by the generic
clients in the ``industrial_robot_client`` package. Vendor specific
and messages in any of the *freely assignable* ranges are not
described.


Definitions
===========

Controller
    TODO: include this?
Server
    A software component exposing a Simple Message compatible message
    interface that offers services and datastreams for clients to
    connect to.
Client
    A software component implementing a Simple Message compatible
    message interface that wishes to make use of the services and
    datastreams offered by clients.
Trajectory Point
    TODO: improve description. A single point in a trajectory that
    encodes a position in space, with associated velocity and
    acceleration constraints.
Motion group
    TODO


Assumptions
===========

#. The key words *must*, *must not*, *required*, *shall*, *shall not*,
   *should*, *should not*, *recommended*,  *may*, and *optional* in this
   document are to be interpreted as described in RFC-2119 [#RFC2119]_.
#. Where applicable, fields with units will adhere to ROS REP-103 [#REP103]_.
#. Message type identifiers, when given, will always use decimal (base-ten)
   notation.


Overview
========

(TODO: wiki text) The Simple Message (SimpleMessage) protocol defines the
message structure between the ROS driver layer and the robot controller itself
as used by the generic nodes in the ``industrial_robot_client`` package of
ROS-Industrial.

Requirements and constraints that influenced its design were:

#. Format should be simple enough that code can be shared between ROS and the
   controller (for those controllers that support C/C++). For those
   controllers that do not support C/C++, the protocol must be simple enough
   to be decoded with the limited capabilities of the typical robot
   programming language. A corollary to this requirement is that the protocol
   should not be so onerous as to overwhelm the limited resources of the
   robot controller.
#. Format should allow for data streaming (ROS *topic like*).
#. Format should allow for data reply (ROS *service like*).
#. The protocol is not intended to encapsulate version information. It is up
   to individual developers to ensure that code developed for communicating
   platforms does not have any version conflicts (this includes message type
   identifiers).

TODO: extend.


Model of Operation
------------------

TODO: client-server based. Controller-specific programs running on the
controller, generic ROS nodes are provided by ``industrial_robot_client``
package. Nodes (try to) open TCP (by default) connections to the server
programs on the controller. All *state relay*-type server programs broadcast
state periodically in *topic like* messages, clients command motion by
enqueuing trajectory points at the server side using *service like* messages
sent to *trajectory relay* programs, requesting execution of the trajectory
according to the communicated constraints (velocity, time_from_start etc).
Client is *not* in direct control of motion, server makes use of robot
controller facilities (interpolation, etc).


Endianness
----------

TODO: explain that 'default simple message' supports ``<le, 32, 32>`` (default),
``<be, 32, 32>`` (bswap) and ``<le, 32, 64>`` (float64).


Bytestream Layout
-----------------

TODO: explain makeup of bytestream: length, header, payload. No magic or sync
bytes (currently). No section markers, just byte counting.


Shared Types
------------

All message structures are aggregates of fields with a type from the set of
*shared types*. 

The following set has been defined (type sizes are in bytes)::

  Name         Base type        Size

  shared_int   int32               4
  shared_real  float32/float64   4/8

TODO: explain that this is to accomodate systems that have different sizes of
these types

TODO: explain that ``shared_real`` can be either a ``float`` or a ``double``


Packet Layout
=============

The following sections describe the different sub structures that make up
a valid Simple Message packet.


Prefix
------

All packets must start with the *prefix*, which must contain only a single
field: ``length``. Message structure length is defined as the sum in bytes
of the sizes of the individual fields in the *header* and the *body*,
excluding the ``length`` field itself (ie: only actual message bytes are
considered).

Layout::

  length           : shared_int

Notes

#. Client and server implementations shall prefix all outgoing messages with
   the value of ``length``.
#. Refer to section `Shared Types`_ for information on the size of supported
   field types.
#. The size of fields that are arrays or lists shall be defined as the size
   of their base type (ie: ``shared_int``) multiplied by the number of
   elements in the list, or the declared size of the array.


Header
------

The *packet header* etc.

Layout::

  msg_type         : shared_int
  comm_type        : shared_int
  reply_code       : shared_int

Notes

#. Refer to [REP-I0004]_ for valid values for the ``msg_type`` field.
#. Refer to `Communication Types`_ for valid values for the ``comm_type``
   field.
#. Refer to `Reply Codes`_ for valid values for the ``reply_code``
   field.
#. For ``TOPIC`` and ``SERVICE_REQUEST`` type messages, the ``reply_code``
   field must be set to ``INVALID``.
#. The ``SUCCESS`` and ``FAILURE`` reply codes shall only be used with
   ``SERVICE_REPLY`` type messages. They are not valid for any other
   message type.
#. The ``TOPIC`` communication type shall only be used when the sender does
   not need the recipient to acknowledge the message.
#. Receivers shall ignore (ie: take no action upon receipt) incoming ``TOPIC``
   messages they do not support.
#. Incoming ``SERVICE_REQUEST`` messages requesting use of a service that the
   receiver does not support shall result in a ``SERVICE_REPLY`` being sent
   by the receiver with the ``reply_code`` set to ``FAILURE``. No further
   action shall be taken. TODO: should a 'generic reply' message be defined?
#. Implementations shall ignore incoming ``SERVICE_REPLY`` messages for
   which no outstanding ``SERVICE_REQUEST`` exists.
#. Implementations shall warn the user of any incoming messages with the
   ``comm_type`` field set to either invalid or unsupported values. The
   message itself is then to be ignored.


Body
----

The *body* is that part of the packet which consists of all fields that are
not part of either the prefix or the message header. Most message structures
described in the `Message Definitions`_ section have a body part, but this is
not required. Messages may consist of only a prefix and a header, for
example in the case of pure acknowledgements that carry no data.

In cases where fixed-size messages are required, an array of ``shared_int``
dummy values may be used. All elements must be initialised to zero (``0``).


Message Definitions
===================

The following sections describe the message structures that make up
the standard set of the Simple Message protocol.

Values given as *assigned message identifiers* are further described in
[#REP-I0004]_.


PING
----

This message may be used by clients to test communication with the server.

Server implementations should respond to incoming ``PING`` messages with
minimal delay.

Message type: *synchronous service*

Assigned message identifier: 1

Status: active, in use

Supported by generic nodes: yes

Request::

  Prefix
  Header
  data             : shared_int[10]

Reply::

  Prefix
  Header
  data             : shared_int[10]

Notes

#. The contents of ``data`` is to be ignored by both client and server.
#. All elements in ``data`` must be initialised to zero (``0``).


GET_VERSION
-----------

Allows clients to determine the specific version of a server implementation
running on the remote system.

Message type: *synchronous service*

Assigned message identifier: 2

Status: active, in use

Supported by generic nodes: no

Request::

  Prefix
  Header

Reply::

  Prefix
  Header
  major            : shared_int
  minor            : shared_int
  patch            : shared_int

Notes

#. Fields not used by the server shall be set to zero (``0``).
#. Server implementations may return alphanumeric version info in any of the
   ``major``, ``minor`` or ``patch`` fields, but this may result in rendering
   artefacts on the client side. The generic clients in
   ``industrial_robot_client`` will always interpret these fields as signed
   integers.


JOINT_POSITION
--------------

Description.

Only used for relaying server state, NOT for enqueueing trajectory points.

One of the two message used for broadcasting joint states.

See `Example: JOINT_POSITION`_ for bytestream example.

Message type: *asynchronous publication*

Assigned message identifier: 10

Status: active, in use

Supported by generic nodes: yes

Message::

  Prefix
  Header
  sequence         : shared_int
  joint_data       : shared_real[10]

Notes

#. Use of this message structure for enqueuing trajectory points is deprecated
   and **not** supported by the generic nodes in the ``industrial_robot_client``
   package. Drivers should use the `JOINT_TRAJ_PT`_ or `JOINT_TRAJ_PT_FULL`_
   messages instead.
#. The ``sequence`` field uses zero-based numbering.
#. The ``sequence`` field is not used when reporting joint state and shall be
   set to zero (``0``) by server implementations.
#. Elements of ``joint_data`` that are not used must be initialised to zero
   (``0``) by the sender.
#. The size of the ``joint_data`` array is ``10``, even if the server
   implementation does not need that many elements (for instance because it
   only has six joints).
#. Controllers that support or are configured with more than a single motion
   group should use the `JOINT_FEEDBACK`_ message if they wish to report joint
   state for all configured motion groups.
#. The elements of the ``joint_data`` field shall represent the joint space
   positions of the corresponding joint axes of the controller. Units are
   *radians* for rotational or revolute axes, and *meters* for translational
   or prismatic axes (see also [#REP103]_).
#. TODO: what should authors / drivers do when there are more than 10 joints
   in a single motion group?


JOINT_TRAJ_PT
-------------

Clients may use this message to enqueue trajectory points for execution on
the server.

See `Example: JOINT_TRAJ_PT`_ for bytestream example.

Message type: *synchronous service*

Assigned message identifier: 11

Status: active, in use

Supported by generic nodes: yes

Request::

  Prefix
  Header
  sequence         : shared_int
  joint_data       : shared_real[10]
  velocity         : shared_real
  duration         : shared_real

Reply::

  Prefix
  Header
  dummy_data       : shared_real[10]

Notes

#. Drivers shall set the value of the ``reply_code`` field in the ``Header``
   of the reply messages to *the result of the enqueueing operation* of the
   trajectory point that was transmitted in the request. It is *not* to be
   used to report the success or failure of the *execution* of the motion.
   Drivers may use the appropriate fields in `STATUS`_ for that.
#. TODO: the IRC is not setup to support this currently. Also: does this only
   hold for drivers that use a trajectory buffering approach? What about
   direct streaming?
#. Refer to `Special Sequence Numbers`_ for valid values for the ``sequence``
   field.
#. Driver authors must abort any motion executing on the controller on receipt
   of a message with ``sequence`` set to ``STOP_TRAJECTORY``. Note that such
   messages must also be acknowledged with a reply message.
#. Servers must abort any motion executing on the controller on receipt of an
   out-of-order trajectory point (ie: ``(seq(msg_n) - seq(msg_n-1)) != 1``).
#. Elements of ``joint_data`` that are not used must be initialised to zero
   (``0``) by the sender.
#. The size of the ``joint_data`` array is ``10``, even if the server
   implementation does not need that many elements (for instance because it
   only has six joints).
#. Controllers that support or are configured with more than a single motion
   group should use the `JOINT_TRAJ_PT_FULL`_ message if they wish to relay
   trajectories for all configured motion groups.
#. The elements of the ``joint_data`` field shall represent the joint space
   positions of the corresponding joint axes of the controller. Units are
   *radians* for rotational or revolute axes, and *meters* for translational
   or prismatic axes (see also [#REP103]_).
#. The ``duration`` field represents total segment duration for all joints in
   seconds [#REP103]_. The generic nodes calculate this duration based on the
   time needed by the slowest joint to complete the segment.
   As an alternative to the ``duration`` field, the value of the ``velocity``
   field is a value representing the fraction ``(0.0, 1.0]`` of maximum joint
   velocity that should be used when executing the motion for the current
   segment. Driver authors may use whichever value is more conveniently mapped
   onto motion primitives supported by the controller.
#. TODO: problem with 'velocity': is that max velocity over segment, average
   velocity, or does it encode desired state of manipulator at a specific point
   in time?


JOINT_TRAJ
----------

Used to encode entire ROS ``JointTrajectory`` messages.

Message type: *synchronous service*

Assigned message identifier: 12

Status: deprecated

Supported by generic nodes: no

Message::

  Header
  sequence         : shared_int
  TODO

Reply::

  Header
  TODO


STATUS
------

Description.

Also: ``ROBOT_STATUS``. Not for joint states.

See `Example: STATUS`_ for bytestream example.

Message type: *asynchronous publication*

Assigned message identifier: 13

Status: active, in use

Supported by generic nodes: yes

Message::

  Prefix
  Header
  drives_powered   : shared_int
  e_stopped        : shared_int
  error_code       : shared_int
  in_error         : shared_int
  in_motion        : shared_int
  mode             : shared_int
  motion_possible  : shared_int

Valid values for ``mode`` are::

  Val  Name     Description

   -1  UNKNOWN  Controller mode cannot be determined or is not one of those
                defined in ISO 10218-1
    1  MANUAL   Controller is in ISO 10218-1 'manual' mode
    2  AUTO     Controller is in ISO 10218-1 'automatic' mode

All other values are reserved for future use.

Notes

#. The fields ``drives_powered``, ``e_stopped``, ``in_error``, ``in_motion``
   and ``motion_possible`` are tri-states. Refer to `Tri-states`_ for valid
   values for these fields.
#. Fields for which a driver cannot determine a value shall be set to
   ``UNKNOWN``.
#. The ``error_code`` field should be used to store the integer representation
   (id, number or code) of the error that caused the robot to go into an error
   mode.
#. If the controller can be set to modes other than those defined in ISO
   10218-1, drivers shall report ``UNKNOWN`` for those modes.


JOINT_TRAJ_PT_FULL
------------------

Meant to be an almost 1-to-1 copy of the ROS ``JointTrajectoryPoint`` message
type. But without the ``names`` field (we rely on indices).

TODO: extend.

Message type: *synchronous service*

Assigned message identifier: 14

Status: active, in use

Supported by generic nodes: no (motoman_driver only)

Request::

  Prefix
  Header
  robot_id         : shared_int
  sequence         : shared_int
  valid_fields     : shared_int
  time             : shared_real
  positions        : shared_real[10]
  velocities       : shared_real[10]
  accelerations    : shared_real[10]

Reply::

  Prefix
  Header
  dummy_data       : shared_real[10]

Notes

#. Drivers shall set the value of the ``reply_code`` field in the ``Header``
   of the reply messages to the result of the *enqueueing operation* of the
   trajectory point that was transmitted in the request. It is *not* to be
   used to report the success or failure of the *execution* of the motion.
   Drivers may use the appropriate fields in `STATUS`_ for that.
#. TODO: the IRC is not setup to support this currently. Also: does this only
   hold for drivers that use a trajectory buffering approach? What about
   direct streaming?
#. The value of the ``robot_id`` field shall match that of the numeric
   identifier of the corresponding motion group on the controller. This field
   uses zero-based counting.
   In cases where motion groups are not identified by numeric ids on the
   controller, drivers shall implement an appropriate mapping (ie:
   alphabetical sorting of group names, etc).
#. Refer to `Special Sequence Numbers`_ for valid values for the ``sequence``
   field.
#. Driver authors must abort any motion executing on the controller on receipt
   of a message with ``sequence`` set to ``STOP_TRAJECTORY``. Note that such
   messages must also be acknowledged with a reply message.
#. Servers must abort any motion executing on the controller on receipt of an
   out-of-order trajectory point (ie: ``(seq(msg_n) - seq(msg_n-1)) != 1``).
#. Refer to `Valid fields`_ for defined bit positions for the ``valid_fields``
   field.
#. Drivers shall set all undefined bit positions in ``valid_fields`` to zero
   (``0``).
#. Drivers shall set all elements of invalid fields (as encoded by
   ``valid_fields``) to zero (``0``).
#. Elements of ``positions``, ``velocities`` and ``accelerations`` that are
   not used must be initialised to zero (``0``) by the sender.
#. The size of the ``positions``, ``velocities`` and ``accelerations`` arrays
   is ``10``, even if the server implementation does not need that many
   elements (for instance because it only has six joints).


JOINT_FEEDBACK
--------------

Only used for broadcasting server state.

Supports multiple motion groups.

Message type: *asynchronous publication*

Assigned message identifier: 15

Status: active, in use

Supported by generic nodes: no (motoman_driver only)

Message::

  Prefix
  Header
  robot_id         : shared_int
  valid_fields     : shared_int
  time             : shared_real
  positions        : shared_real[10]
  velocities       : shared_real[10]
  accelerations    : shared_real[10]

Notes

#. Refer to `Special Sequence Numbers`_ for valid values for the ``sequence``
   field.
#. The value of the ``robot_id`` field shall match that of the numeric
   identifier of the corresponding motion group on the controller. This field
   uses zero-based counting.
   In cases where motion groups are not identified by numeric ids on the
   controller, drivers shall implement an appropriate mapping (ie:
   alphabetical sorting of group names, etc).
#. Refer to `Valid fields`_ for defined bit positions for the ``valid_fields``
   field.
#. Drivers shall set all undefined bit positions in ``valid_fields`` to zero
   (``0``).
#. Drivers shall set all elements of invalid fields (as encoded by
   ``valid_fields``) to zero (``0``).
#. Elements of ``positions``, ``velocities`` and ``accelerations`` that are
   not used must be initialised to zero (``0``) by the sender.
#. The size of the ``positions``, ``velocities`` and ``accelerations`` arrays
   is ``10``, even if the server implementation does not need that many
   elements (for instance because it only has six joints).


Defined Constants
=================

This section documents all shared constants as defined in the Simple Message
protocol. Constants defined in this section are recognised by the generic
nodes in the ``industrial_robot_client`` package and shall be used by
compliant drivers.


Communication Types
-------------------

::

  Val  Name             Description

    0  INVALID          Reserved value. Do not use.
    1  TOPIC            Message needs no acknowledgement
    2  SERVICE_REQUEST  Sender requires acknowledgement
    3  SERVICE_REPLY    Message is a reply to a request

All other values are reserved for future use.


Reply Codes
-----------

::

  Val  Name     Description

    0  INVALID  Also encodes UNUSED
    1  SUCCESS  Receiver processed the message succesfully
    2  FAILURE  Receiver encountered a failure processing the message

All other values are reserved for future use.


Special Sequence Numbers
------------------------

::

  Val  Name                        Description

    N                              Index into current trajectory
   -1  START_TRAJECTORY_DOWNLOAD   Downloading drivers only: signals start
   -2  START_TRAJECOTRY_STREAMING  TODO (typo is on purpose)
   -3  END_TRAJECTORY              Downloading drivers only: signals end
   -4  STOP_TRAJECTORY             Driver must abort any currently executing motion

All other *negative* values are reserved for future use.


Tri-states
----------

::

  Val  Name     Description

   -1  UNKNOWN  -
    0  ON       Also encodes TRUE, ENABLED or HIGH
    1  OFF      Also encodes FALSE, DISABLED or LOW

All other values are reserved for future use.


Valid fields
------------

Bit positions are counted starting from LSB::

  Pos  Name          Description

    0  TIME          The 'time' field contains valid data
    1  POSITION      The 'positions' field contains valid data
    2  VELOCITY      The 'velocities' field contains valid data
    3  ACCELERATION  The 'accelerations' field contains valid data

All other positions are reserved for future use.


Application Procedure
=====================

TODO.


References
==========

.. [#RFC2119] Key words for use in RFCs to Indicate Requirement Levels, on-line, retrieved 5 October 2015
   (https://tools.ietf.org/html/rfc2119)
.. [#REP103] Standard Units of Measure and Coordinate Conventions, on-line, retrieved 5 October 2015
   (https://github.com/ros-infrastructure/rep/blob/cde09a4b18eea68ca37c4ab2d1b70d7ce7a5738c/rep-0103.rst)
.. [#simple_message] ROS-Industrial simple_message package, ROS Wiki, on-line, retrieved 5 October 2015
   (http://wiki.ros.org/simple_message)
.. [#rosi_ml] ROS-Industrial mailing list (Google Group)
   (https://groups.google.com/forum/?fromgroups#!forum/swri-ros-pkg-dev)
.. [#REP-I0004] REP-I0004 - Assigned Message Identifiers for the Simple Message Protocol, on-line, retrieved 5 October 2015
   (https://github.com/ros-industrial/rep/blob/7894644f4937c1d910b3e55ad4494788637f89ef/rep-I0004.rst)


Appendix A - Bytestream Examples
================================

This section provides three annotated examples of bytestreams driver authors can
expect to be sent and received by the generic nodes in the
``industrial_robot_client`` package.

Note that the hexadecimal numbers are displayed in big-endian byte-order.


Example: JOINT_POSITION
-----------------------

This shows a stream for a ``JOINT_POSITION`` message, sent by a server to
broadcast joint state for a six-axis robot that is close to its zero position.

Direction: server → client

::

  Hex       Field              Description

            Prefix
  00000038    length           56 bytes

            Header
  0000000A    msg_type         Joint Position
  00000001    comm_type        Topic
  00000000    reply_code       Unused / Invalid

            Body
  00000000    sequence          0 (unused)
  B81AD9FA    joint_data[0]    -0.000036919
  B6836312    joint_data[1]    -0.000003916
  B7C043F5    joint_data[2]    -0.000022920
  B8B81516    joint_data[3]    -0.000087777
  B865D055    joint_data[4]    -0.000054792
  B8B6365E    joint_data[5]    -0.000086886
  00000000    joint_data[6]     0.000000000
  00000000    joint_data[7]     0.000000000
  00000000    joint_data[8]     0.000000000
  00000000    joint_data[9]     0.000000000


Example: JOINT_TRAJ_PT
----------------------

The following is a bytestream for a serialised ``JOINT_TRAJ_PT`` sent be a
client to a server to request the second trajectory point in a trajectory be
queued for execution by the controller. This is for a six-axis robot.

Direction: client → server

::

  Hex       Field              Description

            Prefix
  00000040    length           64 bytes

            Header
  0000000B    msg_type         Joint Trajectory Point
  00000002    comm_type        Service Request
  00000000    reply_code       Unused / Invalid

            Body
  00000001    sequence          1 (second TrajectoryPoint)
  A7600000    joint_data[0]    -0.000000000
  3EA7CDE8    joint_data[1]     0.327742815
  BF5D9E57    joint_data[2]    -0.865697324
  C0490FDB    joint_data[3]    -3.141592741
  3F34815F    joint_data[4]     0.705099046
  C0490FDB    joint_data[5]    -3.141592741
  00000000    joint_data[6]     0.000000000
  00000000    joint_data[7]     0.000000000
  00000000    joint_data[8]     0.000000000
  00000000    joint_data[9]     0.000000000
  3DCCCCCD    velocity          0.1
  40A00000    duration          5.0


Example: STATUS
---------------

This is a bytestream encoding a ``STATUS`` message for a six-axis robot that is
in auto-mode, not moving, not in an error mode, of which the servo drives are
powered and is ready to execute a new trajectory. Note that the state of the
e-stop could not be determined by the driver, and is thus reported as
``UNKNOWN``.

Direction: server → client

::

  Hex       Field              Description

            Prefix
  00000028    length           40 bytes

            Header
  0000000D    msg_type         Status
  00000001    comm_type        Topic
  00000000    reply_code       Unused / Invalid

            Body
  00000001    drives_powered   True
  FFFFFFFF    e_stopped        Unknown
  00000000    error_code       0
  00000000    in_error         False
  00000000    in_motion        False
  00000002    mode             Auto
  00000001    motion_possible  True


Revision History
================

::

  2016-Apr-15   Initial revision


Copyright
=========

This document has been placed in the public domain.
