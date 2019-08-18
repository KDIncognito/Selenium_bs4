# Harvesting data from Ryder's website

As part of my work, I encountered a need where somebody needed a list of Ryder's locations. Part of Ryder's website has content that gets rendered only through user interaction with certain elements. This JavaScript enabled content is not easy to scrape from their website, so I needed to use Selenium with bs4

### Dependencies
BeautifulSoup, Requests, Random, Numpy, Pandas and geckodriver.
Geckodriver is necessary for the Slenium to work with a browser. I use Firefox in this example.

### whenever you're ready
load all required modules
```
import random
from bs4 import BeautifulSoup
import requests
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd
import numpy as np
```
```
states=[]
driver= webdriver.Firefox(executable_path=r'/.../AppData/Local/Continuum/anaconda3/Lib/site-packages/geckodriver.exe')
#please change executable_path in the above line
=random.randint(3,25)


driver.get('https://ryder.com/locations/usa')

driver.refresh()

html=driver.page_source

soup= BeautifulSoup(html)

#Get States
for litag in soup.find_all('li'):
    for link in litag.find_all('a', href=True):
        if '/locations/usa/' in link['href']:
            source_link= 'https://ryder.com'+link['href']
            states.append(source_link)
            print('{} - {}'.format(link.get_text(strip=True), link['href']))

#Get cities
cities=[]
for state in states:
    driver.get(state)
    t=random.randint(10,50)
    driver.implicitly_wait(t)
    WebDriverWait(driver,10).until(EC.element_to_be_clickable((By.XPATH,'//*[@id="maincontent"]/div/section[2]/div[1]/div[2]/div[1]/div/ul/li/a')))
    soup= BeautifulSoup(driver.page_source,'html.parser')
    for litag in soup.find_all('li'):
        for link in litag.find_all('a', href=True):
            if '/locations/usa/' in link['href']:
                source_link= 'https://ryder.com'+link['href']
                if source_link not in cities:
                    cities.append(source_link)
    driver.implicitly_wait(t)
  

address_out= pd.DataFrame()
address_out= pd.DataFrame(columns=['address_loc','address_plc','address_zip','address_tel'])   

#Harvest data          
for city in cities:
    address_plc=[]
    address_loc=[]
    address_zip=[]
    address_tel=[]
    driver.implicitly_wait(t)
    print(city)
    driver.get(city)
    t=random.randint(10,30)
    driver.implicitly_wait(t)
    #WebDriverWait(driver,20).until(EC.element_to_be_clickable((By.XPATH,'//*[@id="location__link-315"]/address/div[1]/span[1]')))
    soup= BeautifulSoup(driver.page_source,'html.parser')
    for add_l in soup.find_all("span", itemprop="streetAddress"):
        address_plc.append(add_l.text)
        driver.implicitly_wait(t)
    for add_l in soup.find_all("span", itemprop="addressLocality"):
        address_loc.append(add_l.text)
        driver.implicitly_wait(t)
    for add_l in soup.find_all("span", itemprop="postalCode"):
        address_zip.append(add_l.text)
        driver.implicitly_wait(t)
    for add_l in soup.find_all("span.tel"):
        print(add_l.text)
        address_tel.append(add_l.text)
        driver.implicitly_wait(t)
    print(address_plc)
    address_out=address_out.append({'address_loc':address_loc,'address_plc':address_plc,'address_zip':address_zip,'address_tel':address_tel},ignore_index=True)

towns=pd.DataFrame()
towns= cities.append(address_out)    
```
After this step, you might want to change how the data is formatted, and I found this neat piece of code to do that. I am unable to recall from where, but it works.

Create an explode function 
```
def explode(df, lst_cols, fill_value='', preserve_index=False):
    # make sure `lst_cols` is list-alike
    if (lst_cols is not None
        and len(lst_cols) > 0
        and not isinstance(lst_cols, (list, tuple, np.ndarray, pd.Series))):
        lst_cols = [lst_cols]
    # all columns except `lst_cols`
    idx_cols = df.columns.difference(lst_cols)
    # calculate lengths of lists
    lens = df[lst_cols[0]].str.len()
    # preserve original index values    
    idx = np.repeat(df.index.values, lens)
    # create "exploded" DF
    res = (pd.DataFrame({
                col:np.repeat(df[col].values, lens)
                for col in idx_cols},
                index=idx)
             .assign(**{col:np.concatenate(df.loc[lens>0, col].values)
                            for col in lst_cols}))
    # append those rows that have empty lists
    if (lens == 0).any():
        # at least one list in cells is empty
        res = (res.append(df.loc[lens==0, idx_cols], sort=False)
                  .fillna(fill_value))
    # revert the original index order
    res = res.sort_index()
    # reset index if requested
    if not preserve_index:        
        res = res.reset_index(drop=True)
    return res

```
abdl= explode(address_out,['address_loc','address_plc','address_zip','address_tel'],fill_value='', preserve_index=True)
driver.close() #close the selenium instance

```
