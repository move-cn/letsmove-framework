# Sui Framework
- [Sui Framework](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages)
  什么是 Sui Framework， 大家第一次看到 Framework的时候会觉得奇怪，不知道是什么，
  简单来说就是Sui Move的标准库，官方已经封装好了一些常用的库供大家使用
- Sui Framework 是经过了长期打磨，和安全验证的代码，阅读Framework不仅仅能掌握常用的工具库，
  而且能更好的学习Sui Move，Framework每一个包和库都值得大家精心阅读和学习


# move-stdlib
-  [move-stdlib](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/move-stdlib)

> move stdlib 是来自Move上游的核心标准库，可以理解成最核心的标准库，基本上是需要大家都掌握的


## 总览，可以这样说 就是最好把这个系列的都完全掌握
| 模块名          |            大概用途 | 需要掌握程度 |
|:-------------|----------------:|:------:|
| address.move |            地址长度 |   了解   |
| ascii.move   |     ascii编码的字符串 |  完全掌握  |
| bcs.move     |      把数据序列化成二进制 |   掌握   |
| bit_vector.move   |      bit 位标记的数组 |   掌握   |
| debug.move |       调试代码，打印输出 |  完全掌握  |
| fixed_point32.move   |             浮点数 |   掌握   |
| hash.move |          hash函数 |   掌握   |
| option.move   |             可选值 |  完全掌握  |
| type_name.move     |         获取结构的类型 |  完全掌握  |
| unit_test.move   | 单元测试生成测试signers |   了解   |
| vector.move |              数组 |  完全掌握  |

## address.move
    基本上没什么用处
```move
module std::address {
    public fun length(): u64 {
        32
    }
}

```

## ascii.move
ascii 编码规则的字符串,说白了就是只有a-z A-Z 0-9这些编码的字符串，只支持早期的英文字符

```Move
module std::ascii {
    use std::option::{Self, Option};

    // Allows calling `.to_string()` to convert an `ascii::String` into as `string::String`
    public use fun std::string::from_ascii as String.to_string;

    /// An invalid ASCII character was encountered when creating an ASCII string.
    const EINVALID_ASCII_CHARACTER: u64 = 0x10000;

    /// The `String` struct holds a vector of bytes that all represent
    /// valid ASCII characters. Note that these ASCII characters may not all
    /// be printable. To determine if a `String` contains only "printable"
    /// characters you should use the `all_characters_printable` predicate
    /// defined in this module.
    public struct String has copy, drop, store {
        bytes: vector<u8>,
    }

    /// An ASCII character.
    public struct Char has copy, drop, store {
        byte: u8,
    }

    /// Convert a `byte` into a `Char` that is checked to make sure it is valid ASCII.
    public fun char(byte: u8): Char {
        assert!(is_valid_char(byte), EINVALID_ASCII_CHARACTER);
        Char { byte }
    }

    /// Convert a vector of bytes `bytes` into an `String`. Aborts if
    /// `bytes` contains non-ASCII characters.
    public fun string(bytes: vector<u8>): String {
       let x = try_string(bytes);
       assert!(x.is_some(), EINVALID_ASCII_CHARACTER);
       x.destroy_some()
    }

    /// Convert a vector of bytes `bytes` into an `String`. Returns
    /// `Some(<ascii_string>)` if the `bytes` contains all valid ASCII
    /// characters. Otherwise returns `None`.
    public fun try_string(bytes: vector<u8>): Option<String> {
        let len = bytes.length();
        let mut i = 0;
        while (i < len) {
            let possible_byte = bytes[i];
            if (!is_valid_char(possible_byte)) return option::none();
            i = i + 1;
        };
        option::some(String { bytes })
    }

    /// Returns `true` if all characters in `string` are printable characters
    /// Returns `false` otherwise. Not all `String`s are printable strings.
    public fun all_characters_printable(string: &String): bool {
        let len = string.bytes.length();
        let mut i = 0;
        while (i < len) {
            let byte = string.bytes[i];
            if (!is_printable_char(byte)) return false;
            i = i + 1;
        };
        true
    }

    public fun push_char(string: &mut String, char: Char) {
        string.bytes.push_back(char.byte);
    }

    public fun pop_char(string: &mut String): Char {
        Char { byte: string.bytes.pop_back() }
    }

    public fun length(string: &String): u64 {
        string.as_bytes().length()
    }

    /// Get the inner bytes of the `string` as a reference
    public fun as_bytes(string: &String): &vector<u8> {
       &string.bytes
    }

    /// Unpack the `string` to get its backing bytes
    public fun into_bytes(string: String): vector<u8> {
       let String { bytes } = string;
       bytes
    }

    /// Unpack the `char` into its underlying byte.
    public fun byte(char: Char): u8 {
       let Char { byte } = char;
       byte
    }

    /// Returns `true` if `b` is a valid ASCII character. Returns `false` otherwise.
    public fun is_valid_char(b: u8): bool {
       b <= 0x7F
    }

    /// Returns `true` if `byte` is an printable ASCII character. Returns `false` otherwise.
    public fun is_printable_char(byte: u8): bool {
       byte >= 0x20 && // Disallow metacharacters
       byte <= 0x7E // Don't allow DEL metacharacter
    }
}

```


### bcs
> bcs 大家可能比较模糊，有的人不太了解是什么东西，我举一个例子，就是你定义好一个json的数据结构，然后把它转成json字符串
> 字符串这个类型就是一个通用的数据类型
> bsc存在的意义就是把Move的数据结构转成通用的序列化好的一种二进制
> 好处就是单一类型了，通用性很强，能在不同的编程语言之间传递，反序列化回来了也具有唯一的数据对应


```move
module std::bcs {
    native public fun to_bytes<MoveValue>(v: &MoveValue): vector<u8>;
}
```

### bit_vector.move

数组的一种结构，  [bool,length] [ture,false,ture],就是会标注index位置有没有被使用

```move
 public struct BitVector has copy, drop, store {
        length: u64,
        bit_field: vector<bool>,
    }

    public fun new(length: u64): BitVector {
        assert!(length > 0, ELENGTH);
        assert!(length < MAX_SIZE, ELENGTH);
        let mut counter = 0;
        let mut bit_field = vector::empty();
        while (counter < length) {
            bit_field.push_back(false);
            counter = counter + 1;
        };

        BitVector {
            length,
            bit_field,
        }
    }

    /// Set the bit at `bit_index` in the `bitvector` regardless of its previous state.
    public fun set(bitvector: &mut BitVector, bit_index: u64) {
        assert!(bit_index < bitvector.bit_field.length(), EINDEX);
        let x = &mut bitvector.bit_field[bit_index];
        *x = true;
    }

    /// Unset the bit at `bit_index` in the `bitvector` regardless of its previous state.
    public fun unset(bitvector: &mut BitVector, bit_index: u64) {
        assert!(bit_index < bitvector.bit_field.length(), EINDEX);
        let x = &mut bitvector.bit_field[bit_index];
        *x = false;
    }
}
```

### debug.move
提供了在开发的时候，人工打印调试信息

```move
module std::debug {
    native public fun print<T>(x: &T);

    native public fun print_stack_trace();
}
```

### fixed_point32.move
32位的定点数，也就是简单的浮点数构造
```move

module std::fixed_point32 {

    /// Define a fixed-point numeric type with 32 fractional bits.
    /// This is just a u64 integer but it is wrapped in a struct to
    /// make a unique type. This is a binary representation, so decimal
    /// values may not be exactly representable, but it provides more
    /// than 9 decimal digits of precision both before and after the
    /// decimal point (18 digits total). For comparison, double precision
    /// floating-point has less than 16 decimal digits of precision, so
    /// be careful about using floating-point to convert these values to
    /// decimal.
    public struct FixedPoint32 has copy, drop, store { value: u64 }

    ///> TODO: This is a basic constant and should be provided somewhere centrally in the framework.
    const MAX_U64: u128 = 18446744073709551615;

    /// The denominator provided was zero
    const EDENOMINATOR: u64 = 0x10001;
    /// The quotient value would be too large to be held in a `u64`
    const EDIVISION: u64 = 0x20002;
    /// The multiplied value would be too large to be held in a `u64`
    const EMULTIPLICATION: u64 = 0x20003;
    /// A division by zero was encountered
    const EDIVISION_BY_ZERO: u64 = 0x10004;
    /// The computed ratio when converting to a `FixedPoint32` would be unrepresentable
    const ERATIO_OUT_OF_RANGE: u64 = 0x20005;




    public fun create_from_rational(numerator: u64, denominator: u64): FixedPoint32 {
        // If the denominator is zero, this will abort.
        // Scale the numerator to have 64 fractional bits and the denominator
        // to have 32 fractional bits, so that the quotient will have 32
        // fractional bits.
        let scaled_numerator = numerator as u128 << 64;
        let scaled_denominator = denominator as u128 << 32;
        assert!(scaled_denominator != 0, EDENOMINATOR);
        let quotient = scaled_numerator / scaled_denominator;
        assert!(quotient != 0 || numerator == 0, ERATIO_OUT_OF_RANGE);
        // Return the quotient as a fixed-point number. We first need to check whether the cast
        // can succeed.
        assert!(quotient <= MAX_U64, ERATIO_OUT_OF_RANGE);
        FixedPoint32 { value: quotient as u64 }
    }

    /// Create a fixedpoint value from a raw value.
    public fun create_from_raw_value(value: u64): FixedPoint32 {
        FixedPoint32 { value }
    }

    /// Accessor for the raw u64 value. Other less common operations, such as
    /// adding or subtracting FixedPoint32 values, can be done using the raw
    /// values directly.
    public fun get_raw_value(num: FixedPoint32): u64 {
        num.value
    }

    /// Returns true if the ratio is zero.
    public fun is_zero(num: FixedPoint32): bool {
        num.value == 0
    }
}

```


### hash.move
提供了两个常用的hash函数

```move
module std::hash {
    native public fun sha2_256(data: vector<u8>): vector<u8>;
    native public fun sha3_256(data: vector<u8>): vector<u8>;
}
```

### option.move

一种容器类，表达的是，一个东西能为空，而且用这种类型会约束你强制处理为空的情况
简单来说一个NFT的图片，允许为空，或者必须填写

```move
// Copyright (c) Mysten Labs, Inc.
// SPDX-License-Identifier: Apache-2.0

// Copyright (c) Mysten Labs, Inc.
// SPDX-License-Identifier: Apache-2.0

/// This module defines the Option type and its methods to represent and handle an optional value.
module std::option {
    /// Abstraction of a value that may or may not be present. Implemented with a vector of size
    /// zero or one because Move bytecode does not have ADTs.
    public struct Option<Element> has copy, drop, store {
        vec: vector<Element>
    }

    /// The `Option` is in an invalid state for the operation attempted.
    /// The `Option` is `Some` while it should be `None`.
    const EOPTION_IS_SET: u64 = 0x40000;
    /// The `Option` is in an invalid state for the operation attempted.
    /// The `Option` is `None` while it should be `Some`.
    const EOPTION_NOT_SET: u64 = 0x40001;

    /// Return an empty `Option`
    public fun none<Element>(): Option<Element> {
        Option { vec: vector::empty() }
    }

    /// Return an `Option` containing `e`
    public fun some<Element>(e: Element): Option<Element> {
        Option { vec: vector::singleton(e) }
    }

    /// Return true if `t` does not hold a value
    public fun is_none<Element>(t: &Option<Element>): bool {
        t.vec.is_empty()
    }

    /// Return true if `t` holds a value
    public fun is_some<Element>(t: &Option<Element>): bool {
        !t.vec.is_empty()
    }

}


```

### string.move

- 字符串的UTF-8版本
- 提供了一些ascii转换的方法和定义
```move
module std::string {
    use std::ascii;
    use std::vector;
    use std::option::{Self, Option};

    /// An invalid UTF8 encoding.
    const EINVALID_UTF8: u64 = 1;

    /// Index out of range.
    const EINVALID_INDEX: u64 = 2;

    /// A `String` holds a sequence of bytes which is guaranteed to be in utf8 format.
    public struct String has copy, drop, store {
        bytes: vector<u8>,
    }

    /// Creates a new string from a sequence of bytes. Aborts if the bytes do not represent valid utf8.
    public fun utf8(bytes: vector<u8>): String {
        assert!(internal_check_utf8(&bytes), EINVALID_UTF8);
        String{bytes}
    }

    /// Convert an ASCII string to a UTF8 string
    public fun from_ascii(s: ascii::String): String {
        String { bytes: ascii::into_bytes(s) }
    }

    /// Convert an UTF8 string to an ASCII string.
    /// Aborts if `s` is not valid ASCII
    public fun to_ascii(s: String): ascii::String {
        let String { bytes } = s;
        ascii::string(bytes)
    }
}
```

### type_name.move

```move
module std::type_name {
    use std::ascii::{Self, String};
    use std::address;

    /// ASCII Character code for the `:` (colon) symbol.
    const ASCII_COLON: u8 = 58;

    /// ASCII Character code for the `v` (lowercase v) symbol.
    const ASCII_V: u8 = 118;
    /// ASCII Character code for the `e` (lowercase e) symbol.
    const ASCII_E: u8 = 101;
    /// ASCII Character code for the `c` (lowercase c) symbol.
    const ASCII_C: u8 = 99;
    /// ASCII Character code for the `t` (lowercase t) symbol.
    const ASCII_T: u8 = 116;
    /// ASCII Character code for the `o` (lowercase o) symbol.
    const ASCII_O: u8 = 111;
    /// ASCII Character code for the `r` (lowercase r) symbol.
    const ASCII_R: u8 = 114;

    /// The type is not from a package/module. It is a primitive type.
    const ENonModuleType: u64 = 0;

    public struct TypeName has copy, drop, store {
        /// String representation of the type. All types are represented
        /// using their source syntax:
        /// "u8", "u64", "bool", "address", "vector", and so on for primitive types.
        /// Struct types are represented as fully qualified type names; e.g.
        /// `00000000000000000000000000000001::string::String` or
        /// `0000000000000000000000000000000a::module_name1::type_name1<0000000000000000000000000000000a::module_name2::type_name2<u64>>`
        /// Addresses are hex-encoded lowercase values of length ADDRESS_LENGTH (16, 20, or 32 depending on the Move platform)
        name: String
    }

    /// Return a value representation of the type `T`.  Package IDs
    /// that appear in fully qualified type names in the output from
    /// this function are defining IDs (the ID of the package in
    /// storage that first introduced the type).
    public native fun get<T>(): TypeName;

    /// Return a value representation of the type `T`.  Package IDs
    /// that appear in fully qualified type names in the output from
    /// this function are original IDs (the ID of the first version of
    /// the package, even if the type in question was introduced in a
    /// later upgrade).
    public native fun get_with_original_ids<T>(): TypeName;

    /// Returns true iff the TypeName represents a primitive type, i.e. one of
    /// u8, u16, u32, u64, u128, u256, bool, address, vector.
    public fun is_primitive(self: &TypeName): bool {
        let bytes = self.name.as_bytes();
        bytes == &b"bool" ||
        bytes == &b"u8" ||
        bytes == &b"u16" ||
        bytes == &b"u32" ||
        bytes == &b"u64" ||
        bytes == &b"u128" ||
        bytes == &b"u256" ||
        bytes == &b"address" ||
        (bytes.length() >= 6 &&
         bytes[0] == ASCII_V &&
         bytes[1] == ASCII_E &&
         bytes[2] == ASCII_C &&
         bytes[3] == ASCII_T &&
         bytes[4] == ASCII_O &&
         bytes[5] == ASCII_R)

    }

    /// Get the String representation of `self`
    public fun borrow_string(self: &TypeName): &String {
        &self.name
    }

    /// Get Address string (Base16 encoded), first part of the TypeName.
    /// Aborts if given a primitive type.
    public fun get_address(self: &TypeName): String {
        assert!(!self.is_primitive(), ENonModuleType);

        // Base16 (string) representation of an address has 2 symbols per byte.
        let len = address::length() * 2;
        let str_bytes = self.name.as_bytes();
        let mut addr_bytes = vector[];
        let mut i = 0;

        // Read `len` bytes from the type name and push them to addr_bytes.
        while (i < len) {
            addr_bytes.push_back(str_bytes[i]);
            i = i + 1;
        };

        ascii::string(addr_bytes)
    }

    /// Get name of the module.
    /// Aborts if given a primitive type.
    public fun get_module(self: &TypeName): String {
        assert!(!self.is_primitive(), ENonModuleType);

        // Starts after address and a double colon: `<addr as HEX>::`
        let mut i = address::length() * 2 + 2;
        let str_bytes = self.name.as_bytes();
        let mut module_name = vector[];

        loop {
            let char = &str_bytes[i];
            if (char != &ASCII_COLON) {
                module_name.push_back(*char);
                i = i + 1;
            } else {
                break
            }
        };

        ascii::string(module_name)
    }

    /// Convert `self` into its inner String
    public fun into_string(self: TypeName): String {
        self.name
    }
}
```

### vector.move
```move
// Copyright (c) Mysten Labs, Inc.
// SPDX-License-Identifier: Apache-2.0

#[defines_primitive(vector)]
/// A variable-sized container that can hold any type. Indexing is 0-based, and
/// vectors are growable. This module has many native functions.
module std::vector {

    /// Allows calling `.to_string()` on a vector of `u8` to get a utf8 `String`.
    public use fun std::string::utf8 as vector.to_string;

    /// Allows calling `.try_to_string()` on a vector of `u8` to get a utf8 `String`.
    /// This will return `None` if the vector is not valid utf8.
    public use fun std::string::try_utf8 as vector.try_to_string;

    /// Allows calling `.to_ascii_string()` on a vector of `u8` to get an `ascii::String`.
    public use fun std::ascii::string as vector.to_ascii_string;

    /// Allows calling `.try_to_ascii_string()` on a vector of `u8` to get an
    /// `ascii::String`. This will return `None` if the vector is not valid ascii.
    public use fun std::ascii::try_string as vector.try_to_ascii_string;

    /// The index into the vector is out of bounds
    const EINDEX_OUT_OF_BOUNDS: u64 = 0x20000;

    #[bytecode_instruction]
    /// Create an empty vector.
    native public fun empty<Element>(): vector<Element>;

    #[bytecode_instruction]
    /// Return the length of the vector.
    native public fun length<Element>(v: &vector<Element>): u64;

    #[syntax(index)]
    #[bytecode_instruction]
    /// Acquire an immutable reference to the `i`th element of the vector `v`.
    /// Aborts if `i` is out of bounds.
    native public fun borrow<Element>(v: &vector<Element>, i: u64): &Element;

    #[bytecode_instruction]
    /// Add element `e` to the end of the vector `v`.
    native public fun push_back<Element>(v: &mut vector<Element>, e: Element);

    #[syntax(index)]
    #[bytecode_instruction]
    /// Return a mutable reference to the `i`th element in the vector `v`.
    /// Aborts if `i` is out of bounds.
    native public fun borrow_mut<Element>(v: &mut vector<Element>, i: u64): &mut Element;

    #[bytecode_instruction]
    /// Pop an element from the end of vector `v`.
    /// Aborts if `v` is empty.
    native public fun pop_back<Element>(v: &mut vector<Element>): Element;

    #[bytecode_instruction]
    /// Destroy the vector `v`.
    /// Aborts if `v` is not empty.
    native public fun destroy_empty<Element>(v: vector<Element>);

    #[bytecode_instruction]
    /// Swaps the elements at the `i`th and `j`th indices in the vector `v`.
    /// Aborts if `i` or `j` is out of bounds.
    native public fun swap<Element>(v: &mut vector<Element>, i: u64, j: u64);

    /// Return an vector of size one containing element `e`.
    public fun singleton<Element>(e: Element): vector<Element> {
        let mut v = empty();
        v.push_back(e);
        v
    }

    /// Reverses the order of the elements in the vector `v` in place.
    public fun reverse<Element>(v: &mut vector<Element>) {
        let len = v.length();
        if (len == 0) return ();

        let mut front_index = 0;
        let mut back_index = len -1;
        while (front_index < back_index) {
            v.swap(front_index, back_index);
            front_index = front_index + 1;
            back_index = back_index - 1;
        }
    }

    /// Pushes all of the elements of the `other` vector into the `lhs` vector.
    public fun append<Element>(lhs: &mut vector<Element>, mut other: vector<Element>) {
        other.reverse();
        while (!other.is_empty()) lhs.push_back(other.pop_back());
        other.destroy_empty();
    }

    /// Return `true` if the vector `v` has no elements and `false` otherwise.
    public fun is_empty<Element>(v: &vector<Element>): bool {
        v.length() == 0
    }

    /// Return true if `e` is in the vector `v`.
    /// Otherwise, returns false.
    public fun contains<Element>(v: &vector<Element>, e: &Element): bool {
        let mut i = 0;
        let len = v.length();
        while (i < len) {
            if (&v[i] == e) return true;
            i = i + 1;
        };
        false
    }

    /// Return `(true, i)` if `e` is in the vector `v` at index `i`.
    /// Otherwise, returns `(false, 0)`.
    public fun index_of<Element>(v: &vector<Element>, e: &Element): (bool, u64) {
        let mut i = 0;
        let len = v.length();
        while (i < len) {
            if (&v[i] == e) return (true, i);
            i = i + 1;
        };
        (false, 0)
    }

    /// Remove the `i`th element of the vector `v`, shifting all subsequent elements.
    /// This is O(n) and preserves ordering of elements in the vector.
    /// Aborts if `i` is out of bounds.
    public fun remove<Element>(v: &mut vector<Element>, mut i: u64): Element {
        let mut len = v.length();
        // i out of bounds; abort
        if (i >= len) abort EINDEX_OUT_OF_BOUNDS;

        len = len - 1;
        while (i < len) v.swap(i, { i = i + 1; i });
        v.pop_back()
    }

    /// Insert `e` at position `i` in the vector `v`.
    /// If `i` is in bounds, this shifts the old `v[i]` and all subsequent elements to the right.
    /// If `i == v.length()`, this adds `e` to the end of the vector.
    /// This is O(n) and preserves ordering of elements in the vector.
    /// Aborts if `i > v.length()`
    public fun insert<Element>(v: &mut vector<Element>, e: Element, mut i: u64) {
        let len = v.length();
        // i too big abort
        if (i > len) abort EINDEX_OUT_OF_BOUNDS;

        v.push_back(e);
        while (i < len) {
            v.swap(i, len);
            i = i + 1
        }
    }

    /// Swap the `i`th element of the vector `v` with the last element and then pop the vector.
    /// This is O(1), but does not preserve ordering of elements in the vector.
    /// Aborts if `i` is out of bounds.
    public fun swap_remove<Element>(v: &mut vector<Element>, i: u64): Element {
        assert!(!v.is_empty(), EINDEX_OUT_OF_BOUNDS);
        let last_idx = v.length() - 1;
        v.swap(i, last_idx);
        v.pop_back()
    }
}

```



# sui-framework
- [sui-framework](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/sui-framework)
> sui 独有的标准库，和其他链的不一样


| 模块名                       |              大概用途 | 需要掌握程度 |
|:--------------------------|------------------:|:------:|
| address.move              |   提供了一些地址和其他类型的转换 |   掌握   |
| bag.move                  |    map结构，值可以类型不一样 |  完全掌握  |
| bcs.move                  |       在核心标准库上做了补充 |  完全掌握  |
| borrow.move               |                   |   掌握   |
| clock.move                |            获取链上时间 |  完全掌握  |
| coin.move                 |         类似ERC20标准 |  完全掌握  |
| display.move              |           NFT展现标准 |  完全掌握  |
| dynamic_field.move        |              动态属性 |   掌握   |
| dynamic_object_field.move |            动态对象属性 |   掌握   |
| event.move                |              打印日志 |  完全掌握  |
| hex.move                  |             hex编码 |  完全掌握  |
| linked_table.move         |          table的一种 |   掌握   |
| math.move                 |          常用数学工具函数 |  完全掌握  |
| object.move               |             对象工具库 |  完全掌握  |
| object_bag.move           | map结构，值是对象类型可以不一样 |  完全掌握  |
| object_table.move         |  map结构，值是对象类型必须一样 |  完全掌握  |
| package.move              |            包管理和升级 |   掌握   |
| pay.move                  |     对Coin的快捷处理函数， |  完全掌握  |
| prover.move               |           用不上自行了解 |   了解   |
| sui.move                  |           SUI 的定义 |   了解   |
| table.move                |     map结构，值类型必须一样 |  完全掌握  |
| table_vec.move            |       用table实现的数组 |   掌握   |
| transfer.move             |          转移对象所有权， |  完全掌握  |
| tx_context.move           |    取得当前交易钱包信息的上下文 |  完全掌握  |
| types.move                |  类型工具，目前只有判断OWT类型 |  完全掌握  |
| vec_map.move              |      底层是vec数组的map |   掌握   |
| vec_set.move              |      底层是vec数组的Set |   掌握   |
| versioned.move            |          版本管理的工具类 |   掌握   |
| kiosk 目录                  |       NFT交易的基础工具类 |  完全掌握  |
| crypto 目录                 |           高阶的加密算法 |   了解   |
| test.move                 |           测试相关工具库 |  完全掌握  |


### borrow.move
- 一个简单的库，支持烫手山芋的借用机制。
- 在可编程事务中，可以在内部借用值
- 一个事务，使用它并在最后放回。“Borrow”是个烫手山芋
- 确保返回的对象没有被替换为另一个对象。
> 如何理解呢？ 就是用确定一个对象在不同的合约之间传递到还回来的时候没有被改变id


### clock.move
- 提供了一个获取链上时间的方法,Sui上链上的时间在 0x6对象实例，只有这一个唯一的对象实例

```move
module sui::clock {
    public struct Clock has key {
        id: UID,
        timestamp_ms: u64,
    }

    public fun timestamp_ms(clock: &Clock): u64 {
        clock.timestamp_ms
    }

```

### hex.move
base16编码   就是把二进制的数据转成16进制表示让肉眼方便阅读和简短，比如 address

### linked_table.move
链表数据结构
类似于`sui::table`，但值是链接在一起的，允许有序插入和删除

### math.move
- 最大值，最小值，平均值，差值，指数，开平方




## 集合的类型如何选择？
- bag 和 table的选择
> 如果值类型一样就选table,如果值类型不一样就选bas

- object_table  和 table的选择
> 如果值确定是 object就选 object_table ，否则选 table，  table范围更广泛

- object_bag  和 bag
> 如果值确定是 object就选 object_bag ，否则选 table，  bag范围更广泛

- vec相关的和 table 和bag
> 大小已知而且小于1000用 vec ，大小未知 数据比较大 用 table


## vec相关的集合
- 使用vec相关的集合都要非常的小心，
  不能存储大量的数据， 理论上必须小于1000，而且最好不要提供让用户来自行添加数据，也就是不确定长度的，很容易产生gas不足的安全攻击