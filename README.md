# Parsing_kinopoisk
Данный проект представляет собой простой парсер, который собирает информацию о голосах пользователя на сайте Кинопоиск. Он извлекает данные о фильмах, включая название, дату выпуска, оценку и голосование, и сохраняет собранные данные в формате JSON. Это может быть полезно для анализа предпочтений пользователя и получения статистики.
Для работы с проектом вам необходимо установить следующие библиотеки Python:

- `requests`: для выполнения HTTP-запросов.
- `beautifulsoup4`: для парсинга HTML-страниц.
- `lxml`: для обработки XML и HTML.
- После успешного выполнения скрипта вы увидите следующий вывод в консоли:
Найдено записей: 15
```

А также первые 15 записей о фильмах, например:

[
    {
        "film_name": "Фильм 1",
        "release_date": "2023",
        "div_vote": "Голос: 10",
        "rating": "8.5"
    },
    {
        "film_name": "Фильм 2",
        "release_date": "2022",
        "div_vote": "Голос: 9",
        "rating": "7.5"
    }
    ...
]
Данные будут сохранены в файле user_votes.json в следующем формате:

[
    {
        "film_name": "Фильм 1",
        "release_date": "2023",
        "div_vote": "Голос: 10",
        "rating": "8.5"
    },
    {
        "film_name": "Фильм 2",
        "release_date": "2022",
        "div_vote": "Голос: 9",
        "rating": "7.5"
    }
    ...
]
import requests
from bs4 import BeautifulSoup
import json

def collect_user_votes(user_login):
    url = f'https://www.kinopoisk.ru/user/{user_login}/votes/'

    r = requests.get(url)
    soup = BeautifulSoup(r.text, 'lxml')

    entries = soup.find_all('div', class_='item')
    print(f'Найдено записей: {len(entries)}')

    data = []
    for entry in entries:
        div_film_info = entry.find('div', class_='info')
        film_name = div_film_info.find('a').text.strip()

        release_date = entry.find('div', class_='date').text.strip()
        div_vote = entry.find('div', class_='vote').text.strip()

        rating = None
        rating_div = div_film_info.find('div', class_='rating')
        if rating_div:
            rating_text = rating_div.find('b')
            if rating_text:
                rating = rating_text.text.strip()
            else:
                classes = rating_div.get('class', [])
                if len(classes) > 1:
                    rating = classes[1].split('-')[1]

        data.append({
            'film_name': film_name,
            'release_date': release_date,
            'div_vote': div_vote,
            'rating': rating
        })

    return data

user_votes = collect_user_votes(user_login='2206841')

print(user_votes[:15])

print(f'Количество собранных голосов: {len(user_votes)}')
with open('user_votes.json', 'w', encoding='utf-8') as json_file:
    json.dump(user_votes, json_file, ensure_ascii=False, indent=4)
