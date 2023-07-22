import requests
from bs4 import BeautifulSoup
import time

class Scraper:
    def __init__(self, url, bot_token, channel_id):
        self.url = url
        self.bot_token = bot_token
        self.channel_id = channel_id

    def scrape_data(self):
        data = []
        response = requests.get(self.url)
        content = response.content
        soup = BeautifulSoup(content, 'html.parser')
        paragraphs = soup.find_all('p')

        for paragraph in paragraphs:
            item = {}
            heading = paragraph.find_previous(['h3'])
            if heading:
                heading_text = heading.get_text()
                item['heading'] = heading_text
            image = paragraph.find_previous('img')
            if image:
                image_url = image.get('src')
                item['image'] = image_url
            paragraph_text = paragraph.get_text()
            item['paragraph'] = paragraph_text
            data.append(item)
        return data

    def post_to_telegram(self, data):
        for item in data:
            message = ""
            if 'heading' in item: 
                message += f" {item['heading']}\n"
            if 'image' in item:
                message += f" {item['image']}\n"
            if 'paragraph' in item:
                message += f" {item['paragraph']}\n"
            base_url = f'https://api.telegram.org/bot{self.bot_token}/sendMessage?chat_id={self.channel_id}&text={message}'
            requests.get(base_url)
            time.sleep(2)

if __name__ == '__main__':
    url = 'https://www.ethiopianreporter.com/'
    bot_token = "6316874111:AAE0G8HfH1nQfgRXsHXlijn3JAQpx0uDM90"
    channel_id = "-1001832802081"

    scraper = Scraper(url, bot_token, channel_id)

    while True:
        data = scraper.scrape_data()
        scraper.post_to_telegram(data)
        time.sleep(3600)
