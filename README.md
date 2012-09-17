## Introduction

libencoding aims to provide a generic and extensible encoding conversion library.

## Motivation and Scope

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

    struct utf8    { }; 
    struct utf16   { }; 
    struct utf16le { }; 
    struct utf16be { }; 
    struct utf32   { }; 
    struct utf32le { }; 
    struct utf32be { }; 

    std::ostream& operator<<(std::ostream& out, const utf8&);
    std::ostream& operator<<(std::ostream& out, const utf16&);
    std::ostream& operator<<(std::ostream& out, const utf16le&);
    std::ostream& operator<<(std::ostream& out, const utf16be&);
    std::ostream& operator<<(std::ostream& out, const utf32&);
    std::ostream& operator<<(std::ostream& out, const utf32le&);
    std::ostream& operator<<(std::ostream& out, const utf32be&);

    template<class Encoding> struct max_code_unit_per_code_point; 
 
    template<> struct max_code_unit_per_code_point<utf8>;
    template<> struct max_code_unit_per_code_point<utf16>;
    template<> struct max_code_unit_per_code_point<utf16le>; 
    template<> struct max_code_unit_per_code_point<utf16be>; 
    template<> struct max_code_unit_per_code_point<utf32>; 
    template<> struct max_code_unit_per_code_point<utf32le>; 
    template<> struct max_code_unit_per_code_point<utf32be>; 

    template<
        class FromCharT,
        class FromEncoding,
        class ToCharT,
        class ToEncoding> 
    struct converter 
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
                        std::error_code& e) ;

        template<class InputIterator, class OutputIterator> 
        void operator()(InputIterator first,
                        InputIterator last,
                        OutputIterator out) ;

        template<std::size_t N, class OutputIterator> 
        void operator()(const from_char_type (&str)[N],
                        OutputIterator out,
                        std::error_code& e) ;

        template<std::size_t N, class OutputIterator> 
        void operator()(const from_char_type (&str)[N],
                        OutputIterator out) ;
    }; 

    // UTF-8 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8, ToCharT, utf8>; 

    // UTF-16 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8, ToCharT, utf16>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf16, ToCharT, utf16>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf16, ToCharT, utf8>; 

    // UTF-16LE 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8, ToCharT, utf16le>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf16le, ToCharT, utf16le>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf16le, ToCharT, utf8>; 

    // UTF-16BE 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8, ToCharT, utf16be>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf16be, ToCharT, utf16be>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf16be, ToCharT, utf8>; 

    // UTF-32 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8, ToCharT, utf32>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf32, ToCharT, utf32>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf32, ToCharT, utf8>; 

    // UTF-32LE 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8, ToCharT, utf32le>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf32le, ToCharT, utf32le>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf32le, ToCharT, utf8>; 

    // UTF-32BE 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf8,  ToCharT, utf32be>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf32be, ToCharT, utf32be>; 
    template<class FromCharT, class ToCharT>
        struct converter<FromCharT, utf32be, ToCharT, utf8>; 
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