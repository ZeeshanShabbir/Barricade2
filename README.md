# Barricade2
![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/mutualmobile/Barricade2?label=version)

Barricade is a library for Android apps that allows you to get responses to API requests locally by running a local server. Barricade will intercept your API calls using an OkHttp Network Interceptor and can provide a local response from a local file.

It also supports multiple responses per request, unlike other local server implementations and presents the user with an UI for modifying which response to return for a request. at runtime.

<img src="https://github.com/mutualmobile/Barricade2/blob/master/art/Barricade2Demo.gif" width="250"/>

## When to use

* During **development**, Barricade is useful for easily exercising all edge cases and responses of a feature without waiting for those server devs to finish implementing the APIs

* For **integration and system tests**, Barricade allows you to easily toggle through each predefined response for a request to cover every possible edge cases

* To build a **demo mode** so that users can explore the app with dummy data

#### How is it different from OkHttp's MockWebServer?

* MockWebServer is queue-based which is ok for simple apps, but hard to use predictably when you have multiple calls getting fired at the same time. Barricade is call-specific so it'll always return the response you configure irrespective of the number of requests your app is making.

* Barricade gives you a UI to easily change the configuration whenever you want so even your QA can test different scenarios easily,

* Barricade can be used outside of tests. For example, you can easily build a full-fledged demo mode to allow users to try out the app without creating an account.

* Barricade allows you to specify responses in files instead of plain strings which keeps your codebase clean.


## Adding Barricade to your project

Include the following dependencies in your app's build.gradle :

```
def barricadeVersion = "0.0.3" // Get the latest version from tags
dependencies {
    implementation ("com.mutualmobile:barricade2:$barricadeVersion")
    implementation ("com.mutualmobile:barricade-annotations:$barricadeVersion")
    ksp ("com.mutualmobile:barricade-compiler:$barricadeVersion")
}
```

Since Barricade2 internally uses [Kotlin Symbol Processing (KSP)](https://kotlinlang.org/docs/ksp-overview.html#:~:text=Kotlin%20Symbol%20Processing%20(KSP)%20is,up%20to%202%20times%20faster.) and generates Config classes for us, therefore we also need to include the sourceSets it generates the files into in order to make them visible to our codebase.
We can do that by including the following inside the `android` block in the app-level build gradle (or wherever we've put the dependencies in the step above) i.e.

```
android {
    ...
    sourceSets {
        debug {
            kotlin { srcDir(file("build/generated/ksp/debug/kotlin")) }
        }
        release {
            kotlin { srcDir(file("build/generated/ksp/release/kotlin")) }
        }
    }
}
```

## How to use

1. Install Barricade in your `Application` class' `#onCreate()`

  ```
  override fun onCreate() {
        super.onCreate()
        Barricade.Builder(this, BarricadeConfig.getInstance())
            .enableShakeToStart(this)
            .install()
            ...
    }
  ```

2. Add your API response files to `assets/barricade/` for each type of response (success, invalid, error etc). You should consider creating subdirectories for each endpoint and putting the responses in them to organise them properly.

3. In your Retrofit API interface, for the required methods, add annotation `@Barricade` which mentions which endpoint to intercept and the possible responses using `@Response` annotation.

Example -
```
@GET("/users/{user}/repos")
@Barricade(endpoint = "repos", responses = {
    @Response(fileName = "success.json", isDefault = true),
    @Response(fileName = "failure.json", statusCode = 500)
}) fun getUserRepositories(@Path("user") String userId): Single<ReposResponse>
```

To configure multiple / non-JSON responses -
```
@GET("/users/{user}/repos")
@Barricade(endpoint = "repos", responses = {
    @Response(fileName = "success.xml", type = "application/xml", isDefault = true),
    @Response(fileName = "failure.xml", type = "application/xml", statusCode = 500)
}) fun getUserRepositories(@Path("user") String userId): Single<ReposResponse>
```
Default type is "application/json"


4. Add `BarricadeInterceptor` to your `OkHttpClient`

```
var okHttpClient = OkHttpClient.Builder()
    .addInterceptor(BarricadeInterceptor())
    ...
    .build()
```

#### Enable/ Disable Barricade
Barricade can be enabled or disabled at runtime using `Barricade.getInstance().setEnabled(true/false)`

#### Change Settings
Barricade allows you to change the response time and the required response for a request at runtime.
* Open Barricade settings by shaking the device
* Change the response time in milliseconds by clicking on the timer on top right
* To change the required response for a request, click on the request from list and then select the response you want from
the list of responses. This list is populated from the response files in assets folder

You can also change the above settings programmatically which can be helpful for testing -
```
Barricade.getInstance()
    .setDelay(100)
    .withResponse(BarricadeConfig.Endpoints.REPOS, BarricadeConfig.Responses.Repos.GetReposSuccess)
```
* `withResponse()` changes the response of the endpoint passed in the first parameter.


**Note:** Using the above technique will also save the settings and apply to other responses as well, not just the
next response, until changed.

#### Migration from Barricade1
If you're planning to migrate to Barricade2 from the older version(s) of [Barricade](https://github.com/mutualmobile/Barricade),
you might want to take into consideration that Barricade2 has completely been rewritten in Kotlin
(which might not be feasible for Java-only projects) and has
its own set of dependencies. Therefore, you will have to [include the new dependencies](https://github.com/mutualmobile/Barricade2/tree/setup/readme#adding-barricade-to-your-project)
and remove the old ones from your project now (again, only if you plan to or already use Kotlin).

License
-------

    Copyright 2022 Mutual Mobile

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
