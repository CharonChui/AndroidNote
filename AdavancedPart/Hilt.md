## Hilt

Hilt是Google基于Dagger2封装的Android专用依赖注入框架


- Dagger2的痛点: 需要手动定义Component、SubComponent，模板代码多且复杂
- Hilt的价值: 预定义了与Android生命周期绑定的标准Component，开箱即用。 

### 核心原理: 编译期代码生成

Hilt的本质是注解处理器，在编译阶段扫描注解，自动生成Dagger的模板代码。 

```kotlin
// 你写的代码
class UserRepository @Inject constructor(
    private val api: ApiService
)

// Hilt 编译后自动生成（伪代码，实际在 build/generated 目录）
class UserRepository_Factory : Factory<UserRepository> {
    private val apiProvider: Provider<ApiService>

    constructor(apiProvider: Provider<ApiService>) {
        this.apiProvider = apiProvider
    }

    override fun get(): UserRepository {
        return UserRepository(apiProvider.get())
    }

    companion object {
        fun create(apiProvider: Provider<ApiService>) =
            UserRepository_Factory(apiProvider)
    }
}
```


你写的注解 -> 编译器生成工厂类 -> 运行时Hilt调用工厂类创建对象并注入。   

零反射，全部在编译期完成，这是Hilt/Dagger性能优于其他DI框架的根本原因。  



