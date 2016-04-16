# Руководство для Android

SoulSDK предоставляет удобно серверное решение для ваших мобильных приложений дейтинг тематики.

## Инициализация

 Добавте следующие строки в классе Application в метод onCreate():
```java
 @Override
    public void onCreate() {
        super.onCreate();
        SoulSDK.initialize("API_KEY", this);
    }
```
Вместо “API_KEY” вставте ключ, соответствующий вашему аккаунту в Soul.



## Клиентская кастомизация

Soul предоставляет вам возможность создавать свое уникальное приложение на базе существующих решений, для того, чтобы кастомизировать ваше приложение вы можете  устанавливать те или иные значения параметров на сайте в админ-панели Soul, а также настроить SoulSDK при помощи  специального класса SoulConfigs. 

После того, как вы инициализировали SoulSDK, вы можете вызвать методы установки параметров. Это действие не является обязательным, если его проигнорировать все параметры получат значения по умолчанию.

```java
        SoulConfigs.setGCMSenderId(ApplicationConfig.SENDER_ID);
        SoulConfigs.setSearchLifeTimeValue(SoulConfigs.ONE_HOUR);
        SoulConfigs.setLikeReactionLifeTime(SoulConfigs.ONE_MONTH);
        SoulConfigs.setDislikeReactionLifeTime(SoulConfigs.ONE_YEAR);
        SoulConfigs.setUserAgentAppVersion(BuildConfig.VERSION_NAME);
       
        // other
        
```

## Список возможностей

* `SoulSDK` связывает ваше приложение с Soul Backend и предоставляет возможность использовать в вашем приложении готовые функции, которые почти всегда используются практически в каждом приложении для знакомств.

Вот основные возможности реализованные на данный момент:

* `SoulAuth` - набор методов необъодимых для авторизации пользователя в системе. На данный момент доступна возможность авторизации через подтверждение номера мобильного телефона через sms.

* `SoulCurrentUser` - набор методов для работы с сущностью авторизированного пользователя. После авторизации из люого места приложения можно легко получить обект авторизированного пользователя, а также отдельно его свойства. При использовании методов SoulCurrentUser SDK само синхронизирует информацию с сервером, поэтому производя любые действия с авторизированным пользователем через класс SoulCurrentUser, вы всегда можете быть уверены, что вы работаете с актуальными данными и любые изменения свойств пользователя будут сохранены.

* `SoulMedia` - набор методов для работы с изображением, аудио и видео. Получение объектов альбомов пользователя, создание, удаление и редактирование альбомов с изображением, а также самих изображений.

* `SoulUsers` - набор методов для поиска других пользователей по определенным кретериям. SoulSDK избавляет вас от необходимости описывать логику получения новых пользователей, а также случаи с обрывом соединения. 

* `SoulReactions` - набор методов для отправки ваших “реакций” на других пользователей. Сами реакции могут быть абсолютно произвольными и указанными вами на сайте Soul в админ панеле. Однако самые очевидные и популярные вынесены в SoulReactions в простые удобные методы, не требующие излишней информации.

* `SoulCommunication` - набор методов для коммуникации между пользователями. На данный момент SoulSDK поддерживает коммуникацию через текстовые чаты. SoulSDK упрощает разработку чатов, достаточно указать с каким пользователем нужно создать чат и подписаться на прием/отправку сообщений вызвав один простой метод.

* `SoulEvents` - набор методов для быстрой и удобной актуализации информации по всем событиям и объектам, которые так или иначе касаются текущего авторизированного пользователя.

* `SoulPurchases` - набор методов для упрощения и безопастности (валидации на сервере) произведенных покупок в Google Play.

* `SoulSystem` - набор методов для получения информации о настройках, параметрах связанных с сервером Soul. Например: getServerTime()


### Возвращаемые значения

Любой метод из перечисенных выше классов вызывается в качестве статического метода этого класса и не требует никаких дополнительных зависимостей. 
Например:  SoulReactions.likeUser(userId) 

Все методы `SoulSDK` предоставляют возможность получения результатов в двух вариантах: 

* через универсальный интерфейс `SoulCallback`

```java
public interface SoulCallback<T> {
    void onSuccess(T responseEntity);
    void onError(SoulError error);
}
```

```java
public class SoulResponse<T> {
          private SoulError error;
          private boolean hasError = false;
          private T response;

      // ...
}
```

```java
public class SoulError {
          private String description = "";
          private int code = 0;

      // ...
}
```

* для поклонников реактивности через универсальный `Observable<SoulResponse<T>>`

```java
 SoulUsers.getNextSearchResult()
                .subscribe(
                        res -> sowUsers(res.getResponse().getUsers()),
                        err -> Log.e(TAG, "error " + err.getMessage()));
}
```



## Пример авторизации

Ниже рассмотрен пример реализации авторизации через подтверждение мобильного телефона на примере `SoulResponse`:

```java
public class AuthActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // при старте приложения автоматически пробуем авторизироваться
        SoulAuth.login(authCallback);
    }


    private SoulCallback authCallback = new SoulCallback<AuthorizationResponse>() {
        @Override
        public void onSuccess(AuthorizationResponse responseEntity) {
            // после каждого успешного SoulSDK автоматически сохраняет все необходимые данные 
            // для успешной авторизации в следующий раз, поэтому  никаких параметров в данный метод 
            // передавать не требуется
            startActivity(new Intent(this, NextBeautifulActivity.class));
        }

        @Override
        public void onError(SoulError error) {
            switch (error.getCode()) {
                case SoulError.NO_LOGIN_CREDENTIALS:
                    // в случае отсутствия необходимых данны для логина в фоне, показываем экран 
                    // с просьбой ввести номер телефона для авторизации
                    showPhoneInput();
                    break;
                case SoulError.PHONE_WRONG_CODE:
                    showWrongCodeError(error.getDescription());
                    break;
                default:
                    showError(error.getDescription());
            }
        }
    };

    // после того, как пользователь ввел номер телефона отправляем запрос SMS на этот 
    // номер мобильного телефона ...
    public void requestPhoneNumber(String phoneNumber) {
        SoulAuth.requestPhone(phoneNumber);
        //... и отображаем форму для ввода проверочного кода
        showVerificationCodeInput();
    }

    // после того, как пользователь получил проверочный код на указанный ранее номер телефона, 
    // отправляем код на валидацию
    public void verifyPhone(String verificationCode) {
        SoulAuth.verifyPhone(verificationCode, authCallback);
    }
}
```


## Пример работы с авторизированным пользователем

### CurrentUser

Авторизированного пользователя всегда можно получать данным методом здесь находится актуальная информация, т.к. объект всегда синхронизируется с сервером после каждой успешной авторизации.
        
```java
    User authorizedUser = SoulCurrentUser.getCurrentUser();
    Log.e(TAG, "Usersid is " + authorizedUser.getId());
```



```java
    SoulCurrentUser.updateSelfGender(myGenderET.getText().toString())
            .subscribe(
                    res -> Log.e(TAG, "success " + res.getResponse().getCurrentUser().getId()),
                    err -> Log.e(TAG, "error " + err.getMessage())));
```

### UsersParameters

Объект `CurrentUser` содержит в себе `UsersParameters`,

public class User {

```java
    private String id;
    private int recordId;
    private NotificationTokens notificationTokens;
    private SubscriptionServices subscriptionServices;
    private UsersParameters parameters; // <- параметры
    private List<Album> albums;
    private Reactions reactions;
    private Chat chat;
    
    // ...
}
```

которые в свою очередь представлены четырьмя разными группами параметров:

* __filterable__ - параметры, которые влияют или могут влиять на фильтрацию результатов при поиске пользователей  
* __publicVisible__ - информация о пользователях, доступная другим пользователям
* __publicWritable__ - данные пользователя, которые могут быть изменены другими пользователями
* __privateVisible__ - данные, доступные для изменения и просмотра только самим пользователем

Данные пармаметры являются свободным JSON и могут иметь любую структуру.

```java
    String filterable = SoulCurrentUser.getCurrentUser().getParameters().getFilterable();
    JSONObject jObject = new JSONObject(filterable);
    String gender = jObject.getString("gender");
    Log.d(TAG, "gender: " + gender);
}
```

Создать свое произвальное свойство пользователя можно очень просто при помощи класса `SoulParameterObject`:

```java
    UsersParameters parameters = new UsersParameters();
    SoulParameterObject soulParameterObject = new SoulParameterObject();
    soulParameterObject.setProperty("myOwnProperty", "value Of Property");
    parameters.setFilterable(soulParameterObject);
    
    SoulCurrentUser.updateUserParameters(parameters).subscribe();
    // Обратите внимание, что ничего не сохраниться без вызова .subscribe()
    // т.к. метод можвращает Observable<SoulResponse<CurrentUserRESP>>
    // иначе с параметрами нужно передать дополнительно SoulCallback
```

Объект созданный при помощи `SoulParameterObject` можно наполнять свойствами при помощи метода `setProperty(...)`,
который принимает на вход String, людые Numbers, Boolean, а также любой валидный `SoulParameterObject`, в который также может быть вложены вышеперечисленные свойства или опять же `SoulParameterObject`.

```java
    UsersParameters parameters = new UsersParameters();
    SoulParameterObject soulParameterObject = new SoulParameterObject();
    soulParameterObject.setProperty("myOwnProperty", "value Of Property");

    SoulParameterObject nestedObject = new SoulParameterObject();
    nestedObject.setProperty("prop_1", 38);
    nestedObject.setProperty("prop_2", true);

    soulParameterObject.setProperty("myOwnProperty", "value Of Property");
    soulParameterObject.setProperty("nested", nestedObject);

    parameters.setFilterable(soulParameterObject);
        
    SoulCurrentUser.updateUserParameters(parameters).subscribe();
```

Получать результаты из SoulSDK (полученные от сервера по API) также гораздо удобней не в виде Json-строки, а в качестве SoulParameterObject.
В этом случае вам не требуется заниматься парсингом json.

```java
    SoulParameterObject filterable = SoulCurrentUser.getCurrentUser().getParameters().getFilterable();
    String gender = filterable.getString("myOwnProperty");
    Log.d(TAG, "myOwnProperty: " + gender);

    SoulParameterObject nestedObj = filterable.getSoulParameterObject("nested");
    int prop1 =  nestedObj.getInt("prop_1");
    boolean prop2 =  nestedObj.getBoolean("prop_2");
    Log.d(TAG, "prop1: " + prop1 + " prop2: " + prop2);
```

Таким образом можно сохранить местоположение пользователя:

```java
    UsersParameters parameters = new UsersParameters();
    SoulParameterObject soulParameterObject = new SoulParameterObject();
    SoulParameterObject nestedObject = new SoulParameterObject();
    nestedObject.setProperty("lat", 38.74123);
    nestedObject.setProperty("lng", -9.1400233);
    soulParameterObject.setProperty("location", nestedObject);
    parameters.setFilterable(soulParameterObject);
    SoulCurrentUser.updateUserParameters(parameters).subscribe();

    SoulParameterObject filterable = SoulCurrentUser.getCurrentUser().getParameters().getFilterable();
```

А затем прочитать его:

```java
    SoulParameterObject locationObject = filterable.getSoulParameterObject("location");
    Location location = new Location("");
    location.setLatitude(locationObject.getDouble("lat"));
    location.setLongitude(locationObject.getDouble("lng"));
```

### Популярные свойста пользователя

Однако, существуют такие свойства пользователей, которые используются почти любими приложениями для знакомств.
Например, пол пользователя или рассмотренное выше местоположение. Работа с такими наиболее популярными свойствами в SoulSDK представлена специальными методами.  

На данный момент реализованные следующие "быстрые" методы для работы с такими свойствами:

В Soulplatform существует такое понятие как __`availability`__ - это способность находить других пользователей и быть видимым для поиска другими пользователями. Соответствует свойству {"filterable":{"availableTill": [unix time here]}},
где `availableTill` - это временная метка, время до которого пользователь обладает данной способностью.

* __`SoulCurrentUser.getTimeLeft()`__ - возвращает время оставшееся до конца __availability__

* __`SoulCurrentUser.turnSearchOn()`__ - устанавливает значение __availableTill__ равным сумме текущего времени сервера и длительности __availability__. Значение длительности заданно по умолчанию в SoulSDK, однако ее всегда можно сменить вызовом метода  `SoulConfigs.setSearchLifeTimeValue(long val)`

* __`SoulCurrentUser.turnSearchOff()`__ - сбрасывает значение __availableTill__ и пользователь теряет способность __availability__

* __`SoulCurrentUser.updateGCMToken(String token)`__ - самый простой и быстрый способ сообщить серверу о токене для получения push уведомлений. Альтернативой может быть создание объекта `UsersParameters` самостоятельно и передачей собранного json на сервер.

* __`SoulCurrentUser.updateSelfGender(String gender)`__ - самый простой и быстрый способ передать на сервер пол авторизированного пользователя

* __`SoulCurrentUser.updateTargetGender(String gender)`__ - самый простой и быстрый способ передать на сервер пол пользователей, которые будут найдены в процессе поиска

* __`SoulCurrentUser.updateLocation(Location location)`__ - нет проще способа сообщить серверу о своем местоположении



