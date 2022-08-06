# Retrofit

## Retrofit概述

- Retrofit最初的样子

```kotlin
val retrofit = Retrofit.Builder()
  .baseUrl("http://api.devio.org/as/")
  .build();

interface Api{
  //默认情况下方法返回类型为Call<T>，response类型为ResponseBody
  @GET("/user/login")
  fun login():Call<ResponseBody>
}
```

- Retrofit的扩展玩法

  ![image-20210223214731627](../img/image-20210223214731627.png)

- RxJavaCallAdapterFactory

  ![image-20210223214844425](../img/image-20210223214844425.png)

- HiCallAdapterFactory

  ![image-20210223214919299](../img/image-20210223214919299.png)

- HiCall

  ```kotlin
  class HiCall :Call<T>{
    
    private var delegate:Call<T>
    
    fun HiCall(delegate : Call<T>){
      this.delegate = delegate
    }
    
    override fun enqueue(callback: Callback<T>) {
      delegate.enqueue(response ->
      	mainHandler.post{
        	callback.onSuccess(response)
        }
      )
    }
  
    override fun execute(): Response<T> {
      //...
    }
  }
  ```

  



![image-20210223215117281](../img/image-20210223215117281.png)

![image-20210223215139938](../img/image-20210223215139938.png)