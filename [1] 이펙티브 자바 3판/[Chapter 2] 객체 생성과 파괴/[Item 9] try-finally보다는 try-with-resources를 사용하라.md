---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 9] try-finally보다는 try-with-resources를 사용하라'
created: '2022-01-04T10:02:50.279Z'
modified: '2022-01-07T12:48:56.018Z'
---

# [Item 9] try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 `InputStream`, `OutputStream`, `java.sql.Connection` 등 `close()` 메서드를 이용해 닫아야 하는 자원이 많다. 그러나 자원 닫기는 예측할 수 없는 예외가 발생하면 클라이언트가 놓치기 쉽다. 이런 자원 중 대부분이 [finalizer]()를 사용하지만 믿을만하지 못하다.

## 자원을 닫기 위해 try-finally를 사용하는 경우

아래 코드에서는 자원이 제대로 닫히는 것을 보장하는 수단으로 항상 실행됨을 보장하는 try-finally를 사용했다. 그러나 예외는 try문과 finally 문에서 모두 발생할 수 있다.

만약 기기에 물리적인 문제가 생긴다면 `readLine` 메서드에 문제가 생겨 예외를 던지고 `close()`도 실패할 것이다. 이 경우 스택 추적 내역은 두번째 예외만을 기록할 것이고 디버깅을 어렵게 한다. 

```java
// 자원을 한 개 사용하는 경우
static String firstLineOfFile(String path) throws IOException {
  BufferReader bt = new BufferReader(new FileReader(path));
  try {
    return br.readLine();
  } finally {
    br.close();
  }
}

```
```java
// 자원을 두 개 사용하는 경우
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStrieam(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    } finally {
      cout.close();
    }
  }
}
```

## 자원을 닫기 위해 try-with-resources를 사용하는 경우

자바 7은 리소스를 닫아야만 하는 자원은 `AutoCloseable` 인터페이스를 사용하게끔 했다. 아래 코드처럼 `AutoClosable`을 구현한 리소스에 try-with-resources를 사용하면 길이가 짧아져서 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 쉽다.

이전 상황과 달리, 만약 `close()` 호출 양쪽에서 예외가 발생하면, `close()`에서 발생한 예외는 숨겨지고 `readLine`에서 발생한 예외만 기록된다.

try-finally에서처럼 try-with-resources에도 catch절을 쓸 수 있으므로 try문을 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path) throws IOException {
  try (BufferReader br = new BufferedReader(
    new FileReader(path))) {
      return br.readLine();
    }
}
```
```java
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
      OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
          out.write(buf, 0, n);
      }
}
```
