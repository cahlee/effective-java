## 아이템6. 불필요한 객체 생성을 피하라
### 불변 객체는 재사용 하자
``` java
String s1 = new String("bikini");  // bad
String s2 = "bikini" // good
```

### 불변 클래스에서는 정적 팩터리 메서드를 사용하자
``` java
Boolean bool1 = new Boolean("true");  // bad
Boolean bool2 = Boolean.valueOf("true");  // good
```

### 생성 비용이 비싼 객체는 캐싱하자
``` java
// bad
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
// good
private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
```

### 오토박싱을 조심하자
``` java
Long sum = 0L;  // 기본 타입인 long을 사용해야 함
for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;  // 오토박싱
return sum;
```
박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자

### 참고
- 방어적 복사[아이템 50]
