import urllib3
from bs4 import BeautifulSoup
import pandas as pd
from datetime import datetime
import dateutil.relativedelta
from calendar import monthrange
import boto3

# CONNECTING TO WEATHER WEBSITE
http = urllib3.PoolManager()
url = 'https://alma-ata.nuipogoda.ru/%D0%B0%D1%80%D1%85%D0%B8%D0%B2-%D0%BF%D0%BE%D0%B3%D0%BE%D0%B4%D1%8B'
data = http.request('GET',url=url)
if data.status == 200:
    # PARSING DATA FROM WEATHER WEBSITE FOR LAST 1 MONTH
    soup = BeautifulSoup(data.data)
    days = soup.find_all("tbody", {"class": "tbody-forecast"})[0].find_all("td")
    my_dict = []
    for i in days:
        weather_status = []
        try:
            a = i.find("span").get_text()
            b = i.find('div', {"class": "cl_title"}).get_text()
            minimum = i.find("span", {"class": "min"}).get_text()
            maximum = i.find("span", {"class": "max"}).get_text()
            for i in i.find('div', {"class": "cl_title"}):
                if i.get_text() != '':
                    weather_status.append(i.get_text())
            my_dict.append([a, weather_status, minimum, maximum])

        except:
            pass

    # SAVING THE PARSED DATA INTO DATAFRAME
    df = pd.DataFrame(my_dict)

    # PREPROCESSING
    df[['Облака', 'Осадки']] = pd.DataFrame(df[1].to_list(), columns=['Облака', 'Осадки'])
    df.drop(1, axis=1, inplace=True)
    df.columns = ['Date', 'Min', 'Max', 'Clouds', 'Precipitation']
    
    # FIXING DATE COLUMN
    months = [i for i in df['Date'].to_list() if i.isalpha()]
    first_month_to = datetime.now().day
    second_month_to = 1
    
    df['Date'] = df['Date'].replace(months[0], first_month_to)
    df['Date'] = df['Date'].replace(months[1], second_month_to)
    
    now = datetime.now()
    temp = now + dateutil.relativedelta.relativedelta(months=-1)
    
    first_day, number_of_months = monthrange(temp.year, temp.month)
    new_days = []
    days = 0
    for i in df['Date']:
        if i== 1:
            days=number_of_months
        new_days.append(int(i)+days)

    df['Date'] = [temp + pd.Timedelta(days=int(day)-temp.day)  for day in new_days]
    df['Date'] = df['Date'].dt.strftime('%Y-%m-%d')
    
    # FIXING OTHER COLUMNS
    df['Min'] = df['Min'].str.replace(chr(176), '').astype('int')
    df['Max'] = df['Max'].str.replace(chr(176), '').astype('int')
    
    filename = "ala_2.csv"
    df.to_csv(filename, date_format='%Y-%m-%d', encoding='utf-8-sig', index=False, header=False)
    
