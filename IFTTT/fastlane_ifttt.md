# Fastlane + IFTTT

Что такое IFTTT? 

Это сервис, позволяющий делать автоматизации путем связывания Api разных сервисов. К примеру, когда вы загрузили фото в Instagram, оно автоматически появится в вашей ленте Twitter. Или можно взывать действие написав сообщение боту в Telegram. А также многое другое.

Для удобства я вынес настройку пользовательской части (приложение IFTTT) в отдельный файл

[Настройка IFTTT](https://www.notion.so/IFTTT-f2bb385a11024820ada7b4efef0a7e81)

В этой части гайда мы настроим Fastlane для вызова WebHooks в IFTTT, что позволит нам сделать много интересных интеграций со сторонними сервисами. Начнем с примеров того, что после настройки интеграции мы сможем сделать.

- Вышел новый релиз в App Store? Напишите об этом в социальные сети.
- Стала доступна новая бета? Напишите бета тестерам в их закрытый канал или чат.
- Завершилась сборка для разработчиков? IFTTT пришлет Push-уведомление, перейдя по которому, можно будет моментально установить новую сборку.
- Интеграция сообщений с Telegram.
- И многое другое.

Fastlane по умолчанию поддерживает IFTTT, так что нам не придется устанавливать дополнительные плагины. Так что, для интеграции нам нужно написать лишь этот код в Lane:

```ruby
ifttt(
  api_key: account_token, #Api ключ полученный из гайда по настройке IFTTT
  event_name: event_name, #Название event введенное вами при создании автоматизации
	#значения которые будут передаваться в автоматизацию
  value1: "#{ITMS-SERVICES}#{plist_url}", 
	# ссылка по открытию которой сразу начнется установка приложения
  value2: "#{QR_GEN_URL}#{plist_url}", 
	# ссылка генерирующая QR-код начинающего загрузку приложения
  value3: "#{full_build_number}"
	# версия сборки в формате "Версия(номер сборки)" "1.0(15)"
 )
```

Это был общий пример из документации, а теперь, я покажу свою, модифицированную версию, для более удобного использования автоматизации

```ruby
platform :ios do
	# Переменные
  IFTTT_TOKENS = ['токены рядовых пользователей']
  IFTTT_ACTIONS = ['fastlane_push', 'название команд из WebHooks']
  IFTTT_ADMIN_TOKENS = ['токены админов']
  IFTTT_ADMIN_ACTIONS = ['fastlane_telegram', 'название команд из WebHooks']
  ITMS-SERVICES = "itms-services://?action=download-manifest&url="
  QR_GEN_URL = "https://linkbridge.ru/item-serviece.php?url="
  before_all do
    xcodeproj = "файл.xcodeproj"
    version_number = get_version_number(
      xcodeproj: xcodeproj,
      target: "таргет приложения"
    )
    build_number = get_build_number()
    full_build_number = "#{version_number}(#{build_number})"
    ENV['FULL_BUILD_NUMBER'] = full_build_number
	end
  lane :ifttt_actions do
     plist_url = ENV['PLIST_URL']
     full_build_number = ENV['FULL_BUILD_NUMBER']
     ENV['FULL_BUILD_NUMBER'] = full_build_number
     IFTTT_TOKENS.each do |account_token|
		 	IFTTT_ACTIONS.each do |event_name|
				ifttt(
					api_key: account_token,
					event_name: event_name,
					value1: "#{ITMS-SERVICES}#{plist_url}",
					value2: "#{QR_GEN_URL}#{plist_url}",
					value3: "full_build_number"
				)
			end
		end
		IFTTT_ADMIN_TOKENS.each do |account_token|
			IFTTT_ADMIN_ACTIONS.each do |event_name|
				ifttt(
					api_key: account_token,
					event_name: event_name,
					value1: "#{ITMS-SERVICES}#{plist_url}",
					value2: "#{QR_GEN_URL}#{plist_url}",
					value3: "full_build_number"
				)
			end
		end
	end
	lane :beta do
		#другой код Lane
		ifttt_actions()
		#другой код Lane
	end
end
```

Теперь быстро расскажу, что тут и как:

Для удобства я вынес все переменные в начало, те переменные, которые не являются постоянными, лежат в контейнере, который запускается автоматически, каждый раз, при запуске любого Lane.

Далее в Lane `ifttt_actions` для каждого пользователя `IFTTT_TOKENS` из массива выполняем все действия из массива `IFTTT_ACTIONS`. Таким образом, можно внести всех пользователей в один массив, и для каждого из них запускать одинаковый набор команд.

Также я добавил отдельные массивы администраторов и администраторских действий, сделано это для тех действий, которые надо выполнять 1 раз на сборку. 

Пример:

Разослать Push-уведомления с ссылкой на установку новой сборки надо всем бета-тестерам, а написать в канал о том, что вышла новая сборка, нужно один раз.

На последок расскажу о параметрах value. 

Это переменные String которые передаются автоматизации IFTTT. Количество Value ограничено тремя. Больше IFTTT воспринимать не будет. В эти переменные можно загрузить ссылку, номер сборки или еще что-то, что вы захотите.
