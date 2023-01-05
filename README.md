## Веб-скрапинг

Для начала необходимо определится, что такое Веб-скрапинг.

**Веб-скрапинг** - это автоматизированный процесс извлечения данных с какой-либо веб-страницы. Когда не было возможности автоматизировано извлекать данные со страниц, извлекали данные вручную, что занимало достаточно большое количество времени. Сейчас же можно ускорить извлечение данных в разы при помощи различных языков программирования и специальных инструментов.

Информации в интернете становится все больше, как следствие, время на извлечение данных увеличивается. Возникает необходимость в решении данной задачи.

В данной задаче рассмотрим способ с применением Python библиотеки BeautifulSoup4 и Requests.

1. Задание

Задание будет состоять в том, чтобы выгрузить данные о фильмах на TMDB (ссылка - https://www.themoviedb.org/): название фильма, дату и время создания, номер фильма. А также для другогих запросов - название сериалов, дата создания и номер сериала.
На самом деле, можно разбить работу на 2 этапа:
* Этап 1: выгрузить и сохранить HTML-страницы;
* Этап 2: распарсить HTML в удобный для дальнейшего анализа формат (csv, json, pandas dataframe etc.).

## Инструменты
Для отправки HTTP-запросов есть немало Python-библиотек, наиболее известные urllib/urllib2 и Requests. На мой вкус Requests удобнее и лаконичнее, так что, буду использовать ее.
Также необходимо выбрать библиотеку для парсинга HTML. Анализ веб-данных реализуем с помощью Beautiful Soup.

Библиотека Beautiful Soup создает дерево синтаксического разбора из проанализированных HTML и XML-документов (включая документы с открытыми тегами, tag soup и неправильной разметкой). Beautifulsoup предполагает универсальное средство для веб-скрапинга, 
реализованного по модели парсинга DOM. Эта библиотека сделает текст веб-страницы, извлеченный с помощью Requests, более удобочитаемым.

## Пример парсинга сайта TMDB

1. Загрузка данных

Библиотека Requests дает вам возможность посылать HTTP/1.1-запросы, 
используя Python. С ее помощью можно добавлять контент, например заголовки, формы, многокомпонентные файлы и параметры, используя только 
простые библиотеки Python. Для начала, получаем страницу по URL.
```
url = 'https://www.themoviedb.org/movie'
page = requests.get(url)
page.status_code
```
Запрос был получен и успешно обработан сервером, если мы получем код - 2хх.

Теперь перейдем непосредственно к получению данных из HTML. Проще всего понять как устроена HTML-страница используя функцию "Просмотр кода страницы" в браузере.
В данном случае: вся таблица с названиями фильмов и их датами создания заключена в теге </div class = "content">.
Пример кода представлен ниже:
```
  books_data = []
items = soup.find_all('div', {'class': 'card style_1'})
for item in items:
    movie_name = item.find('div', {'class': 'content'}).find('a').text
    movie_desc = item.find('div', {'class': 'content'}).find('p').text
    movie_link = item.find('div', {'class': 'content'}).find('a').get('href')
    movie_id = re.findall('\d+', movie_link)[0]
    books_data.append({
        'movie_id': movie_id,
        'movie_name': movie_name,
        'movie_desc': movie_desc
    })
    with open(f'labirint_{cur_time}.csv', 'a', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(
            (
                movie_id,
                movie_name,
                movie_desc
            )
        )
with open(f'labirint_{cur_time}.json', 'w') as file:
        json.dump(books_data, file, indent=4, ensure_ascii=False)
```
Из приведенного кода видно, что после считывания мы выгружаем все данные промежуточную модель. А далее данные выгружаем в файлы - labirint_11_11_2022_16_19.csv и labirint_11_11_2022_16_19.json.

2. Результат работы

Результатом работы являются CSV-файлы с извлеченными данными. Пример файла *"labirint_11_11_2022_16_19.csv"* представлен ниже.
```
id,Название фильма,Дата создания

663712,Terrifier 2,"Oct 06, 2022"

436270,Black Adam,"Oct 19, 2022"

675054,MexZombies,"Oct 26, 2022"

49046,All Quiet on the Western Front,"Oct 07, 2022"

732459,Blade of the 47 Ronin,"Oct 25, 2022"

505642,Black Panther: Wakanda Forever,"Nov 09, 2022"

829280,Enola Holmes 2,"Nov 04, 2022"

985939,Fall,"Aug 11, 2022"

420634,Terrifier,"Mar 15, 2018"

1033107,Wild Is the Wind,"Oct 28, 2022"

830788,The Invitation,"Aug 24, 2022"

575322,Project Gemini,"Jan 06, 2022"

717728,Jeepers Creepers: Reborn,"Sep 15, 2022"

642885,Hocus Pocus 2,"Sep 27, 2022"

361743,Top Gun: Maverick,"May 24, 2022"

760161,Orphan: First Kill,"Jul 27, 2022"

960704,Fullmetal Alchemist: The Final Alchemy,"Jun 24, 2022"

879538,Crawlspace,"Mar 31, 2022"

718930,Bullet Train,"Jul 03, 2022"

928123,The Battle at Lake Changjin: Water Gate Bridge,"Feb 01, 2022"
```
