# Find and Fix MEMORY LEAKS with Leak Canary in Android π

μλλ‘μ΄λ κ°λ° μ μΌμ΄λλ λ©λͺ¨λ¦¬ λ¦­μ λν΄ μμλ³΄μ.

μλ°μλ λ°±κ·ΈλΌμ΄λλ‘ λμνλ κ°λΉμ§ μ»¬λ ν°κ° μ‘΄μ¬νλ€. κ°λΉμ§ μ»¬λ ν°λ μ¬μ©νμ§ μλ μ€λΈμ νΈλ λ³μλ€μ λͺ¨μ λ©λͺ¨λ¦¬μμ free ν΄μ£Όλ μμμ μννλ€. c λλ c++μμ λ©λͺ¨λ¦¬ μ¬μ© ν μλμΌλ‘ freeλ₯Ό
ν΄μ€μΌ νμ§λ§ μλ°λ κ°λΉμ§ μ»¬λ ν°κ° μλμΌλ‘ κ·Έ μ­ν μ μννλ€.

νμ§λ§ μ±μμ destroy λμλλ°λ μ λ¦¬λμ§ μλ νΉμ  μ€λΈμ νΈκ° μλ€. μ΄λ κ² λλ©΄ λ©λͺ¨λ¦¬λ₯΄ μ μ νκ³  μκΈ° λλ¬Έμ μ’μ§ μλ€.

λ€μκ³Ό κ°μ΄ `companion object`μ `Context`λ₯Ό λ³μλ‘ μ μ₯νλ κ²½μ° μλμ΄ λ°μνλ€. μ΄λ κ² κ΅¬ννλ©΄ μ‘ν°λΉν°κ° clear λλ `context` λ³μκ° clear λμ§ μλλ€.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }

    companion object {
        lateinit var context: Context
    }
}
```

`SecondActivity`μμ `MainActivity.context`μ `this`λ‘ ν λΉλ ν `SecondActivity`κ° destroy λλ κ²½μ° μ΄ μ‘ν°λΉν°μ `context`
λ `MainActivity.context`μ μ¬μ ν λ¨μμκ² λλ€. λ°λΌμ κ°λΉμ§ μ»¬λ ν°λ `context`κ° μ¬μ ν μ¬μ©λλ μ€ μκ³  clear νμ§ μλλ€.

```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        MainActivity.context = this
    }
}
```

μ΄μ  `MainActivity`μμ `SecondActivity`λ₯Ό μμνλλ‘ μμ±ν΄λ³΄μ.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Intent(this, SecondActivity::class.java).also {
            startActivity(it)
        }
    }

    companion object {
        lateinit var context: Context
    }
}
```

μ€ν ν `SecondActivity`μμ λ€λ‘κ°κΈ°λ₯Ό `SecondActivity`λ destory λμμ§λ§ `context`κ° `MainActivity.context`μ λ¨μμμ΄ λλ₯΄λ©΄ λ©λͺ¨λ¦¬ λ¦­μ΄ λ°μνλ€.

νμ§λ§ μ§κΈ μνμμ  μ΄ λ©λͺ¨λ¦¬ λ¦­μ νμ§νλ λ°©λ²μ΄ μλ€. νμ§λ§ **leakcanary**λΌλ λΌμ΄λΈλ¬λ¦¬κ° μ‘΄μ¬νλ©° νλ‘μ νΈμ ν¬ν¨νμ¬ μλμΌλ‘ λ©λͺ¨λ¦¬ λ¦­μ νμ§ν  μ μλ€.

```groovy
debugImplementation "com.squareup.leakcanary:leakcanary-android:2.8.1"
```

<div align="center">
<img src="img/result.gif" width="40%">
</div>

## References

* [Find and Fix MEMORY LEAKS with Leak Canary in Android π](https://www.youtube.com/watch?v=VvkRe9vP5Oc)