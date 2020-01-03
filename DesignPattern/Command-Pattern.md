# 커맨드 패턴

**실행될 기능을 캡슐화**함으로써 기능의 실행을 요구하는 호출자(Invoker) 클래스와 실제 기능을 실행하는 수신자(Receiver) 클래스 사이의 의존성을 제거한다.

어떤 객체가 다른 객체의 메서드를 호출하려면 그 객체를 참조하는 의존성이 생기는데 커맨드 패턴으로 이 의존성을 없앨 수 있다.

기능 변경 시 기능 클래스만 변경하면 되므로 수정과 확장에 유연하다.

작업취소 기능이나 로그 & 복구 기능을 구현하기 쉽다.

## 클래스 다이어그램

![command-pattern-diagram](https://user-images.githubusercontent.com/22253556/71721572-ee6fc400-2e68-11ea-8cef-b06ca740a7c7.png)

`Command`는 모든 커맨드 객체에서 구현해야하는 인터페이스이다.

클라이언트가 `ConcreteCommand`를 생성하고 `Receiver`를 설정한다.

호출자(`Invoker`)에는 명령이 들어있어, `execute()`를 호출해 커맨드 객체에게 작업 수행 요청을 한다.

`ConcreteCommand`의 `execute()` 메서드는 수신자(`Receiver`)에 있는 메서드를 호출하여 요청된 작업을 수행한다.

## 구현

![watchman](https://search3.kakaocdn.net/argon/600x0_65_wr/Hw16btZqj4A)

당직사관이 전파한다. 전 병사들은 생활관 청소와 침구류 일광건조를 실시할 것

### 커맨드 인터페이스

모든 커맨드 객체가 구현해야 하는 인터페이스다.

```typescript
interface Command {
  execute(): void;
}
```

### 요청을 받은 내용을 처리할 수신자(Receiver)

청소와 일광건조를 하는 병사

```typescript
class Soldier {
  cleaning() {
    console.log("생활관 대청소");
  }

  sunDrying() {
    console.log("침구류 일광건조");
  }
}
```

### 명령을 위한 커맨드 클래스 구현

청소를 시키는 명령

```typescript
class CleaningCommand implements Command {
  soldier: Soldier;

  constructor(soldier: Soldier) {
    this.soldier = soldier;
  }

  execute() {
    this.soldier.cleaning();
  }
}
```

일광건조를 시키는 명령

```typescript
class SunDryingCommand implements Command {
  soldier: Soldier;

  constructor(soldier: Soldier) {
    this.soldier = soldier;
  }

  execute() {
    this.soldier.sunDrying();
  }
}
```

### 커맨드 객체를 사용하는 호출자(Invoker)

당직사관이 명령을 모아서 전파한다.

```typescript
class Watchman {
  commands: Command[] = [];

  addCommand(command: Command) {
    this.commands.push(command);
  }

  propagateCommands() {
    this.commands.forEach(command => command.execute());
  }
}
```

## 테스트

아래와 같이 사용하는 것이 커맨드 패턴에서 클라이언트에 해당한다.

호출자(당직사관)와 수신자(병사) 사이에 커맨드 객체(청소, 일광건조 명령)가 매개가 된다.

```typescript
// 호출자
const watchman = new Watchman();

// 수신자
const soldier1 = new Soldier();
// 커맨드 객체
const cleaningCommand = new CleaningCommand(soldier1);

// 또 다른 수신자와 커맨드 객체
const soldier2 = new Soldier();
const sunDryingCommand = new SunDryingCommand(soldier2);

// 커맨드 객체를 호출자에 전달
watchman.addCommand(cleaningCommand);
watchman.addCommand(sunDryingCommand);

// 명령 실행
watchman.propagateCommands();
```

호출자는 어떤 명령이 와도 명령을 요청할 수 있으므로 명령을 실행하는 수신자는 누구든지 상관없어진다. 훈련된 누구든 명령을 할 수 있게 된다.

만약 커맨드 패턴을 사용하지 않았다면 호출자 내부에 수신자의 참조가 있을 거고, 수신자의 기능을 변경할 때마다 호출자의 코드도 변경해야 한다.

## 작업취소(Undo) 기능 추가하기

`Command` 인터페이스에 `undo()` 메서드를 추가한다.

```typescript
interface Command {
  execute(): void;
  undo(): void;
}
```

`Command` 클래스에서 `undo()` 메서드를 구현한다.

명령의 반대 행동만 하면 된다.

```typescript
class CleaningCommand implements Command {
  soldier: Soldier;

  constructor(soldier: Soldier) {
    this.soldier = soldier;
  }

  execute() {
    this.soldier.cleaning();
  }

  undo() {
    this.soldier.uncleaning();
  }
}
```

만약 했거나 안 했거나로 나눠지는 작업이 아닌 단계가 있는 작업이라면 그 전 상태를 기억하는 변수를 두고, 실행취소 시 그 변수를 불러온다.
