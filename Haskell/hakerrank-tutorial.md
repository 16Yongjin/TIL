# HakerRank Haskcell

## IO 모나드로 입력 받기

```haskell
solveMeFirst a b = a + b

main = do
    val1 <- readLn
    val2 <- readLn
    let sum = solveMeFirst val1 val2
    print sum
```

## 따옴표 없이 문자열 출력하기

```haskell
putStrLn "Hello World"
```

## n번 출력하기

`replicate n str`로 문자열 n개가 든 리스트를 만들고 각 문자열을 '\n'으로 연결한 것을 출력한다.

```haskell
hello_worlds n = putStrLn $ unlines (replicate n "Hello World")
```

## n 보다 작은 요소 필터링하기

```haskell
f :: Int -> [Int] -> [Int]
f n arr = [x | x <- arr, x < n]
```

## 인덱스가 홀수인 요소 필터링하기

입력 리스트와 인덱스를 나타낼 `[1..리스트길이]` 둘을 `zip`하고 인덱스 리스트가 짝수인 요소만 리스트에 넣는다.

```haskell
f :: [Int] -> [Int]
f lst = [v | (v, idx) <- zip lst [1..], even idx]
```

## 리스트 뒤집기

리스트를 분해해서 해드 요소를 뒤에 연결한다. 테일 요소들은 재귀로 해드 요소처럼 뒤에 연결되게 한다.

```haskell
rev :: [Int] -> [Int]
rev (x:xs) = (rev xs) ++ [x]
rev [] = []
```

## 리스트 길이 세기

```haskell
len :: [a] -> Int
len (x:xs) = 1 + len xs
len [] = 0
```
