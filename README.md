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

Soul предоставляет вам возможность создавать свое уникальное приложение на базе существующих решений, для того, чтобы кастомизировать ваше приложение вы можете  устанавливать те или иные значения параметров на сайте в админ-панеле Soul, а также настоить SoulSDK при помощи  специального класса SoulConfigs. 

После того, как вы инициализировали SoulSDK, вы можете вызвать методы установки параметро. Это дейсвие не является обязательным, если его проигнорировать все параметры получат значения по умолчанию.

```java
        SoulConfigs.setSearchLifeTimeValue(SoulConfigs.ONE_HOUR);
        SoulConfigs.setLikeReactionLifeTime(SoulConfigs.ONE_MONTH);
        SoulConfigs.setDislikeReactionLifeTime(SoulConfigs.ONE_YEAR);
        SoulConfigs.setUserAgentAppVersion(BuildConfig.VERSION_NAME);
        SoulConfigs.setUserAgentAppName("Pure");
```

## Список возможностей

* `SoulSDK` связывает ваше приложение с Soul Backend и предоставляет возможность использовать в вашем приложении готовые функции, которые почти всегда используются практически в каждом приложении для знакомств.

Вот основные возможности реальзованные на данный момент:

* `SoulAuth` - набор методов необъодимых для авторизации пользователя в системе. На данный момент доступна возможность авторизации через подтверждение номера мобильного телефона через sms.

* `SoulCurrentUser` - набор методов для работы с сущностью авторизированного пользователя. После авторизации из люого места приложения можно легко получить обект авторизированного пользователя, а также отдельно его свойства. При использовании методов SoulCurrentUser SDK само синхронизирует информацию с сервером, поэтому производя любые действия с авторизированным пользователем через класс SoulCurrentUser, вы всегда можете быть уверены, что вы работаете с актуальными данными и любые изменения свойств пользователя будут сохранены.

* `SoulMedia` - набор методов для работы с изображением, аудио и видео. Получение объектов альбомов пользователя, создание, удаление и редактирование альбомов с изображением, а также самих изображений.

* `SoulUsers` - набор методов для поиска других пользователей по определенным кретериям. SoulSDK избавляет вас от необходимости описывать логику получения новых пользователей, а также случаи с обрывом соединения. 

* `SoulReactions` - набор методов для отправки ваших “реакций” на других пользователей. Сами реакции могут быть абсолютно произвольными и указанными вами на сайте Soul в админ панеле. Однако самые очевидные и популярные вынесены в SoulReactions в простые удобные методы, не требующие излишней информации.

* `SoulCommunication` - набор методов для коммуникации между пользователями. На данный момент SoulSDK поддерживает коммуникацию через текстовые чаты. SoulSDK упрощает разработку чатов, достаточно указать с каким пользователем нужно создать чат и подписаться на прием/отправку сообщений вызвав один простой метод.

* `SoulEvents` - набор методов для быстрой и удобной актуализации информации по всем событиям и объектам, которые так или иначе касаются текущего авторизированного пользователя.

* `SoulPurchases` - набор методов для упрощения и безопастности (валидации на сервере) произведенных покупок в Google Play.

* `SoulSystem` - набор методов для получения информации о настройках, параметрах связанных с сервером Soul. Например: getServerTime()


## Возвращаемые значения

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

## Пример реализации авторизации

Ниже рассмотрен пример реализации авторизации на примере SoulResponse:

```java
public class AuthActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

/* при старте приложения автоматически пробуем авторизироваться */

        SoulAuth.login(authCallback);
    }


    private SoulCallback authCallback = new SoulCallback<AuthorizationResponse>() {
        @Override
        public void onSuccess(AuthorizationResponse responseEntity) {

/* после каждого успешного SoulSDK автоматически сохраняет все необходимые данные для успешной авторизации в следующий раз, поэтому  никаких параметров в данный метод передавать не требуется  */

            startActivity(new Intent(this, NextBeautifulActivity.class));
        }

        @Override
        public void onError(SoulError error) {

            switch (error.getCode()) {
                case SoulError.NO_LOGIN_CREDENTIALS:

/* в случае отсутствия необходимых данны для логина в фоне, показываем экран с просьбой ввести номер телефона для авторизации  */

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

    public void requestPhoneNumber(String phoneNumber) {

/* после того, как пользователь ввел номер телефона отправляем запрос SMS на это номер мобильного телефона ... */

        SoulAuth.requestPhone(phoneNumber);

/* … и отображаем форму для ввода проверочного кода */

        showVerificationCodeInput();
    }

    public void verifyPhone(String verificationCode) {

/* после того, как пользователь ввел номер телефона отправляем запрос SMS на это номер мобильного телефона  */

        SoulAuth.verifyPhone(authCallback);
    }
}
```



