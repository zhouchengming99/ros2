ROS IDL stuff
=============

Goals
-----

* Support specifying ROS messages, services (and actions) using a subset of the `Interface Defintion Language <https://www.omg.org/spec/IDL/About-IDL/>`_ in `.idl` files

  * Utilize format features

    * Group constants into enums
    * Annotations:

      * Comment
      * Unit
      * Range (?)

  * Reference constant from other `.idl` files as default value
  * Determine need for Connext/OpenSplice-specific `.idl` files

* Continue to support the existing feature set of `.msg` and `.srv` files
* Provide a script to convert `.msg` / `.srv` (/ `.action`) files to `.idl`
* Determine types and their mapping into each language

  * Unicode support

    * Ensure that this is feasible with member-based access and doesn't require method-based access (http://design.ros2.org/articles/serialization.html#member-based-vs-method-based-access)

  * Efficient handling of binary data (e.g. the image messsage)

* Distinguish a "strict" (default?) mode where ROS conventions apply vs. a "relaxed" mode where the user can diverge from those (?)

* Support evolution of message definitions (?)
* Support for optional fields (which don't appear on the wire if not set) (?)
* Configure custom upper bounds (at generation time, not specified in the IDL) (?)
* Support keys (?)
* Default initizlization if message fields in C (?)
* Default values for complex fields
* Provide e.g. `to_yaml` functions
* Support providing additional "helper" functions (e.g. C++ constructor with poitional arguments for Vector3)

Discussions to be considered
----------------------------

* Discussion about how to add a new PRIMITIVE_TYPES to rosidl(msg): https://discourse.ros.org/t/discussion-about-how-to-add-a-new-primitive-types-to-rosidl-msg/4607

Tickets to be considered
------------------------

* Review primitive types: https://github.com/ros2/rosidl/issues/190

  * Deprecated byte char: https://github.com/ros2/rosidl/pull/191
  * remap `char` and `byte` to `int8_t` and `uint8_t`: https://github.com/ros2/design/pull/130
  * use `uint8` instead of deprecated `byte`: https://github.com/ros2/rcl_interfaces/pull/13
  * Deprecate char and byte: https://github.com/ros2/system_tests/pull/185
  * Change Image (and other) messages to use `byte[]` rather than `uint8[]`: https://github.com/ros2/common_interfaces/issues/55
  * Use buffer protocol and byte type for byte arrays in python: https://github.com/ros2/rosidl/pull/276

* ROS2 message format should support enums: https://github.com/ros2/rosidl/issues/260

* Wide string: https://github.com/ros2/design/pull/117

  * add multibyte tests to Primitives: https://github.com/ros2/system_tests/pull/194
  * encode/decode all string as utf8: https://github.com/ros2/rosidl/pull/209

* Optional Fields in Message: https://discourse.ros.org/t/optional-fields-in-message/991

* Design doc for C mapping: https://github.com/ros2/design/issues/126

* Collisions with keywords: https://github.com/ros2/design/pull/172

  * Consider narrowing the scope of `#undef ERROR` in generated message code: https://github.com/ros2/rosidl/issues/118

* [rosidl_generator_py] Provide field type in the generated message classes: https://github.com/ros2/rosidl/issues/243

* [rosidl_generator_py] Can bypass setter checks by setting individual list elements: https://github.com/ros2/rosidl/issues/279

* Error on all msg/srv names with underscores: https://github.com/ros2/rosidl/pull/288
* Support placing message files in subfolders of "msg": https://github.com/ros2/rosidl/issues/213


ROS IDL
-------

Design article: http://design.ros2.org/articles/interface_definition.html

* `bool`
* `byte`
* `char`
* `float32`, `float64`
* `int8`, `uint8`
* `int16`, `uint16`
* `int32`, `uint32`
* `int64`, `uint64`
* `string`

Notes:

* string does not specify any encoding
* consider wchar, wstring, u16string, u32string
* consider removing byte, char

ROS IDL -> C
------------

No design article.

Primitives
~~~~~~~~~~
* `bool` -> `bool`
* `byte` -> `uint8_t`
* `char` -> `signed char`
* `float32` -> `float`
* `float64` -> `double`
* `int8` -> `int8_t`
* `uint8` -> `uint8_t`
* `int16` -> `int16`
* `uint16` -> `uint16`
* `int32` -> `int32`
* `uint32` -> `uint32`
* `int64` -> `int64`
* `uint64` -> `uint64_t`
* `string` -> `rosidl_generator_c__String`

Arrays / bounded strings
~~~~~~~~~~~~~~~~~~~~~~~~
* `static array` -> `[N]`
* `unbounded dynamic array` -> `<typename>__Array`
* `bounded dynamic array` -> `<typename>__Array`
* `bounded string` -> `<typename>__Array`

ROS IDL -> C++
--------------

Design article: http://design.ros2.org/articles/generated_interfaces_cpp.html

Primitives
~~~~~~~~~~
* `bool` -> `bool`
* `byte` -> `uint8_t`
* `char` -> `char`
* `float32` -> `float`
* `float64` -> `double`
* `int8` -> `int8_t`
* `uint8` -> `uint8_t`
* `int16` -> `int16`
* `uint16` -> `uint16`
* `int32` -> `int32`
* `uint32` -> `uint32`
* `int64` -> `int64`
* `uint64` -> `uint64_t`
* `string` -> `std::string`  // actually: `std::basic_string<char, std::char_traits<char>, typename ContainerAllocator::template rebind<char>::other>`

Arrays / bounded strings
~~~~~~~~~~~~~~~~~~~~~~~~
* `static array` -> `std::array<T, N>`
* `unbounded dynamic array` -> `std::vector`
* `bounded dynamic array` -> `custom_class<T, N>`
* `bounded string` -> `std::string`

ROS IDL -> Python
-----------------

Design article: http://design.ros2.org/articles/generated_interfaces_python.html

Primitives
~~~~~~~~~~
* `bool` -> `builtins.bool`
* `byte` -> `builtins.bytes with length 1`
* `char` -> `builtins.str with length 1`
* `float32` -> `builtins.float`
* `float64` -> `builtins.float`
* `int8` -> `builtins.int`
* `uint8` -> `builtins.int`
* `int16` -> `builtins.int`
* `uint16` -> `builtins.int`
* `int32` -> `builtins.int`
* `uint32` -> `builtins.int`
* `int64` -> `builtins.int`
* `uint64` -> `builtins.int`
* `string` -> `builtins.str`

Arrays / bounded strings
~~~~~~~~~~~~~~~~~~~~~~~~
* `static array` -> `builtins.list`
* `unbounded dynamic array` -> `builtins.list`
* `bounded dynamic array` -> `builtins.list`
* `bounded string` -> `builtins.str`

* Implementation differs from design doc:
  * `byte[]` -> `bytes`
  * `char[]` -> `builtins.str`

Notes:

* Common messages use `uint8[]` for binary data which is expensive since it maps to a list of ints.

ROS IDL -> DDS IDL
------------------

Primitives
~~~~~~~~~~
* `bool` -> `boolean`
* `byte` -> `octet`
* `char` -> `char`
* `float32` -> `float`
* `float64` -> `double`
* `int8` -> `octet`
* `uint8` -> `octet`
* `int16` -> `short`
* `uint16` -> `unsigned short`
* `int32` -> `long`
* `uint32` -> `unsigned long`
* `int64` -> `long long`
* `uint64` -> `unsigned long long`
* `string` -> `string`

Arrays / bounded strings
~~~~~~~~~~~~~~~~~~~~~~~~
* `static array` -> `T[N]`
* `unbounded dynamic array` -> `sequence`
* `bounded dynamic array` -> `sequence<T, N>`
* `bounded string` -> `string`



RTI DDS gen
===========

DDS -> C++
----------

This is what `rosidl_typesupport_connext_cpp` currently uses.

``rtiddsgen -d cpp -language C++ -namespace -update typefiles -unboundedSupport Primitives.idl``

The C++ class has **public** members ending with `_`.
The C++ class has (const) getter and setter methods.

* `boolean` -> `DDS_Boolean`
* `char` -> `DDS_Char`
* `double` -> `DDS_Double`
* `float` -> `DDS_Float`
* `long double` -> `DDS_LongDouble`
* `long` -> `DDS_Long`
* `long long` -> `DDS_LongLong`
* `octet` -> `DDS_Octet`
* `short` -> `DDS_Short`
* `string` -> `DDS_Char *`
* `unsigned long long` -> `DDS_UnsignedLongLong`
* `unsigned long` -> `DDS_UnsignedLong`
* `unsigned short` -> `DDS_UnsignedShort`
* `wchar` -> `DDS_Wchar`
* `wstring` -> `DDS_Wchar *`

DDS -> C++11
------------

``rtiddsgen -d cpp11 -language C++11 -update typefiles -unboundedSupport Primitives.idl``

The C++ class has **private** members ending with `_`.
The C++ class has (const) getter and setter methods.

* `boolean` -> `bool`
* `char` -> `char`
* `double` -> `double`
* `float` -> `float`
* `long double` -> `rti::core::LongDouble`
* `long` -> `int32_t`
* `long long` -> `rti::core::int64`
* `octet` -> `uint8_t`
* `short` -> `int16_t`
* `string` -> `dds::core::string`
* `unsigned long long` -> `rti::core::uint64`
* `unsigned long` -> `uint32_t`
* `unsigned short` -> `uint16_t`
* `wchar` -> `DDS_Wchar`
* `wstring` -> `dds::core::wstring`

ROS 1 Message Description Specification
---------------------------------------

See http://wiki.ros.org/msg
