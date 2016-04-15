::

  REP: Ixxxx
  Title: Message Structures of the ROS-Industrial Simple Message Protocol
  Author: G.A. vd. Hoorn <g.a.vanderhoorn@tudelft.nl>
  Status: Draft
  Type: Process
  Content-Type: text/x-rst
  Created: 5-Jan-2016
  Post-History: 


Outline
=======

#. Abstract_
#. Motivation_
#. Definitions_
#. Assumptions_
#. `Shared Types`_
#. Endianness_
#. `Bytestream Layout`_
#. `Packet Layout`_

   #. Preamble_
   #. Header_
   #. Body_

#. `Message Definitions`_

   #. `Reply Codes`_
   #. `Communication Types`_
   #. PING_
   #. GET_VERSION_
   #. JOINT_POSITION_
   #. JOINT_TRAJ_PT_
   #. JOINT_TRAJ_
   #. STATUS_
   #. JOINT_TRAJ_PT_FULL_
   #. JOINT_FEEDBACK_

#. `Application Procedure`_
#. References_
#. `Revision History`_
#. Copyright_


Abstract
========

This REP documents the Simple Message message structures that are part
of the *standard set* as defined in REP-I0004, and supported by the
generic clients in the ``industrial_robot_client`` package. Both
syntax and semantics of message structures and fields are described.

Driver authors may treat this document as the normative reference for
message type structures in the standard set within the ROS-Industrial
Simple Message protocol.


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

Identifiers are documented in the [#REP-I0004]_ (TODO: copy title).
Message structures and their semantics are described in this document.

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


Assumptions
===========

#. The key words "must", "must not", "required", "shall", "shall not",
   "should", "should not", "recommended",  "may", and "optional" in this
   document are to be interpreted as described in RFC-2119 [#RFC2119]_.
#. Where applicable, fields with units will adhere to ROS REP-103 [#REP103]_.
#. Message type identifiers, when given, will always use decimal (base-ten)
   notation.
#. TODO: others


Shared Types
============

All message structures are aggregates of fields with a type from the set of
*shared types*. 

The following set has been defined (type sizes are in bytes)::

  Name         Base type        Size

  shared_int   int32               4
  shared_real  float32/float64   4/8

TODO: explain that this is to accomodate systems that have different sizes of
these types
TODO: explain that ``shared_real`` can be either a ``float`` or a ``double``


Endianness
==========

TODO: explain that 'default simple message' supports ``<le, 32, 32>`` (default),
``<be, 32, 32>`` (bswap) and ``<le, 32, 64>`` (float64).


Bytestream Layout
=================

TODO: explain makeup of bytestream: length, header, payload. No magic or sync
bytes (currently). No section markers, just byte counting.


Packet Layout
=============

The following sections describe the different sub structures that make up
a valid Simple Message packet.


Preamble
--------

All packets must start with the *preamble*, which must contain only a single
field: ``length``. Message structure length is defined as the sum (in bytes)
of the sizes of the individual fields in the *header* and the *body*,
excluding the ``length`` field itself (ie: only actual message bytes are
considered).

Layout::

  length           : shared_int

Notes

#. Client and server implementations are required to add the preamble to all
   outgoing messages.
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

Communication Types
^^^^^^^^^^^^^^^^^^^

Valid values for the ``comm_type`` field are::

  Val  Name             Description

    0  INVALID          Reserved value. Do not use.
    1  TOPIC            Message needs no acknowledgement
    2  SERVICE_REQUEST  Sender requires acknowledgement
    3  SERVICE_REPLY    Message is a reply to a request

All other values are reserved for future use.

Reply Codes
^^^^^^^^^^^

Valid values for the ``reply_code`` field are::

  Val  Name     Description

    0  INVALID  See notes below
    1  SUCCESS  Receiver processed the message succesfully
    2  FAILURE  Receiver encountered a failure processing the message

All other values are reserved for future use.

Notes

#. Refer to [#REP-I0004]_ for valid values for the ``msg_type`` field.
#. For ``TOPIC`` and ``SERVICE_REQUEST`` type messages, the ``reply_code``
   field should be set to ``INVALID``.
#. The ``SUCCESS`` and ``FAILURE`` reply codes may only be used with
   ``SERVICE_REPLY`` type messages. They are not valid for any other
   message type.
#. The ``TOPIC`` communication type should only be used when the sender does
   not need the recipient to acknowledge the message.
#. Implementations should ignore incoming ``SERVICE_REPLY`` messages for
   which no outstanding ``SERVICE_REQUEST`` exists.
#. Implementations shall warn the user of any incoming messages with the
   ``comm_type`` field set to either invalid or unsupported values. The
   message itself is then to be ignored.


Body
----

The *body* is that part of the packet which consists of all fields that are
not part of either the preamble or the message header. Most message structures
described in the `Message Definitions`_ section have a body part, but this is
not required. Messages may consist of only a preamble and a header, for
example in the case of pure acknowledgements that carry no data.

In cases where fixed-size messages are required, an array of ``shared_int``
dummy values may be used. All elements must be initialised to zero (``0``).


Message Definitions
===================

The following sections describe the message structures that make up
the standard set of the Simple Message protocol.


PING
----

This message may be used by clients to test communication with the server.

Server implementations should respond to incoming ``PING`` messages with
minimal delay.

Message type: *synchronous service*

Assigned identifier (see [#REP-I0004]_): 1

Request::

  Preamble
  Header
  data             : shared_int[10]

Reply::

  Preamble
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

Assigned identifier (see [#REP-I0004]_): 2

Request::

  Preamble
  Header

Reply::

  Preamble
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

One of the two message used for broadcasting joint states

Message type: *asynchronous publication*

Assigned identifier (see [#REP-I0004]_): 10

Msg::

  Preamble
  Header
  sequence         : shared_int
  joint_data       : shared_real[10]

Valid values for the ``sequence`` field are::

  Val  Name                        Description

    N                              Index into current trajectory
   -1  START_TRAJECTORY_DOWNLOAD   Downloading drivers only: signals start
   -2  START_TRAJECOTRY_STREAMING  TODO (typo is on purpose)
   -3  END_TRAJECTORY              Downloading drivers only: signals end
   -4  STOP_TRAJECTORY             TODO

Notes

#. Elements of ``joint_data`` that are not used must be initialised to zero
   (``0``) by the sender.
#. The size of the ``joint_data`` array is ``10``, even if the server
   implementation does not need that many elements (fi because it only has 6
   joints).
#. Driver authors must abort any motion executing on the controller on receipt
   of a message with ``sequence`` set to ``STOP_TRAJECTORY``.
#. Server behaviour is undefined for trajectory points that arrive
   out-of-order (ie: ``seq(msg_n) < seq(msg_n-1)``).
#. TODO


JOINT_TRAJ_PT
-------------

Clients may use this message to enqueue trajectory points for execution on
the server.

Message type: *synchronous service*

Assigned identifier (see [#REP-I0004]_): 11

Msg::

  Preamble
  Header
  sequence         : shared_int
  joint_data       : shared_real[10]
  velocity         : shared_real
  duration         : shared_real

Reply::

  Preamble
  Header
  dummy_data       : shared_real[10]

Notes

#. See `JOINT_POSITION`_ for valid values for ``sequence``.
#. TODO: ``joint_data`` is een array van joint angles in radians.
#. TODO: een server ACK is alleen een ACK van de enqueuing operation, NIET
   van de reachability of execution completion. Daar is ``STATUS`` voor.
#. TODO: vaak zal ``velocity`` de inverse zijn van ``duration``. Driver
   authors may use which ever is more convenient to map onto motion controller
   primitives.
#. TODO: problem with 'velocity': is that max velocity over segment, average
   velocity, or does it encode desired state of manipulator at a specific point
   in time?
#. TODO: 'duration' only makes sense when it encodes total motion execution
   time for the segment defined by ``(p_n; p_n+1)``.


JOINT_TRAJ
----------

Used to encode entire ROS ``JointTrajectory`` messages.

Deprecated.

Message type: *synchronous service*

Assigned identifier (see [#REP-I0004]_): 12

Msg::

  Header
  sequence         : shared_int
  TODO

Reply::

  Header

Notes

#. None


STATUS
------

Description.

Also: ``ROBOT_STATUS``. Not for joint states.

Message type: *asynchronous publication*

Assigned identifier (see [#REP-I0004]_): 13

Msg::

  Preamble
  Header
  drives_powered   : shared_int
  e_stopped        : shared_int
  error_code       : shared_int
  in_error         : shared_int
  in_motion        : shared_int
  mode             : shared_int
  motion_possible  : shared_int

The fields ``drives_powered``, ``e_stopped``, ``in_error``,
``in_motion`` and ``motion_possible`` are treated as tri-states. Valid values
are::

  Val  Name     Description

   -1  UNKNOWN  -
    0  ON       Also encodes TRUE, ENABLED or HIGH
    1  OFF      Also encodes FALSE, DISABLED or LOW

All other values are reserved for future use.

Valid values for ``mode`` are::

  Val  Name     Description

   -1  UNKNOWN  -
    1  MANUAL   Controller is in ISO 10218-1 'manual' mode
    2  AUTO     Controller is in ISO 10218-1 'automatic' mode

All other values are reserved for future use.

Notes

#. The ``error_code`` field should be used to store the numerical
   representation (id, number or code) of the error that caused the robot to
   go into an error mode.


JOINT_TRAJ_PT_FULL
------------------

Meant to be an almost 1-to-1 copy of the ROS ``JointTrajectoryPoint`` message
type. But without the ``names`` field (we rely on indices).

TODO: extend.

Message type: *synchronous service*

Assigned identifier (see [#REP-I0004]_): 14

Msg::

  Preamble
  Header
  robot_id         : shared_int
  sequence         : shared_int
  valid_fields     : shared_int
  time             : shared_real
  positions        : shared_real[10]
  velocities       : shared_real[10]
  accelerations    : shared_real[10]

Reply::

  Preamble
  Header
  dummy_data       : shared_real[10]

Defined bit positions in ``valid_fields`` are::

  Pos  Name          Description

    0  TIME          The 'time' field contains valid data
    1  POSITION      The 'positions' field contains valid data
    2  VELOCITY      The 'velocities' field contains valid data
    3  ACCELERATION  The 'accelerations' field contains valid data

All other positions are reserved for future use.

TODO: bit position counting is from LSB.

Notes

#. See `JOINT_POSITION`_ for valid values for ``sequence``.


JOINT_FEEDBACK
--------------

Only used for broadcasting server state.

Supports multiple motion groups.

Message type: *asynchronous publication*

Assigned identifier (see [#REP-I0004]_): 15

Msg::

  Preamble
  Header
  robot_id         : shared_int
  valid_fields     : shared_int
  time             : shared_real
  positions        : shared_real[10]
  velocities       : shared_real[10]
  accelerations    : shared_real[10]

Notes

#. See `JOINT_TRAJ_PT_FULL`_ for defined bit positions in ``valid_fields``.


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


Revision History
================

::

  2016-Jan-5   Initial revision


Copyright
=========

This document has been placed in the public domain.
