# Настройка IFTTT

В этом гайде мы получим Api ключ Webhooks для IFTTT, а также настроим необходимые автоматизации.

## Установка IFTTT

IFTTT — Это сервис, позволяющий делать автоматизации путем связывания Api разных сервисов. К примеру, когда вы загрузили фото в Instagram, оно автоматически появится в вашей ленте Twitter. Или можно взывать действие написав сообщение боту в Telegram. И.Т.П.

Загрузить приложение можно по ссылкам или при помощи QR кода:
### <a href="https://itunes.apple.com/app/apple-store/id660944635">App Store</a>

<p float="left">
    <img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/qrAppStore.png" width="300">
</p>

### <a href="https://play.google.com/store/apps/details?id=com.ifttt.ifttt&utm_source=/&utm_medium=web">Google Play</a> 
<p float="left">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/qrPlayMarket.png" width="300">
</p>

Далее регистрируемся или заходим под своим существующим аккаунтом, на главной странице находим кнопку **Get More** и нажимаем её.

Ищем в поиске сервис Webhooks и выбираем его.

Далее нажимаем Documentation и копируем строку после **Your key is: ...**
<p float="left">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/1.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/2.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/3.JPG" width="250">
</p>

Передаем эту строку разработчику системы автоматизации приложения. 

Следом нужно создать автоматизации: в этом гайде я подробно опишу пример создания автоматизации, с использованием Push уведомлений. Другие автоматизации вам поможет настроить разработчик системы автоматизации приложения.

## Создание автоматизации с использованием Push уведомлений

Далее на главной странице находим кнопку **Get More** и нажимаем её.
На открывшейся странице есть кнопка **Create**, именно она нам и нужна.

Если вы не нашли кнопку, попробуйте нажать кнопку **Explore** в главном меню.

В открывшемся меню нажимаем на **+ This** и пишем в поиске **Webhooks.**

После выбора Webhooks жмем на единственный вариант **Receive a web request.**
<p float="left">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/4.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/5.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/6.PNG" width="250">
</p>
Далее в поле **Event Name**, пишем то, что вам передал разработчик системы автоматизации приложения. 
<p float="left">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/7.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/8.PNG" width="250">
</p>
После этого открываем **That** и пишем **Notifications**. В открывшемся списке нам нужен вариант **Send a rich Notification from the IFTTT app.**
<p float="left">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/9.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/10.PNG" width="250">
<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/IFTTT/11.PNG" width="250">
</p>
Поля можно заполнять по вашему желанию, но я рекомендую заполнять их так:

*Tilte*: Новая сборка приложения доступна

*Message*: Нажмите на уведомление для установки

***Link URL***: ВАЖНО уточните, что нужно заполнять в это поле у разработчика системы автоматизации приложения.

После заполнения всех полей нажимаем на Create Action.

Последним шагом будет подтверждение действия на открывшейся странице (внизу), также на этой странице можно переименовать автоматизацию.
