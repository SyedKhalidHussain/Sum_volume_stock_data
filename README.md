# Sum_volume_stock_data
# Compare the total volume of two stocks for each month through bar chart


import re
import requests
import json
import pandas as pd
import time
from datetime import date
import matplotlib.pyplot as plt


def stock_data(stock_code):                    # defining the function to retrieve stock_data
    json_data = list()                         # an empty list for data in json format
    url = 'https://finance.yahoo.com/quote/%s/history?p=%s' % (stock_code, stock_code)                   # modified url with address/parameter=stock_code
    fhandle = requests.get(url)                            # Get the file handle of url
    str_data = re.findall('"HistoricalPriceStore":{"prices":(.*?),"isPending"', fhandle.text)            # using regular expression to find the string="search_pattern" in the fhandle.text
    if str_data:              # False if str_data is empty
        json_data = json.loads(str_data[0])           # Convert the string data into JSON data
        json_data = json_data[::-1]                   # reverse the order/index of Json_data
    return [item for item in json_data if not 'type' in item]                 # return those item from json data not containing any string "type"

def sum_volumes_permonth(stock_code):                              # a function for calculating the total volume in each month of the stock_code
    json_data = stock_data(stock_code)                             # calling the function for stock_data in JSON format
    str_date = list()                                              # empty list for string date
    for i in range(len(json_data)):                                # iterate through the length of json_data
        time_format = date.fromtimestamp(json_data[i]['date'])                   # extracting the date in time format from json_data
        str_format = date.strftime(time_format,'%Y-%m-%d')                       # convert the time_format date into string_format
        str_date.append(str_format)                                         # append the string format date into list ("str_date")
    json_data_df = pd.DataFrame (json_data,index=str_date)                       #  make the data frame of json data with index str_date
    list_months = list()                                           # an empty list for month list
    for i in range(len(json_data_df)):                                        # iterate through the length of json_data_df
        time_format = time.strptime(json_data_df.index[i],'%Y-%m-%d')                     # convert the date in string format from index of json_data_df into time format
        list_months.append(time_format.tm_mon)                                    # extract the month from time format date and append into list of months
    temp_df = json_data_df.copy()
    temp_df['month'] = list_months                                                 # add another column "month" with values of list_months in temporary DataFrame
    total_volume = temp_df.groupby('month').volume.sum()                         # sum the values of volume column in temporary dataframe groupby w.r.t month
    return total_volume

INTC_volume = sum_volumes_permonth('INTC')                                 # calling function for 'INTC' to calculate it's sum of volume for each month
IBM_volume = sum_volumes_permonth('IBM')
volume_df = pd.DataFrame()                                   # empty data frame
volume_df['INTC'] = INTC_volume                               # add column to store the total volume for INTC for each month
volume_df['IBM'] = IBM_volume
volume_df.plot(kind = 'bar')                                   # bar-plot for data frames
plt.show()
