# VGA 텍스트 모드

VGA 텍스트 모드를 사용하면 간단하게 텍스트를 화면에 출력할 수 있다.

안전하지 않은 연산을 모듈로 분리하고 감싸서 안전하고 간편하게 텍스트 출력을 할 수 있는 인터페이스를 만들어 본다.

추가로 러스트의 포매팅 매크로도 구현한다.

## VGA 텍스트 버퍼

VGA 텍스트 모드의 화면에 문자를 출력하려면 VGA 장치의 텍스트 버퍼에 문자를 작성해야 한다.

VGA 텍스트 버퍼는 보통 25 x 80 크기의 2차원 배열이다.

배열의 각 요소는 다음과 같은 형식으로 화면에 출력되는 문자 하나를 나타낸다.

| 비트  | 값          |
| ----- | ----------- |
| 0-7   | 아스키 코드 |
| 8-11  | 전경색      |
| 12-14 | 배경색      |
| 15    | 깜빡이      |

첫 번째 바이트는 아스키 인코딩으로 출력되는 문자를 나타낸다.

> 정확히 아스키 코드는 아니고 code page 437이라는 문자 세트다.

두 번째 바이트는 문자의 모양을 정한다.

앞 4비트는 전경색을, 뒤 3비트는 배경색을, 마지막 1비트는 문자 깜빡임을 정한다.

다음 표에 나온 색깔을 사용할 수 있다.

| Number | Color      | Number + Bright Bit | Bright Color |
| ------ | ---------- | ------------------- | ------------ |
| 0x0    | Black      | 0x8                 | Dark Gray    |
| 0x1    | Blue       | 0x9                 | Light Blue   |
| 0x2    | Green      | 0xa                 | Light Green  |
| 0x3    | Cyan       | 0xb                 | Light Cyan   |
| 0x4    | Red        | 0xc                 | Light Red    |
| 0x5    | Magenta    | 0xd                 | Pink         |
| 0x6    | Brown      | 0xe                 | Yellow       |
| 0x7    | Light Gray | 0xf                 | White        |

VGA 텍스트 버퍼는 메모리에 매핑된 입출력(memory-mapped I/O)
을 통해 `0xb8000` 주소로 접근할 수 있다.

메모리 연산으로 RAM이 아닌 VGA 하드웨어의 텍스트 버퍼에 직접 읽고 쓸 수 있다.

## 러스트 모듈

VGA 텍스트 버퍼 처리를 위한 모듈을 만든다.

`vga_buffer.rs` 파일에 `mod vga_buffer;` 코드를 추가해서 모듈을 선언한다.

다양한 색을 나타내는 열거형을 선언한다.

```rust
#[allow(dead_code)]
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
#[repr(u8)]
pub enum Color {
    Black = 0,
    Blue = 1,
    Green = 2,
    Cyan = 3,
    Red = 4,
    ...
}
```

안 쓰는 변형타입 경고를 끄기 위해 `allow(dead_code)` 속성을 추가했다.

열거형을 복사하고 비교하고 출력할 수 있게 하기 위해 `derive`로 트레잇들을 이끌어냈다.

러스트에 색 표현에 4비트면 충분하지만 `u4`가 없어서 색을 `u8`로 표현했다.

전경색, 배경색을 표현하기 위해 `u8` 타입에다 뉴타입(`ColorCode(u8)`)을 만든다.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
#[repr(transparent)]
struct ColorCode(u8);

impl ColorCode {
    fn new(foreground: Color, background: Color) -> ColorCode {
        ColorCode((background as u8) << 4 | (foreground as u8))
    }
}
```

`ColorCode`가 안에 있는 `u8`과 똑같은 데이터 레이아웃을 갖게하려고 `repr(transparent)` 속성을 추가했다.

## 텍스트 버퍼

화면 문자와 텍스트 버퍼를 구조체로 만든다.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
#[repr(C)]
struct ScreenChar {
    ascii_character: u8,
    color_code: ColorCode,
}

const BUFFER_HEIGHT: usize = 25;
const BUFFER_WIDTH: usize = 80;

#[repr(transparent)]
struct Buffer {
    chars: [[ScreenChar; BUFFER_WIDTH]; BUFFER_HEIGHT],
}
```

러스트의 구조체 필드 순서는 기본적으로 안 정해져 있다.
`repr(C)` 속성으로 구조체 필드가 C 구조체와 동일하게 나열되게 한다.

화면에 작성할 때 사용할 `Writer` 타입도 만든다.

```rust
pub struct Writer {
    column_position: usize,
    color_code: ColorCode,
    buffer: &'static mut Buffer,
}
```

writer는 마지막 줄에 작성하고 줄이 꽉차거나 `\n`이 나왔을 때 윗줄로 바꾼다.

`column_position` 필드로 마지막 행의 현재 위치를 추적한다.

전경색, 배경색은 `color_code` 필드로, VGA 버퍼는 `buffer` 필드로 나타낸다.

버퍼의 라이프타임을 프로그램 전체동안 유지하기 위해 `'static` 라이프타임을 사용했다.
