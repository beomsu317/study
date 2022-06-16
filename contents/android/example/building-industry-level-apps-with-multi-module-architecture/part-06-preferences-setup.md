# Preferences Setup

키, 몸무게 등 사용자의 정보를 저장하기 위해 Shared Preferences를 사용하며, 이는 전체 모듈에서 사용되므로`core` 모듈에 작성한다.

지금의 `core`에는 많은 `data`, `domain`의 많은 기능이 들어가 있지 않아 `core` 모듈만 생성해놓은 상태이다. 만약 앱이 커지게 되면 `core-data`, `core-domain`으로 나누어야
한다.

`core` 모듈에 `domain/model` 패키지를 생성하고 하위에 `Gender`, `GoalType`, `ActivityLevel` sealed class를 생성한다.

```kotlin
sealed class Gender(val name: String) {
    object Male : Gender("male")
    object Female : Gender("female")

    companion object {
        fun fromString(name: String): Gender {
            return when (name) {
                "male" -> Male
                "female" -> Female
                else -> Female
            }
        }
    }
}
```

```kotlin
sealed class GoalType(val name: String) {
    object LoseWeight : GoalType("lose_weight")
    object KeepWeight : GoalType("keep_weight")
    object GainWeight : GoalType("gain_weight")

    companion object {
        fun fromString(name: String): GoalType {
            return when (name) {
                "lose_weight" -> LoseWeight
                "keep_weight" -> KeepWeight
                "gain_weight" -> GainWeight
                else -> KeepWeight
            }
        }
    }
```

```kotlin
sealed class ActivityLevel(val name: String) {
    object Low : ActivityLevel("low")
    object Medium : ActivityLevel("medium")
    object High : ActivityLevel("high")

    companion object {
        fun fromString(name: String): ActivityLevel {
            return when (name) {
                "low" -> Low
                "medium" -> Medium
                "high" -> High
                else -> Medium
            }
        }
    }
}
```

그 후 `model` 패키지 하위에 사용자 정보를 저장할 수 있는 `UserInfo` data class를 생성한다.

```kotlin
data class UserInfo(
        val gender: Gender,
        val age: Int,
        val weight: Float,
        val height: Int,
        val activityLevel: ActivityLevel,
        val goalType: GoalType,
        val carbRatio: Float,
        val proteinRatio: Float,
        val fatRatio: Float
)
```

그 후 `core` 모듈 `domain` 패키지 하위에 `preference` 패키지를 생성한다.

```kotlin
interface Preferences {

    fun saveGender(gender: Gender)
    fun saveAge(age: Int)
    fun saveWeight(weight: Float)
    fun saveHeight(height: Int)
    fun saveActivityLevel(level: ActivityLevel)
    fun saveGoalType(type: GoalType)
    fun saveCarbRatio(ratio: Float)
    fun saveProteinRatio(ratio: Float)
    fun saveFatRatio(ratio: Float)

    fun loadUserInfo(): UserInfo

    companion object {
        const val KEY_GENDER = "gender"
        const val KEY_AGE = "age"
        const val KEY_WEIGHT = "weight"
        const val KEY_HEIGHT = "height"
        const val KEY_ACTIVITY_LEVEL = "activity_level"
        const val KEY_GoalType = "goal_type"
        const val KEY_CARB_RATIO = "carb_ratio"
        const val KEY_PROTEIN_RATIO = "protein_ratio"
        const val KEY_FAT = "fat_ratio"
    }

}
```

그 다음 `core` 모듈에 `data/preferences` 패키지를 생성한 후 `Preferences` 구현체를 구현한다. 만약 Shared Preferences가 아니라 DB 또는 원격 저장소에 저장하고 싶은
경우 `Preferences`를 사용하는 모든 곳에서 `Preferences` 인터페이스만 알고 있기 때문에 이 구현체만 변경하여 구현해주면 된다.

```kotlin
class DefaultPreferences(
        private val sharedPref: SharedPreferences
) : Preferences {
    override fun saveGender(gender: Gender) {
        sharedPref.edit()
                .putString(Preferences.KEY_GENDER, gender.name)
                .apply()
    }

    override fun saveAge(age: Int) {
        sharedPref.edit()
                .putInt(Preferences.KEY_AGE, age)
                .apply()
    }

    override fun saveWeight(weight: Float) {
        sharedPref.edit()
                .putFloat(Preferences.KEY_WEIGHT, weight)
                .apply()
    }

    override fun saveHeight(height: Int) {
        sharedPref.edit()
                .putInt(Preferences.KEY_HEIGHT, height)
                .apply()
    }

    override fun saveActivityLevel(level: ActivityLevel) {
        sharedPref.edit()
                .putString(Preferences.KEY_ACTIVITY_LEVEL, level.name)
                .apply()
    }

    override fun saveGoalType(type: GoalType) {
        sharedPref.edit()
                .putString(Preferences.KEY_GoalType, type.name)
                .apply()
    }

    override fun saveCarbRatio(ratio: Float) {
        sharedPref.edit()
                .putFloat(Preferences.KEY_CARB_RATIO, ratio)
                .apply()
    }

    override fun saveProteinRatio(ratio: Float) {
        sharedPref.edit()
                .putFloat(Preferences.KEY_PROTEIN_RATIO, ratio)
                .apply()
    }

    override fun saveFatRatio(ratio: Float) {
        sharedPref.edit()
                .putFloat(Preferences.KEY_FAT_RATIO, ratio)
                .apply()
    }

    override fun loadUserInfo(): UserInfo {
        val age = sharedPref.getInt(Preferences.KEY_AGE, -1)
        val height = sharedPref.getInt(Preferences.KEY_HEIGHT, -1)
        val weight = sharedPref.getFloat(Preferences.KEY_WEIGHT, -1f)
        val genderString = sharedPref.getString(Preferences.KEY_GENDER, null)
        val activityLevelString = sharedPref.getString(Preferences.KEY_ACTIVITY_LEVEL, null)
        val goalType = sharedPref.getString(Preferences.KEY_GoalType, null)
        val carbRatio = sharedPref.getFloat(Preferences.KEY_CARB_RATIO, -1f)
        val proteinRatio = sharedPref.getFloat(Preferences.KEY_PROTEIN_RATIO, -1f)
        val fatRatio = sharedPref.getFloat(Preferences.KEY_FAT_RATIO, -1f)

        return UserInfo(
                gender = Gender.fromString(genderString ?: "male"),
                age = age,
                weight = weight,
                height = height,
                activityLevel = ActivityLevel.fromString(activityLevelString ?: "medium"),
                goalType = GoalType.fromString(goalType ?: "keep_weight"),
                carbRatio = carbRatio,
                proteinRatio = proteinRatio,
                fatRatio = fatRatio
        )
    }
}
```

이제 `Preferences`를 Dependency Injection을 통해 injection 해보자.

모든 모듈들을 컴파일 타임에 알아야 하기 때문에 hilt는 모놀리틱이다.

Dagger-Hilt는 dynamic feature module을 지 원하지 않기 때문에 Dagger2를 사용해야 한다. dynamic feature module은 특정 모듈의 scope에서만 동작하는 기능이다.
우리는 dynamic feature module을 사용하지 않으므로 Dagger-Hilt를 사용한다.

`app` 모듈에 `di` 패키지를 만들고 하위에 `AppModule`을 작성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideSharedPreferences(
            app: Application
    ): SharedPreferences {
        return app.getSharedPreferences("shared_pref", MODE_PRIVATE)
    }

    @Provides
    @Singleton
    fun providePreferences(sharedPreferences: SharedPreferences): Preferences {
        return DefaultPreferences(sharedPreferences)
    }
} 
```

`app` 모듈에 `Application()`를 상속하는 `CaloryTrackerApp` 클래스를 생성한 후 `@HiltAndroidApp` 어노테이션을 달아준다.

```kotlin
// Hilt의 entry point
@HiltAndroidApp
class CaloryTrackerApp : Application()
```

그 후 `AndroidManifest.xml`의 `application` 태그에 `android:name=".CaloryTrackerApp"`을 추가해준다.

```xml
        <!-- ... -->
<application
        android:name=".CaloryTrackerApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher">
        <!-- ... -->
</application>
        <!-- ... -->
```

실행 후 크래시 없이 정상적으로 동작하는지 확인한다.

<div align="center">
<img src="img/part-06/result.png" width="40%">
</div>
