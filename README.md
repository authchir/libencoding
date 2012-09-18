## Introduction

libencoding aims to provide a generic and extensible encoding conversion library. It's main component
is the `converter<> class template` which perform the conversion from an input encoding to an output
encoding. It is declare as follow:

```c++
    template <
        class FromCharT,
        class FromEncoding,
        class ToCharT,
        class ToEncoding
    > struct converter;
```

Instance of this function object provide an `operator()` that accept an input iterator range
`[first, last)` of character type for the input encoding and an output iterator where character type
for the output encoding will be placed.

```c++
    using namespace encoding;
    
    converter<char, utf8, std::uint32_t, utf32> conv;
    
    const char utf8_buffer[] = u8"Lorem ipsum";
    std::vector<std::uint32_t> utf32_buffer;
    
    conv(std::begin(utf8_buffer), std::end(utf8_buffer), std::back_inserter(utf32_buffer));
```

Since the conversion of a string literal is a common task and tedious to write, an overload is
provide which replace the input iterator range is replace by a `const from_char_type (&)[N]`
parameter. So the above example could have been simply write:

```c++
    conv(u8"Lorem ipsum", std::back_inserter(utf32_buffer));
```

The default behaviour is to throw an exception if an error occure or if the operation result in a
partial conversion. For a fine grain error handling, an overload is provide with a `std::error_code&`
parameter where the corresponding error code will be place.

## Motivation and Scope

I am working on this library for two main purpose:
- Learning about unicode encoding and libary design.
- Try to get a simple libary that offer many possibilities for modern C++ development.

### The possibility to add a new encoding easily

### The possibility to optimize selected conversions

### The possibility to write generic code independant of the encoding

### The possibility to use modern and standard practices

## Design Decisions

## Technical Specifications

```c++
namespace encoding
{
    enum class encoding_errc
    {
        trailing_byte_expected = 1,
        leading_byte_expected,
        partial_conversion,
        ill_formed_code_unit,
        ill_formed_code_point,
        ill_formed_byte_sequence
    };

    const std::error_category& encoding_category();

    std::error_code make_error_code(encoding_errc e);
    std::error_code make_error_condition(encoding_errc e);

    struct utf32   { };
    struct utf32le { };
    struct utf32be { };
    struct utf16   { };
    struct utf16le { };
    struct utf16be { };
    struct utf8    { };

    template<class Encoding> struct max_code_unit_per_code_point;

    template<> struct max_code_unit_per_code_point<utf32>;
    template<> struct max_code_unit_per_code_point<utf32le>;
    template<> struct max_code_unit_per_code_point<utf32be>;
    template<> struct max_code_unit_per_code_point<utf16>;
    template<> struct max_code_unit_per_code_point<utf16le>;
    template<> struct max_code_unit_per_code_point<utf16be>;
    template<> struct max_code_unit_per_code_point<utf8>;

    template<class Encoding> struct min_code_unit_per_code_point;

    template<> struct min_code_unit_per_code_point<utf32>;
    template<> struct min_code_unit_per_code_point<utf32le>;
    template<> struct min_code_unit_per_code_point<utf32be>;
    template<> struct min_code_unit_per_code_point<utf16>;
    template<> struct min_code_unit_per_code_point<utf16le>;
    template<> struct min_code_unit_per_code_point<utf16be>;
    template<> struct min_code_unit_per_code_point<utf8>;

    template<class Encoding>
    struct is_variable_length
        : std::integral_constant<bool,
            max_code_unit_per_code_point<Encoding>::value
            != min_code_unit_per_code_point<Encoding>::value
          >
    {
    };

    template <
        class FromCharT,
        class FromEncoding,
        class ToCharT,
        class ToEncoding
    > struct converter
    {
        typedef FromCharT    from_char_type;
        typedef FromEncoding from_encoding;
        typedef ToCharT      to_char_type;
        typedef ToEncoding   to_encoding;

        converter() = default;
        converter(const converter&) = default;
        converter(converter&&) = default;

        ~converter() = default;

        converter& operator=(const converter&) = default;
        converter& operator=(converter&&) = default;

        template<class InputIterator, class OutputIterator>
        void operator()(InputIterator first,
                        InputIterator last,
                        OutputIterator out,
                        std::error_code& e);

        template<class InputIterator, class OutputIterator>
        void operator()(InputIterator first,
                        InputIterator last,
                        OutputIterator out);

        template<std::size_t N, class OutputIterator>
        void operator()(const from_char_type (&str)[N],
                        OutputIterator out,
                        std::error_code& e);

        template<std::size_t N, class OutputIterator>
        void operator()(const from_char_type (&str)[N],
                        OutputIterator out);
    };

    // UTF-32
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf32>;

    // UTF-32LE
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf32le>;
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32le, ToCharT, utf32>;

    // UTF-32BE
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf32be>;
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32be, ToCharT, utf32>;

    // UTF-16
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf16>;
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf16,   ToCharT, utf32>;

    // UTF-16LE
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf16le>;
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf16le, ToCharT, utf32>;

    // UTF-16BE
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf16be>;
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf16be, ToCharT, utf32>;

    // UTF-8
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf32,   ToCharT, utf8>;
    template<class FromCharT, class ToCharT> struct converter<FromCharT, utf8,    ToCharT, utf32>;
} // namespace encoding

namespace std 
{ 
    template<> 
    struct is_error_code_enum<encoding::encoding_errc> : true_type { }; 
} // namespace std
```

## References

The Unicode Consortium. The Unicode Standard.

[http://www.unicode.org/versions/latest/](http://www.unicode.org/versions/latest/)