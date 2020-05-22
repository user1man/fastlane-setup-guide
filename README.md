# fastlane-setup-guide

## Установка

Установите последнюю версию Xcode command line tools.

```bash
xcode-select --install
```

Далее устанавливаем сам Fastlane:

```bash
# Через RubyGems
sudo gem install fastlane -NV

# Альтернативно через Brew
brew install fastlane
# Лично я рекомендую 2 варинант
```

Переходим в папку проекта. Это можно легко сделать при помощи команды cd и перетаскивания папки проекта в командную строку.

## Настройка

Находясь в папке системы, пишем:

```bash
fastlane init
```

> Для тех, кто верит в себя, есть бета-версия Fastlane для Swift. Лично я предпочитаю версию на Ruby, ибо информации по ней больше, чем о Swift версии. [Документация на Swift версию](https://docs.fastlane.tools/getting-started/ios/fastlane-swift/)

```bash
fastlane init swift
```

Далее Fastlane спросит вас, какой пресет использовать для настройки. Я выбирал 4 вариант, и этот вариант не сложнее всех остальных.

### **Перед тем, как продолжим**

Если в **shell** профайле **locale** не **UTF-8**, то будут возникать проблемы со сборкой и загрузкой билдов. Заходим в файл вашего shell profile (*~/.bashrc*, *~/.bash_profile*, *~/.profile* или *~/.zshrc*) и добавляем следующие строчки:

```
export LC_ALL=en_US.UTF-8 
export LANG=en_US.UTF-8
```

Теперь все готово к написанию непосредственных шагов автоматизации сборки.

# Пишем автоматизацию

## **Fastfile**

Папка *fastlane* содержит в себе *Fastfile* и *Appfile*. В Appfile мы будем прописывать необходимые для сборки и публикации значения: Bundle IDs, App ID, Team ID и другие. В *Fastfile* мы будем описывать наши скрипты. После первоначальной установки он выглядит так:

```
default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"
  lane :custom_lane do
    # add actions here: https://docs.fastlane.tools/actionsend
end
```

- **default_platform(:ios)** - задаем платформу по умолчанию, чтобы не указывать ее из командной строки.
- **platform :ios do … end** - здесь описываются "lanes" для платформы iOS.
- **desc "Description of what the lane does"** - краткое описание "lane". Список всех "lanes" с описаниями можно посмотреть с помощью команды `$ fastlane lanes`.
- **lane :custom_lane do … end**: Lane (путь, полоса) — это, проще говоря, метод. У него есть имя, параметры и тело. В теле мы будем вызывать необходимые нам команды для сборки, выкладки, запуска тестов и др. Lanes вызываются из командной строки вызовом `$ fastlane [lane_name] [parameters]` . Именно с вызова одной из lanes начинается выполнение автоматизированных шагов.
- Название Lane можно поменять, к примеру, один из моих называется `lane :notify_slack`

## Appfile

Для удобства заполним Appfile.

```ruby
  app_identifier("бандл вашего приложения") # The bundle identifier of your app
  apple_id("ваш dev apple id") # Your Apple email address

# For more information about the Appfile, see:
#     https://docs.fastlane.tools/advanced/#appfile
```

## Команды

Начинаем с изучения основных команд. Все команды можно легко найти в [документации](https://docs.fastlane.tools/actions/)

[increment_version_number](https://docs.fastlane.tools/actions/increment_version_number/) -  повышает номер сборки на 1

[gym](https://docs.fastlane.tools/actions/gym/#gym)(scheme: "MyApp") - делает сборку приложения

[slack](https://docs.fastlane.tools/actions/slack/)(message: "App successfully released!") - отправляет сообщение в slack

[upload_to_app_store](https://docs.fastlane.tools/actions/upload_to_app_store/) - название говорит само за себя

[upload_to_testflight](https://docs.fastlane.tools/actions/upload_to_testflight/) - не менее говорящее название

[snapshot](https://docs.fastlane.tools/actions/snapshot/#snapshot): запускает UI-тесты и делает скриншоты, которые можно использовать при отправке на review в App Store

Остальные команды можно глянуть в [документации](https://docs.fastlane.tools/actions/).

### Match

Специально вынес эту команду в отдельный заголовок, так как лично мне она далась нелегко, поэтому сделаю подробный гайд.
Что делает команда? Автоматически подписывает код, и при этом требует ноль действий от вас, только первоначальная настройка.

**Подготовка**

На этапе подготовки я расскажу, как вам создать репозиторий где и будут храниться наши сертификаты для подписи приложения. Я буду делать это на примере GitHub, Match по умолчанию поддерживает Git, GitHub, Google Cloud Storage, Amazon S3.

Открываем GitHub и создаем **приватный** репозиторий. Называем как-нибудь, там будут храниться наши сертификаты. Для каждого проекта будет свой репозиторий.

Далее переходим по [ссылке](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line), где написано, как получить заветный ключ доступа к git через ssh. Храним этот ключ.

**Основной этап**

Заходим в нашу любимую командную строку и пишем команду, внутри папки проекта:

```bash
fastlane match init
```

Далее в ответ на вопрос даем веб ссылку на только что созданный приватный репозиторий.

<img src="https://github.com/user1man/fastlane-setup-guide/blob/master/match_init.gif">

Эта команда создала `Matchfile` внутри папки `./fastlane/` . Находим и открываем этот файл. И редактируем его.

```ruby
git_url("ссылка с https на ваш приватный git репозиторий для сертификатов")

storage_mode("git") 

type("development") # The default type, can be: appstore, adhoc, enterprise or development

 app_identifier("bundle id вашего приложения") #com.yourOrg.MyApp
 username("Ваш Apple ID зарегистрированный в программе apple developer") 

# For all available options run `fastlane match --help`
# Remove the # in the beginning of the line to enable the other options

# The docs are available on https://docs.fastlane.tools/actions/match
```

Далее, чтобы залогиниться в git, в командной строке пишем:

```bash
curl -u username:token https://api.github.com/user
# где username это ваше имя github
# и token это тот, который мы создали этаром выше
```

Далее в командной строке пишем :

```bash
fastlane match development
```

После чего заботливо вводим пароль от связки ключей, он же пароль от Mac.

Далее вас попросят придумать пароль для сертификата. Лайфхак: пароль "1" подойдет.

# Пишем Lane для Ad Hoc

## Получаем Api ключ Slack

Открываем [эту страницу](https://api.slack.com/messaging/webhooks)

Далее кликаем на зеленую кнопку "[Create your Slack App](https://api.slack.com/apps/new)"

Далее создаем приложение

На странице приложения открываем *Add features and functionality* 

Выбираем *Incoming Webhooks*

Нажимаем Add New Webhooks to Workspace и копируем его в переменную SLACK_URL (да поможет вам поиск по коду).

```bash
before_all do
	ENV['SLACK_URL'] = 'URL кооторый мы получили от slack'
end
```

## Настраиваем загрузку в AWS S3.

Так как я не занимался созданием и первичной настройкой S3, не буду обманывать вас своими догадками, и пропущу этот момент и перейду сразу к настройке Bucket.

Ваш Bucket должен разрешать доступ на загрузку файлов с бакета по URL

Копируем название бакета в переменную в кода.

Далее создаем папку и так же копируем название в переменную.

После настройки командной строки система сама будет загружать сборки по следующей системе: https://названиебакета.s3.amazonaws.com/названиеПапки/ВерсияПриложения/номерСборки/dev/название приложения.plist

### Установка AWS CLI

Лично я рекомендую устанавливать через графический интерфейс (вариант:"Install and update the AWS CLI version 2 using the macOS user interface")

Далее вводим ключи доступа, которые можно получить тут:

**Для создания ключей доступа для IAM пользователя**

1. Войдите в аккаунт AWS Management Console и откройте IAM консоль в [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
2. В навигационной панели выберите **Users**.
3. Выберите имя пользователя, для которого вы хотите создать ключ доступа, и после этого выберите вкладку **Security credentials**
4. В разделе **Access keys**, выберите **Create access key**.
5. Для того, чтобы увидеть новый ключ доступа, выберите **Show**. Вы больше не получите доступ к секретному ключу после того, как диалоговое окно исчезнет. Ваши ключи будут выглядеть примерно так:
    - Access key ID: AKIAIOSFODNN7EXAMPLE
    - Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
6. Для загрузки пары ключей выберите **Download .csv file**. Храните ключи в защищенном месте. Вы больше не получите доступ к секретному ключу после того как диалоговое окно исчезнет.

    Храните ключи конфиденциально, чтобы защитить свой аккаунт AWS, и никогда не отправляйте их по электронной почте. Не делитесь ими за пределами своей организации, даже если запрос поступает из AWS или других источников. [Amazon.com](http://amazon.com/) Никто из тех, кто законно представляет Amazon, никогда не попросит у вас ваш секретный ключ.

7. После того, как вы скачаете  `.csv` file, выберите **Close**. Когда вы создаете ключ доступа, пара ключей активна по умолчанию, и вы можете использовать эту пару сразу же.

Далее настраиваем CLI, подробнее тут: [https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

А если по-быстрому, вводим это в командную строку

```bash
aws configure
```

Самое главное в последнем пункте `Default output format [json]:` указать text.

## Сам код fastlane

Вот код который используется у меня, готовый к употреблению, так сказать. Нужно только заменить переменные на свои

```ruby
# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

default_platform(:ios)

platform :ios do
  
  APP_NAME = 'Имя вашего приложения, которое отображается под иконкой'
#Display Name в General> Idnetity вашего Target приложения
  SCHEME = 'Такое же, как и Target приложения'
  AWS_S3_BUCKET='Название бакета на s3 без доп знаков'
  AWS_FOLDER_NAME = 'папка в s3, специально созданная для сборок'
	# Папка должна иметь публичный доступ на скачивание по url
  APP_ICON_58 = "https://via.placeholder.com/58x58.png" 
#иконки приложений в 2 разрешениях, в примере оставил ссылку на генератор картинок
  APP_ICON_120 = "https://via.placeholder.com/120x120.png"

  before_all do # lane запускается в самом начале, при каждом запуске
		version_number = get_version_number(
      xcodeproj: "файл.xcodeproj",
      target: SCHEME 
    )
    build_number = get_build_number()# получаем номер сборки
		# переменные, которые задают путь на s3
		s3_path = "#{AWS_S3_BUCKET}/#{s3_folder}"
    s3_folder = "#{AWS_FOLDER_NAME}/#{version_number}/#{build_number}/dev"

    
		#пробрасываем переменные
    http_path = "https://#{AWS_S3_BUCKET}.s3.amazonaws.com/#{s3_folder}"
    ENV['S3_folder'] = s3_folder
    ENV['S3_PATH'] = s3_path
    ENV['APP_URL'] = "#{http_path}/#{APP_NAME}.ipa"
    ENV['PLIST_URL'] = "#{http_path}/#{APP_NAME}.plist"
    ENV['SLACK_URL'] = 'URL кооторый мы получили от slack'
		ENV['Build_number'] = build_number
    
  end
  #lane который отвечает за slack
  lane :notify_slack do |options|
		#поля для 
    attachment_properties_fields = [
      {
        title: "App Version",
        value: get_version_number(target: SCHEME),
        short: true
      }, {
        title: "Build",
        value: Build_number,
        short: true
      }
    ]

    if options.key?([:master]) && options[:master] then
      message = 'New build is available in iTunes Connect'
      attachment_properties_fields_environment = 'Prod'
    else

      require 'cgi'
    #получение url для qr
      plist_url = ENV['PLIST_URL']

      message = 'New build is ready to install from S3'
      attachment_properties_fields_environment = 'deployment'
      attachment_properties_fields.push({
        title: 'QR code', #ссылка на qr код для загрузки, а если с iphone то на прямую
        value: "<https://linkbridge.ru/item-serviece.php?url=#{plist_url}|link> \n\nitms-services://?action=download-manifest&ampurl=#{plist_url}",
        short: false
      })
    end
      
    attachment_properties_fields.push({
      title: 'Environment',
      value: attachment_properties_fields_environment,
      short: true
    })

    attachment_properties = {
      thumb_url: APP_ICON_58,
      fields: attachment_properties_fields
    }

    slack(message: message, attachment_properties: attachment_properties)

  end
  
   lane :upload_s3 do #загрузка 

      s3_path = ENV['S3_PATH']
      s3_folder = ENV['S3_folder']
      ['ipa', 'plist'].each do |extension|
				#загрузка на s3 и открытие доступа загрузки по url
        sh "aws s3 cp './build/#{APP_NAME}.#{extension}' 's3://#{s3_path}/#{APP_NAME}.#{extension}'"
        sh "aws s3api put-object-acl --bucket #{AWS_S3_BUCKET} --key #{s3_folder}/#{APP_NAME}.#{extension} --acl public-read"
      end
    end
    
    private_lane :build_gym_dev do
			get_certificates # invokes cert
			get_provisioning_profile # invokes sigh
      match(type: "adhoc") # подписываем для AdHoc
      gym(
        scheme: SCHEME,
        clean:true,
        export_options: {
          method: "ad-hoc",
          manifest: { #генерация plist для загрузки с сервера s3
            appURL: ENV["APP_URL"],
            displayImageURL: APP_ICON_58,
            fullSizeImageURL: APP_ICON_120
          },
        },
        output_directory: "./fastlane/build"
      )

    end
  
  lane :tobeta do
    increment_build_number  #повышаем номер сборки на 1
    build_gym_dev() #собираем сборку
    sh "mv ./build/manifest.plist ./build/#{APP_NAME}.plist" # 
    upload_s3() # загрузка на s3
    notify_slack() # шлем уведомление в slack
    require 'cgi' # получаем qr 
    #получение url для QR для уведомления в MacOS
      plist_url = ENV['PLIST_URL']
      plist_url_escaped = CGI::escape plist_url
    notification(subtitle: "Build completed", message: "Ready", open: "https://linkbridge.ru/item-serviece.php?url=#{plist_url}")
  end
  
  lane :beta do

    increment_build_number #повышаем номер сборки на 1
      build_number: latest_testflight_build_number(
        username: "почта apple developer",
        app_identifier: "bundle id") + 1,
      xcodeproj: "файл.xcodeproj"
    

    match(type: "appstore")

    gym(scheme: SCHEME,
      clean:true,
      export_options: {
        method: "appstore",
      },
      output_directory: "./fastlane/build"
    )

    upload_to_testflight(# отправкляем в testflight
			username: "Ваш apple id",
			beta_app_feedback_email: "почта для обратной связи тестеров",
		  beta_app_description: "Описание приложения",
		  demo_account_required: true, #если false то beta_app_review_info удаляем
		  notify_external_testers: false,
		  beta_app_review_info: {
		    contact_email: "email для обратной связи модераторов",
		    contact_first_name: "Имя",
		    contact_last_name: "Фамилия",
		    contact_phone: "телефон для связи с модераторами",
		    demo_account_name: "demo@email.com",
		    demo_account_password: "connectapi",
		    notes: "this is review note for the reviewer <3 thank you for reviewing"
  })

    slack(message: "Successfully distributed a new beta build")
  end
end
```
