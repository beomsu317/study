# First Unit Test

## Dependencies

다음과 같이 디펜던시를 추가한다.

```groovy
testImplementation 'junit:junit:4.13.2'  
androidTestImplementation 'androidx.test.ext:junit:1.1.3'
androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'

testImplementation 'com.google.truth:truth:1.0.1'
androidTestImplementation 'com.google.truth:truth:1.0.1'
```

## Registration Implementation

`RegistrationUtil` object를 하나 생성한 후 다음과 같이 `username`과 `password`/`confirmedPassword`를 파라미터로 입력받아 검증하는 함수를 생성한다.

```kotlin
object RegistrationUtil {

    private val existingUsers = listOf("Peter", "Carl")
    /**
     * the input is not valid if...
     * ...the username/password is empty
     * ...the username is already take
     * ...the confirmed password is not the same as the real password
     * ...the password contains less than 2 digits
     */
    fun validateRegistrationInput(
        username: String,
        password: String,
        confirmedPassword: String
    ): Boolean {
        return true
    }
}
```

## Unit Test Implementation

다음과 같이 `username`이 비어있을 경우에 대한 테스트 케이스를 만들고, `assertThat`을 사용해 false를 반환하는지 검증한다. Truth 라이브러리의 `assertThat`을 사용하면 더 직관적으로 코드를 작성할 수 있다.

```kotlin
class RegistrationUtilTest {

    @Test
    fun `empty username returns false`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "",
            "123",
            "123"
        )
        // Truth library
        assertThat(result).isFalse()
    }
}
```

결과는 다음과 같이 false로 예상되지만 true를 반환하여 테스트가 실패하였다. 이는 `RegistrationUtil`에서 구현되지 않았으므로 당연한 결과이며, 여러 테스트 케이스 작성 후 구현할 것이다.

```
expected to be false
	at com.example.testing.RegistrationUtilTest.empty username returns false(RegistrationUtilTest.kt:16)
```

필요한 테스트 케이스를 생성한다.

```kotlin

class RegistrationUtilTest {

    @Test
    fun `empty username returns false`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "",
            "123",
            "123"
        )
        assertThat(result).isFalse()
    }

    @Test
    fun `valid username and correctly repeated password returns true`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "Philipp",
            "123",
            "123"
        )
        assertThat(result).isTrue()
    }

    @Test
    fun `username already exists returns false`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "Carl",
            "123",
            "123"
        )
        assertThat(result).isFalse()
    }

    @Test
    fun `incorrectly confirmed password returns false`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "Philipp",
            "1555",
            "123"
        )
        assertThat(result).isFalse()
    }

    @Test
    fun `empty password returns false`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "Philipp",
            "",
            ""
        )
        assertThat(result).isFalse()
    }

    @Test
    fun `less than 2 digits password returns false`() {
        val result = RegistrationUtil.validateRegistrationInput(
            "Philipp",
            "1sadasd",
            "1sadasd"
        )
        assertThat(result).isFalse()
    }
}
```

## Actual Function Implementation

이제 `RegistrationUtil.validateRegistrationInput`을 구현해보자.

```kotlin
object RegistrationUtil {

    private val existingUsers = listOf("Peter", "Carl")
    /**
     * the input is not valid if...
     * ...the username/password is empty
     * ...the username is already take
     * ...the confirmed password is not the same as the real password
     * ...the password contains less than 2 digits
     */
    fun validateRegistrationInput(
        username: String,
        password: String,
        confirmedPassword: String
    ): Boolean {
        if (username.isEmpty() || password.isEmpty()) {
            return false
        }
        if (username in existingUsers) {
            return false
        }
        if (password != confirmedPassword) {
            return false
        }
        if (password.count { it.isDigit() } < 2) {
            return false
        }
        return true
    }
}
```

## References

* [Writing Our First Unit Tests - Testing on Android - Part 3](https://www.youtube.com/watch?v=W0ag98EDhGc&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=3)