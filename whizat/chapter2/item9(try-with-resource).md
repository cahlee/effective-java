## 아이템9: try-finally보다는 try-with-resource를 사용하라
### try-finally
``` java
InputStream in = new FileInputStream(src);
try {
    OutputStream out = new FileOutputStream(dst);
    try {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    } finally {
        out.close();
    }
} finally {
    in.close();
}
```
- 지저분하다.
- 물리적 에러(하드웨적인 에러인가?)가 발생하면 out.write에서 예외를 던지고 같은 이유로 out.close도 in.close도 실패한다.
  - 첫번째 예외에 관한 정보가 남지 않아서 디버깅을 어렵게 한다.
### try-with-resource
``` java
try (InputStream   in = new FileInputStream(src);
     OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
} catch (IOException e) {
    System.out.println("catch 절도 사용 가능");
}
```
- 짧다
- 여전히 catch절 사용 가능
- 첫번째 예외에 관한 정보가 출력된다
  - 다른 예외에 관한 정보는 숨겨졌다(suppressed) 꼬리표를 달고 출력된다.
  - Throwale에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드에서 가져올 수 있다.
``` java
static void testSuppressedException() {
    try (TestAutoCloseable testAutoCloseable = new TestAutoCloseable()) {
      throw new ArrayIndexOutOfBoundsException("ArrayIndexOutOfBoundsException가 주에러");
    } catch (Throwable e) {
      Throwable[] suppressedException = e.getSuppressed();
      for (Throwable exception : suppressedException) {
        System.out.println("여기에 suppressed exception 출력");
        System.out.println(exception);
      }
    }
}
static class TestAutoCloseable implements AutoCloseable {
  @Override
  public void close() throws Exception {
    throw new NullPointerException("AutoCloseable close 할 때 NullPointerException 발생");
  }
}
```
