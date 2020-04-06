# 크기가 작은 러스트 커널

x86 아키텍처에서 돌아가는 작은 크기의 64비트 러스트 커널을 만들어본다.

## 부팅 과정

컴퓨터를 켜면, 메인보드 ROM에 저장되어있는 펌웨어 코드가 실행된다.

이 코드는 램 사용량을 확인하고 CPU와 하드웨어를 초기화하는 시동 자체 시험(power-on self-test)을 수행한다.

그다음에 부팅 가능한 디스크를 찾아서 운영체제 커널을 부팅한다.

x86에는 "Basic Input/Output System"(BIOS)와 최신식인 "Unified Extensible Firmware Interface"(UEFI)라는 두 가지 펌웨어 표준이 있다.

BIOS는 오래됐어도 단순하고 x86 기기에서 지원도 잘 된다. 반면에, UEFI는 현대적이고 기능도 많지만, 설정이 복잡해서 BIOS 부팅을 사용한다.

## BIOS 부팅

대부분의 x86 시스템은 BIOS 부팅을 지원하고, 최신식 UEFI 기반 기기도 BIOS를 예뮬레이트해서 사용할 수 있다.

BIOS 부팅의 폭넓은 호환성은 장점이지만 CPU가 16비트 호환성 모드인 Real Mode에 들어간다는 것이 단점이다.

## A Minimal Kernel

## Rust Nightly 설치하기

`rustup override add nightly` 명령어로 현재 디렉터리의 컴파일러를 설정할 수 있다.

나이틀리 컴파일러를 써야 실험적 기능들을 끄고 켤 수 있다.

## 컴파일 타깃 명시하기

`x86_64-blog_os.json`에 다음과 같은 타깃 설정 파일을 작성한다.

```rust
{
  "llvm-target": "x86_64-unknown-none",
  "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
  "arch": "x86_64",
  "target-endian": "little",
  "target-pointer-width": "64",
  "target-c-int-width": "32",
  "os": "none",
  "executables": true,
  "linker-flavor": "ld.lld",
  "linker": "rust-lld",
  "panic-strategy": "abort",
  "disable-redzone": true,
  "features": "-mmx,-sse,+soft-float"
}
```

베어 메탈에서 동작하기에 `llvm-target`과 `os`는 `none`으로 설정했다.

```json
"panic-strategy": "abort",
```

스택 되감기를 하지 않겠다는 설정이다.

```json
"disable-redzone": true,
```

"레드 존"이라는 특정 스택 포인터 최적화 기능을 꺼서 인터럽트 발생 시 스택 오염을 막는다.

```json
"features": "-mmx,-sse,+soft-float",
```

`mmx`와 `sse` 앞에 `-`를 붙여서 끄고, `soft-float` 앞에는 `+`를 붙여서 켰다.

`mmx`와 `sse`는 Single Instruction Multiple Data (SIMD) 기능 사용 여부를 결정한다.

켜놓으면 성능 향상이 크지만, 인터럽트 처리 후 전에 동작하던 프로그램의 레지스터를 모두 복구해야하는데, SIMD 레지스터는 용량이 너무 커서(512-1600바이트) 인터럽트마다 오버헤드가 크다.

`x86_64`의 부동소수점 연산은 SIMD이 기본 요구사항이므로, 부동소수점 연산을 정수로 에뮬레이트하는 `soft-float` 기능을 사용한다.

## 커널 빌드하기

`cargo build --target x86_64-blog_os.json` 명령어로 커널을 빌드하면 `` error[E0463]: can't find crate for `core` `` 에러가 발생한다.

`no_std` 속성으로 코어 라이브러리도 같이 날려버려서 생긴 에러다.

코어 라이브러리는 미리 컴파일된 상태로 러스트 컴파일러와 같이 배포되는데, 이 라이브러리가 현재 호스트 시스템에만 사용가능하고 우리가 만드려는 타깃에는 사용할 수 없다.

그래서 원하는 타깃에 맞는 코어 라이브러리를 직접 다시 컴파일해서 써야 한다.

## Cargo xbuild

`cargo xbuild`는 `cargo build`를 감싸서 코어는 물론 다른 라이브러리까지 같이 자동으로 크로스 컴파일해준다.

`cargo install cargo-xbuild`

`cargo xbuild`는 `core`, `compiler_builtin`, `alloc` 라이브러리를 크로스 컴파일하는데, 이 라이브러리들은 내부적으로 불안정한 기능을 사용하므로 나이틀리 러스트가 필요하다.

## 기본 타깃 설정

`.cargo/config`에 다음과 같이 작성하면 `cargo xbuild` 시 `--target x86_64-blog_os.json` 인자를 붙여준다

```
[build]
target = "x86_64-blog_os.json"
```

## 화면에 출력하기

VGA 텍스트 버퍼를 사용해서 화면에 텍스트 출력을 할 수 있다.

VGA 텍스트 버퍼는 화면에 표시된 내용을 포함하는 VGA 장치에 매핑된 특수 메모리 영역이다.

다음은 버퍼의 주소인 `0xb8000`에 `Hello World!`를 출력하는 코드이다.

```rust
static HELLO: &[u8] = b"Hello World!";

#[no_mangle]
pub extern "C" fn _start() -> ! {
    let vga_buffer = 0xb8000 as *mut u8;

    for (i, &byte) in HELLO.iter().enumerate() {
        unsafe {
            *vga_buffer.offset(i as isize * 2) = byte;
            *vga_buffer.offset(i as isize * 2 + 1) = 0xb;
        }
    }

    loop {}
}
```

코드에서 메모리에 직접 접근해서 값을 설정했는데, 이런 식으로 하면 코드가 엉망이 되기 쉽다.

나중에 `unsafe`를 사용하는 메모리 직접 접근 부분만 추상화해서, 안전하게 VGA 버퍼를 사용하도록 할 것이다.

## 커널 실행하기

디스크 이미지를 QEMU 가상머신이나 USB를 이용해 실제 하드웨어에 실행해볼 수 있다.

## 부트 이미지 만들기

컴파일된 커널을 부트 디스크 이미지로 만들기 위해서는 부트로더와 연결해야 한다.

참고로, 부트로더는 CPU를 초기화하고 커널을 불러온다.

부트로더를 직접 작성하지 않고 `bootloader` 크레이트를 사용한다.

이 크레이트는 C 디펜던시 없는 BIOS 부트로더를 구현했다.

Cargo.toml에 다음 코드를 추가한다.

```
[dependencies]
bootloader = "0.8.0"
```

부트로더만으로는 작동가능한 부팅 디스크 이미지를 만들 수는 없고, 컴파일 이후에 작성한 커널과 부트로더를 연결해야 한다.

Cargo에는 포스트 빌드 스크립트가 없으므로 `bootimage` 툴을 사용한다.

추가로 `llvm-tools-preview` 컴포넌트도 사용한다.

```
cargo install bootimage --version "^0.7.7"
rustup component add llvm-tools-preview
```

이후엔 `cargo bootimage` 명령어 만으로 부팅 가능한 디스크 이미지를 만들 수 있다

이 툴은 커널 코드를 `cargo xbuild`로 재컴파일 하고 부트로더를 컴파일 한 뒤, 둘을 합쳐서 부팅 가능한 디스크 이미지를 만든다.

처음 툴 실행 시 모두 컴파일 하느라 느리고 다음에 빌드할 때는 빠르다.

툴 명령어 실행 뒤 `target/x86_64-blog_os/debug` 디렉터리에 `bootimage-blog_os.bin` 파일이 생성된다.

## QEMU에서 부팅하기

QEMU 설치 후 QEMU가 설치된 `C:\Program Files\qemu` 디렉터리를 환경 변수에 지정한다.

`bootimage-blog_os.bin`이 있는 디렉터리로 가서 다음 명령어를 실행한다.

```
qemu-system-x86_64 -drive format=raw,file=bootimage-blog_os.bin
```

그러면 다음과 같은 감격스러운 화면이 나온다.

![qemu-hello-world]](https://user-images.githubusercontent.com/22253556/78552910-51477180-7843-11ea-844f-5f70a2a57e03.png)
