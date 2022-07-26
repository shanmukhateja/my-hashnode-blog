## MyStudio IDE: Let's talk about UTF-16

Hello,

In a previous [post](https://surya-dev-journey.hashnode.dev/mystudio-ide-windows-utf-16-encoding-software), I mentioned using a workaround when dealing with non UTF-8 files.  It's about time this was fixed.

Let's talk about about UTF-16 and how it works. Later, I'll show you how I added UTF-16 support to MyStudio IDE [project](https://github.com/shanmukhateja/mystudio-ide).

## Brief History

### Character Codes

If I wanted to save the characters "abc" in a text file with UTF-8 encoding, a typical notes taking application would first iterate over the typed characters (in this case, 'a', 'b' & 'c'), find each character's unique numeric representation and store the result in the file.

Each character has a numeric representation (usually a number) called a **character code** that can uniquely identify it.

From asciitable.com:

ASCII was developed in 1960s for use in teleprinters for sending text over phone lines.   

> ASCII stands for **A**merican **S**tandard **C**ode for **I**nformation **I**nterchange. 

> Computers can only understand numbers, so an ASCII code is the numerical representation of a character such as 'a' or '@' or an action of some sort. A

### ASCII Table

When you press 'a' on your keyboard, its "character code" (`97`) is sent to the operating system by the keyboard and any application can "listen" to these events and process them.

For example, your web browser has a "listener" which is listening to key presses and it shares them to the website anytime you type something.

My browser was doing the same thing when I was typing this article :) 

ASCII Table has a complete list of valid characters [here](https://www.asciitable.com/).

> ASCII contains 127 characters.

### UTF-8

> ASCII is the first 128 (0x00 – 0x7f) characters of UTF-8. UTF-8 is actually a variable-length, multi-byte encoding of Unicode. UTF-8 uses from 1 to 4 bytes for each character. 

> Wikipedia

## Theory

Let's see what gets saved to a UTF-8 encoded text file with `"abc"` as it's contents. 

```rust
[97, 98, 99, 10]

`97` is character code of `a`
`98` is character code of `b`
`99` is character code of `c`
`10` is character code of `LF` or **L**ine **F**eed

```

Interesting..isn't it?

Let's see what UTF-16 does.

We'll try to save the same text "abc" and save it with UTF-16 encoding.

```rust
[97, 0, 98, 0, 99, 0]
```

So..it's the same characters as before..but with a 0 "suffix" byte.

In computer terminology, a byte number with value 0 is called a null byte.

Now, lets see how Wikipedia defines UTF-16

> UTF-16 is a character encoding capable of encoding all 1,112,064 valid character code points of Unicode.

> The encoding is variable-length, as code points are encoded with one or two 16-bit code units. _UTF-16 arose from an earlier obsolete fixed-width 16-bit encoding, now known as UCS-2 (for 2-byte Universal Character Set)_.

That makes sense! We were seeing a 2-byte sequence of characters where all the trailing bytes were null bytes.

So, for a given "abc", it needs to produce `[97, 0, 98, 0, 99, 0]` to be considered a valid UTF-16 file.

### What about LE or BE?

From my previous post: 

> In simple terms, it determines the order or sequence with which set of bytes are stored in a file. In this case, the UTF-16 LE means its a text which is UTF-16 encoded whose byte ordering is in Little Endian format.

For example:

**Text**                :   "bc"

**Input**               :   [98, 0, 99, 0]

**Little Endian**   :   [98, 0, 99, 0]

**Big Endian**      :   [0, 98, 0, 99]


### Solution

Let's write a Rust function that takes a string and returns LE and BE byte slices.

```rust

// Borrowed from content_inspector crate
static UTF16_BYTE_ORDER_MARKS: &[(&[u8], ContentType)] = &[
    (&[0xFF, 0xFE], ContentType::UTF_16LE),
    (&[0xFE, 0xFF], ContentType::UTF_16BE),
];

fn encode_to_utf16<B: byteorder::ByteOrder>(
    text: String,
    content_type: ContentType,
) -> Option<Vec<u8>> {
    let mut buf: Vec<u8> = vec![];

    // Inject BOM
    let byte_order = UTF16_BYTE_ORDER_MARKS
        .iter()
        .find(|r| r.1 == content_type)?;
    for bom_char in byte_order.0 {
        buf.push(bom_char.to_owned());
    }

    for char in text.encode_utf16() {
        // Assume UTF16 has 2 byte sequences
        let mut data_bytes = vec![0, 0];

        // Write bytes to buffer with provided byteorder (LE or BE)
        B::write_u16(&mut data_bytes, char);

        let _ = buf.write(&data_bytes[..]);
    }

    // return buffer
    Some(buf)
}
```
Credit: https://github.com/udoprog/ptscan/blob/46a3a7652d5ece03842e72a38b9b1f67a5519027/lib/src/encoding.rs#L91

This function takes any String `text`, creates a empty `buffer` `Vec` (array), injects appropriate BOM bytes to the beginning of the `Vec`, stores the encoded UTF-16 bytes to buffer and returns it.

### BOM?

BOM stands for **B**yte **O**rder **M**ark. It's a convenient way to recognize a file's byte order (LE or BE) by reading the first few characters of a file.

BOM is a set of 2 bytes - `0xFF` (numeric: 255) & `0xFE` (numeric: 254).  In LE format, its stored as `[255, 254]` and in the case of BE, its stored as `[254, 255]`.

Finally, the buffer would be:

```rust
[255, 254, 99, 0, 98, 0]
```

You can find my Git commit for this feature [here](https://github.com/shanmukhateja/mystudio-ide/commit/769a7f709d44c6cbdc115a4e5a63c9fdacf6ff7f).

## Conclusion

I hope you've learned something interesting. Give a Like to this post (on the right) and don't forget to add a comment on your thoughts about this.

You can also @ me on [Twitter](https://twitter.com/shanmukhateja94).

Bye for now :-)