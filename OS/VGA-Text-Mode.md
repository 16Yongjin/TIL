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

앞 4 bit는 전경색을, 뒤 3비트는 배경색을, 마지막 1 bit는 문자 깜빡임을 정한다.

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

`main.rs` 파일에 `mod vga_buffer;` 코드를 추가해서 모듈을 선언한다.

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

러스트의 구조체 필드가 메모리 상에서 배치되는 순서는 정해져 있지 않다.

`repr(C)` 속성으로 구조체 필드가 C 구조체와 동일하게 메모리 상에서 나열되게 한다.

이제 구조체 변수 그대로 메모리에 대입할 수 있다.

버퍼를 조작해서 화면에 글자를 작성하는 `Writer` 타입도 만든다.

```rust
pub struct Writer {
    column_position: usize,
    color_code: ColorCode,
    buffer: &'static mut Buffer,
}
```

`Writer`는 문자 작성 중에 줄이 꽉차거나 `\n`이 나오면 그 줄을 위로 올린다.

`column_position` 필드는 마지막 행의 인덱스를 추적해서 개행 여부를 파악하는데 쓰인다.

전경색, 배경색은 `color_code` 필드로, VGA 버퍼는 `buffer` 필드로 나타낸다.

프로그램이 작동하는 내내 버퍼를 사용하기 위해 `'static` 라이프타임을 사용했다.

## 출력하기

`write_byte`는 문자 하나를 작성하는데 쓰인다.

```rust
impl Writer {
    pub fn write_byte(&mut self, byte: u8) {
        match byte {
            b'\n' => self.new_line(),
            byte => {
                if self.column_position >= BUFFER_WIDTH {
                    self.new_line();
                }

                let row = BUFFER_HEIGHT - 1;
                let col = self.column_position;

                let color_code = self.color_code;
                self.buffer.chars[row][col] = ScreenChar {
                    ascii_character: byte,
                    color_code,
                };
                self.column_position += 1;
            }
        }
    }

    fn new_line(&mut self) {/* TODO */}
}
```

`\n`이 나오거나 현재 줄이 꽉차면 `new_line` 메서드를 호출해서 줄바꿈을 한다.

`write_string` 메서드는 문자열을 화면에 작성한다.

```rust
impl Writer {
    pub fn write_string(&mut self, s: &str) {
        for byte in s.bytes() {
            match byte {
                // printable ASCII byte or newline
                0x20..=0x7e | b'\n' => self.write_byte(byte),
                // not part of printable ASCII range
                _ => self.write_byte(0xfe),
            }

        }
    }
}
```

VGA 텍스트 버퍼는 아스키랑 코드 페이지 437만 지원하는 반면, 러스트는 UTF-8 문자열을 지원한다.

VGA 텍스트 버퍼가 지원하지 않는 문자는 `■`(헥스 코드로 `0xfe`)가 출력되게 한다.

## 출력 시도해보기

임시로 화면에 문자를 출력하는 함수를 만든다.

```rust
pub fn print_something() {
    let mut writer = Writer {
        column_position: 0,
        color_code: ColorCode::new(Color::Yellow, Color::Black),
        buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
    };

    writer.write_byte(b'H');
    writer.write_string("ello ");
    writer.write_string("Wörld!");
}
```

`buffer`에 `0xb8000` 주소의 가변 로우 포인터를 `*`로 역참조해서 가변 참조를 넘긴다.

컴파일러는 로우 포인터가 유효한지 모르므로 `unsafe`를 써야한다.

`_start` 함수에서 `print_something`을 호출하면 `Hello W■■rld!`가 출력된다.

```rust
#[no_mangle]
pub extern "C" fn _start() -> ! {
    vga_buffer::print_something();

    loop {}
}
```

## Volatile

VGA 버퍼 메모리를 나타내는 `Buffer`에 쓰기는 하지만 읽지는 하지 않는다.

러스트 컴파일러는 `Buffer`를 사용하지 않는 값으로 생각해서 최적화 시 없애버린다.

이런 무지막지한 최적화를 피하기 위해 메모리 쓰기를 `volatile`로 표시해야 한다.

`volatile` 라이브러리의 `Volatile` 래퍼 타입은 `read`와 `write` 메서드를 가지고 있다.

이 메서드 호출 시 코어 라이브러리의 `read_volatile`과 `write_volatile` 함수가 호출되므로 컴파일러 최적화로 인한 참조자 제거를 막을 수 있다.

`Cargo.toml`에 `volatile = "0.2.6"`를 추가하고 `ScreenChar`를 `Volatile`로 감싼다.

```rust
use volatile::Volatile;

struct Buffer {
    chars: [[Volatile<ScreenChar>; BUFFER_WIDTH]; BUFFER_HEIGHT],
}
```

버퍼에 쓸 때는 대입 연산자 `=` 대신에 다음과 같이 `write` 메서드를 사용한다.

```rust
self.buffer.chars[row][col].write(ScreenChar {
    ascii_character: byte,
    color_code: color_code,
});
```

## 포매팅 매크로

정수랑 부동소수점을 출력하기 위해 포매팅을 지원하면 좋다.

`Writer` 타입에 `core::fmt::Write` 트레잇을 구현하기만 하면된다.

```rust
use core::fmt;

impl fmt::Write for Writer {
    fn write_str(&mut self, s: &str) -> fmt::Result {
        self.write_string(s);
        Ok(())
    }
}
```

`write_str`에 들어오는 문자열 `s`를 출력하고 `Ok(())`를 반환한다.

이러면 다음처럼 `write!/writeln!` 포매팅 매크로를 사용할 수 있다.

```rust
write!(writer, "The numbers are {} and {}", 42, 1.0/3.0).unwrap();
```

`write!`는 `Result`를 반환하는데, 사용하지 않으면 경고가 뜨기도 하고 VGA 버퍼에 쓰는 게 실패할 일이 없으므로 마지막에 `unwarp`을 호출한다.

## 개행하기

문자가 한 줄에 다 안들가면 이미 쓴 문자를 모조리 윗줄로 옮긴 다음 다시 마지막 줄 처음부터 쓴다.

```rust
impl Writer {
    fn new_line(&mut self) {
        for row in 1..BUFFER_HEIGHT {
            for col in 0..BUFFER_WIDTH {
                let character = self.buffer.chars[row][col].read();
                self.buffer.chars[row - 1][col].write(character);
            }
        }
        self.clear_row(BUFFER_HEIGHT - 1);
        self.column_position = 0;
    }

    fn clear_row(&mut self, row: usize) {/* TODO */}
}
```

마지막 줄을 비우기 위해 `row`는 1 부터 시작한다.

`clear_row` 메서드는 해당 행에 공백 문자를 넣어서 비게 만든다.

```rust
impl Writer {
    fn clear_row(&mut self, row: usize) {
        let blank = ScreenChar {
            ascii_character: b' ',
            color_code: self.color_code,
        };
        for col in 0..BUFFER_WIDTH {
            self.buffer.chars[row][col].write(blank);
        }
    }
}
```

## 전역 인터페이스

다른 모듈에서 인스턴스를 만들지않고도 `Writer`를 사용할 수 있게 `WRITER` 전역 인터페이스를 만든다.

```rust
pub static WRITER: Writer = Writer {
    column_position: 0,
    color_code: ColorCode::new(Color::Yellow, Color::Black),
    buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
};
```

이대로 컴파일하면 엄청나게 많은 에러가 발생한다.

정적값(Statics)은 컴파일 타임에 초기화되지만, 일반 변수는 런타임에 초기화되므로 문제가 발생한다.

`ColorCode::new` 정도야 `const` 함수를 사용하면 해결되지만, 로우 포인터를 컴파일 타임에 참조하도록 할 수 없다.

## Lazy Statics

`lazy_static` 크레이트의 `lazy_static!` 매크로를 사용하면 값이 접근될 때 초기화되도록 지연할 수 있다.

`Cargo.toml`에 크레이트를 추가한다.

```
[dependencies.lazy_static]
version = "1.0"
features = ["spin_no_std"]
```

`spin_no_std` 옵션으로 표준 라이브러리 연결을 제거한다.

`WRITER`를 `lazy_static!`으로 감싸기만 하면 된다.

```rust
use lazy_static::lazy_static;

lazy_static! {
    pub static ref WRITER: Writer = Writer {
        column_position: 0,
        color_code: ColorCode::new(Color::Yellow, Color::Black),
        buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
    };
}
```

한편, 쓰기 메서드는 `&mut self`를 요구하는데 `WRITER`가 불변이라서 화면에 쓰는 것을 할 수 없다.

가변 정적 변수는 데이터를 레이스를 일으키므로 사용하지 말아야 한다.

`RefCell`이나 `UnsafeCell`은 내부 가변성을 지원하지만, `Sync`하지 않다.

## Spinlocks

동기적인 내부 가변성을 위해 표준 라이브러리의 `Mutex`를 사용할 수 있다.

하지만, 뮤텍스는 스레드를 막아서 상호 배제를 지원하는데, 커널에는 스레드가 없다.

운영체제 없이도 뮤텍스를 사용할 수 있는 Spinlock을 사용해야 한다.

Spinlock은 스레드를 막는게 아니라 뮤텍스가 풀릴 때까지 CPU 시간을 잡아먹기 위해 루프를 계속 돌리서 락을 얻는다.

Cargo.toml에 `spin = "0.5.2"`를 추가하고 `Writer`를 `Mutext::new`로 감싼다.

```rust
use spin::Mutex;
...
lazy_static! {
    pub static ref WRITER: Mutex<Writer> = Mutex::new(Writer {
        column_position: 0,
        color_code: ColorCode::new(Color::Yellow, Color::Black),
        buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
    });
}
```

## 안전성

`0xb8000`를 가르키는 `Buffer` 참조자를 만드는 unsafe 블록 하나를 제외하고 나머지 모든 연산이 안전하다.

요구사항을 타입 시스템으로 인코딩했으므로 외부에 안전한 인터페이스를 제공할 수 있다.

## `println` 매크로

```rust
#[macro_export]
macro_rules! print {
    ($($arg:tt)*) => ($crate::vga_buffer::_print(format_args!($($arg)*)));
}

#[macro_export]
macro_rules! println {
    () => ($crate::print!("\n"));
    ($($arg:tt)*) => ($crate::print!("{}\n", format_args!($($arg)*)));
}

#[doc(hidden)]
pub fn _print(args: fmt::Arguments) {
    use core::fmt::Write;
    WRITER.lock().write_fmt(args).unwrap();
}
```

`#[macro_export]` 속성은 매크로를 루트 크레이트에 위치시켜서 크레이트 어디서든 매크로를 사용할 수 있게한다.

`use crate::vga_buffer::println` 대신에 `use crate::println`로 매크로를 가져올 수 있다.

`println` 매크로 정의 시 `print` 매크로 앞에 `$crate`를 붙여서 `println`을 사용할 때 `print`를 가져오지 않아도 되게 했다.

`doc(hidden)` 속성으로 구현을 비공개로 만들어서 문서 생성 시 생략되게 할 수 있다.

## `println`로 Hello Wolrd 출력하기

```rust
#[no_mangle]
pub extern "C" fn _start() {
    println!("Hello World{}", "!");

    loop {}
}
```

메인 함수에 매크로를 가져오지 않아도 매크로는 이미 루트 네임스페이스에 들어있다.

## 정리

- VGA 텍스르 구조
- 안전하지 않은 모듈을 감싸서 안전한 인터페이스로 만들기
- `lazy_static`, `spin` 크레이트
  를 다뤘다.
