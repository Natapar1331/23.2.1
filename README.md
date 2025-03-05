import requests
import csv
import json
from bs4 import BeautifulSoup

class LetterboxdParser:
    def __init__(self, user_id):
        self.user_id = user_id
        self.url = f'https://letterboxd.com/{self.user_id}/ratings/'

    def fetch_data(self):
        response = requests.get(self.url)
        if response.status_code == 200:
            return response.text
        else:
            raise Exception("Ошибка при получении данных: " + response.status_code)

    def parse_data(self, html):
        soup = BeautifulSoup(html, 'html.parser')
        ratings = []
        for rating in soup.find_all('div', class_='rating'):
            movie_title = rating.find('a', class_='film-title').text.strip()
            rating_value = rating.find('span', class_='rating-value').text.strip()
            ratings.append({'title': movie_title, 'rating': rating_value})
        return ratings

    def save_to_json(self, data, filename):
        with open(filename, 'w', encoding='utf-8') as json_file:
            json.dump(data, json_file, indent=4, ensure_ascii=False)

    def save_to_csv(self, data, filename):
        with open(filename, 'w', newline='', encoding='utf-8') as csv_file:
            fieldnames = ['title', 'rating']
            writer = csv.DictWriter(csv_file, fieldnames=fieldnames)
            writer.writeheader()
            for row in data:
                writer.writerow(row)

    def run(self):
        html = self.fetch_data()
        ratings = self.parse_data(html)
        return ratings
